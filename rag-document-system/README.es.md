# Sistema RAG estándar (n8n)

[English](README.md) · **Español**

## Descripción general

Este proyecto es un sistema completo de **Retrieval-Augmented Generation (RAG)** hecho con **n8n**. Automatiza la importación de documentos guardados en Google Drive, vectoriza su contenido con embeddings de OpenAI y los guarda en una base de datos PostgreSQL con la extensión PGVector. Después, un agente conversacional de IA permite consultar esos documentos desde un chat.

El sistema está configurado como **asistente jurídico** que ayuda a los abogados a consultar documentos legales internos, con respuestas precisas y de estructura formal basadas únicamente en el contenido importado.

---

## Funcionalidades

- Importación de documentos manual y programada  
- Integración con Google Drive para localizar los documentos  
- Detección de cambios: no vuelve a procesar ficheros que no han cambiado  
- Limpieza automática de los vectores obsoletos cuando se actualiza un fichero  
- División de los documentos en fragmentos con división recursiva por caracteres  
- Generación de embeddings con OpenAI  
- Almacenamiento vectorial en PostgreSQL con PGVector  
- Agente conversacional de IA con memoria de sesión  
- Respuestas basadas únicamente en los documentos importados  
- Referencia al documento de origen incluida en las respuestas  

---

## Arquitectura del flujo

El sistema se divide en tres fases independientes pero conectadas:

### 1. Selección y validación de documentos

Se lanza manualmente o de forma programada. Recorre una carpeta de Google Drive, identifica los ficheros y aplica la lógica de detección de cambios antes de procesarlos.

**Lógica de decisión:**

| Condición | Acción |
|---|---|
| El fichero no está en la base de datos | Se procesa e importa |
| El fichero está y tiene el mismo checksum MD5 | Se omite: no hay cambios |
| El fichero está y tiene otro checksum MD5 | Se borran los vectores antiguos y se vuelve a importar |

**Nodos:**

| Nodo | Descripción |
|---|---|
| `ejecutar_manualmente` | Disparador manual para ejecutarlo bajo demanda |
| `ejecutar_programado` | Disparador programado para la ejecución automática |
| `buscar_archivos_drive` | Busca ficheros y carpetas en Google Drive |
| `es_archivo_valido` | Descarta las carpetas y deja solo los ficheros |
| `buscar_archivos_subcarpetas` | Busca dentro de las subcarpetas si hace falta |
| `obtener_registro_bd` | Consulta la base de datos para saber si el fichero ya se importó |
| `fue_procesado_anteriormente` | Evalúa si el fichero hay que volver a procesarlo |
| `sin_cambios_omitir` | Termina el proceso si no hay cambios |
| `extraer_variables_archivo` | Extrae los metadatos del fichero para el resto del proceso |
| `eliminar_vectores_anteriores` | Borra los vectores existentes del fichero antes de volver a importarlo |
| `esperar_eliminacion` | Espera a que terminen las dos ramas de borrado antes de seguir |
| `registrar_archivo_bd` | Inserta o actualiza el registro del fichero en la base de datos |
| `descargar_archivo_drive` | Descarga el fichero de Google Drive |
| `actualizar_file_id_vectores` | Asocia los vectores generados a su fichero de origen |
| `fin_proceso` | Marca el final del proceso de importación |

---

### 2. Importación a la base de datos

Procesa el fichero descargado, lo divide en fragmentos, genera los embeddings y los guarda en PostgreSQL con PGVector.

**Nodos:**

| Nodo | Descripción |
|---|---|
| `cargar_documento` | Carga el fichero descargado para procesarlo |
| `dividir_texto_en_chunks` | Divide el documento en fragmentos con división recursiva por caracteres |
| `vector_store_insercion` | Genera los embeddings e inserta los fragmentos en PostgreSQL PGVector |

---

### 3. Interfaz de chat con IA

Un agente conversacional que recibe los mensajes del usuario, busca el contenido relevante en el almacén vectorial y genera respuestas basadas en él.

**Nodos:**

| Nodo | Descripción |
|---|---|
| `recibir_mensaje_chat` | Recibe el mensaje del usuario desde el chat |
| `agente_ia` | Coordina el razonamiento y la generación de la respuesta |
| `modelo_chat_openai` | Modelo de OpenAI que usa el agente para razonar |
| `memoria_conversacion` | Mantiene el contexto de la conversación entre mensajes |
| `responder_desde_vector_store` | Herramienta que consulta el almacén vectorial para encontrar los fragmentos relevantes |
| `vector_store_postgres` | Almacén vectorial PostgreSQL PGVector usado para la recuperación |
| `modelo_embeddings_consulta` | Modelo de OpenAI usado para generar el embedding de la consulta |
| `embeddings_openai` | Modelo de embeddings conectado al almacén vectorial |

