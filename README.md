# DHAP-34 — Email Threads Data Pipeline

## Overview

This pipeline ingests two locally downloaded CSV files from SharePoint:

* `email_thread_details.csv`
* `email_thread_summaries.csv`

The data is validated and loaded into PostgreSQL using a Dockerized Airflow setup. All pipeline assets, configuration, and runbook are kept in this directory.

The target tables are:

* `email_thread_details`
* `email_thread_summaries`

## Dataset Purpose

This dataset contains exported email thread metadata and summaries used for reporting and analysis. New CSV files will occasionally be dropped into the Airflow data directory and processed via the DAG.

## Directory Structure

```
extraction/email_threads/
│
├── load_email_threads.py     # Airflow DAG
├── schema.yaml               # Defines schema expectations
├── manifest.yaml             # Dataset metadata
├── ddl.sql                   # PostgreSQL DDL statements
├── README.md                 # This documentation
└── data/                     # (Optional) Local CSV files
```

## CSV File Location

CSV files must be placed in:

```
/opt/airflow/data/
```

Inside the repository, this corresponds to:

```
./data/
```

## Target Database

PostgreSQL database in the Docker environment:

* **Host:** `postgres`
* **User:** `airflow`
* **Password:** `airflow`
* **Port:** `5432`
* **Database:** `airflow`

## Environment Variables

Create `.env` based on `.env.sample` in the root project directory.

Example:

```
AIRFLOW_UID=50000
POSTGRES_USER=airflow
POSTGRES_PASSWORD=airflow
POSTGRES_DB=airflow
```

## Running the Pipeline

### 1. Start Docker Services

```
docker compose up -d
```

This starts:

* Airflow webserver
* Scheduler
* PostgreSQL DB
* Triggerer
* Worker

### 2. Access Airflow UI

Open:

```
http://localhost:8080
```

Default login (unless changed):

```
username: airflow
password: airflow
```

### 3. Enable the DAG

In the Airflow UI:

```
DAGs → load_email_thread_data → Switch ON
```

### 4. Trigger the Pipeline

Click:

```
▶ Trigger DAG
```

### 5. Verify Load in PostgreSQL

Connect inside the running container:

```
docker exec -it <postgres_container_name> psql -U airflow
```

Example:

```sql
SELECT COUNT(*) FROM email_thread_details;
SELECT COUNT(*) FROM email_thread_summaries;
```

## DAG Summary

### Task Flow

```
load_email_data
```

The DAG:

* Reads both CSVs using `pandas`
* Loads them into Postgres using SQLAlchemy `to_sql()`
* Runs on-demand (no schedule)

### Code Entry Point

`load_email_threads.py`

## Troubleshooting

### Error: Table not found

Run DDL manually:

```
psql -U airflow -d airflow -f ddl.sql
```

### Error: CSV not found

Ensure files exist in:

```
/opt/airflow/data/
```

If they were added after the container started, restart:

```
docker compose restart
```

### Error: Invalid credentials

Check `.env` and `docker-compose.yaml` match:

```
POSTGRES_USER
POSTGRES_PASSWORD
POSTGRES_DB
```

### Stuck DAG Run

Reset from Airflow UI:

```
Graph → Right-Click Task → Clear
```

Or delete runs from:

```
Browse → DAG Runs
```

## Runbook

### Updating Schema

1. Modify `schema.yaml`
2. Update table definition in `ddl.sql`
3. Apply:

```
psql -U airflow -d airflow -f ddl.sql
```

4. Commit both schema and DDL together.

### New CSV Drop

When new data is downloaded:

1. Place CSVs into:

```
./data/
```

2. Restart Docker (optional)
3. Trigger the DAG again from UI.

### Adding a New Dataset

Before committing:

* `manifest.yaml` created
* `schema.yaml` defined
* `ddl.sql` verified and tested
* Pipeline runs successfully locally
* Sample CSV included if possible

## Success Criteria (Met)

✔ DAG runs end-to-end: CSV → PostgreSQL
✔ Reproducible via Docker Compose
✔ Credentials stored in environment variables
✔ Clear documentation and runbook
✔ Files stored in standard extraction directory

## Maintainer

Data Engineering Team
LCW
