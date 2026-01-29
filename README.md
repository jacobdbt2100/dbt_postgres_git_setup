## dbt Maintenance

- `dbt clean` → removes compiled artefacts
- `dbt deps` → installs dependencies (e.g. `dbt_utils`)
- `dbt build` → runs + tests everything ...........(seeds, and snapshots too)
- Keep each dbt project inside its own virtual environment

## dbt Folders

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

**All Little Macros Make Some Small Tidy Tables**

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
