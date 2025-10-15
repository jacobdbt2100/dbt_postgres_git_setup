# dbt_postgres_git_setup

# 🧭 Local Setup Guide: PostgreSQL ↔ dbt Core ↔ Git

This guide provides a complete, step-by-step workflow for setting up **dbt Core** with **PostgreSQL** locally and managing your project using **Git**.  
It’s designed for quick reference — from environment setup to pushing your code to GitHub.

---

## ⚙️ 1. Environment Setup

### 1.1 Install the required tools

Make sure you have the following installed on your system:

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
cd dbt_postgres_project

# Create virtual environment
python -m venv venv

# Activate it (Windows)
venv\Scripts\activate

# (Mac/Linux)
source venv/bin/activate
