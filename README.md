# 🧾 InvoScan AI — End-to-End GST Invoice Intelligence System

> **Hacktober Fest 4 | Open Source AI Hackathon | Organized by Elevate**  
> **Problem Statement 3 — VYOM+ GST Invoice Intelligence**

---

## 📌 Problem Statement

Businesses in India deal with GST invoices across a wide variety of formats — printed PDFs, scanned images, handwritten documents, Excel sheets, and CSVs. Manually extracting, validating, and structuring this data is error-prone, time-consuming, and difficult to scale.

**VYOM+** needs a complete invoice intelligence pipeline that:
- Accepts diverse input formats (`.xlsx`, `.csv`, `.pdf`, `.jpeg`, `.jpg`, `.png`)
- Automatically identifies the document type and routes it through the right processing pipeline
- Extracts all relevant GST, tax, financial, and line-item information
- Validates and standardizes the output into machine-readable structured records (JSON + tabular)

---

## 🎯 Target Users

| User | Pain Point Solved |
|---|---|
| **Small & Medium Businesses (SMBs)** | No dedicated accountant; need automated GST record creation |
| **CA Firms & Tax Consultants** | Handle hundreds of invoices per client; manual extraction wastes hours |
| **ERP / Accounting Platforms (like VYOM+)** | Need structured data from raw documents to feed downstream workflows |
| **GST Auditors** | Require validated, consistent data across multiple invoice sources |

---

## 💡 Proposed Solution

We propose **InvoScan AI** — a modular, AI-powered document intelligence pipeline that ingests raw invoice documents and outputs validated, structured financial records.

The system is divided into three layers:
1. **Input Router** — detects file type and directs to the right sub-pipeline
2. **AI Extraction Core** — uses a Vision-Language Model (VLM) and OCR for unstructured inputs, and pandas-based parsing for structured inputs
3. **Validation & Output Engine** — validates GST numbers, invoice totals, tax calculations, and exports clean JSON + tabular data

---

## 🤖 Selected Open-Source AI Technology

### Primary Model: [Qwen2.5-VL](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct) (Vision-Language Model)

**Why Qwen2.5-VL?**
- State-of-the-art open-source VLM with exceptional document understanding capabilities
- Handles both printed and **handwritten** text in images/PDFs — which is the core technical challenge of this problem
- Supports multi-page documents and complex table layouts natively
- Can be run locally via `transformers` + `bitsandbytes` (4-bit quantized) on a single GPU
- Openly licensed (Apache 2.0) — fully compliant with the open-source requirement

