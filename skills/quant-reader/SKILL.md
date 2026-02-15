---
name: quant-reader
description: Read and extract insights from quantitative finance papers and books (PDFs). Extracts abstract, methodology, formulas, data sources, backtesting results, and generates actionable trading notes. Triggers on "read paper", "analyze PDF", "quant paper", "extract insights", "summarize paper", "read chapter", "trading strategy paper", "factor model paper".
argument-hint: <pdf-path> [section] [pages] - e.g., "paper.pdf", "paper.pdf methodology", "book.pdf pages 45-65"
user-invocable: true
allowed-tools: Bash(python3 *), Bash(pdftotext *), Read, Grep, Glob
---

# Quant Reader

Extract structured insights from quantitative finance papers and books (PDFs).

**Primary tool**: Claude's built-in `Read` tool (natively reads PDFs with page ranges).
**Fallback**: `pdfplumber` for dense table extraction, `pypdf` for metadata.
**Output format**: Structured markdown following [extraction-template.md](extraction-template.md).
**Examples**: See [examples.md](examples.md).

## Prerequisites

The `Read` tool handles most PDFs natively. For advanced table extraction:

```bash
pip install pdfplumber pypdf
# Optional: poppler-utils for layout-preserving text
brew install poppler  # macOS
```

## Argument Parsing

Parse `$ARGUMENTS` to extract the PDF path, optional section filter, and optional page range.

| Pattern | Meaning |
|---------|---------|
| `paper.pdf` | Full structured extraction |
| `paper.pdf methodology` | Deep dive into methodology only |
| `paper.pdf formulas` | Extract formulas and model specifications |
| `paper.pdf tables` | Extract and format all tables |
| `paper.pdf results` | Key findings and backtesting results |
| `paper.pdf actionable` | Implementation notes and trading insights only |
| `book.pdf pages 100-120` | Extract from specific page range |
| `paper.pdf abstract` | Abstract and core thesis only |
| `paper.pdf limitations` | Limitations, caveats, and red flags |

If no section filter is provided, produce the full structured extraction.

## Workflow

### Phase 1: Reconnaissance

Always start here. Read the first few pages to understand the document.

1. **Read pages 1-3** using the `Read` tool with `pages: "1-3"` to get title, abstract, and structure
2. **Get metadata** (optional, if title/authors unclear from reading):
```python
python3 -c "
from pypdf import PdfReader
r = PdfReader('$PDF_PATH')
m = r.metadata
print(f'Pages: {len(r.pages)}')
print(f'Title: {m.title}' if m and m.title else 'Title: (not in metadata)')
print(f'Author: {m.author}' if m and m.author else 'Author: (not in metadata)')
"
```
3. **Determine document type**:
   - Short paper (≤20 pages): read entirely
   - Long paper (21-40 pages): read in two passes
   - Book/monograph (>40 pages): requires page range

### Phase 2: Reading Strategy

Choose strategy based on document length and user request.

**Short paper (≤20 pages):**
- Read entire PDF: `Read(file_path, pages="1-20")`

**Long paper (21-40 pages):**
- Pass 1: `Read(file_path, pages="1-20")`
- Pass 2: `Read(file_path, pages="21-40")`

**Book or monograph (>40 pages):**
- If user specified page range: read that range in 20-page chunks
- If no page range specified:
  1. Read pages 1-5 to find the table of contents
  2. Present the TOC to the user
  3. Ask which chapter/section to analyze
  4. Then read the specified range

**Section-specific request:**
- For `methodology`, `results`, `tables`: still read broadly first (Phase 1), then focus on the relevant pages
- Use the abstract and section headings from Phase 1 to locate the target section

### Phase 3: Structured Analysis

After reading the content, produce output following the template in [extraction-template.md](extraction-template.md).

Fill each section:

**Metadata**
- Extract title, authors, date, publication venue, DOI/arXiv ID from first page

**Abstract / Core Thesis**
- Locate the abstract (usually page 1)
- Distill into 2-3 sentences capturing the main claim and contribution

**Methodology**
- Identify the methods/model section
- Document: model specification, estimation technique, assumptions, robustness checks
- Note whether it's cross-sectional, time-series, panel, or event study

**Key Formulas**
- Extract all important mathematical expressions
- Define every variable
- Note equation numbers for reference
- Use LaTeX-style notation: `$r_{i,t} = \alpha_i + \beta_i f_t + \epsilon_{i,t}$`

**Data Sources**
- Identify datasets (CRSP, Compustat, Bloomberg, TEJ, etc.)
- Document time periods, asset universes, data frequency
- Note any filters or exclusions applied

**Key Findings**
- Extract main results with effect sizes
- Include statistical significance (t-stats, p-values, R-squared)
- Note in-sample vs out-of-sample results separately

