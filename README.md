# PharmaCo Data Product Portal (ODPS)

A reference implementation of a **Customer 360 data product** for a global pharmaceutical company, combining the **[Data Product Portal](https://github.com/conveyordata/data-product-portal)** (Dataminded/Conveyor) operational pattern with the **[Open Data Product Specifications](https://opendataproducts.org/)** (ODPS / ODPC / ODPG / ODPV).

The product unifies HCP/HCO engagement, prescribing, transfer-of-value, consent, and segmentation data into a single governed, consent-aware view consumed by Sales, Medical Affairs, Marketing, and Compliance.

## Interactive Portal

A self-contained, single-file portal visualizes the entire portfolio. **Open [`portal.html`](portal.html) in any browser** — no build step, no server, no dependencies beyond a CDN-loaded D3.js for the graph.

The portal renders 15 views: dashboard, an interactive product graph, product/dataset/output catalogs, use cases, KPIs, objectives, signals, governance policies, domains, access scopes, AI agents, the common data model, and the delivery roadmap.

## Repository Structure

```
.
├── portal.html                       # Self-contained interactive portal (open in browser)
├── plan.txt                          # Project plan: architecture, governance, roadmap, risks
│
├── portal_config/                    # Data Product Portal native config (operational)
│   ├── data_product_glossary.yaml    # Product registration: envs, access scopes, tooling
│   ├── datasets.yaml                 # Input dataset definitions (6 datasets)
│   └── data_outputs.yaml             # Output surface definitions (4 outputs)
│
├── catalog/                          # Open Data Product Specification artifacts
│   ├── odps-c360-hcp.yaml            # ODPS 4.1 spec for the flagship C360 product
│   ├── odps-onekey.yaml              # ODPS spec: OneKey HCP/HCO master
│   ├── odps-rx-feed.yaml             # ODPS spec: IQVIA prescription claims
│   ├── odps-consent.yaml             # ODPS spec: OneTrust consent records
│   ├── odps-nba.yaml                 # ODPS spec: Next-Best-Action model
│   ├── odps-medinfo.yaml             # ODPS spec: Medical Information inquiries
│   ├── odpc-pharma-commercial.yaml   # ODPC catalog: product refs, use cases, objectives, signals
│   ├── odpg-pharma-c360.yaml         # ODPG graph: nodes + edges across the ecosystem
│   ├── odpv-pharma-extension.yaml    # ODPV vocabulary: 26 pharma-specific terms
│   ├── policies/                     # Governance policy definitions
│   │   ├── gdpr.yaml
│   │   ├── hipaa.yaml
│   │   ├── sunshine-act.yaml
│   │   ├── efpia-code.yaml
│   │   └── iqvia-license.yaml
│   ├── domains/                      # Organizational domain definitions
│   │   ├── commercial.yaml
│   │   ├── medical.yaml
│   │   ├── privacy.yaml
│   │   └── compliance.yaml
│   └── agents/                       # AI agent definitions
│       ├── field-rep-copilot.yaml
│       └── msl-assistant.yaml
│
└── cdm/
    └── tables.md                     # Common Data Model: 14 tables across 3 layers
```

## The Open Data Product Standards

| Standard | File(s) | Purpose |
|----------|---------|---------|
| **ODPS** (Specification) | `catalog/odps-*.yaml` | Machine-readable description of each data product — contract, KPIs, SLAs, data quality, access ports |
| **ODPC** (Catalog) | `catalog/odpc-pharma-commercial.yaml` | Portfolio context — product references, use cases, business objectives, signals |
| **ODPG** (Graph) | `catalog/odpg-pharma-c360.yaml` | Relationship graph connecting products, KPIs, policies, domains, and agents for impact analysis and GraphRAG discovery |
| **ODPV** (Vocabulary) | `catalog/odpv-pharma-extension.yaml` | Pharma-specific term definitions (HCP, HCO, ToV, Consent, Pharmacovigilance, etc.) |

## Data Products

| Product | ID | Version | Type | Owner |
|---------|-----|---------|------|-------|
| Customer 360 - HCP & HCO | `DP-C360-HCP-001` | 1.0.0 | derived | Commercial Data & Analytics |
| OneKey HCP/HCO master | `DP-MDM-ONEKEY-001` | 2.3.0 | source-aligned | MDM Operations |
| Prescription claims feed | `DP-RX-FEED-001` | 1.4.0 | source-aligned | External Data Ops |
| Consent records (OneTrust) | `DP-CONSENT-001` | 1.2.0 | source-aligned | Privacy Office |
| Next-Best-Action | `DP-NBA-MODEL-001` | 0.9.0 | derived | Data Science - Commercial |
| Medical Information inquiries | `DP-MEDINFO-001` | 1.0.0 | source-aligned | Medical Information |

## Governance

The product is governed by 5 policies enforced through the Portal's access scopes:

- **GDPR** (EU/EEA) — data residency, right-to-be-forgotten, consent-based processing
- **HIPAA** (US) — patient pseudonymization, IAM boundary separation, PHI audit logging
- **Sunshine Act** (US) — annual CMS Open Payments ToV reporting with KMS-signed ledger
- **EFPIA Disclosure Code** (EU5) — country-level transfer-of-value disclosure
- **IQVIA License** (Global) — third-party data usage restrictions (IQ-2025-MASTER, IQ-2025-ONEKEY)

Four access scopes (`c360_read_commercial`, `c360_read_medical`, `c360_read_compliance`, `c360_read_pv`) translate into IAM, Unity Catalog, and Snowflake roles.

## Key Outcomes Targeted

- **+15%** commercial productivity via Next-Best-Action for field reps
- **100%** Sunshine Act / EFPIA-compliant transfer-of-value reporting
- **Sub-24h** adverse-event signal correlation for pharmacovigilance
- **360°** HCP profile available across Sales, Medical, Marketing, and Compliance

## Validation

All YAML files are syntactically valid and internally consistent — every `$ref` link in the graph and catalog resolves to an existing file, and all entity IDs are consistent across the ODPS/ODPC/ODPG artifacts.

```bash
# Validate all YAML parses
python3 -c "import yaml,glob; [yaml.safe_load(open(f)) for f in glob.glob('**/*.yaml',recursive=True)]"
```

## License

Internal reference implementation. The ODPV pharma vocabulary extension is proposed under Apache 2.0; IQVIA-sourced data fields are subject to their respective license terms.
