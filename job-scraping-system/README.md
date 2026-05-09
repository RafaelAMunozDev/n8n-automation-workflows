# Job Scraping System (n8n)

## Overview

This project is an end-to-end automation workflow built with **n8n** that scrapes job offers from tech employment portals, extracts structured information using AI, and stores the results in Google Sheets along with a detailed Google Docs file per offer.

It simulates a real-world job monitoring system that runs automatically every day, keeping a structured and up-to-date record of tech job offers.

---

## Features

- Scheduled daily execution  
- URL management via Google Sheets  
- Web scraping using Firecrawl  
- AI-powered data extraction (OpenAI)  
- Support for multiple job portals  
- Per-offer processing loop  
- Automatic Google Docs creation with full offer detail  
- Google Sheets update with extracted data and document link  

---

## Workflow Architecture

The workflow is divided into three main stages:

### 1. Input & Site Fetching
- Reads job portal URLs from Google Sheets  
- Filters out invalid or empty entries  
- Iterates over each URL using a loop  
- Fetches the full page content via Firecrawl  

### 2. AI Processing
- Converts the scraped content and sends it to OpenAI  
- Extracts a structured list of job offers including:
  - Job title, company, location  
  - Work mode, salary, technologies  
  - Description and offer URL  
- Normalizes the AI response and splits it into individual items  

### 3. Per-Offer Processing
- For each job offer:
  - Registers the general data in Google Sheets  
  - Scrapes the individual offer URL for full details  
  - Creates a Google Docs document with the structured content  
  - Updates the Google Sheets row with the link to the document  

---

## Google Sheets Structure

### Sheet 1 — URLs
| Sitio Web |
|---|
| https://www.tecnoempleo.com/ofertas-trabajo/ |
| https://www.infojobs.net/... |

### Sheet 2 — Results
| Título del puesto | Empresa | Ubicación | Modalidad | Salario | Tecnologías | Descripción | Enlace oferta | Google Docs - Detalle |
|---|---|---|---|---|---|---|---|---|

---

## AI Prompts

### First AI Node — Offer Extraction
Receives the full page markdown and extracts a structured JSON array of job offers, including title, company, location, work mode, salary, technologies, description and offer URL.

### Second AI Node — Offer Detail
Receives the markdown of an individual offer page and extracts all relevant information in a format ready to be written into a Google Docs document, with clear sections and line breaks.

---

## Integrations

This workflow integrates with:

- **Firecrawl**  
  Scrapes job portal pages and individual offer pages, bypassing bot protection  

- **OpenAI**  
  Extracts and structures job offer data from raw markdown content  

- **Google Sheets**  
  Stores the list of portals to scrape and the extracted job offer data  

- **Google Docs**  
  Creates one document per offer with the full structured detail  

---

## Credentials Required

To run this workflow, you must configure your own credentials:

- Firecrawl API key  
- OpenAI API key  
- Google Sheets (OAuth2)  
- Google Docs (OAuth2)  

Credentials are not included in the exported workflow JSON for security reasons.

---

## How to Use

1. Import the workflow into n8n  
2. Configure all required credentials  
3. Set up the Google Sheets document with the structure above  
4. Add the job portal URLs to Sheet 1  
5. Execute the workflow or wait for the scheduled trigger  

---

## Notes

- The workflow uses Firecrawl instead of a standard HTTP request to avoid bot detection on job portals  
- The AI extraction may vary depending on the structure of each portal  
- The internal node naming is in Spanish, while documentation is in English  

---

## Purpose

This project was built as part of hands-on practice with n8n, focusing on real-world web scraping, AI-powered data extraction and automated document generation.

---

## Future Improvements

- Email or Slack notification when new offers are found  
- Duplicate detection to avoid re-processing the same offer  
- Filtering by technology stack or location  
- Dashboard to visualize extracted offers