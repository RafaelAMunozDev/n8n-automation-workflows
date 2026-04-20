# Vacation & Sick Leave Request Management System (n8n)

## Overview

This project is an end-to-end automation workflow built with **n8n** that manages employee leave requests, including both **vacation days** and **sick leave**.

It simulates a real-world HR system where employees submit requests, validations are applied, HR reviews the request, and the system automatically processes the outcome.

---

## Features

- Request submission via form  
- Data normalization and transformation  
- Business rule validation:
  - Minimum notice period  
  - Date range validation  
  - Available days validation  
- Support for two types of requests:
  - Vacation leave  
  - Sick leave  
- HR approval flow via Discord  
- Automated decision handling:
  - Calendar event creation  
  - Database update  
  - Email notifications  
- Error handling and user feedback  

---

## Workflow Architecture

The workflow is divided into four main stages:

### 1. Input & Data Preparation
- Receives request from a form  
- Normalizes input fields  
- Calculates total requested days  

### 2. Validation Layer
- Validates:
  - Minimum 7-day advance request  
  - Valid date range  
  - Available leave days from database  
- Rejects invalid requests with detailed error messages  

### 3. HR Approval Process
- Sends request to HR via Discord  
- Waits for manual approval/rejection (using n8n resume form)  
- Includes contextual information (employee, dates, type)  

### 4. Final Processing
- If approved:
  - Creates event in Google Calendar  
  - Updates database (vacation_days or sick_days)  
  - Sends confirmation email  
- If rejected:
  - Sends rejection email with reason  

---

## Database Structure

This workflow uses a SQL database table to track available leave days:

```sql
CREATE TABLE days_off (
  id SERIAL PRIMARY KEY,
  email TEXT,
  vacation_days SMALLINT,
  sick_days SMALLINT
);
```

### Key Points

- Each employee is identified by their email address  
- The system uses the email provided in the form to:
  - Retrieve available leave days  
  - Update remaining balance after approval  
- Two types of balances are managed:
  - `vacation_days`  
  - `sick_days`  

---

## Business Logic

### Days Calculation

The workflow calculates the number of requested days using JavaScript:

- Converts dates to avoid timezone issues  
- Includes both start and end dates  

```js
Math.ceil((endDate - startDate) / (1000 * 60 * 60 * 24)) + 1
```

### Validation Rules

- Requests must be submitted at least 7 days in advance  
- End date must be greater than start date  
- Requested days cannot exceed available balance  

The system dynamically selects:
- `vacation_days` or `sick_days` depending on request type  

---

## Dynamic Database Update

A single SQL query updates the correct field:

```sql
UPDATE days_off
SET 
  vacation_days = CASE 
    WHEN $3 = 'Vacaciones' THEN vacation_days - $1 
    ELSE vacation_days 
  END,
  sick_days = CASE 
    WHEN $3 = 'Baja por enfermedad' THEN sick_days - $1 
    ELSE sick_days 
  END
WHERE email = $2;
```

---

## Integrations

This workflow integrates with:

- **Google Calendar**  
  Automatically creates events for approved requests  

- **Discord**  
  Sends approval requests to HR  
  Includes a link to approve/reject  

- **Email**  
  Sends notifications:
  - Validation errors  
  - Approval confirmation  
  - Rejection with reason  

- **SQL Database**  
  Stores and updates leave balances  

---

## Credentials Required

To run this workflow, you must configure your own credentials:

- Google Calendar API  
- Database (PostgreSQL / MySQL)  
- Discord webhook or bot  
- Email service (SMTP / Gmail)  

Credentials are not included in the exported workflow JSON for security reasons.

---

## How to Use

1. Import the workflow into n8n  
2. Configure all required credentials  
3. Ensure the database table exists  
4. Execute the workflow  
5. Submit a leave request via the form  

---

## Notes

- The workflow is designed to simulate a real-world HR automation system  
- It follows clean architecture principles:
  - Separation of concerns  
  - Clear validation layer  
  - Structured processing  
- The internal node naming is in Spanish, while documentation is in English  

---

## Purpose

This project was built as part of hands-on practice with n8n, focusing on solving a realistic business problem through automation.

---

## Future Improvements

- Multi-user role system (HR vs employee)  
- UI dashboard for requests  
- Holiday calendar integration  
- Slack integration  