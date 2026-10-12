# Personal Finance Observability

Self-hosted stack to record personal transactions and chart them: PostgreSQL stores the data, NocoDB provides a spreadsheet-style interface for entering and editing records, and Grafana builds dashboards on top of the database. Everything runs from one `docker compose up`.

Status: work in progress. The stack runs and the dashboards work; ingestion is manual (NocoDB) and there is no provisioning of dashboards yet.

## Architecture

```
NocoDB (:8080)  --writes-->  PostgreSQL  <--reads--  Grafana (:3000)
 data entry                  (finance-db)             dashboards
```

| Service | Image | Role | Memory limit |
| ------- | ----- | ---- | ------------ |
| `db` | `postgres:15` | Persistent storage | 256 MB |
| `nocodb` | `nocodb/nocodb` | Data entry UI | 512 MB |
| `grafana` | `grafana/grafana-oss` | Dashboards | 256 MB |

All services share the `finance-network` bridge network and use `restart: always`. Database and Grafana data are kept in bind-mounted volumes (`postgres_data/`, `grafana_data/`).

## Run

```bash
cp .env.exemplo .env     # set POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB
docker compose up -d
```

- NocoDB: http://localhost:8080 (create the admin account on first visit)
- Grafana: http://localhost:3000 (add a PostgreSQL data source pointing to `db:5432`)

## Data model

A single `transactions` table, managed in NocoDB:

| Column | Type | Notes |
| ------ | ---- | ----- |
| `title` | text | Merchant or description |
| `date_column` | date | |
| `value` | decimal | Negative for expenses |
| `category` | single select | Groceries, Housing & Utilities, Health, Savings, ... |

## Dashboards

Monthly income, expenses and net balance (bar chart) and spending by category, filtered by `year` and `month` dashboard variables. A mock-up with synthetic data is available at https://samanta-bordallo.github.io/resume/finance-demo.html.

## Next steps

- Provision the Grafana data source and dashboards as code (`provisioning/`)
- Import bank statements from CSV instead of typing records
- Pin image versions instead of `latest`
