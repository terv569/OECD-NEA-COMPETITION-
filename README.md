# OECD NEA Coding Competition — Risk Register Normalizer

**Team Submission | March 2026**

## Overview

This project was built for the OECD Nuclear Energy Agency (NEA) Coding Competition. The goal was to build an automated pipeline that reads diverse, inconsistently formatted risk registers (Excel, PDF) and converts them into standardised, machine-readable Excel files using Python and NLP techniques.

## Competition Brief

Participating teams were provided with five raw input risk registers and three corrected output risk registers. The task was to build an algorithm that ingests the input files and produces standardised outputs matching the corrected register format. Two of the five files (4 and 5) served as blind tests and form the final submission.

Grading criteria:
- 60% — Automated comparison against correct outputs (field-level accuracy)
- 40% — Human expert review (logical coherence and completeness)

## Submission Files

| File | Description |
|------|-------------|
| `4. Moorgate Crossrail Register (Final).xlsx` | Blind test output — Moorgate Crossrail project |
| `5. Corporate Risk Register (Final).xlsx` | Blind test output — Corporate PDF register |
| `code/model.py` | Self-contained Python pipeline |

## How It Works

The pipeline runs in 9 stages (Jupyter notebook cells):

### Cell 1 — Setup
Installs and imports all required libraries: `openpyxl`, `pandas`, `pdfplumber`, `python-docx`.

### Cell 2 — File Upload
Creates `input/` and `output/` directories, copies the 5 input files from the local source folder, and verifies all files are present before proceeding.

### Cell 3 — Inspect Outputs
Reads the 3 corrected output files to establish the gold-standard schema — column headers, sheet structure, and mandatory fields.

### Cell 4 — Parsers
Three parser functions extract raw structured content from each file type:
- `parse_xlsx()` — reads all sheets row by row via `openpyxl`
- `parse_pdf()` — extracts tables and text per page via `pdfplumber`
- `parse_docx()` — extracts paragraphs and tables via `python-docx`

### Cell 5 — Normalisation Utilities
Pure-logic helper functions with no external dependencies:
- `to_num()` — converts qualitative labels (e.g. "Possible", "Minor") and numeric strings to float 1–10
- `to_pri()` — derives "Low", "Med", or "High" from a label or likelihood × impact score
- `to_date()` — parses multiple date formats to a standard `datetime.date`
- `derive_output_name()` — maps input filenames to correct output filenames

### Cell 6 — Extraction
File-specific extraction logic for all 5 inputs:

| File | Format | Key Challenge |
|------|--------|---------------|
| 1. IVC DOE R2 | `.xlsx` | Multi-row merged headers, pre and post mitigation columns, 5-point severity/frequency scale |
| 2. City of York Council | `.xlsx` | Column order differs from output schema |
| 3. Digital Security IT | `.xlsx` | Missing project stage and category — inferred from description |
| 4. Moorgate Crossrail | `.xlsx` | Qualitative likelihood/impact labels converted to numeric |
| 5. Corporate Risk Register | `.pdf` | Rotated column headers, multi-column table layout across 21 pages |

### Cell 7 — Validation
Compares extracted data for files 1–3 against the known correct outputs. Checks row count match and field coverage (%) for all mandatory columns. All 3 files passed at 100% row match.

### Cell 8 — Excel Writer
Writes all 5 standardised output files with:
- `Simplified Register` sheet — styled with dark blue headers, alternating row fills, frozen panes, auto-sized columns
- `Output Requirements` sheet — mandatory column specification as provided in the competition brief
- Pre/post mitigation columns for files that have them (files 1 and 5)
- Result column added where source data contains outcome notes (files 2 and 5)

### Cell 9 — Quality Check
Final validation pass across all 5 output files confirming both sheets exist, data is present, and all mandatory columns are populated.

## Output Schema

Every output file contains these mandatory columns:

| Column | Notes |
|--------|-------|
| Date Added | Where available in source |
| Risk ID | Assigned sequentially if not present |
| Risk Description | Full text, newlines preserved |
| Project Stage | Pre-construction / Construction / Operations etc. |
| Project Category | Inferred from description where blank |
| Risk Owner | Person or team responsible |
| Likelihood (1-10) | Normalised to numeric scale |
| Impact (1-10) | Normalised to numeric scale |
| Risk Priority (low, med, high) | Computed from score or mapped from label |
| Mitigating Action | Full mitigation text |

Files 1 and 5 additionally include pre-mitigation and post-mitigation column variants.

## Dependencies

```
openpyxl
pandas
pdfplumber
python-docx
```

Install with:
```bash
pip install openpyxl pandas pdfplumber python-docx
```

## Running the Code

```bash
# Option 1 — Jupyter Notebook (recommended)
# Run cells 1 through 9 in order

# Option 2 — standalone script
export LLM_API_KEY="your-key"   # only needed if using model.py with LLM fallback
python model.py
```

Input files go in `input/`, outputs are written to `output/`.

## Repository Structure

```
├── code/
│   └── model.py                          # standalone pipeline script
├── input/
│   ├── 1. IVC DOE R2 (Input).xlsx
│   ├── 2. City of York Council (Input).xlsx
│   ├── 3. Digital Security IT Sample Register (Input).xlsx
│   ├── 4. Moorgate Crossrail Register (Input).xlsx
│   └── 5. Corporate_Risk_Register (Input).pdf
├── output/
│   ├── 1. IVC DOE R2 (Final).xlsx
│   ├── 2. City of York Council (Final).xlsx
│   ├── 3. Digital Security IT Sample Register (Final).xlsx
│   ├── 4. Moorgate Crossrail Register (Final).xlsx        ← submission
│   └── 5. Corporate Risk Register (Final).xlsx            ← submission
├── notebooks/
│   └── pipeline.ipynb                    # Jupyter notebook (cells 1–9)
├── README.md
└── Attestation.pdf
```

## Validation Results

| File | Extracted Rows | Correct Rows | Match |
|------|---------------|--------------|-------|
| 1. IVC DOE R2 | 33 | 33 | ✓ |
| 2. City of York Council | 45 | 45 | ✓ |
| 3. Digital Security IT | 3 | 3 | ✓ |
| 4. Moorgate Crossrail | 10 | — | blind test |
| 5. Corporate Risk Register | 15 | — | blind test |

