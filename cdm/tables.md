# Customer 360 Pharma - Common Data Model

Three layers: **HUB** (master), **ENGAGEMENT** (interactions), **COMMERCIAL** (outcomes).
All times in UTC. All money in USD with `currency_iso` column where applicable.
SCD-2 tables carry `valid_from`, `valid_to`, `is_current`.

---

## HUB LAYER

### 1. `dim_hcp` - Healthcare Professional master

| column                 | type         | pk | nullable | description                                              |
|------------------------|--------------|----|----------|----------------------------------------------------------|
| hcp_id                 | string       | Y  | N        | Internal golden ID                                       |
| onekey_id              | string       |    | Y        | IQVIA OneKey ID (primary cross-ref)                      |
| npi                    | string(10)   |    | Y        | US National Provider Identifier                          |
| rpps                   | string(11)   |    | Y        | France Repertoire Partage des Professionnels de Sante    |
| gmc_number             | string       |    | Y        | UK General Medical Council number                        |
| first_name             | string       |    | N        | Given name                                               |
| last_name              | string       |    | N        | Family name                                              |
| primary_specialty_code | string       |    | N        | Coded specialty (taxonomy: NUCC US, RPPS FR)             |
| primary_specialty_name | string       |    | N        | Human label                                              |
| email                  | string       |    | Y        | Professional email                                       |
| country_iso            | string(2)    |    | N        | ISO 3166-1 alpha-2                                       |
| valid_from             | timestamp    | Y  | N        | SCD-2 effective from                                     |
| valid_to               | timestamp    |    | Y        | SCD-2 effective to (NULL when current)                   |
| is_current             | boolean      |    | N        | True if row is current                                   |
| consent_status         | string       |    | N        | Materialized from dim_consent (latest)                   |
| match_confidence       | decimal(3,2) |    | N        | 0.00-1.00 identity resolution score                      |

**Quality rules**: `npi` unique among `country_iso='US'`; `onekey_id` recommended.

---

### 2. `dim_hco` - Healthcare Organization master

| column            | type         | pk | nullable | description                                       |
|-------------------|--------------|----|----------|---------------------------------------------------|
| hco_id            | string       | Y  | N        | Internal golden ID                                |
| onekey_hco_id     | string       |    | Y        | IQVIA OneKey HCO ID                               |
| dea_number        | string       |    | Y        | US DEA registration                               |
| hco_name          | string       |    | N        | Legal organization name                           |
| hco_type          | string       |    | N        | hospital / clinic / pharmacy / group_practice     |
| hco_class_of_trade| string       |    | Y        | Class of trade (academic, IDN, community, etc.)   |
| address_line1     | string       |    | N        | Street                                            |
| city              | string       |    | N        | City                                              |
| region            | string       |    | Y        | State / region / Land                             |
| postal_code       | string       |    | N        | Postal / ZIP                                      |
| country_iso       | string(2)    |    | N        | ISO 3166-1 alpha-2                                |
| parent_hco_id     | string       |    | Y        | FK self-ref for parent org                        |
| valid_from        | timestamp    | Y  | N        | SCD-2                                             |
| valid_to          | timestamp    |    | Y        | SCD-2                                             |
| is_current        | boolean      |    | N        | SCD-2 current flag                                |

---

### 3. `dim_hcp_hco_affiliation` - HCP-HCO membership over time (SCD-2)

| column           | type      | pk | nullable | description                                              |
|------------------|-----------|----|----------|----------------------------------------------------------|
| affiliation_id   | string    | Y  | N        | Surrogate key                                            |
| hcp_id           | string    |    | N        | FK -> dim_hcp.hcp_id                                     |
| hco_id           | string    |    | N        | FK -> dim_hco.hco_id                                     |
| affiliation_role | string    |    | N        | attending / consulting / employed / privileges           |
| is_primary       | boolean   |    | N        | True if primary affiliation                              |
| valid_from       | date      |    | N        | Effective from                                           |
| valid_to         | date      |    | Y        | Effective to (NULL if current)                           |
| is_current       | boolean   |    | N        | SCD-2 current flag                                       |
| source           | string    |    | N        | onekey / claims_inference / self_attest                  |

