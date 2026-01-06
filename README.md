# dbt_postgres_git_setup

# Local Setup Guide: PostgreSQL ↔ dbt Core ↔ Git

This guide provides a complete, step-by-step workflow for setting up **dbt Core** with **PostgreSQL** locally and managing your project using **Git**.  
It’s designed for quick reference — from environment setup to pushing your code to GitHub.

---
## 1. Environment Setup

### 1.1 Install the required tools

| Dependency                                  | cmd Check if exist                                                                    |
| ------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Python 3.8+**                             | `python --version`, `python3 --version`, or `py -0` (to check all versions installed) |
| **pip**                                     | `pip --version` or `pip3 --version`                                                   |
| **PostgreSQL**                              | `psql --version`                                                                      |
| **Git**                                     | `git --version`                                                                       |
| **VS Code** (or your preferred IDE)         | `code --version`                                                                      |

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

# Activate it (Windows); notice the prefix "venv" after activation
venv/Scripts/activate

# (Mac/Linux)
source venv/bin/activate
```
**Fix Execution Error in PowerShell for Windows:**

```bash
If `Get-ExecutionPolicy` is `Restricted`
Run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted -Force`

Reverse command after venv activation: `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Undefined`
```

### 1.3 Install dbt for PostgreSQL

```bash
# Install both dbt and PostgreSQL adapters
pip install dbt-core dbt-postgres

# Freeze dependencies (to make venv reproducible)
pip freeze > requirements.txt
```
Alternatively, edit existing requirements.txt (for a previously used venv) to contain required dependencies ( dbt-core, dbt-postgres) and run;
```bash
pip install -r requirements.txt
```

> **pip freeze > requirements.txt** displays list of installed packages and save the list into a file called requirements.txt

> **Later (for reuse)**
Anyone cloning this Git repo just needs to do:
```bash
# Create virtual environment
python3 -m venv venv

#Activate it
venv\Scripts\activate (for Windows) or source venv/bin/activate (for Mac/Linux)

# Install dependencies from requirements.txt
pip install -r requirements.txt
```
> That recreates the same dbt environment exactly.

```bash
# Verify installed adapters
pip freeze
```

### 1.4 Verify installation

```bash
dbt --version
```
Expected output (example):

```yml
Core:
  - installed version: 1.8.5
Plugins:
  - postgres: 1.8.5
```

## 2. Connect dbt to PostgreSQL
### 2.1 Create a database and user in PostgreSQL

Log in to PostgreSQL (via **pgAdmin** or **psql**) and run:

```sql
CREATE DATABASE analytics;
CREATE USER dbt_user WITH PASSWORD 'dbt_password';
GRANT ALL PRIVILEGES ON DATABASE analytics TO dbt_user;
```
Then create a sample **schema** and **table**:

```sql
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
```
### 2.2 Initialize a dbt project

```bash
# Initialize
dbt init (Alternatively, dbt init my_dbt_project)
```

dbt will ask for:

- **project name**: `my_dbt_project`
- **host**: `localhost`
- **port**: same as system suggestion `Enter`
- **username**: `dbt_user`
- **password**: `dbt_password`
- **dbname**: `analytics` (**case sensitive** for **PostgreSQL**)
- **schema**: `dbt_schema`
- **threads**: `1 to 4` for free or small setup (like **local Postgres**) is good to avoid overloading the system. In the **cloud** or **production**, users often set **8–16** (depending on the power of the data warehouse).
- **Adapter**: `postgres`
- **Profile**: same as project name (`my_dbt_project`)

`dbt init my_dbt_project` skips the first question: “Enter a name for your project”
`threads` is the number of simultaneous queries dbt runs to make builds faster.

### 2.3 Configure dbt profile

Your **profiles.yml** is located here:

**Windows**:
> C:\Users\<YourUser>\.dbt\profiles.yml

**Example configuration**:

```yml
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
```

### 2.4 Test the connection

