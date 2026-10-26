# Sistema de scraping de ofertas de empleo (n8n)

[English](README.md) · **Español**

## Descripción general

Este proyecto es un flujo de automatización completo hecho con **n8n** que extrae ofertas de empleo de portales tecnológicos, obtiene la información de forma estructurada con IA y guarda los resultados en Google Sheets, junto con un documento de Google Docs detallado por oferta.

Simula un sistema real de seguimiento de ofertas que se ejecuta solo cada día y mantiene un registro estructurado y actualizado de las ofertas de empleo tecnológico.

---

## Funcionalidades

- Ejecución diaria programada  
- Gestión de las URL desde Google Sheets  
- Web scraping con Firecrawl  
- Extracción de datos con IA (OpenAI)  
- Soporte para varios portales de empleo  
- Bucle de procesamiento por oferta  
- Creación automática de un documento de Google Docs con todo el detalle de la oferta  
- Actualización de Google Sheets con los datos extraídos y el enlace al documento  

---

## Arquitectura del flujo

El flujo se divide en tres fases principales:

### 1. Entrada y descarga de los portales
- Lee las URL de los portales de empleo desde Google Sheets  
- Descarta las entradas no válidas o vacías  
- Recorre cada URL con un bucle  
- Descarga el contenido completo de la página con Firecrawl  

### 2. Procesamiento con IA
- Convierte el contenido descargado y lo envía a OpenAI  
- Extrae una lista estructurada de ofertas con:
  - Puesto, empresa y ubicación  
  - Modalidad, salario y tecnologías  
  - Descripción y URL de la oferta  
- Normaliza la respuesta de la IA y la divide en elementos individuales  

### 3. Procesamiento por oferta
- Para cada oferta:
  - Registra los datos generales en Google Sheets  
  - Descarga la URL de la oferta para obtener todo el detalle  
  - Crea un documento de Google Docs con el contenido estructurado  
  - Actualiza la fila de Google Sheets con el enlace al documento  

---

## Estructura de Google Sheets

### Hoja 1 — URLs
| Sitio Web |
|---|
| https://www.tecnoempleo.com/ofertas-trabajo/ |
| https://www.infojobs.net/... |

### Hoja 2 — Resultados
| Título del puesto | Empresa | Ubicación | Modalidad | Salario | Tecnologías | Descripción | Enlace oferta | Google Docs - Detalle |
|---|---|---|---|---|---|---|---|---|

---

## Prompts de IA

### Primer nodo de IA — Extracción de ofertas
Recibe el markdown completo de la página y extrae un array JSON estructurado con las ofertas: puesto, empresa, ubicación, modalidad, salario, tecnologías, descripción y URL de la oferta.

### Segundo nodo de IA — Detalle de la oferta
Recibe el markdown de la página de una oferta y extrae toda la información relevante en un formato listo para escribirse en un documento de Google Docs, con secciones claras y saltos de línea.

---

## Integraciones

Este flujo se integra con:

- **Firecrawl**  
  Descarga las páginas de los portales y de cada oferta, sorteando la protección contra bots  

- **OpenAI**  
  Extrae y estructura los datos de las ofertas a partir del markdown sin procesar  

- **Google Sheets**  
  Guarda la lista de portales que se van a analizar y los datos extraídos de las ofertas  

- **Google Docs**  
  Crea un documento por oferta con todo el detalle estructurado  

---

## Credenciales necesarias

Para ejecutar este flujo tienes que configurar tus propias credenciales:

- Clave de la API de Firecrawl  
- Clave de la API de OpenAI  
- Google Sheets (OAuth2)  
- Google Docs (OAuth2)  

Por seguridad, las credenciales no se incluyen en el JSON exportado del flujo.

---

## Cómo usarlo

1. Importa el flujo en n8n  
2. Configura todas las credenciales necesarias  
3. Prepara el documento de Google Sheets con la estructura de arriba  
4. Añade las URL de los portales de empleo en la Hoja 1  
5. Ejecuta el flujo o espera a la ejecución programada  

---

## Notas

- El flujo usa Firecrawl en lugar de una petición HTTP normal para evitar la detección de bots en los portales de empleo  
- La extracción con IA puede variar según la estructura de cada portal  
- Los nombres internos de los nodos están en español; la documentación está en inglés y en español  

---

## Objetivo

Este proyecto se hizo como práctica con n8n, centrado en el web scraping real, la extracción de datos con IA y la generación automática de documentos.

---

## Mejoras futuras

- Aviso por correo o Slack cuando aparezcan ofertas nuevas  
- Detección de duplicados para no volver a procesar la misma oferta  
- Filtro por tecnologías o por ubicación  
- Panel para visualizar las ofertas extraídas  
