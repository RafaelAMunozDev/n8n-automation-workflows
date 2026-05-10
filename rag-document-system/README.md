# Standard RAG System (n8n)

## Overview

This project is an end-to-end **Retrieval-Augmented Generation (RAG)** system built with **n8n**. It automates the ingestion of documents stored in Google Drive, vectorizes their content using OpenAI embeddings and stores them in a PostgreSQL database with the PGVector extension. A conversational AI agent then allows querying those documents through a chat interface.

The system is configured as a **legal assistant** that helps lawyers consult internal legal documents, delivering precise and formally structured answers based exclusively on the ingested content.

---

## Features

- Manual and scheduled document ingestion  
- Google Drive integration for document discovery  
- Change detection: avoids reprocessing unchanged files  
- Automatic cleanup of outdated vectors when a file is updated  
- Document chunking with recursive character text splitting  
- Embedding generation with OpenAI  
- Vector storage in PostgreSQL using PGVector  
- Conversational AI agent with session memory  
- Answers grounded exclusively in ingested documents  
- Document source reference included in responses  

---

## Workflow Architecture

The system is divided into three independent but connected stages:

### 1. Document Selection & Validation

Triggered manually or on a schedule. Scans a Google Drive folder, identifies files, and applies change detection logic before processing.

**Decision logic:**

| Condition | Action |
|---|---|
| File not found in database | Process and ingest |
| File found with same MD5 checksum | Skip — no changes detected |
| File found with different MD5 checksum | Delete old vectors and re-ingest |

**Nodes:**

| Node | Description |
|---|---|
| `ejecutar_manualmente` | Manual trigger for on-demand execution |
| `ejecutar_programado` | Scheduled trigger for automatic execution |
| `buscar_archivos_drive` | Searches files and folders in Google Drive |
| `es_archivo_valido` | Filters out folders, keeps only files |
| `buscar_archivos_subcarpetas` | Searches inside subfolders if needed |
| `obtener_registro_bd` | Queries the database to check if the file was previously ingested |
| `fue_procesado_anteriormente` | Evaluates whether the file needs to be reprocessed |
| `sin_cambios_omitir` | Ends the process if no changes are detected |
| `extraer_variables_archivo` | Extracts file metadata for further processing |
| `eliminar_vectores_anteriores` | Deletes existing vectors for the file before re-ingesting |
| `esperar_eliminacion` | Waits for both deletion branches to complete before continuing |
| `registrar_archivo_bd` | Inserts or updates the file record in the database |
| `descargar_archivo_drive` | Downloads the file from Google Drive |
| `actualizar_file_id_vectores` | Links the generated vectors to their source file |
| `fin_proceso` | Marks the end of the ingestion process |

---

### 2. Database Ingestion

Processes the downloaded file, splits it into chunks, generates embeddings and stores them in PostgreSQL with PGVector.

**Nodes:**

| Node | Description |
|---|---|
| `cargar_documento` | Loads the downloaded file for processing |
| `dividir_texto_en_chunks` | Splits the document into fragments using recursive character splitting |
| `vector_store_insercion` | Generates embeddings and inserts chunks into PostgreSQL PGVector |

---

### 3. AI Chat Interface

A conversational agent that receives user messages, searches the vector store for relevant content and generates grounded responses.

**Nodes:**

| Node | Description |
|---|---|
| `recibir_mensaje_chat` | Receives the user message via chat interface |
| `agente_ia` | Orchestrates the reasoning and response generation |
| `modelo_chat_openai` | OpenAI model used by the agent for reasoning |
| `memoria_conversacion` | Maintains conversation context across messages |
| `responder_desde_vector_store` | Tool that queries the vector store to find relevant chunks |
| `vector_store_postgres` | PostgreSQL PGVector store used for retrieval |
| `modelo_embeddings_consulta` | OpenAI model used to embed the user query |
| `embeddings_openai` | Embedding model connected to the vector store |

---

## Database Structure

### `ingested_files` — Tracks processed documents

    CREATE TABLE ingested_files (
      file_id       TEXT PRIMARY KEY,
      name          TEXT,
      mime_type     TEXT,
      md5_checksum  TEXT,
      modified_time TIMESTAMPTZ,
      last_ingested TIMESTAMPTZ DEFAULT now()
    );

### `n8n_vectors` — Stores document chunks and embeddings

    CREATE TABLE n8n_vectors (
      id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      text      TEXT,
      metadata  JSONB,
      embedding VECTOR,
      file_id   TEXT
    );

### Key Points

- Files are identified by their Google Drive `file_id`  
- Change detection uses `md5_checksum` to compare file versions  
- Each vector chunk is linked to its source file via `file_id`  
- When a file changes, all its associated vectors are deleted before re-ingestion  

---

## Change Detection Logic

    -- Check if file was previously ingested with the same content
    SELECT COUNT(*) AS ingested
    FROM ingested_files
    WHERE file_id = '{{ $json.id }}'
    AND md5_checksum = '{{ $json.md5Checksum }}';

    -- Delete outdated vectors before re-ingesting
    DELETE FROM n8n_vectors
    WHERE file_id = '{{ $json.id }}';

    -- Link new vectors to their source file
    UPDATE n8n_vectors
    SET file_id = '{{ $json.id }}'
    WHERE metadata->'loc'->'lines' = '{{ $json.metadata.loc.lines }}'
    AND file_id IS NULL;

---

## AI Agent Configuration

The agent is configured as a **legal assistant** designed to help lawyers query internal legal documents.

**Behavior:**

- Searches the vector store for relevant content based on the user query  
- Returns concise, formally structured answers  
- Always includes a reference to the source document and section  
- If the information is not found, responds with a standardized message  
- Does not generate information outside of what is contained in the documents  

**This prompt can be adapted** to any other domain simply by changing the system message of the `agente_ia` node.

---

## Integrations

| Service | Usage |
|---|---|
| **Google Drive** | Document discovery and download |
| **OpenAI** | Embedding generation and chat model |
| **PostgreSQL + PGVector** | Vector storage and similarity search |

---

## Credentials Required

- OpenAI API key  
- Google Drive (OAuth2)  
- PostgreSQL connection  

Credentials are not included in the exported workflow JSON for security reasons.

---

## How to Use

1. Import the workflow into n8n  
2. Configure all required credentials  
3. Create the database tables using the SQL above  
4. Place the documents you want to ingest in a Google Drive folder  
5. Execute the ingestion workflow manually or wait for the scheduled trigger  
6. Open the chat interface and start querying your documents  

---

## Notes

- The system avoids duplicate processing using MD5 checksum comparison  
- The AI agent only answers based on ingested documents — it does not hallucinate outside content  
- The legal assistant configuration can be replaced with any other domain prompt  
- Internal node naming is in Spanish, while documentation is in English  

---

## Purpose

This project was built as part of hands-on practice with n8n, focusing on building a production-ready RAG pipeline that combines document management, vector search and conversational AI.

---

## Future Improvements

- Support for multiple file formats beyond PDF  
- Multi-folder monitoring in Google Drive  
- User authentication for the chat interface  
- Admin dashboard to monitor ingested documents  
- Automatic re-ingestion when a file is deleted from Drive