```bash
# Switch to project directory
cd my_dbt_project
# test connection
dbt debug
```
Alternatively, use this to test the specified project directory;
```bash
# Switch to project directory
cd my_dbt_project
# test connection
dbt debug --profile my_dbt_project
```

If successful, you’ll see:
```css
All checks passed!
```

## 3. Working with dbt Models
### 3.1 Create a model file

In **models/**, create **customers_view.sql**:

```sql
--{{ config(materialized='view') }} --(optional; to override the setting in schema.yml for view/table model to the transformed schema, if any (configuring in schema.yml is optional)).

SELECT
    customer_id,
    name,
    gender,
    annual_income
FROM {{ source('raw', 'customers') }} --analytics.raw.customers
WHERE annual_income > 50000
```

### 3.2 Define the source

Create **models/schema.yml**:

```yml
version: 2

sources:
  - name: raw_customers_table --(or some other descriptive name; not restricted)
    schema: raw --this is the "staging" schema
    description: "customers raw data"
    tables:
      - name: customers

--Optional
models:
  - name: customers_view
    description: --add description here
    columns:
      - name: customer_id
        tests: --(this test is inside schema.yml; other options are- inside the "/tests" folder, as a standalone test macro in "macros/tests/"
          -  not_null
          -  unique
      - name: annual_income
        tests:
          - not_null 
    config:
      materialized: view --can be overridden or simply configured in the model(.sql) file
      alias: --transformed_customers --optional; to change the model name in the transformed schema
```

### 3.3 Run and test the model

```bash
dbt run
```
Alternatively, to run a specific model;
```bash
dbt run --select customers_view
```
`customers_view` is the only model in this example.

Generally, to run multiple selected models;
```bash
dbt run -m model1 model2 model3 ...
```
Or equivalently;
```bash
dbt run --select model1 model2 model3 ...
```

`-m`(or `--models`) is the older, common shortcut.

Check in PostgreSQL:
```sql
SELECT * FROM dbt_schema.customers_view;
```

### 3.4 Add tests (optional but recommended)

In **schema.yml**, add:
```yml
models:
  - name: customers_view
    columns:
      - name: customer_id
        tests:
          - not_null
          - unique
```

Then run:
```bash
dbt test
```
`dbt tests` help **validate data quality** — like `missing values`, `duplicates`, or `mismatched relationships between tables`.

> **Where Tests Live:**

**(a) Inside schema.yml**

✅ This is the most common and recommended way.

Example:

```yml
version: 2

models:
  - name: customers
    description: "Customer data"
    columns:
      - name: customer_id
        tests:
          - not_null
          - unique
      - name: email
        tests:
          - not_null
```
Here, dbt will automatically generate and run tests for:
- Missing customer_id values
- Duplicate customer_id values
- Missing email values

These are **generic tests** (built-in).

**(b) Inside the /tests folder**

This is for custom SQL tests you create manually.

Example folder:

```pgsql
dbt_project/
├── models/
│   ├── staging/
│   └── marts/
├── tests/
│   └── customers_email_valid.sql
```
Content of `customers_email_valid.sql`:

```sql
-- Fail if any invalid emails exist
select *
from {{ ref('customers') }}
where email not like '%@%.%'
```
**(c) As a standalone test macro**

If you want reusable logic (e.g., check pattern validity across multiple tables), you can write a custom test macro in `/macros/tests/`.

Example macros/tests/test_email_pattern.sql:

```sql
{% test email_pattern(model, column_name) %}
    select *
    from {{ model }}
    where {{ column_name }} not like '%@%.%'
{% endtest %}
```
Then, reference it in schema.yml:

```yml
columns:
  - name: email
    tests:
      - email_pattern
```
> **Summary — What Goes Where**

| Type              | Purpose                                                                     | Location                | Defined in  |
| ----------------- | --------------------------------------------------------------------------- | ----------------------- | ----------- |
| Generic Test      | Common checks like `not_null`, `unique`, `accepted_values`, `relationships` | `schema.yml`            | YAML syntax |
| Custom SQL Test   | One-off data quality checks                                                 | `/tests/` folder        | SQL query   |
| Custom Macro Test | Reusable test logic                                                         | `/macros/tests/` folder | Jinja macro |

> **How to Run Tests:**

| **Goal**                                               | **Command**                                                 |
| ------------------------------------------------------ | ----------------------------------------------------------- |
| All tests in project                                   | `dbt test`                                                  |
| All tests for `customers` model                        | `dbt test -m customers`                                     |
| All tests for `orders` model                           | `dbt test -m orders`                                        |
| Tests for a specific column (`customer_id`)            | `dbt test -m customers.customer_id`                         |
| Only `not_null` tests for `customers`                  | `dbt test -m customers -s test_type:not_null`               |
| Only `unique` tests for `customers`                    | `dbt test -m customers -s test_type:unique`                 |
| Only `not_null` tests for `orders`                     | `dbt test -m orders -s test_type:not_null`                  |
| Only `unique` tests for `orders`                       | `dbt test -m orders -s test_type:unique`                    |
| Specific SQL test file (e.g., `invalid_email.sql`)     | `dbt test -m invalid_email`                                 |
| Run tests for both models together                     | `dbt test -m customers orders`                              |
| Run only `not_null` and `unique` tests for both models | `dbt test -m customers orders -s test_type:not_null,unique` |

## 4. Version Control with Git
### 4.1 Initialize Git

From the **root** of your dbt project folder:
```bash
git init
```

### 4.2 Create a .gitignore file

```bash
echo "venv/" >> .gitignore
echo "target/" >> .gitignore
echo "dbt_packages/" >> .gitignore
echo "logs/" >> .gitignore
```

### 4.3 Commit your initial project

```bash
git add .
git commit -m "Initial dbt project setup with PostgreSQL connection"
```

### 4.4 Connect to GitHub

1. Create a new empty GitHub repository (no README or .gitignore).
Example: *my_dbt_project_repo*

2. Link your local repo:
```bash
git remote add origin https://github.com/yourusername/my_dbt_project_repo.git
```

### 4.5 Push your first commit

```bash
git branch -M main
git push -u origin main
```

### 4.6 Typical Git Workflow (Full Cycle)
Step	Command	Description
| Step | Command                          | Description                |
| ---- | -------------------------------- | -------------------------- |
| 1    | `git status`                     | Check changed files        |
| 2    | `git add .`                      | Stage all files            |
| 3    | `git commit -m "Your message"`   | Save changes locally       |
| 4    | `git pull origin main`           | Sync updates from GitHub   |
| 5    | `git push origin main`           | Push new commits to GitHub |
| 6    | `git checkout -b feature_branch` | Create new branch for work |
| 7    | `git merge feature_branch`       | Merge updates to main      |

**Example workflow in words**:

> You edit your dbt model → `run dbt` run to test → commit your change → push to GitHub.

## 5. Maintenance Tips

- `dbt clean` → removes compiled artefacts
- `dbt deps` → installs dependencies (e.g. `dbt_utils`)
- `dbt build` → runs + tests everything
- Keep each dbt project inside its own virtual environment

## Summary Checklist
| Step | Description                   | Command                                        |
| ---- | ----------------------------- | ---------------------------------------------- |
| 1    | Create and activate venv      | `python -m venv venv`                          |
| 2    | Install dbt-postgres          | `pip install dbt-core dbt-postgres`            |
| 3    | Create PostgreSQL DB and user | SQL statements                                 |
| 4    | Configure profiles.yml        | Connection settings                            |
| 5    | Initialise dbt project        | `dbt init my_dbt_project`                      |
| 6    | Create model + schema.yml     | In `/models`                                   |
| 7    | Run dbt                       | `dbt run`                                      |
| 8    | Test dbt                      | `dbt test`                                     |
| 9    | Initialise git                | `git init`                                     |
| 10   | Commit + push to GitHub       | `git add . && git commit -m "msg" && git push` |


## Curate this
| Description                   | Command            |
| ----------------------------- | ------------------ |
| Clear                         | `cls`              |
| Clear (PowerShell)            | `cls` or `clear`   |
