# Gas Station Inventory Analysis — n8n Workflow

An automated inventory management and financial reporting system for gas stations, built as a single n8n workflow. Zero manual input — the system processes vendor invoices, daily cashier reports, and monthly analysis requests entirely through email.

---

## What It Does

The workflow runs 24/7, scanning Gmail every minute for incoming emails. A Switch node routes each email to the correct processing branch based on subject line:

| Email Type | What Happens |
|---|---|
| Vendor Invoice | OCR extracts PDF → AI parses line items → stored to Google Sheets → confirmation sent |
| Daily Cashier Report | OCR extracts PDF → AI parses sales data → stored to Google Sheets → confirmation sent |
| Monthly Analysis Request | Pulls all stored data → AI crunches financials → HTML report generated → emailed back |
| Unrecognized Email | Auto-reply with correct subject format instructions |

---

## Architecture

**Email-triggered, single-canvas, multi-branch** — one workflow handles all three document types via Switch routing. No subworkflows. Designed as a portable single JSON export for client deployment.
Gmail Trigger (every 1 min)
│
Switch Router
┌────┼────┬────────────┐
Invoice  Sales  Monthly   Unrecognized
Branch  Branch  Analysis   → Error Reply
│      │      │
OCR    OCR   Pull Sheets
│      │      │
AI     AI   Financial
Parse  Parse  Analyst AI
│      │      │
Sheets Sheets Report
Store  Store  Writer AI
│      │      │
Confirm Confirm Email Report

---

## Tech Stack

| Layer | Technology |
|---|---|
| Workflow Platform | n8n Cloud (v2.34.5) |
| Trigger | Gmail OAuth2 (polling, every 1 minute) |
| OCR / File Parsing | n8n built-in `extractFromFile` (PDF → text) |
| AI / LLM | OpenAI GPT-3.5-turbo (4 separate AI agents) |
| Data Storage | Google Sheets (OAuth2) — 3 tabs |
| Notifications | Gmail (outbound confirmations + reports) |
| Logic / Transform | JavaScript Code nodes |

---

## AI Agents

This workflow uses **4 specialized AI agents**, each with a specific role:

1. **Invoice Parser** — extracts vendor, date, line items, and totals from raw OCR text
2. **Sales Parser** — extracts date, line items, shift hours, and total revenue from cashier reports
3. **Financial Analyst** (temp 0.1) — calculates gross profit, margin, sell-through % per item; uses a Calculator tool for precision math
4. **Report Writer** (temp 0.4) — generates a full inline-CSS styled HTML report with summary cards, data tables, color-coded insights, and reorder recommendations

---

## Key Features

- **OCR Quality Gate** — rejects poor scans before AI processing (fails if < 10 words or > 30% special characters)
- **Structured Output Enforcement** — AI responses parsed with autoFix to guarantee clean JSON
- **Sell-Through Categorization** — items automatically classified as Fast Mover (>60%), Slow Mover (20–40%), or Dead Stock (<20%)
- **Top 5 Revenue Items** — computed per month from aggregated sales data
- **Reorder Alerts** — auto-generated for fast movers and low-margin items
- **Error Handling** — every branch has a dedicated error email with instructions for the sender

---

## Data Model (Google Sheets)

| Tab | What's Stored |
|---|---|
| `Invoices` | Vendor, date, line items, totals, OCR quality score |
| `Sales` | Date, shift, line items, total revenue, OCR quality score |
| `Monthly_Analysis` | Month, gross profit, margin, sell-through by item, recommendations |

---

## Test Data Included

| File | Description |
|---|---|
| `test-data/sample-invoice.txt` | Mock vendor invoice — ABC Distributors, $1,438.10, 12 line items |
| `test-data/sample-cashier-report.txt` | Mock cashier shift report — $656.93, 12 products, 6am–10pm shift |

---

## Setup & Deployment

1. Import `gas-station-inventory-analysis.json` into your n8n instance
2. Connect your Gmail account via OAuth2 (trigger + sender nodes)
3. Connect your Google Sheets account via OAuth2
4. Add your OpenAI API key to the credential store
5. Create a Google Sheet with three tabs: `Invoices`, `Sales`, `Monthly_Analysis`
6. Update the Sheets node references to point to your Sheet ID
7. Activate the workflow

> **Note:** Credential IDs in the JSON are internal n8n references. You will need to re-link all credentials to your own accounts after import.

---

## Email Subject Format

The Switch router matches on these exact subject patterns:

| Subject Contains | Route |
|---|---|
| `invoice` | Invoice branch |
| `cashier report` | Sales branch |
| `monthly analysis` | Monthly Analysis branch |
| Anything else | Error reply |

---

## Products Tracked (Sample Data)

Cigarettes (Marlboro, Camel) · Beverages (Coca-Cola, Pepsi, Red Bull, Monster) · Snacks (Doritos, Lay's, Snickers, Jack Links) · Energy (5-Hour Energy) · Alcohol (Bud Light)

---

## Built With

- [n8n](https://n8n.io) — workflow automation platform
- [OpenAI](https://openai.com) — GPT-3.5-turbo
- [Google Sheets](https://sheets.google.com) — data storage
- [Gmail](https://gmail.com) — email trigger and delivery