---

### 4. `dim_product` - PharmaCo drug / SKU master

| column            | type         | pk | nullable | description                                       |
|-------------------|--------------|----|----------|---------------------------------------------------|
| product_id        | string       | Y  | N        | Internal product ID                               |
| brand_name        | string       |    | N        | Marketed brand                                    |
| generic_name      | string       |    | N        | INN / USAN                                        |
| therapeutic_area  | string       |    | N        | TA (oncology, cardio, immuno, etc.)               |
| atc_code          | string       |    | Y        | WHO ATC code                                      |
| ndc_codes         | array<string>|    | Y        | US NDC SKUs                                       |
| ema_product_id    | string       |    | Y        | EU EMA product ID                                 |
| launch_date       | date         |    | Y        | First marketing date                              |
| status            | string       |    | N        | active / discontinued / paused                    |

---

### 5. `dim_geography` - Country / region / territory hierarchy

| column         | type      | pk | nullable | description                                         |
|----------------|-----------|----|----------|-----------------------------------------------------|
| geography_id   | string    | Y  | N        | Surrogate                                           |
| country_iso    | string(2) |    | N        | ISO 3166-1                                          |
| region_code    | string    |    | Y        | Internal region (e.g. EU5, NA, APAC)                |
| state_province | string    |    | Y        | State or province                                   |
| territory_code | string    |    | Y        | Sales territory                                     |
| territory_name | string    |    | Y        | Human label                                         |

---

### 6. `dim_consent` - Consent records (OneTrust)

| column              | type         | pk | nullable | description                                          |
|---------------------|--------------|----|----------|------------------------------------------------------|
| consent_id          | string       | Y  | N        | OneTrust consent record ID                           |
| hcp_id              | string       |    | N        | FK -> dim_hcp.hcp_id                                 |
| channel             | string       |    | N        | email / phone / sms / inperson / web                 |
| purpose             | string       |    | N        | marketing / scientific / regulatory                  |
| opt_in_status       | string       |    | N        | granted / denied / withdrawn / pending               |
| consent_collected_at| timestamp    |    | N        | When consent was captured                            |
| valid_from          | timestamp    |    | N        | Effective from                                       |
| valid_to            | timestamp    |    | Y        | Effective to (NULL if current)                       |
| is_current          | boolean      |    | N        | Current row flag                                     |
| jurisdiction        | string       |    | N        | GDPR / HIPAA / PIPEDA / etc.                         |
| evidence_url        | string       |    | Y        | Pointer to signed consent record (S3 with KMS)       |

---

## ENGAGEMENT LAYER

### 7. `fact_engagement` - Unified interactions (calls, email, web, events)

| column           | type      | pk | nullable | description                                              |
|------------------|-----------|----|----------|----------------------------------------------------------|
| engagement_id    | string    | Y  | N        | Surrogate                                                |
| hcp_id           | string    |    | N        | FK -> dim_hcp                                            |
| hco_id           | string    |    | Y        | FK -> dim_hco (location of engagement)                   |
| product_id       | string    |    | Y        | FK -> dim_product (product discussed)                    |
| channel          | string    |    | N        | rep_call / email / web / event / phone / chatbot         |
| sub_channel      | string    |    | Y        | f2f / virtual / approved_email / etc.                    |
| direction        | string    |    | N        | inbound / outbound                                       |
| occurred_at      | timestamp |    | N        | When the interaction happened                            |
| duration_seconds | integer   |    | Y        | Duration if measurable                                   |
| rep_id           | string    |    | Y        | Internal rep / MSL ID                                    |
| rep_role         | string    |    | Y        | sales_rep / msl / kam                                    |
| consent_id       | string    |    | N        | FK -> dim_consent at time of interaction                 |
| outcome_code     | string    |    | Y        | Coded outcome (interested / declined / question / etc.)  |
| campaign_id      | string    |    | Y        | Source campaign                                          |
| source_system    | string    |    | N        | veeva / sfmc / web_tracking / etc.                       |
| consent_valid    | boolean   |    | N        | Derived: was consent valid at occurred_at?               |

