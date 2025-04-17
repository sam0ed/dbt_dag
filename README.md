# dbt Snowflake Data Pipeline

This repository contains dbt models that transform data in a Snowflake data warehouse. This README explains how to run these models using Apache Airflow with the Cosmos package.

## Local Development with dbt

For local development without Airflow or Cosmos:

1. **Setup your local environment**:
   ```bash
   # Create a virtual environment
   python -m venv dbt_env
   source dbt_env/bin/activate  # On Windows: dbt_env\Scripts\activate
   
   # Install dbt-snowflake
   pip install dbt-snowflake
   ```

2. **Configure dbt profile**:
   Create or update `~/.dbt/profiles.yml` with your Snowflake credentials:
   ```yaml
   data_pipeline:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: <account_locator>-<account_name>
         user: <username>
         password: <password>
         role: dbt_role
         database: dbt_db
         warehouse: dbt_wh
         schema: dbt_schema
         threads: 4
   ```

3. **Test your connection**:
   ```bash
   cd dags/dbt/data_pipeline
   dbt debug
   ```

4. **Develop and test models**:
   ```bash
   # Run specific models
   dbt run --models model_name
   
   # Run tests
   dbt test
   
   # Generate documentation
   dbt docs generate
   dbt docs serve
   ```

5. **Commit changes to the repository**:
   After testing locally, commit your changes to the repository.
   Airflow will use the latest models on the next run.

## Running with Airflow and Cosmos

To run this dbt project with Apache Airflow using the Cosmos package, follow these steps:

### 1. Set Up Airflow Environment

After cloning the repository, move it into dags folder of an existing airflow project. Then modify the project as follows:

#### Dockerfile Setup
Add this to your Airflow Dockerfile to install dbt-snowflake:

```dockerfile
FROM quay.io/astronomer/astro-runtime:12.8.0

RUN python -m venv dbt_venv && source dbt_venv/bin/activate && \
    pip install --no-cache-dir dbt-snowflake && deactivate
```

#### Requirements
Ensure these packages are in your `requirements.txt`:

```
astronomer-cosmos
apache-airflow-providers-snowflake
```

### 2. Configure Snowflake Connection

Create a Snowflake connection in Airflow UI with the ID `snowflake_conn` and these details:

```json
{
  "account": "<account_locator>-<account_name>",
  "warehouse": "dbt_wh",
  "database": "dbt_db",
  "role": "dbt_role",
  "insecure_mode": false
}
```

### 3. Create the DAG File
Create a file named `dbt_dag.py` in your Airflow DAGs directory:

### 4. Running the DAG

1. Start your Airflow environment
2. The DAG will automatically appear in the Airflow UI
3. Cosmos will parse the dbt project and create individual tasks for each model
4. You can trigger the DAG manually or wait for the scheduled run

