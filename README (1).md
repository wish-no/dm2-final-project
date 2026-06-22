# Data Management 2 — Final Project

**Course:** Data Curation and Data Management (Esam Sharaf, SRH University Campus Hamburg, Summer 2026)
**Platform:** Databricks Free Edition · Medallion architecture (Bronze → Silver → Gold)
**Author:** Vishnu Prem Nair

---

## 1. Objective

Combine **two data sources of different types** and report **two business objectives** on a single dashboard, built as a production-style pipeline on Databricks.

| Source | Type | What it is |
|--------|------|-----------|
| Online Retail II transactions | **Stream** (Auto Loader) | Real UK online-retailer transactions, ingested incrementally |
| Country → continent mapping | **Object** (file) | Static reference CSV, used to add a `region` column |

In pipeline terms, the retail transactions are the **dynamic** source — incrementally ingested via Auto Loader and the one the test adds new data to — while the country mapping is the **static** reference lookup. This is the classic **fact + dimension** pattern.

**Business objectives (reported on the dashboard):**
1. **Revenue by region over time** — uses the join between both sources.
2. **Top products by revenue** — plus a Total Revenue KPI.

---

## 2. Architecture (Medallion)

```
Source 1 (Stream)        Source 2 (Object)
 retail CSV via           country_continent.csv
 Auto Loader              (batch file read)
        |                        |
   bronze_sales            bronze_country        <- Bronze: raw, untouched
        \                        /
            silver_sales_enriched                <- Silver: cleaned, curated, joined
              /                  \
 gold_revenue_by_region   gold_top_products      <- Gold: one table per objective
              \                  /
                AI/BI Dashboard                  <- two objectives + KPI
```

- **Bronze** — raw ingest. Transactions via Auto Loader (incremental, checkpointed, schema location); country reference via batch read.
- **Silver** — drop cancellations/returns, compute `line_revenue`, curate country names, join continent → `region`.
- **Gold** — aggregate into the two objective tables the dashboard reads.

### ETL mapping (Extract → Transform → Load)

The full Bronze → Silver → Gold pipeline is the ETL process:

- **Extract** — `01_bronze` ingests both sources: the retail transactions as a stream (Auto Loader) and the country reference via a batch read.
- **Transform** — `02_silver` cleans, curates, and joins the two sources; `03_gold` aggregates the result into the objective tables.
- **Load** — every stage is written to managed Delta tables in Unity Catalog, and the Gold tables are served to the dashboard.

*(Strictly, this is the modern **ELT** variant — raw data is loaded into Bronze first, then transformed in place — which is the standard Databricks / Medallion approach.)*

---

## 3. Notebooks

| Notebook | Role |
|----------|------|
| `00_setup` | Creates the schema and volumes (run once) |
| `prep_slice` | One-time data prep: samples ~4.5k rows across the full dataset, writes the load file + a held-back test file |
| `01_bronze` | Ingests both sources into Bronze (Extract) |
| `02_silver` | Cleans, curates, joins → Silver (Transform) |
| `03_gold` | Builds the two objective tables (Transform / serve) |

The job **`retail_pipeline`** chains `01_bronze → 02_silver → 03_gold` with task dependencies and a **daily** schedule.

---

## 4. How to run

1. Run `00_setup` (creates `workspace.dm2_project` + volumes `landing`, `reference`, `chk`).
2. Upload the retail CSV and `country_continent.csv` to the `reference` volume.
3. Run `prep_slice` (creates `landing/sales/sales_part1.csv` and `reference/sales_part2_TEST.csv`).
4. Run `01_bronze` → `02_silver` → `03_gold` (or run the `retail_pipeline` job).
5. Build / open the AI/BI dashboard on the two gold tables.

---

## 5. Test (data change → dashboard reacts)

The instructions require that adding new data to a source changes the output.

1. Record the baseline: **Total Revenue = £88,353**.
2. Drop the held-back file into the stream folder:
   ```python
   import shutil
   base = "/Volumes/workspace/dm2_project"
   shutil.copy(f"{base}/reference/sales_part2_TEST.csv",
               f"{base}/landing/sales/sales_part2.csv")
   ```
3. Run `retail_pipeline`. Auto Loader ingests only the new file; silver/gold rebuild.
4. Result: **Total Revenue rose to £130,768**, a new region (North America) appeared, and the top-products ranking shifted — the dashboard reacted to the new data.

---

## 6. Data sources

See `citations.md`. No data was fabricated; the test increment is real rows sampled from the cited retail dataset (permitted by the instructions).

---

## 7. Tech / patterns used

ETL (Extract → Transform → Load) · Medallion architecture · Unity Catalog (catalog / schema / volumes) · Auto Loader (incremental, checkpointed ingestion) · idempotent Spark SQL transforms · orchestrated multi-task Job with a daily schedule · native AI/BI dashboard · Git version control · Serverless compute.