---

### 8. `fact_sample_disbursement` - Drug samples (Sunshine Act source)

| column            | type      | pk | nullable | description                                          |
|-------------------|-----------|----|----------|------------------------------------------------------|
| disbursement_id   | string    | Y  | N        | Surrogate                                            |
| hcp_id            | string    |    | N        | Receiving HCP                                        |
| hco_id            | string    |    | Y        | Location                                             |
| product_id        | string    |    | N        | Product sampled                                      |
| ndc_code          | string    |    | N        | Specific SKU                                         |
| quantity_units    | integer   |    | N        | Units handed                                         |
| disbursed_at      | timestamp |    | N        | When                                                 |
| rep_id            | string    |    | N        | Rep responsible                                      |
| signature_evidence| string    |    | N        | URL to e-signature PDF                               |
| reportable        | boolean   |    | N        | True if reportable under Sunshine Act / EFPIA        |

---

### 9. `fact_transfer_of_value` - Meals, fees, travel (ToV reporting)

| column            | type         | pk | nullable | description                                       |
|-------------------|--------------|----|----------|---------------------------------------------------|
| tov_id            | string       | Y  | N        | Surrogate                                         |
| hcp_id            | string       |    | N        | Recipient                                         |
| hco_id            | string       |    | Y        | Recipient organization (if HCO-level)             |
| tov_category      | string       |    | N        | meal / honorarium / travel / consulting / royalty |
| amount            | decimal(12,2)|    | N        | Amount transferred                                |
| currency_iso      | string(3)    |    | N        | ISO 4217                                          |
| amount_usd        | decimal(12,2)|    | N        | Converted to USD at FX of transfer date           |
| transferred_at    | date         |    | N        | Date of transfer                                  |
| nature_of_payment | string       |    | N        | CMS Open Payments code                            |
| product_id        | string       |    | Y        | Related product (if applicable)                   |
| reporting_year    | integer      |    | N        | Sunshine Act / EFPIA reporting year               |
| reported_status   | string       |    | N        | pending / reported / disputed / withdrawn         |
| signature_kms_key | string       |    | N        | KMS key alias used to sign the row                |
| signature_hash    | string       |    | N        | SHA-256 of canonical row + signature              |

---

### 10. `fact_medical_inquiry` - Med-info questions

| column            | type      | pk | nullable | description                                          |
|-------------------|-----------|----|----------|------------------------------------------------------|
| inquiry_id        | string    | Y  | N        | Surrogate                                            |
| hcp_id            | string    |    | Y        | FK -> dim_hcp (may be unknown for new HCPs)          |
| product_id        | string    |    | N        | Product asked about                                  |
| topic_code        | string    |    | N        | Coded topic (dosing / safety / interaction / etc.)   |
| received_at       | timestamp |    | N        | When question received                               |
| responded_at      | timestamp |    | Y        | When response sent                                   |
| channel           | string    |    | N        | phone / email / portal                               |
| ae_suspected      | boolean   |    | N        | True if forwarded to pharmacovigilance               |
| pv_case_id        | string    |    | Y        | FK -> pharmacovigilance case (external)              |
| inquiry_text_url  | string    |    | Y        | S3 pointer; access scope c360_read_medical only      |

---

## COMMERCIAL LAYER

### 11. `fact_prescription` - Rx claims (third-party, restricted)

