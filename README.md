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

- **Tenders:** real line-level data from Costa Rica's public health system (CCSS), published openly through the national procurement system. Costa Rica is the proof of concept. The problem itself is general.
- **Portfolio:** built from scratch using only public sources, namely FDA data (`data/fda-drugs-database/`) and public manufacturer catalogs (`data/pharma-portfolios/`).
