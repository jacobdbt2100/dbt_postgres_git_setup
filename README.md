# dbt_postgres_git_setup

# Local Setup Guide: PostgreSQL ↔ dbt Core ↔ Git

This guide provides a complete, step-by-step workflow for setting up **dbt Core** with **PostgreSQL** locally and managing your project using **Git**.  
It’s designed for quick reference — from environment setup to pushing your code to GitHub.

---
## 1. Environment Setup

### 1.1 Install the required tools

- **Python 3.8+**
- **pip** (Python package manager)
- **PostgreSQL**
- **Git**
- **VS Code** (or your preferred IDE)
---

### 1.2 Create and activate a virtual environment

```bash
# Create a folder for your project
mkdir dbt_postgres_project

# Change directory to created folder
cd dbt_postgres_project

# Alternatively, create folder and change directory to the new folder
mkdir dbt_postgres_project && cd dbt_postgres_project

# Create virtual environment
python3 -m venv venv

# Activate it (Windows)
venv\Scripts\activate

# (Mac/Linux)
source venv/bin/activate
```

### 1.3 Install dbt for PostgreSQL

```bash
pip install dbt-core dbt-postgres
```
✅ This installs both dbt and the PostgreSQL adapter.

### 1.4 Verify installation

```bash
dbt --version
```
Expected output (example):

Core:
  - installed version: 1.8.5
Plugins:
  - postgres: 1.8.5

## 🏦 2. Connect dbt to PostgreSQL
### 2.1 Create a database and user in PostgreSQL

Log in to PostgreSQL (via pgAdmin or psql) and run:

CREATE DATABASE analytics;
CREATE USER dbt_user WITH PASSWORD 'dbt_password';
GRANT ALL PRIVILEGES ON DATABASE analytics TO dbt_user;


Then create a sample schema and table:

CREATE SCHEMA raw;

CREATE TABLE raw.customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT,
    gender TEXT,
    annual_income NUMERIC
);

INSERT INTO raw.customers (name, gender, annual_income) VALUES
('Alice', 'Female', 55000),
('Bob', 'Male', 72000),
('Clara', 'Female', 48000),
('David', 'Male', 88000);

### 2.2 Initialise a dbt project
dbt init my_dbt_project
cd my_dbt_project


dbt will ask for:

Adapter → postgres

Project name → my_dbt_project

Profile → same as project name (my_dbt_project)

### 2.3 Configure dbt profile

Your profiles.yml is located here:

Windows:
C:\Users\<YourUser>\.dbt\profiles.yml

Example configuration:

my_dbt_project:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      user: dbt_user
      password: dbt_password
      port: 5432
      dbname: analytics
      schema: dbt_schema
      threads: 4

### 2.4 Test the connection
dbt debug


If successful, you’ll see:

All checks passed!

## 🧱 3. Working with dbt Models
### 3.1 Create a model file

In models/, create customers_view.sql:

{{ config(materialized='view') }}

SELECT
    customer_id,
    name,
    gender,
    annual_income
FROM {{ source('raw', 'customers') }}
WHERE annual_income > 50000

### 3.2 Define the source

Create models/schema.yml:

version: 2

sources:
  - name: raw
    schema: raw
    tables:
      - name: customers

### 3.3 Run and test the model
dbt run


Check in PostgreSQL:

SELECT * FROM dbt_schema.customers_view;

### 3.4 Add tests (optional but recommended)

In schema.yml, add:

models:
  - name: customers_view
    columns:
      - name: customer_id
        tests:
          - not_null
          - unique


Then run:

dbt test

## 🌳 4. Version Control with Git
### 4.1 Initialise Git

From the root of your dbt project folder:

git init

### 4.2 Create a .gitignore file
echo "venv/" >> .gitignore
echo "target/" >> .gitignore
echo "dbt_packages/" >> .gitignore
echo "logs/" >> .gitignore

### 4.3 Commit your initial project
git add .
git commit -m "Initial dbt project setup with PostgreSQL connection"

### 4.4 Connect to GitHub

Create a new empty GitHub repository (no README or .gitignore).
Example: my_dbt_project_repo

Link your local repo:

git remote add origin https://github.com/yourusername/my_dbt_project_repo.git

### 4.5 Push your first commit
git branch -M main
git push -u origin main

### 4.6 Typical Git Workflow (Full Cycle)
Step	Command	Description
1	git status	Check changed files
2	git add .	Stage all files
3	git commit -m "Your message"	Save changes locally
4	git pull origin main	Sync updates from GitHub
5	git push origin main	Push new commits to GitHub
6	git checkout -b feature_branch	Create new branch for work
7	git merge feature_branch	Merge updates to main

Example workflow in words:

You edit your dbt model → run dbt run to test → commit your change → push to GitHub.

## 🧹 5. Maintenance Tips

dbt clean → removes compiled artefacts

dbt deps → installs dependencies (e.g. dbt_utils)

dbt build → runs + tests everything

Keep each dbt project inside its own virtual environment

✅ Summary Checklist
Step	Description	Command
1	Create and activate venv	python -m venv venv
2	Install dbt-postgres	pip install dbt-core dbt-postgres
3	Create PostgreSQL DB and user	SQL statements
4	Configure profiles.yml	Connection settings
5	Initialise dbt project	dbt init my_dbt_project
6	Create model + schema.yml	In /models
7	Run dbt	dbt run
8	Test dbt	dbt test
9	Initialise git	git init
10	Commit + push to GitHub	git add . && git commit -m "msg" && git push