| column            | type         | pk | nullable | description                                       |
|-------------------|--------------|----|----------|---------------------------------------------------|
| rx_id             | string       | Y  | N        | Surrogate                                         |
| hcp_id            | string       |    | N        | Prescriber FK                                     |
| hco_id            | string       |    | Y        | Location FK                                       |
| product_id        | string       |    | N        | Product FK                                        |
| ndc_code          | string       |    | N        | Dispensed SKU                                     |
| patient_pseudo_id | string       |    | N        | SHA-256 pseudonym; HIPAA boundary                 |
| rx_date           | date         |    | N        | Prescription date                                 |
| dispensed_date    | date         |    | Y        | Pharmacy dispense date                            |
| quantity          | decimal(12,3)|    | N        | Dispensed quantity                                |
| days_supply       | integer      |    | Y        | Days supply                                       |
| payer_channel     | string       |    | Y        | commercial / medicare / medicaid / cash           |
| source_vendor     | string       |    | N        | iqvia / komodo / symphony                         |
| source_load_at    | timestamp    |    | N        | When PharmaCo loaded the record                   |

**Licensing**: bound by IQVIA contract; access scope `c360_read_commercial` only,
with `payer_channel`, `source_vendor` redacted for medical-only scope.

---

### 12. `fact_segmentation` - Decile/segment per HCP per product

| column           | type         | pk | nullable | description                                    |
|------------------|--------------|----|----------|------------------------------------------------|
| segmentation_id  | string       | Y  | N        | Surrogate                                      |
| hcp_id           | string       |    | N        | FK                                             |
| product_id       | string       |    | N        | FK                                             |
| as_of_date       | date         |    | N        | Snapshot date                                  |
| decile           | integer      |    | N        | 1-10                                           |
| segment_code     | string       |    | N        | A / B / C / Adopter / Loyalist / etc.          |
| score            | decimal(6,3) |    | N        | Underlying score                               |
| model_version    | string       |    | N        | Algorithm version (e.g. seg-2026Q1-v3)         |

---

### 13. `fact_nba_recommendation` - Next-Best-Action scores

| column            | type         | pk | nullable | description                                  |
|-------------------|--------------|----|----------|----------------------------------------------|
| recommendation_id | string       | Y  | N        | Surrogate                                    |
| hcp_id            | string       |    | N        | Target HCP                                   |
| product_id        | string       |    | N        | Product context                              |
| recommended_action| string       |    | N        | rep_call / email / event_invite / etc.       |
| channel           | string       |    | N        | Suggested channel                            |
| priority_score    | decimal(5,3) |    | N        | 0.000-1.000                                  |
| rank              | integer      |    | N        | Within HCP's queue at as_of                  |
| reason_codes      | array<string>|    | N        | Explanation tags                             |
| as_of             | timestamp    |    | N        | When computed                                |
| model_version     | string       |    | N        | NBA model version                            |
| consent_filter_applied | boolean |    | N        | True if consent narrowed the channels        |

---

### 14. `fact_adverse_event_link` - Link engagement -> PV case

| column         | type      | pk | nullable | description                                          |
|----------------|-----------|----|----------|------------------------------------------------------|
| link_id        | string    | Y  | N        | Surrogate                                            |
| pv_case_id     | string    |    | N        | External PV case identifier                          |
| engagement_id  | string    |    | Y        | FK -> fact_engagement (if originated)                |
| inquiry_id     | string    |    | Y        | FK -> fact_medical_inquiry (if originated)           |
| hcp_id         | string    |    | N        | Reporter HCP                                         |
| product_id     | string    |    | N        | Suspected product                                    |
| reported_at    | timestamp |    | N        | When linked                                          |
| linkage_method | string    |    | N        | automatic_keyword / manual / pv_reverse_lookup       |

**Access**: scope `c360_read_pv` only.

---

## CROSS-TABLE NOTES

- All FK relationships are enforced in the gold layer via Delta/Iceberg
  constraints where supported, and validated by Soda checks otherwise.
- The patient layer (`patient_pseudo_id`) is a one-way pseudonym. The crosswalk
  to identifiable patient records lives in a separate Glue database with its
  own IAM boundary and is **never** exposed via this product.
- Every fact table joins to `dim_consent` on `(hcp_id, channel, occurred_at)`
  and is filtered by `opt_in_status='granted'` for outbound use cases.
