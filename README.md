# DDR Report Generator — AI-Powered System

An AI workflow that reads property inspection documents and automatically generates a structured, client-ready **Detailed Diagnostic Report (DDR)** in Word format.

---

## What It Does

| Step | What Happens |
|------|-------------|
| 1 | Reads **Inspection PDF** + **Thermal PDF** |
| 2 | Converts all pages to images |
| 3 | Sends images to **Claude AI** for analysis |
| 4 | AI extracts observations, thermal readings, issues |
| 5 | AI generates structured DDR content |
| 6 | Python builds a formatted **Word (.docx) report** with embedded images |

---

## Output Report Structure

The generated DDR contains all 7 required sections:

1. **Property Issue Summary** — overview + color-coded summary table
2. **Area-Wise Observations** — per-area details with site photos + thermal images
3. **Probable Root Cause** — AI-reasoned root cause analysis
4. **Severity Assessment** — HIGH / MEDIUM / LOW with reasoning
5. **Recommended Actions** — prioritized action plan (Immediate / Short-term / Long-term)
6. **Additional Notes** — key contextual observations
7. **Missing or Unclear Information** — all gaps explicitly flagged as "Not Available"

---

## Setup

### 1. Install Python dependencies
```bash
pip install -r requirements.txt
```

### 2. Install system dependency (poppler for PDF conversion)
```bash
# Ubuntu/Debian
sudo apt-get install poppler-utils

# macOS
brew install poppler

# Windows — download from:
# https://github.com/oschwartz10612/poppler-windows
```

### 3. Set your Anthropic API key
```bash
# Option A: Environment variable (recommended)
export ANTHROPIC_API_KEY="sk-ant-your-key-here"

# Option B: Pass via command line flag
python ddr_generator.py ... --api-key sk-ant-your-key-here
```

---

## Usage

```bash
python ddr_generator.py \
  --inspection Sample_Report.pdf \
  --thermal Thermal_Images.pdf \
  --output DDR_Report_Output.docx
```

### All options:
```
--inspection   Path to inspection report PDF      (required)
--thermal      Path to thermal images PDF         (required)
--output       Output .docx filename              (default: DDR_Report_Output.docx)
--api-key      Anthropic API key                  (or set ANTHROPIC_API_KEY env var)
--work-dir     Temp folder for extracted images   (default: ./extracted_images)
```

---

## How the AI Pipeline Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    DDR AI PIPELINE                              │
│                                                                 │
│  INPUT                                                          │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │  Inspection PDF  │    │   Thermal PDF    │                   │
│  └────────┬─────────┘    └────────┬─────────┘                   │
│           │                       │                             │
│           ▼                       ▼                             │
│  ┌────────────────────────────────────────────┐                 │
│  │  PDFExtractor                              │                 │
│  │  • Converts each page to JPEG image        │                 │
│  │  • Resizes for API efficiency              │                 │
│  └────────────────────┬───────────────────────┘                 │
│                       │                                         │
│                       ▼                                         │
│  ┌────────────────────────────────────────────┐                 │
│  │  AIAnalyzer (Claude Vision API)            │                 │
│  │  • analyze_inspection_report()             │                 │
│  │    → Extracts: property info, impacted     │                 │
│  │      areas, checklist findings             │                 │
│  │  • analyze_thermal_report()                │                 │
│  │    → Extracts: hotspot/coldspot temps,     │                 │
│  │      interpretation per scan               │                 │
│  │  • generate_ddr_content()                  │                 │
│  │    → Merges both datasets, generates       │                 │
│  │      full DDR content (all 7 sections)     │                 │
│  └────────────────────┬───────────────────────┘                 │
│                       │                                         │
│                       ▼                                         │
│  ┌────────────────────────────────────────────┐                 │
│  │  DDRDocumentBuilder (python-docx)          │                 │
│  │  • Builds professional Word document       │                 │
│  │  • Embeds site photos per area             │                 │
│  │  • Embeds thermal images per area          │                 │
│  │  • Color-coded severity tables             │                 │
│  │  • Headers, formatting, dividers           │                 │
│  └────────────────────┬───────────────────────┘                 │
│                       │                                         │
│                       ▼                                         │
│  OUTPUT: DDR_Report_Output.docx                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Design Decisions

| Decision | Reason |
|----------|--------|
| Claude Vision API | Can read PDF page images directly — no OCR library needed |
| pdftoppm for extraction | Faster and more reliable than Python PDF libraries |
| Structured JSON prompts | Forces AI to return parseable, consistent output |
| 3-step AI pipeline | Separation of concerns — extract → merge → generate |
| python-docx for output | Produces proper .docx files that open in Word/Google Docs |

---

## Limitations

1. **API cost** — Each run makes ~3 API calls with multiple images (cost: ~$0.10–0.30 per report)
2. **Image matching** — Thermal images are matched to areas by sequence order, not semantic content
3. **Large PDFs** — Capped at 30 pages per PDF for performance; very large reports may need chunking
4. **Handwritten notes** — AI may miss handwritten annotations in PDFs

---

## How to Improve

1. **Smarter image-area matching** — Use CLIP embeddings to semantically match thermal images to inspection areas
2. **Fine-tuned model** — Train on many DDR examples for better domain accuracy
3. **Web UI** — Add a simple Flask/Streamlit front-end for non-technical users
4. **PDF text extraction** — Combine pdfplumber text extraction with vision for higher accuracy
5. **Multi-language support** — Add translation layer for non-English reports
6. **Batch processing** — Process multiple properties in one run

---

## Project Structure

```
ddr_system/
├── ddr_generator.py      ← Main AI pipeline (run this)
├── requirements.txt      ← Python dependencies
├── README.md             ← This file
├── input/                ← Place your PDFs here
│   ├── Sample_Report.pdf
│   └── Thermal_Images.pdf
├── output/               ← Generated reports go here
└── extracted_images/     ← Temp folder (auto-created)
    ├── insp_page-01.jpg
    ├── thermal_page-01.jpg
    ├── inspection_data.json   ← Extracted data (for audit)
    └── thermal_data.json      ← Extracted data (for audit)
```

---

## Tech Stack

- **AI Model**: Claude claude-opus-4-5 (Anthropic) — Vision + Text
- **PDF Processing**: pdftoppm (Poppler) + pdf2image
- **Image Processing**: Pillow (PIL)
- **Document Generation**: python-docx
- **Language**: Python 3.10+

---

*Assignment submission — AI Generalist / Applied AI Builder role*
