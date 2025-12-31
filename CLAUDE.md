# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PDF Credit Card Bill Redactor - A Python tool that performs true PDF redaction (removes text from content stream, not just visual overlay) on ICS credit card statement PDFs based on configurable matching rules.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run redaction on single file
python redact.py bill.pdf

# Batch process with output directory
python redact.py *.pdf --output redacted/

# Dry run with verbose output (preview only)
python redact.py bill.pdf --dry-run --verbose

# Generate Excel report of non-redacted transactions
python redact.py *.pdf --output redacted/ --report transactions.xlsx

# Use custom config
python redact.py bill.pdf --config custom_config.yaml
```

## Architecture

Single-file Python script (`redact.py`) with this processing pipeline:

1. **Text extraction** - `extract_text_spans()` extracts text with bounding boxes using PyMuPDF
2. **Row grouping** - `group_spans_by_row()` clusters spans by Y-coordinate (3px tolerance)
3. **Transaction detection** - `is_transaction_row()` identifies rows starting with Dutch date pattern (e.g., "15 jan.", "02 dec.")
4. **Row parsing** - `parse_transaction_row()` extracts: date1, date2, description, city, country, amounts, cell boundaries
5. **Match logic** - `should_redact()` implements dual-mode matching:
   - `default: none` - redact only matching exceptions (blocklist)
   - `default: all` - redact everything except matching exceptions (allowlist)
6. **Redaction** - Applies PyMuPDF redaction annotations covering full cell widths
7. **Verification** - `verify_redaction()` reopens saved file to confirm text removal

## Key Patterns

- Dutch date regex: `^\d{2}\s+(jan|feb|mrt|apr|mei|jun|jul|aug|sep|okt|nov|dec)\.?$`
- Exchange rate regex: `^Wisselkoers\s+[A-Z]{3}`
- Transaction row format: Date | Date | Description | City | Country | Amounts
- Config supports both substring matching (`exceptions`) and exact matching (`exact_match`)

## Configuration

Uses `config.yaml` (see `config.example.yaml`):
- `redaction.default`: "none" (blocklist) or "all" (allowlist)
- `redaction.color`: RGB list `[0,0,0]` or hex string `"#808080"`
- `redaction.include_exchange_rate`: Also redact currency conversion lines
- `exceptions`: Case-insensitive substring patterns
- `exact_match`: Exact match patterns for short terms