**Supporting Tools:**
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) — for high-accuracy OCR pre-processing on scanned images
- [pdfplumber](https://github.com/jsvine/pdfplumber) — for extracting text and tables from digitally generated PDFs
- [pandas](https://pandas.pydata.org/) — for structured `.xlsx` / `.csv` processing
- [FastAPI](https://fastapi.tiangolo.com/) — lightweight backend for the upload interface

---

## 🧠 AI's Role in the System

The AI (Qwen2.5-VL) is the **core intelligence layer**, not an optional component:

- For **PDFs and images**, the VLM receives the document page as an image and a structured prompt requesting specific GST fields. It outputs a JSON response with extracted values.
- The model is specifically prompted to handle **handwritten invoices** — identifying amounts, GSTIN numbers, HSN codes, and line items even under poor scan quality.
- **No proprietary API is used** — inference runs entirely locally or on a self-hosted server.

> The model is integral to the system's ability to handle real-world, messy invoice documents that rule-based OCR alone cannot reliably parse.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER INTERFACE                           │
│              (FastAPI + Simple HTML Upload Portal)              │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Upload Document
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     INPUT ROUTER                                │
│         Detects: .xlsx / .csv / .pdf / .jpg / .png             │
└────┬──────────────┬──────────────────────┬──────────────────────┘
     │              │                      │
     ▼              ▼                      ▼
┌─────────┐  ┌────────────────┐   ┌────────────────────┐
│ pandas  │  │  pdfplumber    │   │   PaddleOCR        │
│ Parser  │  │  (Digital PDF) │   │ (Image/Scanned PDF)│
│.xlsx/.csv│  └───────┬────────┘   └────────┬───────────┘
└────┬────┘          │                      │
     │               └──────────┬───────────┘
     │                          ▼
     │               ┌─────────────────────┐
     │               │   Qwen2.5-VL (7B)   │
     │               │  Vision-Language     │
     │               │  Model — Extraction  │
     │               │  + Understanding     │
     │               └──────────┬──────────┘
     │                          │
     └──────────────────────────┤
                                ▼
                ┌───────────────────────────┐
                │   VALIDATION ENGINE        │
                │  • GSTIN format check      │
                │  • Tax calculation verify  │
                │  • Missing field flagging  │
                │  • Duplicate detection     │
                └──────────────┬────────────┘
                               ▼
               ┌───────────────────────────────┐
               │        OUTPUT LAYER           │
               │  • JSON (machine-readable)    │
               │  • Structured tabular (CSV)   │
               │  • Confidence scores          │
               │  • Flagged/uncertain fields   │
               └───────────────────────────────┘
```

---

## 🔄 Data Flow

```
1. User uploads document via web interface
2. Input Router identifies file type
3a. [Excel/CSV]       → pandas normalizes columns → Validation Engine
3b. [Digital PDF]     → pdfplumber extracts text/tables → Qwen2.5-VL field mapping → Validation Engine
3c. [Image/Scanned]   → PaddleOCR pre-processes → Qwen2.5-VL vision extraction → Validation Engine
4. Validation Engine checks GSTIN, totals, tax consistency, flags anomalies
5. Output Engine returns structured JSON + tabular CSV with confidence metadata
```

---

## 🗂️ Fields Extracted

| Category | Fields |
|---|---|
| **Invoice Metadata** | Invoice Number, Date, Place of Supply |
| **Seller / Supplier** | Name, Address, GSTIN, PAN |
| **Buyer / Customer** | Name, Address, GSTIN |
| **Line Items** | Description, HSN/SAC Code, Quantity, Unit Price, Total |
| **Tax Details** | CGST, SGST, IGST, Cess, Taxable Value |
| **Financials** | Subtotal, Discount, Freight, Grand Total, Currency |
| **Payment Info** | Payment Mode, Bank Details, Due Date |

---

## 📦 Technology Stack

| Layer | Technology |
|---|---|
| **AI / VLM** | Qwen2.5-VL-7B-Instruct (Apache 2.0) |
| **OCR** | PaddleOCR (Apache 2.0) |
| **PDF Parsing** | pdfplumber |
| **Structured Data** | pandas, openpyxl |
| **Quantization** | bitsandbytes (4-bit / 8-bit) |
| **Backend API** | FastAPI + Uvicorn |
| **Frontend** | HTML + Vanilla JS (upload portal) |
| **Output Formats** | JSON, CSV |
| **Runtime** | Python 3.10+ |

---

## 🗓️ Implementation Plan

| Phase | Duration | Tasks |
|---|---|---|
| **Phase 1 — Setup** | Day 1 | Environment setup, model download, pipeline scaffolding |
| **Phase 2 — Structured Pipeline** | Day 2 | Excel/CSV parser + validation engine |
| **Phase 3 — VLM Integration** | Day 3–4 | Qwen2.5-VL integration, prompt engineering for GST extraction |
| **Phase 4 — OCR Pipeline** | Day 5 | PaddleOCR preprocessing + handwritten invoice testing |
| **Phase 5 — Validation Engine** | Day 6 | GSTIN regex checks, tax math validation, anomaly flagging |
| **Phase 6 — UI + Output** | Day 7 | FastAPI endpoints, upload portal, JSON/CSV export |
| **Phase 7 — Testing** | Day 8 | End-to-end tests across all input types, edge case handling |

---

## 📤 Expected Output

```json
{
  "invoice_number": "INV-2025-00782",
  "invoice_date": "2025-03-15",
  "seller": {
    "name": "Sharma Traders Pvt. Ltd.",
    "gstin": "27AABCS1429B1ZB",
    "address": "123, MG Road, Pune, Maharashtra - 411001"
  },
  "buyer": {
    "name": "Ravi Enterprises",
    "gstin": "29AABCR1234C1Z5",
    "address": "45, Brigade Road, Bengaluru, Karnataka - 560001"
  },
  "line_items": [
    {
      "description": "Office Chair",
      "hsn_code": "9401",
      "quantity": 10,
      "unit_price": 2500.00,
      "total": 25000.00
    }
  ],
  "tax": {
    "cgst": 2250.00,
    "sgst": 2250.00,
    "igst": 0.00,
    "total_tax": 4500.00
  },
  "grand_total": 29500.00,
  "currency": "INR",
  "validation": {
    "gstin_valid": true,
    "tax_calculation_match": true,
    "flagged_fields": []
  },
  "confidence_score": 0.94,
  "source_type": "scanned_image"
}
```

---

## 📈 Scalability & Future Scope

- **Batch Processing**: Accept ZIP archives of multiple invoices and process in parallel using Python `multiprocessing`
- **Model Fine-tuning**: Fine-tune Qwen2.5-VL on a GST-specific dataset using QLoRA for higher domain accuracy
- **Multi-language Support**: PaddleOCR supports Hindi/regional language invoices — can be enabled for Tier-2/3 markets
- **VYOM+ API Integration**: Output JSON is designed to be directly ingestible into VYOM+'s voucher creation workflow
- **Cloud Deployment**: Containerize with Docker; deploy on any GPU-enabled cloud (GCP, AWS, Azure)

---

## ⚠️ Expected Challenges

| Challenge | Mitigation Strategy |
|---|---|
| **Handwritten invoice quality** | PaddleOCR pre-processing + VLM's vision capabilities for layout understanding |
| **Variable invoice formats** | Prompt engineering with few-shot examples across invoice types |
| **GSTIN / HSN code accuracy** | Post-extraction regex + checksum validation |
| **Model inference speed** | 4-bit quantization via bitsandbytes; batch inference where possible |
| **Missing or ambiguous fields** | Explicit confidence scoring + flagging uncertain extractions |
| **Multi-page PDFs** | Page-by-page processing with context aggregation |

---

## 📦 Dependencies

```
transformers>=4.40.0
qwen-vl-utils
paddlepaddle
paddleocr
pdfplumber
pandas
openpyxl
fastapi
uvicorn
bitsandbytes
pillow
torch>=2.0.0
```

---

## 👥 Team

> Hacktober Fest 4 — Problem Statement 3  
> **VYOM+ End-to-End AI-Powered GST Invoice Intelligence System**

---

## 📄 License

This project is open-source and will be released under the **MIT License**.

---

> *Built with ❤️ for Hacktober Fest 4 — Open Source AI Hackathon*
