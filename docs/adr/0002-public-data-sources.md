# Use only public data sources, with FDA NDC as the master reference

The system is built and evaluated only on public data, so the project can be shared and reproduced without access to any company's private portfolio or tender history. The portfolio comes from public Portfolio Catalogs joined by NDC Code to the FDA National Drug Code Directory, which serves as the structured master reference. Tenders come from real CCSS Tender Lines published through SICOP. Because real Tender Lines have no known correct Verdict, accuracy is measured on a synthetic set generated from real Presentations.

## Sources

Portfolio:

- Hikma US Injectable Catalog (June 2025): https://www.hikma.com/media/fvndz2im/hikma-injectable-product-catalog-june-2025.pdf (from https://www.hikma.com/en-us/products/)
- Baxter U.S. Supply Availability Reports, with an NDC Code per product: https://support.baxter.com/en/resources/product-supply-resources/
- FDA National Drug Code Directory (bulk download): https://www.fda.gov/drugs/drug-approvals-and-databases/national-drug-code-directory

Tenders:

- SICOP: https://www.sicop.go.cr/app/
- Observatorio de Compra Pública (historical SICOP procurement and award data, no login): https://www.observatoriocomprapublica.go.cr/acerca-del-observatorio/
- CCSS LOM, searchable online: https://www.ccss.sa.cr/lom
- CCSS LOM 2025 (PDF): https://www.binasss.sa.cr/farmacologia/NORMATIVALOM.pdf

Reference knowledge (agent tools):

- RxNorm API (Reference Brands, Salt versus base): https://lhncbc.nlm.nih.gov/RxNav/APIs/RxNormAPIs.html
- DailyMed web services (full label details such as preservative-free): https://dailymed.nlm.nih.gov/dailymed/app-support-web-services.cfm
- FDA Purple Book (Biosimilars and their reference products, CSV/XLSX): https://purplebooksearch.fda.gov/downloads

## Consequences

- The demo company is a composite of public catalogs, not a real company's portfolio.
- SICOP's open-data download page (https://www.sicop.go.cr/moduloPcont/pcont/rp/CE_MOD_DATOSABIERTOSVIEW.jsp) returned an error when checked. Bulk export of Tender Lines is not yet confirmed.
- The online LOM has no bulk download. The binasss.sa.cr PDF is the downloadable version.
