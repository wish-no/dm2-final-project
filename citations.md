# Data Sources & Citations

## Source 1 — Online Retail II (transactions) — Stream source
- Origin: UCI Machine Learning Repository, "Online Retail II" data set, donated by Daqing Chen.
- Real transactions (2009–2011) from a UK-based online retailer.
- URL: https://archive.ics.uci.edu/dataset/502/online+retail+ii
- Used as the streaming source, ingested incrementally via Auto Loader. Only a small sample (~4.5k rows) was used; size is not the focus, methodology is.

  Accessed: June 2026

## Source 2 — Country → Continent mapping (reference) — Object source
- Countries-Continents list (columns: `Continent`, `Country`).
- Source: dbouquin/IS_608 repository (CUNY MS Data Science coursework), standard continental groupings.
- Repo: https://github.com/dbouquin/IS_608/blob/master/NanosatDB_munging/Countries-Continents.csv
- Raw: https://raw.githubusercontent.com/dbouquin/IS_608/master/NanosatDB_munging/Countries-Continents.csv
- Used as the static object source to add a `region` column via a join on country name.

  Accessed: June 2026

## Note on data generation
No data was fabricated. The test file (`sales_part2_TEST.csv`) consists of real rows
sampled from the Online Retail II dataset and held back to demonstrate the pipeline
reacting to new data — which the project instructions explicitly permit
("generating data can be done based on the used dataset/data source").