---

## Estructura de la base de datos

### `ingested_files` — Registro de los documentos procesados

    CREATE TABLE ingested_files (
      file_id       TEXT PRIMARY KEY,
      name          TEXT,
      mime_type     TEXT,
      md5_checksum  TEXT,
      modified_time TIMESTAMPTZ,
      last_ingested TIMESTAMPTZ DEFAULT now()
    );

### `n8n_vectors` — Fragmentos de los documentos y sus embeddings

    CREATE TABLE n8n_vectors (
      id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      text      TEXT,
      metadata  JSONB,
      embedding VECTOR,
      file_id   TEXT
    );

### Puntos clave

- Los ficheros se identifican por su `file_id` de Google Drive  
- La detección de cambios compara las versiones de los ficheros con `md5_checksum`  
- Cada fragmento vectorial está asociado a su fichero de origen mediante `file_id`  
- Cuando un fichero cambia, se borran todos sus vectores antes de volver a importarlo  

---

## Lógica de detección de cambios

    -- Comprueba si el fichero ya se importó con el mismo contenido
    SELECT COUNT(*) AS ingested
    FROM ingested_files
    WHERE file_id = '{{ $json.id }}'
    AND md5_checksum = '{{ $json.md5Checksum }}';

    -- Borra los vectores obsoletos antes de volver a importar
    DELETE FROM n8n_vectors
    WHERE file_id = '{{ $json.id }}';

    -- Asocia los vectores nuevos a su fichero de origen
    UPDATE n8n_vectors
    SET file_id = '{{ $json.id }}'
    WHERE metadata->'loc'->'lines' = '{{ $json.metadata.loc.lines }}'
    AND file_id IS NULL;

---

## Configuración del agente de IA

El agente está configurado como **asistente jurídico** para ayudar a los abogados a consultar documentos legales internos.

**Comportamiento:**

- Busca en el almacén vectorial el contenido relevante para la consulta del usuario  
- Devuelve respuestas concisas y con estructura formal  
- Incluye siempre una referencia al documento y la sección de origen  
- Si no encuentra la información, responde con un mensaje estándar  
- No genera información fuera de lo que contienen los documentos  

**Este prompt se puede adaptar** a cualquier otro ámbito cambiando simplemente el mensaje de sistema del nodo `agente_ia`.

---

## Integraciones

| Servicio | Uso |
|---|---|
| **Google Drive** | Localización y descarga de los documentos |
| **OpenAI** | Generación de embeddings y modelo de chat |
| **PostgreSQL + PGVector** | Almacenamiento vectorial y búsqueda por similitud |

---

## Credenciales necesarias

- Clave de la API de OpenAI  
- Google Drive (OAuth2)  
- Conexión a PostgreSQL  

Por seguridad, las credenciales no se incluyen en el JSON exportado del flujo.

---

## Cómo usarlo

1. Importa el flujo en n8n  
2. Configura todas las credenciales necesarias  
3. Crea las tablas de la base de datos con el SQL de arriba  
4. Pon los documentos que quieras importar en una carpeta de Google Drive  
5. Ejecuta manualmente el flujo de importación o espera a la ejecución programada  
6. Abre el chat y empieza a consultar tus documentos  

---

## Notas

- El sistema evita procesar duplicados comparando el checksum MD5  
- El agente de IA solo responde a partir de los documentos importados: no inventa contenido externo  
- La configuración de asistente jurídico se puede sustituir por el prompt de cualquier otro ámbito  
- Los nombres internos de los nodos están en español; la documentación está en inglés y en español  

---

## Objetivo

Este proyecto se hizo como práctica con n8n, centrado en construir un pipeline RAG listo para producción que combina gestión documental, búsqueda vectorial e IA conversacional.

---

## Mejoras futuras

- Soporte para más formatos de fichero además de PDF  
- Supervisión de varias carpetas de Google Drive  
- Autenticación de usuarios en el chat  
- Panel de administración para supervisar los documentos importados  
- Volver a importar automáticamente cuando se borra un fichero de Drive  
