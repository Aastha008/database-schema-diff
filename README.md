# Database Schema Diff

[![Python](https://img.shields.io/badge/python-3.10%2B-3776AB.svg?style=flat-square&logo=python&logoColor=white)](https://www.python.org/downloads/)
[![CI](https://img.shields.io/badge/CI-passing-brightgreen.svg?style=flat-square&logo=githubactions&logoColor=white)](https://github.com/daily-project-factory/database-schema-diff/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg?style=flat-square)](https://github.com/psf/black)

A lightweight CLI utility and Python library for inspecting, comparing, and detecting drift across relational database schemas. Parse live database connections or raw DDL SQL files to generate detailed structural diff reports highlighting missing indexes, altered column types, missing tables, and foreign key mismatches.

---

## 📋 Table of Contents

- [Problem Statement & Target Audience](#-problem-statement--target-audience)
- [Key Features](#-key-features)
- [Architecture & System Design](#-architecture--system-design)
- [Technology Stack](#-technology-stack)
- [Installation & Virtual Environment Setup](#-installation--virtual-environment-setup)
- [Environment Variables Configuration](#-environment-variables-configuration)
- [Usage Guide](#-usage-guide)
  - [Command Line Interface (CLI)](#command-line-interface-cli)
  - [Python API Usage](#python-api-usage)
- [Automated Testing Instructions](#-automated-testing-instructions)
- [Project Structure](#-project-structure)
- [Roadmap & Future Enhancements](#-roadmap--future-enhancements)
- [License](#-license)

---

## 🎯 Problem Statement & Target Audience

### The Problem
Development, staging, and production databases frequently suffer from **silent schema drift**. Hotfixes applied directly to production, missed migration scripts during complex deployments, or disparate local development environments often cause subtle structural divergences. Catching these discrepancies prior to deployment typically requires tedious manual inspection, custom SQL queries, or heavy enterprise software suites.

### Target Audience
* **DevOps & SRE Teams**: Automate CI/CD pipeline checks to prevent deployments when schema drift is detected between target environments.
* **Backend Developers**: Verify that generated migration files accurately reflect domain models before committing code.
* **Database Administrators (DBAs)**: Quickly compare live databases against baseline reference `.sql` DDL files without manual `pg_dump` diffing.
* **Data Engineers**: Ensure source transactional schemas match downstream analytical metadata definitions.

---

## ✨ Key Features

* 🔌 **Live Database Introspection**: Seamlessly inspect live SQLite and PostgreSQL database schemas using SQLAlchemy metadata inspection.
* 📄 **Static DDL Parsing & Normalization**: Parse and normalize static DDL `.sql` scripts into strongly-typed Pydantic schema representations using `sqlglot`.
* ⚡ **Deterministic Core Diff Engine**: Isolate missing tables, columns, constraints, foreign key mismatches, primary key drift, and field type changes between target and source schemas.
* 🖥️ **Interactive Terminal Interface**: Render rich, color-coded diff summaries, side-by-side table breakdowns, and structured outputs powered by `Typer` and `Rich`.
* 🤖 **CI/CD Pipeline Ready**: Supports strict exit code flags (`--strict`) to fail CI runner builds automatically when unintended schema drift is detected.
* 🔄 **Programmatic API**: Utilize the core engine as a standalone Python library or access programmatic comparison handlers via FastAPI endpoints.

---

## 🏗️ Architecture & System Design

`Database Schema Diff` decouples schema ingestion from structural diff calculations, enabling cross-dialect and cross-medium comparisons (e.g., comparing a live PostgreSQL instance directly against a static `.sql` snapshot).

```
 ┌──────────────────────┐        ┌──────────────────────┐
 │ Live Database (DB)   │        │   DDL SQL File       │
 │ (SQLite/PostgreSQL)  │        │    (schema.sql)      │
 └──────────┬───────────┘        └──────────┬───────────┘
            │                               │
            ▼                               ▼
 ┌──────────────────────────────────────────────────────┐
 │                 Schema Loaders                       │
 │      (SQLAlchemy Inspection / sqlglot Parser)        │
 └──────────────────────────┬───────────────────────────┘
                            │
                            ▼
 ┌──────────────────────────────────────────────────────┐
 │              Normalized Schema Domain Models         │
 │          (Pydantic: Database, Table, Column)         │
 └──────────────────────────┬───────────────────────────┘
                            │
                            ▼
 ┌──────────────────────────────────────────────────────┐
 │               Core Diff Engine                       │
 │      (Identifies Missing, Modified, Extra Nodes)     │
 └──────────────────────────┬───────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
 ┌──────────────────────┐        ┌──────────────────────┐
 │ Rich Terminal / CLI  │        │ FastAPI / Web API    │
 │ (Color Output / Exit)│        │  (JSON Diff Report)  │
 └──────────────────────┘        └──────────────────────┘
```

### Module Responsibilities

| Module | Purpose |
| :--- | :--- |
| `src/models.py` | Pydantic domain models for database schemas, tables, columns, indexes, foreign keys, and structural diff reports. |
| `src/loaders.py` | Schema extraction adapters for live databases via SQLAlchemy inspection and DDL SQL files via `sqlglot` parsing. |
| `src/core.py` | Deterministic diff calculation engine comparing source and target schemas to detect missing entities and type/constraint drift. |
| `src/presenter.py` | Rich library output formatting for color-coded terminal summaries, side-by-side table comparisons, and JSON export logic. |
| `src/api.py` | Programmatic API interface and REST routes for remote schema analysis and pipeline integrations. |
| `src/main.py` | Typer CLI application entry point handling user commands, parameter parsing, and pipeline exit codes. |

---

## 💻 Technology Stack

* **Language**: Python 3.10+
* **CLI Framework**: [Typer](https://typer.tiangolo.com/)
* **Terminal UI**: [Rich](https://rich.readthedocs.io/)
* **Data Validation**: [Pydantic v2](https://docs.pydantic.dev/)
* **Database Inspection**: [SQLAlchemy 2.0+](https://www.sqlalchemy.org/)
* **SQL Parsing**: [sqlglot](https://github.com/tobymao/sqlglot)
* **Testing & Quality**: [pytest](https://docs.pytest.org/), [Black](https://github.com/psf/black)

---

## ⚙️ Installation & Virtual Environment Setup

### Prerequisites
* Python 3.10 or higher installed.

### Step-by-Step Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/daily-project-factory/database-schema-diff.git
   cd database-schema-diff
   ```

2. **Create and Activate a Virtual Environment**
   ```bash
   # On macOS/Linux
   python3 -m venv venv
   source venv/bin/activate

   # On Windows
   python -m venv venv
   .\venv\Scripts\activate
   ```

3. **Install Dependencies**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

---

## 🔧 Environment Variables Configuration

The application reads configuration from environment variables or a local `.env` file.

1. **Copy the example environment file**:
   ```bash
   cp .env.example .env
   ```

2. **Configure values in `.env`**:
   ```env
   # Default Source Database Connection String
   SOURCE_DB_URL=postgresql://user:password@localhost:5432/source_db

   # Default Target Database Connection String
   TARGET_DB_URL=postgresql://user:password@localhost:5432/target_db

   # App Settings
   STRICT_MODE=true
   LOG_LEVEL=INFO
   ```

---

## 🚀 Usage Guide

### Command Line Interface (CLI)

The CLI tool allows comparing live database connections, static SQL files, or cross-comparing DDL against a live connection.

#### 1. Compare Two Live Databases
```bash
python -m src.main compare-live \
  --source "sqlite:///./dev.db" \
  --target "sqlite:///./prod.db"
```

#### 2. Compare Two Static DDL SQL Files
```bash
python -m src.main compare-ddl \
  --source ./schemas/v1_schema.sql \
  --target ./schemas/v2_schema.sql
```

#### 3. Cross-Compare a DDL File against a Live Database (CI Pipeline Check)
```bash
python -m src.main compare-cross \
  --ddl ./schemas/canonical_schema.sql \
  --db "postgresql://user:pass@localhost:5432/staging_db" \
  --strict
```
> **Note**: When `--strict` is passed, the CLI returns exit code `1` if any structural schema drift is detected, making it ideal for CI/CD integration.

#### 4. Export Diff Output to JSON
```bash
python -m src.main compare-ddl \
  --source ./schemas/v1_schema.sql \
  --target ./schemas/v2_schema.sql \
  --format json \
  --output ./reports/diff_report.json
```

---

### Python API Usage

You can embed the diff engine directly into your custom Python backend application or migration workflow:

```python
from src.loaders import SchemaLoader
from src.core import SchemaDiffEngine
from src.presenter import DiffPresenter

# 1. Load schemas from live connections or files
source_schema = SchemaLoader.from_database("sqlite:///./staging.db")
target_schema = SchemaLoader.from_ddl_file("./schemas/production.sql")

# 2. Compute structural difference
engine = SchemaDiffEngine(source=source_schema, target=target_schema)
diff_report = engine.compute_diff()

# 3. Check for drift programmatically
if diff_report.has_drift:
    print(f"Drift detected! {len(diff_report.missing_tables)} missing tables.")
    
    # Render rich summary in terminal
    presenter = DiffPresenter(diff_report)
    presenter.render_terminal_summary()
```

---

## 🧪 Automated Testing Instructions

The suite contains unit and integration tests covering loader parsing, diff calculation accuracy, edge cases (nullable mismatches, index drops), and API responses.

### Run All Tests
```bash
pytest
```

### Run Tests with Coverage Report
```bash
pytest --cov=src --cov-report=term-missing
```

### Run Specific Test Modules
```bash
# Core diff engine tests
pytest tests/test_core.py

# API endpoint tests
pytest tests/test_api.py
```

---

## 📁 Project Structure

```
database-schema-diff/
├── .env.example            # Sample environment configuration file
├── .gitignore              # Git ignore rules
├── requirements.txt        # Production and development dependencies
├── src/
│   ├── __init__.py         # Package entry point
│   ├── api.py              # Programmatic API and FastAPI endpoint definitions
│   ├── core.py             # Deterministic diff computation engine
│   ├── main.py             # Typer CLI application and command routers
│   └── models.py           # Pydantic domain models for schema representations
└── tests/
    ├── __init__.py         # Test package initialization
    ├── test_api.py         # API tests
    └── test_core.py        # Core diff engine tests
```

---

## 🛣️ Roadmap & Future Enhancements

- [ ] **Auto-Migration SQL Generation**: Automatically output executable `ALTER TABLE` and `CREATE INDEX` SQL patch files to resolve detected drift.
- [ ] **Additional Dialect Support**: Add specialized introspection adapters for MySQL, Oracle, and Microsoft SQL Server.
- [ ] **HTML & Markdown Export**: Generate self-contained HTML reports and GitHub PR-compatible Markdown summaries.
- [ ] **Data Drift Analysis**: Expand beyond structural DDL differences to check reference table row counts and static enum key alignment.
- [ ] **GitHub Action Integration**: Pre-packaged GitHub Action for easy inclusion in PR validation workflows.

---

## 📜 License

This project is open-source software licensed under the [MIT License](LICENSE).