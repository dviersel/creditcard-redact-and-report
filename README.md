# PDF Credit Card Bill Redactor

A Python tool for redacting sensitive transaction information from credit card bill PDFs. Uses true PDF redaction that removes text from the content stream, not just visual overlay.

## Features

- **True redaction** - Removes text from PDF content stream (not just black boxes)
- **Configurable matching** - Substring and exact match patterns
- **Dual mode operation**:
  - `default: none` - Redact only matching transactions (blocklist)
  - `default: all` - Redact everything except matching transactions (allowlist)
- **Exchange rate handling** - Automatically redacts related currency conversion lines
- **Excel reporting** - Export non-redacted transactions to Excel with currency conversion details
- **Batch processing** - Process multiple PDFs at once
- **Verification** - Confirms redacted text is truly removed from output

## Installation

```bash
pip install -r requirements.txt
```

**Requirements:**
- Python 3.10+
- PyMuPDF (fitz)
- PyYAML
- openpyxl

## Usage

### Basic Usage

```bash
# Redact a single PDF
python redact.py bill.pdf

# Redact multiple PDFs
python redact.py *.pdf --output redacted/

# Preview without modifying (dry run)
python redact.py bill.pdf --dry-run --verbose
```

### With Excel Report

```bash
# Generate Excel report of non-redacted transactions
python redact.py *.pdf --output redacted/ --report transactions.xlsx
```

### Command Line Options

| Option | Description |
|--------|-------------|
| `--config, -c` | Path to config file (default: config.yaml) |
| `--output, -o` | Output directory |
| `--suffix, -s` | Suffix for output files (default: _redacted) |
| `--no-suffix` | Don't add suffix to output files |
| `--dry-run, -n` | Preview without modifying files |
| `--verbose, -v` | Show detailed output |
| `--report` | Generate Excel report of kept transactions |

## Configuration

Create a `config.yaml` file (see `config.example.yaml`):

```yaml
redaction:
  # "none" = redact matching terms, "all" = redact everything except matches
  default: "none"
  color: "#808080"
  include_exchange_rate: true

# Terms to match (case-insensitive substring matching)
exceptions:
  - "SENSITIVE_VENDOR"
  - "PRIVATE_MERCHANT"

# Exact match only (for short terms)
exact_match:
  - "XX"
```

### Mode Examples

**Blocklist mode** (`default: none`):
```yaml
redaction:
  default: "none"
exceptions:
  - "CASINO"
  - "BETTING"
```
→ Only redacts transactions containing "CASINO" or "BETTING"

**Allowlist mode** (`default: all`):
```yaml
redaction:
  default: "all"
exceptions:
  - "GROCERY"
  - "UTILITIES"
```
→ Redacts everything *except* transactions containing "GROCERY" or "UTILITIES"

## Excel Report

The `--report` option generates an Excel file with non-redacted transactions:

| Column | Description |
|--------|-------------|
| Source File | Original PDF filename |
| Date | Transaction date |
| Description | Merchant/vendor name |
| City | Transaction location |
| Country | Country code |
| Foreign Amount | Amount in foreign currency |
| EUR Amount | Amount in EUR |
| Exchange Rate | Currency conversion rate |
| Currency | Foreign currency code |

## How It Works

1. **Text extraction** - Extracts text with position data from PDF
2. **Row grouping** - Groups text spans by Y-coordinate into rows
3. **Transaction detection** - Identifies transaction rows by date pattern
4. **Matching** - Checks description/city against configured terms
5. **Redaction** - Adds redaction annotations covering full cell widths
6. **Application** - Applies redactions, removing text from content stream
7. **Verification** - Reopens saved file to confirm text removal

## Supported PDF Format

Currently optimized for ICS (International Card Services) credit card statements with:
- Dutch date format (e.g., "15 jan.", "02 dec.")
- Transaction rows: Date | Date | Description | City | Country | Amounts
- Exchange rate rows: "Wisselkoers [CURRENCY] [RATE]"

## License

MIT
