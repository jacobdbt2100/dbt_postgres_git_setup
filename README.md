# dbt_postgres_git_setup

## 4. Version Control with Git
### 4.1 Initialize Git

From the **root** of your dbt project folder, that is;

`Directory` **dbt_project_name/**

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


## Miscellaneous (Curate this)
| Description                   | Command            |
| ----------------------------- | ------------------ |
| Clear                         | `cls`              |
| Clear (PowerShell)            | `cls` or `clear`   |


## Short Notes about `dbt` folders

**1. analyses**
  - Temporary SQL queries for exploration or reporting.
  - Not models - not materialised in your warehouse.
  - Useful for ad-hoc analysis.

**2. logs**
  - Stores dbt run logs.
  - Mostly for debugging or reviewing past runs.
  - You usually don’t edit anything here.

**3. macros**
  - Reusable SQL snippets (functions).
  - Helps avoid repeating SQL code.
  - Can be called in models, tests, analyses, snapshots.

**4. models**
  - Where your main SQL models live.
  - Materialised as tables or views in your warehouse.
  - Can have subfolders for organisation.

**5. seeds**
  - CSV files that dbt loads into your warehouse.
  - Good for static reference data like country codes or lookup tables.

**6. snapshots**
  - Capture slowly changing data over time.
  - Useful for tracking historical changes.
  - Stored as versioned tables in your warehouse.

**7. targets**
  - Auto-generated folder by dbt.
  - Stores compiled SQL and temporary run artefacts.
  - You usually do not edit it.

**8. tests**
  - Contains singular / custom SQL tests for models or business logic.
  - Runs alongside generic tests in schema.yml.

### Mnemonic sentence for `dbt folders`:

**“All Little Macros Make Some Small Tidy Tables.”**

| Word   | Folder    | Role (quick)              |
| ------ | --------- | ------------------------- |
| All    | analyses  | Ad-hoc queries            |
| Little | logs      | Run logs                  |
| Macros | macros    | Reusable SQL functions    |
| Make   | models    | Main tables/views         |
| Some   | seeds     | Static CSV/reference data |
| Small  | snapshots | Track historical changes  |
| Tidy   | targets   | Compiled/run artefacts    |
| Tables | tests     | Custom/singular tests     |