**Tables & Figures**
- For each important table/figure: describe what it shows and key takeaways
- For tables with numerical data that the Read tool garbles, use the table extraction fallback (see below)

**Backtesting Results**
- Extract strategy performance metrics:
  - Annualized return, Sharpe ratio, information ratio
  - Max drawdown, volatility
  - Turnover, transaction costs
  - In-sample vs out-of-sample periods

**Implementation Notes** (the key value-add)
- What data feeds are needed to implement this?
- Signal construction steps
- Rebalancing frequency and methodology
- Estimated capacity and decay
- Transaction cost considerations
- Implementation complexity: simple / moderate / complex
- Known pitfalls from practical experience

**Limitations & Caveats**
- What the authors acknowledge
- Red flags to watch for:
  - Data mining / p-hacking indicators
  - Survivorship bias
  - Lookahead bias
  - Unrealistic assumptions (zero transaction costs, unlimited shorting)
  - Narrow time period or asset universe

**References of Interest**
- Key papers cited that are worth reading next
- Note why each is relevant

### Phase 4: Section-Specific Deep Dive

When the user requests a specific section (e.g., `/quant-reader paper.pdf methodology`), skip unrelated sections and go deep on the requested one.

**Methodology deep dive:**
- Full model specification with all equations
- Every assumption (stated and unstated)
- Step-by-step estimation procedure
- All robustness checks and alternative specifications
- Comparison to prior approaches in the literature

**Formulas deep dive:**
- Every equation with derivation context
- Variable definitions with units
- Parameter estimation details
- Connections between equations

**Results deep dive:**
- All result tables reproduced or described in detail
- Statistical significance for every finding
- Subperiod analysis, robustness to different specifications
- Economic vs statistical significance

**Actionable deep dive:**
- Detailed implementation roadmap
- Data pipeline requirements
- Signal construction pseudocode
- Portfolio construction rules
- Risk management overlay
- Monitoring and evaluation framework

## Paper Type Guidance

Adjust focus based on paper type:

| Paper Type | Focus Areas |
|-----------|-------------|
| **Factor investing** | Factor construction, sort methodology, long-short returns, factor correlations, factor crowding |
| **Risk models** | Covariance estimation, shrinkage methods, risk decomposition, factor risk vs idiosyncratic |
| **Portfolio optimization** | Objective function, constraints, rebalancing frequency, turnover controls |
| **Statistical arbitrage** | Signal construction, mean reversion speed, holding periods, pair/basket selection |
| **ML in finance** | Feature engineering, train/test split methodology, overfitting controls, feature importance, walk-forward validation |
| **Options/derivatives** | Pricing model, Greeks, calibration procedure, hedging strategy, vol surface |
| **Market microstructure** | Order book data, bid-ask spreads, execution costs, market impact, optimal execution |
| **Crypto/DeFi** | Protocol mechanics, on-chain data sources, liquidity considerations, regulatory risks |

## Table Extraction Fallback

When the `Read` tool output shows garbled or misaligned tabular data, use `pdfplumber`:

```python
python3 -c "
import pdfplumber
with pdfplumber.open('$PDF_PATH') as pdf:
    for i in range($START_PAGE, $END_PAGE):
        page = pdf.pages[i]
        tables = page.extract_tables()
        for idx, table in enumerate(tables):
            print(f'--- Page {i+1}, Table {idx+1} ---')
            for row in table:
                print(' | '.join(str(c or '') for c in row))
            print()
"
```

Use this selectively — only when you detect that a table is important but the Read tool couldn't parse it cleanly.

## Error Handling

| Error | Response |
|-------|----------|
| File not found | Report error, ask user for correct path. Suggest using Glob to search. |
| Password-protected PDF | Inform user the PDF is encrypted. Ask for password or an unprotected copy. |
| Scanned/image-only PDF | The Read tool handles images natively but quality varies. If text extraction fails, suggest OCR with `pytesseract`. |
| >40 pages, no range | Show TOC or structure from first few pages. Ask user which section to analyze. |
| Not a quant paper | Still extract what's possible. Note that the structured template may not fully apply. Adapt sections to the content. |
| pypdf/pdfplumber not installed | Fall back to Read tool only. Note that table extraction may be less precise. |

## Usage Examples

```
# Full extraction
/quant-reader ~/papers/aqr-value-momentum.pdf

# Methodology only
/quant-reader ~/papers/fama-french-five-factor.pdf methodology

# Book chapter
/quant-reader ~/books/advances-financial-ml.pdf pages 45-65

# Just the trading insights
/quant-reader ~/papers/momentum-crashes.pdf actionable

# Extract tables
/quant-reader ~/papers/risk-parity.pdf tables
```
