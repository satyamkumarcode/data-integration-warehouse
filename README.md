# Data integration, record matching and data profiling

Assignment 3 of **INFOMDW Data Wrangling**, MSc Applied Data Science, Utrecht University (2026–27).
Group 8: Theodora Kyriakou, Satyam Kumar, Sultonbek Abdulloev, Sonja Wellman.

Everything runs from one notebook: [`group8_assignment3.ipynb`](group8_assignment3.ipynb). The full write-up is in [`report/Report_Group8_Assignment3.pdf`](report/Report_Group8_Assignment3.pdf).

## Task 1: Record matching (DBLP vs ACM)

Find which papers in two bibliographic databases are the same paper.

- 2,294 ACM records × 2,616 DBLP records = **6,001,104 pairs** compared
- Similarity per field: Levenshtein (title), Jaro (authors), affine gap (venue), exact match (year), written by hand; the Levenshtein and Jaro versions were checked against a library (largest difference 0.0)
- Weighted score with a threshold: **2,560 pairs reported, 2,201 correct**
- **Precision 0.86, recall 0.99**, run time **30.5 s** (a pure-Python version was estimated at 37 minutes)

## Task 2: Data integration (two sources → one warehouse)

Two SQLite sources describe the same Dutch football clubs and players in different formats:

| | Source A (`source_a_league.db`) | Source B (`source_b_transfers.db`) |
|---|---|---|
| Name | `Jordan Henderson` | `Henderson, Jordan` |
| Birth date | `1990-06-17` | `17/06/1990` |
| Height | `182` (cm) | `1.82` (m) |
| Club | `1` → Ajax | `AJX` → AFC Ajax |

The ETL (extract → staging, transform, load) builds `warehouse.db`:

- **club** and **club_map** (both sources' club ids map to one key)
- **player** and **player_source** (lineage: which source rows each player came from)
- **rejected** and **conflict_log** (bad rows and disagreements are recorded, not dropped)

Result: 12 + 12 source players → **20 players, 4 found in both sources**; 12 clubs, 20 club mappings, 24 lineage rows. Reconciliation checks pass.

Robustness tests with deliberately bad data: unmapped team, unsplittable names and a missing birth date go to `rejected`; a simulated crash before commit leaves the warehouse unchanged; an invalid club key is refused by the foreign key.

## Task 3: Data profiling and preparation

UK Department for Transport road collision data for 2025 (**101,525 rows**): summary statistics, distinct values, constancy, Benford's law, disguised missing values (for example 25,105 `-1` codes in `second_road_number`), imputation and correlation.

## Run it

```bash
pip install -r requirements.txt
jupyter notebook group8_assignment3.ipynb
```

Run the notebook from the repository root; it reads `Task1/`, `Task3/` and the `.db` files with relative paths.

## Data sources

- **DBLP-ACM**: entity resolution benchmark from the Database Group, University of Leipzig.
- **Road safety data**: UK Department for Transport, published under the Open Government Licence v3.0.
- The two football sources are small SQLite databases (10 clubs and 12 players each) included in this repository.

The paper used as background reading for Task 3 is not included here; see the reference in the notebook.
