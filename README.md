# Pharma Tender Matcher

An AI system that checks, line by line, whether products in a pharmaceutical portfolio satisfy the lines of a public health tender, and drafts a bid recommendation for human review.

## The Problem

Pharmaceutical companies that sell to hospitals and public health systems must respond to tenders with hundreds or thousands of product lines. Each line describes the requested drug in unstandardized free text, with different units, languages, and naming conventions. Deciding line by line whether a portfolio product satisfies the request is slow and error-prone. This affects the commercial and tender teams that decide which lines to bid on.

## What It Does

The system converts both sides of the comparison (portfolio products and tender lines) into the same structured schema, then checks each tender line against the portfolio. The output is a draft recommendation for human review. Nothing is ever submitted automatically.

A typical interaction: the user provides a tender (a list of requested product lines). The system returns a verdict for each line (exact, equivalent, no match, or needs review) with the matching product and a reason, plus a tender-level summary of the share of lines the company could realistically bid on.

### How it works

1. **Product master model.** An LLM atomizes each portfolio product description into structured fields: active principles (a list, since products may have several), salt, strength per component, fill volume, total amount, dosage form, route, container, dose type, physical form (solid or solution), preservative-free, solvent, and pack conversion.

   Example: *Bendamustine Hydrochloride Injection, 100 mg / 4 mL (25 mg/mL), Multiple-Dose Vial* becomes a record with bendamustine as the active principle, hydrochloride as the salt, 25 mg/mL, a 4 mL fill, a vial container, and a solution form.

2. **Tender atomization.** Each tender line goes through the same extraction, so both sides share one schema.

3. **Deterministic normalization.** Code, not the LLM, converts units within a family (mass/volume to mg/mL, amount per container to mg) and checks consistency (concentration × fill volume = total amount). Units outside those families, such as IU, mEq, or % w/w, are kept as they are and never converted by guessing.

4. **Matching rule: does my product cover the request?** The customer's request is followed strictly. A product that exceeds the request still counts as a match, for example a preservative-free product when preservative-free wasn't required. Rules are defined per field. Some fields must match exactly, such as active principles, salt, and strength.

5. **Verdict per line:** exact, equivalent, no match, or needs review, each with a reason. A value missing from the catalog (unknown) is treated differently from a requirement missing from the tender (not requested).

6. **Agent for ambiguous cases.** Deterministic matching resolves the easy lines. An agent with tools (search the portfolio, normalize units, get product details) handles only the ambiguous ones, with a second reviewer agent and, as a last resort, a person.

7. **Tender-level summary:** the share of lines the company could bid on, with lines pending human review reported separately.

The system judges technical compliance only. Registration status, pricing, and supply are business decisions left to the tender team. Historical awarded prices are reported next to each verdict as input for pricing and never change the verdict. See [docs/adr/0001-technical-compliance-only.md](docs/adr/0001-technical-compliance-only.md). Domain terms are defined in [CONTEXT.md](CONTEXT.md).

### Scope

Injectables and biologics/biosimilars.

## Setup

1. Install uv if you don't have it yet: https://docs.astral.sh/uv/getting-started/installation/

2. Clone this repository (or download the zip and extract it).

3. Create a `.env` file from the template and add your API key:

       cp .env.example .env

4. Install dependencies:

       uv sync

5. Start Jupyter:

       uv run jupyter notebook

## Notebooks

- `notebooks/01-setup.ipynb` - smoke test that confirms your environment works
- `notebooks/02-rag.ipynb` - a minimal RAG baseline you can adapt to your own data

## Data

All data comes from public sources. The system uses three kinds of data: the **portfolio** (what the company sells), the **tenders** (what the customer asks for), and **reference knowledge** used to normalize both. A synthetic set with known answers is used for evaluation. Terms in bold follow the glossary in [CONTEXT.md](CONTEXT.md).

### Portfolio

| Source | What it provides | How I get it | Status |
| --- | --- | --- | --- |
| Manufacturer catalogs (Baxter, Hikma; Pfizer and Fresenius planned) | The demo company portfolio: one row per **Presentation**, in free text, often with an NDC code | Public PDFs downloaded from each manufacturer's website into `data/pharma-portfolios/<company>/`, then extracted to text | Baxter and Hikma collected. The file in `pfizer/` is a copy of Baxter's report, so Pfizer and Fresenius still need real catalogs |
| FDA National Drug Code (NDC) directory | The structured master reference. `product.xls` holds one row per **Product** (active ingredients, strength, dosage form, route); `package.xls` holds one row per **Presentation** (container, **Fill Volume**, **Units per Pack**) | Bulk download `ndcxls.zip` from the FDA NDC page, unzipped into `data/fda-drugs-database/db/`. The `.xls` files are tab-separated text | Collected |

Catalog rows are joined to FDA records by NDC code where one is listed. Each catalog record is then checked against an independent structured source, which gives the extraction step a ground truth. Registration status in Costa Rica is not modeled (see [ADR 0001](docs/adr/0001-technical-compliance-only.md)).

### Tenders

| Source | What it provides | How I get it | Status |
| --- | --- | --- | --- |
| SICOP, Costa Rica's public procurement system | Real CCSS **Tender Lines** in Spanish free text, plus past awards, which provide the **Historical Awarded Price** | Download published CCSS tenders and award records for injectables and biologics. The target is 3 to 5 tenders, about 200 to 500 lines | To collect. Export formats still to be confirmed |
| CCSS official medicines list (*Lista Oficial de Medicamentos*) | The codes and standard descriptions CCSS writes tenders in, including Spanish container and route wording such as "frasco ampolla" or "FA" | Public document from CCSS | To collect. Format still to be confirmed |

Tenders arrive as a **Tender Document** (spreadsheet, PDF, or text). Each line is extracted into the same schema as the portfolio.

### Reference knowledge (called as agent tools)

| Source | What it provides | How I get it |
| --- | --- | --- |
| RxNorm API (NLM) | Maps a **Reference Brand** to its **Active Principle** (Xylocaine → lidocaine), separates the ingredient with **Salt** from the base, and normalizes dose forms and routes | Free REST API at rxnav.nlm.nih.gov, no key needed |
| DailyMed API (NLM) | Full label text by NDC: preservative-free, single- or multiple-dose, and "equivalent to X mg base" wording. It fills fields that are **Unknown** in the catalog | Free REST API at dailymed.nlm.nih.gov, no key needed |
| FDA Purple Book | Biologics with their reference products and **Biosimilars**, which the NDC files do not link | Public download from the FDA Purple Book site |

### Evaluation set (synthetic)

Real tender lines do not come with a correct answer, so I generate labeled **Tender Lines** from real portfolio **Presentations**, each with a known expected **Verdict**. Changes applied:

- Spanish translation and source synonyms ("FA", "intravenoso")
- A **Reference Brand** instead of the active principle
- A dropped **Defining Field** (expected: Equivalent)
- A changed **Salt** or **Container Type** (expected: No Match)
- Ambiguous "pack size" wording

A few hundred lines are enough to measure accuracy per Verdict and to choose the **Confidence Threshold**. The real SICOP lines are used to check that the system holds up on actual CCSS wording.

### Scope

Injectables and biologics/biosimilars only. Costa Rica is the proof of concept, and the problem itself is general.
