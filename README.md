# n8n Automation Workflows

## Overview

This repository contains a collection of automation workflows built with **n8n**, focused on solving real-world business problems through process automation.

Each workflow is designed as an independent solution, covering different use cases such as request management, web scraping, AI-powered data extraction and automated decision-making.

---

## Workflows Included

### 1. Vacation & Sick Leave Request System

Automates the management of employee leave requests, including validation, HR approval and automatic processing.

#### Main features:

- Form-based request submission  
- Validation of business rules  
- Support for vacation and sick leave  
- HR approval via Discord  
- Google Calendar integration  
- Database updates  
- Email notifications  

**Path:** `vacation-request-system/`

---

### 2. Job Scraping System

Automates the daily scraping of tech job portals, extracts structured offer data using AI and stores the results in Google Sheets with a detailed Google Docs file per offer.

#### Main features:

- Scheduled daily execution  
- Multi-portal scraping via Firecrawl  
- AI-powered data extraction with OpenAI  
- Google Sheets integration  
- Automatic Google Docs generation per offer  

**Path:** `job-scraping-system/`

---

### 3. Standard RAG System

A production-ready Retrieval-Augmented Generation pipeline that ingests documents from Google Drive, vectorizes their content using OpenAI embeddings and stores them in PostgreSQL with PGVector. Includes a conversational AI agent that answers questions based exclusively on the ingested documents.

#### Main features:

- Manual and scheduled document ingestion  
- Change detection to avoid reprocessing unchanged files  
- Document chunking and embedding generation  
- Vector storage with PostgreSQL and PGVector  
- Conversational AI agent with session memory  
- Configurable for any domain via system prompt  

**Path:** `rag-document-system/`

---

## Technologies Used

- n8n  
- OpenAI API  
- Firecrawl  
- Google Sheets & Google Docs  
- Google Drive  
- PostgreSQL + PGVector  
- Google Calendar API  
- Discord  
- Email services  

---

## Project Structure

    n8n-automation-workflows/
    ├── vacation-request-system/
    ├── job-scraping-system/
    └── rag-document-system/

Each folder contains:

- `workflow.json` → the n8n workflow  
- `README.md` → explanation of the solution  

---

## Setup Requirements

To run any workflow, you must configure your own credentials:

- Database connection (PostgreSQL)  
- Google Calendar  
- Google Drive (OAuth2)  
- Discord  
- Email service  
- OpenAI API key  
- Firecrawl API key  
- Google Sheets & Google Docs (OAuth2)  

Credentials are not included in the repository for security reasons.

---

## How to Use

1. Import the desired workflow into n8n  
2. Configure the required credentials  
3. Adjust environment-specific values if needed  
4. Execute the workflow  

---

## Purpose

This repository was created as part of hands-on practice with n8n, focusing on building practical automation solutions that simulate real-world scenarios.

---

## Notes

- Workflows are modular and can be adapted to different environments  
- Internal node naming may be in Spanish, while documentation is in English  
- The goal is to demonstrate automation design, not only technical implementation