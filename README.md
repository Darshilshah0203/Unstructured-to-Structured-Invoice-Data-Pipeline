# 🧾 Unstructured-to-Structured Invoice Data Pipeline (n8n + Gemini AI)

> Drop invoices (PDF or image) into a Google Drive folder, click once, and get clean, structured data in Google Sheets. No manual copy-paste, no missed entries.

![n8n](https://img.shields.io/badge/Automation-n8n-EA4B71?logo=n8n&logoColor=white)
![Gemini](https://img.shields.io/badge/AI-Google%20Gemini-4285F4?logo=google&logoColor=white)
![Google Drive](https://img.shields.io/badge/Storage-Google%20Drive-34A853?logo=googledrive&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Output-Google%20Sheets-0F9D58?logo=googlesheets&logoColor=white)
![OCR](https://img.shields.io/badge/OCR-OCR.space-orange)

---

## 📑 Table of Contents

1. [About the Project](#-about-the-project)
2. [Problem Statement](#-problem-statement)
3. [Key Features](#-key-features)
4. [Workflow Architecture](#-workflow-architecture)
5. [Tech Stack](#-tech-stack)
6. [Output Data Schema](#-output-data-schema)
7. [Prerequisites](#-prerequisites)
8. [Setup & Installation](#-setup--installation)
9. [Step-by-Step Build Guide](#-step-by-step-build-guide)
10. [Information Extractor JSON Sample](#-information-extractor-json-sample)
11. [Usage](#-usage)
12. [Duplicate Prevention (UID)](#-duplicate-prevention-uid)
13. [Error Handling & Validation](#-error-handling--validation)
14. [Project Structure](#-project-structure)
15. [Future Improvements](#-future-improvements)
16. [Skills Demonstrated](#-skills-demonstrated)
17. [Acknowledgements](#-acknowledgements)
18. [License](#-license)

---

## 📌 About the Project

This project is a **no-code/low-code invoice automation workflow** built in **n8n**. It automatically reads invoices of different formats (**PDF** and **PNG/image**) stored in a Google Drive folder, uses **OCR** and **Google Gemini** to understand the messy, unstructured content, and extracts key fields such as invoice number, date, billing company details, customer name, tax, total amount, and line items. The clean data is then written into two linked Google Sheets: **Invoice Details** and **Invoice Items**.

It turns a slow, error-prone manual process (download, open, read, type into a sheet) into a **single-click, repeatable pipeline**.

---

## ❗ Problem Statement

Finance and operations teams receive invoices in many layouts, languages, and formats. Entering them manually leads to:

- ⏱️ Hours of repetitive copy-paste work
- ❌ Typing mistakes and missed entries
- 📂 Inconsistent data that is hard to analyze
- 🔁 Duplicate records when files are re-processed

**Goal:** Build a system that opens every file in a Drive folder, reads it like a human would, extracts the important details, and logs them in a tracker, with checks so only valid invoices are processed.

---

## ✨ Key Features

- 🖱️ **One-click trigger** (manual execution, easily switched to a schedule)
- 📁 **Google Drive integration** to pick up all files from a chosen invoice folder
- 🔄 **Loop processing** that handles one file at a time, like a person opening each invoice
- 🔀 **Smart routing** with a Switch node: separate paths for **PDF** and **PNG/image** files
- 📄 **Text extraction** from digital PDFs using the *Extract from File* node
- 🔎 **Fallback OCR** (OCR.space API) for scanned PDFs and images
- 🤖 **AI-powered extraction** with the *Information Extractor* node and the **Google Gemini Chat Model**
- 🧱 **Structured JSON output** guided by a sample schema for consistent results
- 📊 **Two-sheet output**: invoice header data and line-item data
- 🔑 **UID-based matching** to avoid duplicate rows on re-runs
- ✅ **Validation checks** so only valid invoice files are processed

---

## 🏗️ Workflow Architecture

```
┌──────────────────┐   ┌──────────────────────┐   ┌────────────────┐
│ Manual Trigger   │──▶│ Google Drive: Search │──▶│ Loop Over Items│
│ (Execute)        │   │ files and folders    │   │                │
└──────────────────┘   └──────────────────────┘   └───────┬────────┘
                                                          │ (each file)
                                                          ▼
                                                 ┌────────────────┐
                                                 │ Download file  │
                                                 └───────┬────────┘
                                                         ▼
                                                 ┌────────────────┐
                                                 │ Switch (type)  │
                                                 └───┬────────┬───┘
                                                PDF  │        │ PNG
                                                     ▼        │
                                          ┌──────────────────┐│
                                          │ Extract from File││
                                          └────────┬─────────┘│
                                                   ▼          │
                                          ┌──────────────────┐│
                                          │ If (text found?) ││
                                          └───┬──────────┬───┘│
                                         false│          │true│
                                              ▼          ▼    ▼
                                          ┌──────────────────────┐
                                          │ OCR HTTP Request     │
                                          │ (OCR.space API)      │
                                          └──────────┬───────────┘
                                                     ▼
                              ┌─────────────────────────────────────────┐
                              │ Information Extractor                   │◀── Google Gemini
                              │ (structured JSON from raw text)         │    Chat Model
                              └──────────────┬──────────────────────────┘
                                             ▼
                              ┌─────────────────────────────┐
                              │ Append to Sheet 1           │  → "Invoice Details"
                              └──────────────┬──────────────┘
                                             ▼
                              ┌─────────────────────────────┐
                              │ Edit Fields (map items)     │
                              └──────────────┬──────────────┘
                                             ▼
                              ┌─────────────────────────────┐
                              │ Split Out (items list)      │
                              └──────────────┬──────────────┘
                                             ▼
                              ┌─────────────────────────────┐
                              │ Append row in sheet         │  → "Invoice Items"
                              └──────────────┬──────────────┘
                                             ▼
                                   (back to Loop Over Items)
```

> 💡 Add a screenshot of your n8n canvas here: `![Workflow](assets/workflow.png)`

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Workflow automation | **n8n** (Cloud trial or self-hosted) |
| File storage | **Google Drive** |
| Text extraction (PDF) | n8n **Extract from File** node |
| OCR for images / scanned files | **OCR.space API** via HTTP Request node |
| LLM | **Google Gemini Chat Model** |
| Structured extraction | n8n **Information Extractor** (LangChain) node |
| Data store | **Google Sheets** |
| Prompt drafting help | ChatGPT / Perplexity |

---

## 📊 Output Data Schema

### Sheet 1: `Invoice Details`

| Column | Description | Example |
|---|---|---|
| `invoice_number` | Unique invoice ID | `INV-1004` |
| `date` | Invoice date | `2025-06-10` |
| `billing_company` | Issuer of the invoice | `ACME Corp` |
| `billing_company_address` | Issuer address | `123 Industrial Ave, Springfield` |
| `billing_company_email` | Issuer email | `accounts@acmecorp.com` |
| `billing_company_phone` | Issuer phone | `+1 (555) 123-4567` |
| `customer_company_name` | Billed customer | `Sunrise Stores` |
| `total_amount` | Grand total | `125.97` |
| `tax` | Tax amount | `19.21` |

### Sheet 2: `Invoice Items`

| Column | Description | Example |
|---|---|---|
| `uid` | Unique key (invoice number + item) used to prevent duplicates | `INV-1004_Doohickey C` |
| `invoice_number` | Link back to the Invoice Details sheet | `INV-1004` |
| `item_name` | Product or service description | `Doohickey C` |
| `quantity` | Units | `5` |
| `unit_price` | Price per unit | `9.75` |
| `total` | Line total | `48.75` |

> Missing values are written as `Not provided` / `Not specified` so no row is silently dropped.

---

## ✅ Prerequisites

- An **n8n** account: [14-day cloud trial](https://app.n8n.cloud/register) **or** a local/self-hosted install (Docker or npm)
- A **Google account** with access to Drive and Sheets
- **Google Cloud OAuth credentials** for the Drive and Sheets nodes
- A **Google Gemini API key** (from Google AI Studio)
- An **OCR.space API key** (free tier available)
- A few sample invoices (PDF and PNG)

---

## ⚙️ Setup & Installation

### 1. Set up n8n

**Option A: Cloud trial**
Sign up at https://app.n8n.cloud/register

**Option B: Local with Docker**
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```
Open `http://localhost:5678`.

**Option C: npm**
```bash
npx n8n
```

### 2. Prepare Google Drive and Sheets

1. Create a Drive folder, for example `Invoices`, and upload your PDF/PNG invoices.
2. Create a Google Sheet named **Invoice Tracker** with two tabs:
   - `Invoice Details` with the headers listed above
   - `Invoice Items` with the headers listed above

### 3. Configure credentials in n8n

| Credential | Used by |
|---|---|
| Google Drive OAuth2 | Search files, Download file |
| Google Sheets OAuth2 | Append to sheet nodes |
| Google Gemini (PaLM) API | Gemini Chat Model |
| OCR.space API key | HTTP Request node |

### 4. Import the workflow

1. In n8n: **Workflows → Import from File**
2. Select `workflow/invoice_pipeline.json`
3. Re-select your credentials on each node, choose your Drive folder, and pick your Google Sheet and tabs.

---

## 🪜 Step-by-Step Build Guide

**Step 1: Set up your n8n account.** Use the cloud trial or a local setup.

**Step 2: Draft the flow with AI.** Use this prompt in ChatGPT/Perplexity:
> *I want to create an automation in n8n where, when I click a button, it goes to a specific Google Drive folder, picks all PDF invoices, opens each file, reads the content using AI, extracts key fields like Invoice No, Customer Name, Amount, Date, and adds them into a Google Sheet. Only process valid files and ignore anything that doesn't match the pattern. Help me draft this flow step-by-step.*

**Step 3: Trigger.** Add a *Manual Trigger* ("When clicking 'Execute workflow'"). Later you can swap it for a Schedule Trigger.

**Step 4: Locate the invoice folder.** Add a *Google Drive → Search files and folders* node (resource: file/folder) filtered to your invoice folder.

**Step 5: Loop through each file.** Add *Loop Over Items* so one file is processed at a time.

**Step 6: Switch for PNG and PDF.** Add a *Switch* node (mode: Rules) that routes by file extension or MIME type into two paths.

**Step 7: PDF processing.** Add *Extract from File* (operation: Extract from PDF) to pull the text layer. Follow with an *If* node: if text is found, send it to the extractor; if empty (scanned PDF), send it to OCR.

**Step 8: AI information extraction.**
- Add the *Information Extractor* node and pass in the extracted text.
- Attach the **Google Gemini Chat Model** as its language model.
- Provide the JSON sample (see below) for consistent results.

**Step 9: Append to "Invoice Details".** Add a *Google Sheets → Append or Update Row* node and map the header fields.

**Step 10: PNG path (OCR).** Add an *HTTP Request* node (POST to `https://api.ocr.space/parse/image`) with your API key and the file as binary. Feed its parsed text into the same Information Extractor.

**Step 11: Extract the item list.** Use *Edit Fields (Set)* to map the items array, then *Split Out* on the `items` field so each line item becomes its own row.

**Step 12: Append to "Invoice Items".** Add another *Google Sheets* node. Create a **UID** field (for example `{{ $json.invoice_number }}_{{ $json.item_name }}`) and use it as the matching column so re-runs update instead of duplicating.

---

## 🧩 Information Extractor JSON Sample

Paste a sample like this into the Information Extractor's schema/example field. You can generate one by uploading a real invoice to ChatGPT with this prompt:
> *To extract key information from this invoice, draft a JSON format that comprehensively captures all relevant details.*

```json
{
  "invoice_number": "INV-1004",
  "date": "2025-06-10",
  "billing_company": "ACME Corp",
  "billing_company_address": "123 Industrial Ave, Springfield",
  "billing_company_email": "accounts@acmecorp.com",
  "billing_company_phone": "+1 (555) 123-4567",
  "customer_company_name": "Sunrise Stores",
  "total_amount": 125.97,
  "tax": 19.21,
  "items": [
    {
      "item_name": "Doohickey C",
      "quantity": 5,
      "unit_price": 9.75,
      "total": 48.75
    }
  ]
}
```

**Tip:** In the extractor's system prompt, instruct the model to return `"Not provided"` for any missing field and to never invent values.

---

## ▶️ Usage

1. Upload invoices (PDF/PNG) to the configured Drive folder.
2. Open the workflow in n8n and click **Execute workflow**.
3. Watch the **Executions** tab as each file is processed.
4. Open your Google Sheet. `Invoice Details` and `Invoice Items` are now populated.

To automate fully, replace the manual trigger with a **Schedule Trigger** (for example daily at 9 AM) or a **Google Drive Trigger** (on new file).

---

## 🔑 Duplicate Prevention (UID)

Re-running the workflow should not create duplicate rows. The `Invoice Items` sheet uses a **UID** (a combination of invoice number and item) as the *column to match on* in the **Append or Update** operation. If the UID exists, the row is updated; otherwise a new row is appended.

---

## 🛡️ Error Handling & Validation

- **File-type routing:** the Switch node only lets supported formats (PDF, PNG) through.
- **Empty-text check:** the If node detects scanned PDFs and falls back to OCR.
- **Schema-guided AI:** the JSON sample constrains the output structure.
- **Default values:** missing fields are written as `Not provided` / `Not specified`.
- **Sequential loop:** one file at a time makes failures easier to trace in Executions.

---

## 📂 Project Structure

```
invoice-data-pipeline/
├── README.md
├── workflow/
│   └── invoice_pipeline.json        # Exported n8n workflow
├── assets/
│   ├── workflow.png                 # n8n canvas screenshot
│   ├── invoice_details_sheet.png    # Output sheet 1
│   └── invoice_items_sheet.png      # Output sheet 2
├── samples/
│   ├── sample_invoice.pdf
│   └── sample_invoice.png
└── docs/
    └── OCR_Space_API_setup.md
```

---

## 🚀 Future Improvements

- 📧 Pull invoices directly from Gmail attachments
- 🔔 Slack / email alerts on failed or low-confidence extractions
- 🧮 Validation rules (sum of items + tax equals total)
- 🌍 Multi-currency and multi-language support
- 🗂️ Auto-move processed files to a `Processed` folder
- 📈 Dashboard in Looker Studio / Power BI on top of the sheets
- 🔐 Human-in-the-loop approval step for flagged invoices

---

## 🎯 Skills Demonstrated

**Automation & Integration**
- Workflow automation with n8n
- Google Drive and Google Sheets API integration (OAuth2)
- REST API calls with HTTP Request nodes
- Conditional logic, looping, and branching

**AI & Data Extraction**
- Generative AI / LLM integration (Google Gemini)
- Prompt engineering for structured JSON output
- OCR (OCR.space) and document parsing (PDF and image)
- Unstructured-to-structured data transformation

**Data Engineering**
- Data cleaning and schema design
- Header/line-item (one-to-many) data modeling
- Idempotent pipelines and deduplication with UIDs
- ETL pipeline thinking (Extract, Transform, Load)

**Soft Skills**
- Process automation and problem decomposition
- Documentation and technical communication


