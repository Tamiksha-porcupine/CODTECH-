# SQL Internship — From Basics to Advanced

> A hands-on SQL internship project covering core database concepts, advanced querying, Python-SQL integration via SQLAlchemy, and database backup & restoration — all applied to a real-world video game dataset.

---

## About This Project

This repository documents my SQL internship journey, progressing from foundational database concepts to advanced querying and Python integration. Each task builds on the previous one, culminating in a full pipeline: raw data → structured tables → complex queries → Python automation → backup & restore.

**Tech Stack:** `MySQL` · `Python` · `SQLAlchemy` · `Pandas` · `Jupyter Notebook`

---

##  Repository Structure

```
 sql-internship/
├── TASK_1.ipynb              # Database setup & basic SQL queries
├── TASK_2_SQL.ipynb           # Advanced queries: JOINs, CTEs, Window Functions
├── TASK_3_SQL_SQLAlchemy.ipynb   # Python + MySQL via SQLAlchemy
└── TASK_4_SQL_RESTORING.ipynb    # Database backup & restoration
```

---

## Task Breakdown

###  Task 1 — Database Setup & Core SQL
**Tools:** MySQL Workbench / MySQL CLI

- Created a `sets` database with two relational tables:
  - **`imdb`** — video game metadata: `Sr_No`, `Name`, `Year`, `Certificate`, `Rating`, `Votes`
  - **`genres`** — boolean genre flags per game: `Action`, `Adventure`, `Comedy`, `Crime`, `Fantasy`, `Mystery`, `Sci_Fi`, `Thriller`
- Populated tables with 10 top-rated video games (God of War, Red Dead Redemption II, GTA V, etc.)
- Practiced `SELECT`, `WHERE`, `ORDER BY`, and basic filtering

**Sample Data:**

| Sr_No | Name | Year | Rating | Votes |
|-------|------|------|--------|-------|
| 4 | God of War | 2018 | 9.9 | 26,118 |
| 2 | Red Dead Redemption II | 2018 | 9.7 | 35,703 |
| 8 | The Last of Us | 2013 | 9.7 | 60,590 |

---

### Task 2 — Advanced SQL: JOINs, CTEs & Window Functions
**Tools:** MySQL CLI

Applied advanced SQL techniques to extract meaningful insights from the relational dataset.

**Key Concepts Covered:**

**1. JOIN Types** — All four join types demonstrated on `imdb_backup` + `genres`:
- `INNER JOIN` — matched records only
- `LEFT JOIN` — all games, even those missing genre data
- `RIGHT JOIN` — all genre entries, with NULL game info if unmatched
- `FULL OUTER JOIN` (via `JOIN`) — combined result of both sides

Genres were flattened from 9 boolean columns into one readable string using `CONCAT_WS` + `CASE WHEN`:
```sql
CONCAT_WS(', ',
  CASE WHEN gen.Action = 'TRUE' THEN 'Action' ELSE NULL END,
  CASE WHEN gen.Adventure = 'TRUE' THEN 'Adventure' ELSE NULL END,
  ...
) AS Genres
```

**2. CTEs + Window Functions** — Multi-step CTE pipeline to find the top-rated game per year:
```sql
WITH Combined AS (
  SELECT i.Sr_No, i.Name, i.Year, i.Rating, i.Votes, g.Action, g.Adventure, g.Fantasy
  FROM imdb_backup i JOIN genres g ON i.Sr_No = g.Sr_No
),
GenreLabelled AS (
  SELECT *, CONCAT_WS(', ', ...) AS Genres FROM Combined
),
TopGames AS (
  SELECT *, RANK() OVER (PARTITION BY Year ORDER BY Rating DESC) AS RankInYear
  FROM GenreLabelled
)
SELECT Year, Name AS 'Top Game', Rating, Votes, Genres
FROM TopGames WHERE RankInYear = 1 ORDER BY Year;
```

**Output:** Top-rated game per release year — e.g., God of War (9.9) in 2018, The Last of Us (9.7) in 2013.

---

### Task 3 — Python + MySQL via SQLAlchemy
**Tools:** Python · SQLAlchemy 2.0 · mysql-connector-python · Pandas · Jupyter Notebook

Bridged MySQL and Python for programmatic database interaction.

**What Was Done:**
- Connected to MySQL using a SQLAlchemy engine
- Read both `imdb` and `genres` tables directly into Pandas DataFrames
- Created an `imdb_backup` table in MySQL by writing a DataFrame back using `to_sql()`

```python
from sqlalchemy import create_engine
import pandas as pd

engine = create_engine("mysql+mysqlconnector://root:password@localhost/sets")

imdb_df = pd.read_sql("SELECT * FROM imdb", con=engine)
imdb_df.to_sql("imdb_backup", con=engine, if_exists="replace", index=False)
```

**Skills Demonstrated:** ORM engine setup · `read_sql()` · `to_sql()` · error handling with try/except

---

### Task 4 — Database Backup & Restoration
**Tools:** Python · SQLAlchemy · Pandas · CSV

Implemented a complete backup and restore pipeline for database resilience.

**Backup — Tables → CSV:**
```python
for table in ["imdb", "genres"]:
    df = pd.read_sql(f"SELECT * FROM {table}", con=engine)
    df.to_csv(f"{table}_backup.csv", index=False)
```

**Restore — CSV → MySQL (with error handling):**
- Encountered and resolved a real-world foreign key constraint error:  
  `Cannot drop table 'imdb' referenced by foreign key constraint 'genres_ibfk_1'`
- Solved by restoring in the correct dependency order (parent table before child)

```python
df = pd.read_csv("imdb_backup.csv")
df.to_sql("imdb", con=engine, if_exists="replace", index=False)
```

**Skills Demonstrated:** Data durability · FK constraint debugging · CSV-based backup strategy · restore sequencing

---

## Key SQL Concepts Learned

| Concept | Applied In |
|---------|------------|
| DDL — CREATE, USE | Task 1 |
| DML — SELECT, WHERE, ORDER BY | Task 1 |
| INNER / LEFT / RIGHT / FULL JOIN | Task 2 |
| CONCAT_WS + CASE WHEN | Task 2 |
| CTEs (WITH clause) | Task 2 |
| Window Functions — RANK() OVER (PARTITION BY) | Task 2 |
| SQLAlchemy engine & ORM | Task 3 |
| pandas read_sql / to_sql | Task 3 & 4 |
| Backup to CSV / Restore from CSV | Task 4 |
| Foreign Key constraint handling | Task 4 |

---

##  Setup & Run

**Prerequisites:**
```
Python 3.12+
MySQL Server (local)
Anaconda / pip
```

**Install dependencies:**
```bash
pip install sqlalchemy mysql-connector-python pandas jupyter
```

**Configure your connection** in the notebooks:
```python
engine = create_engine("mysql+mysqlconnector://YOUR_USER:YOUR_PASSWORD@localhost/sets")
```

**Run notebooks:**
```bash
jupyter notebook
```

---

##  Dataset

A curated dataset of **10 top-rated video games** with ratings, votes, and multi-genre classification.

Titles include: *God of War, Red Dead Redemption II, Grand Theft Auto V, The Last of Us, Spider-Man, Injustice 2, Horizon Forbidden West, Detroit: Become Human, The Last of Us: Part II, Death Stranding*

---

## Skills Demonstrated

`MySQL` `SQL Joins`  `Window Functions` `SQLAlchemy` `Pandas` `Python` `Database Design` `Backup & Recovery` `Jupyter Notebooks` 

---


*This internship project was completed as part of a structured SQL learning program, progressing from database basics through to Python-integrated data pipelines.*
