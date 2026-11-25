# Vulnera — Analytical Platform for Vulnerability Intelligence and Prioritization

This repository contains the undergraduate capstone project focused on the collection, integration, modeling, analysis, and visualization of public cybersecurity vulnerability data.

The project consolidates multiple authoritative security datasets into a unified relational environment, enabling historical analysis (2015–2026), risk-based remediation prioritization, and correlation between software weaknesses and real-world adversary behavior.

## Project Objectives

The primary objective of **Vulnera** is to provide an interactive analytical platform to evaluate security vulnerabilities across multiple risk dimensions—combining technical severity, exploitation probability, confirmed active exploitation, software weaknesses, and attack patterns.

The system was engineered for:

- **Historical and Temporal Analysis:** Comprehensive coverage of over 290,000 vulnerability records (CVEs) published between 2015 and 2026.
- **Risk-Based Prioritization:** Multi-source data correlation (CVSS, EPSS, and KEV) to distinguish theoretical severity from imminent operational threats.
- **Adversary Behavior Correlation:** Mapping software weaknesses (CWE) to offensive tactics and techniques in the MITRE ATT&CK framework via CAPEC.

## Integrated Data Sources

The platform integrates the following public datasets:

- **NVD / CVE (National Vulnerability Database):** Official repository maintained by NIST, providing detailed vulnerability metadata, descriptions, publication dates, CVSS severity metrics, and initial CWE classifications.
- **EPSS (Exploit Prediction Scoring System):** Maintained by FIRST, providing daily statistical probability scores estimating the likelihood of a vulnerability being exploited within the next 30 days.
- **KEV (Known Exploited Vulnerabilities):** Authoritative catalog maintained by CISA identifying vulnerabilities with confirmed active exploitation in the wild and known ransomware campaign usage.
- **CWE (Common Weakness Enumeration):** Standardized MITRE dictionary categorizing underlying software and hardware programming weaknesses.
- **MITRE ATT&CK & CAPEC:** Frameworks describing attack patterns, adversary objectives (Tactics), and offensive methods (Techniques).

## Project Architecture

The architecture was designed for modularity, reproducibility, analytical query performance, and future scalability.

**Core Components:**

- **Python (`Requests`, `Pandas`, `SQLAlchemy`):** Modular ETL scripts responsible for consuming external APIs/feeds, normalizing schemas, handling conflict resolution (`upsert`), and loading data into the relational database.
- **PostgreSQL:** Relational database storing consolidated vulnerability records and many-to-many associative tables (`cve_cwe_mapping`, `cwe_mitre_mapping`, `mitre_tactic_technique`).
- **Streamlit & Plotly:** Interactive web application and visualization layer supporting global filters, KPI cards, scatter plots, treemaps, and Sankey flow diagrams.
- **Docker & Docker Compose:** Standardized containerized environment for the database and administration tools, ensuring local reproducibility and smooth cloud migration.
- **GitHub Actions & Self-Hosted Runner:** Automated CI/CD data pipeline designed to execute scheduled updates securely against the local Docker database.

## Automation with GitHub Actions

During active development, the project utilized a data CI/CD pipeline to keep the database synchronized with daily updates from external sources such as the NVD.

**Scheduled Workflow and Current Status**  
Defined in `.github/workflows/update_data.yml`, the workflow was configured to run on a scheduled basis using a **Self-Hosted Runner** to safely interact with the local Docker database environment. Because the project has already been defended and concluded as an academic thesis—and the self-hosted runner required a local host machine and Docker containers running at execution time—scheduled automated runs are currently paused, while the workflow remains available in the repository for local or cloud-hosted reuse.

**Security (`GitHub Secrets`)**  
Access credentials are never exposed in the source code. To enable automated runs, configure the following secrets in the GitHub repository settings:

> DB_PASS: PostgreSQL database password.  
> NVD_API_KEY: NIST NVD API access key.

---

## Installation and Execution Guide

Follow the steps below to configure the environment and run the project locally.

### Prerequisites

- [Git](https://git-scm.com/) installed.
- [Docker Desktop](https://www.docker.com/) installed.
- [Python 3.9+](https://www.python.org/) installed.
- [GitHub Actions Runner](https://github.com/settings/actions/runners) configured locally (optional, required only if re-enabling scheduled automation).

### 1. Database Setup (Docker)

From the project root directory, start the PostgreSQL and pgAdmin containers:

```bash
docker-compose up -d
```

#### 1.2. Accessing and Configuring pgAdmin

- Open your browser and navigate to: `http://localhost:5050`
- Log in using the credentials configured in `docker-compose.yml`:

> Email: admin@admin.com  
> Password: admin

#### 1.3. Connecting to the Database Server

- In pgAdmin, right-click **Servers > Register > Server**.
- On the **General** tab:

> Name: Local Docker

- On the **Connection** tab:

> Host name/address: db (or host.docker.internal)  
> Port: 5432  
> Username: admin  
> Password: admin_password

> Database tables and indexes are automatically created on first boot via the `ddl.sql` script located in `docker-entrypoint-initdb/`.

- Click **Save**.

### 2. Installing Dependencies

Install the required Python libraries using the requirements file:

```bash
pip install -r requirements.txt
```

### 3. Data Ingestion and ETL Orchestration

The project uses a central orchestrator (`setup.py`) to manage execution order and preserve foreign-key referential integrity:

```bash
python setup.py
```

`setup.py` automates the execution of the ingestion scripts located in the `populate-tables` directory, ensuring external data is extracted, transformed, and loaded in the proper relational dependency sequence.

**Important — Execution Order Managed by `setup.py`:**

- `cve.py`: Core NVD/CVE vulnerability dataset (must be populated first).
- `kev.py`: CISA Known Exploited Vulnerabilities catalog.
- `epss.py`: FIRST exploitation probability scores.
- `cwe.py`: Software weakness dictionary definitions.
- `mitreattack.py`: MITRE ATT&CK tactics and techniques.
- `capec.py`: Attack pattern mappings linking CWEs to ATT&CK techniques.

If you prefer manual execution (ensure your virtual environment is active):

```bash
# Navigate to the ingestion scripts folder
cd populate-tables

# Run core historical/incremental ingestion (may take several minutes on initial load)
python cve.py

# Run enrichment and framework mapping scripts
python kev.py
python epss.py
python cwe.py
python mitreattack.py
python capec.py

# Return to the project root directory
cd ..
```

### 4. Running the Analytical Dashboard

With the database populated, start the Streamlit application from the project root:

```bash
streamlit run app.py
```

The dashboard will open automatically in your browser at: `http://localhost:8501`.

---

## Authors

- **Izabel Cristina de Souza Soares** — [GitHub](https://github.com/izabel-souza)
- **Isadora Bastos Duarte**
- **Advisors:** Prof. Raphael Carlos Santos Machado and Prof. Clara Mattos Chagas Lino (Universidade Federal Fluminense — UFF, 2026)
