# Analytics Pipeline | dbt, Airflow, and Bigquery

Demo for running an analytics pipeline locally using Apache Airflow, dbt, and BigQuery (BQ).

We're using Google Cloud Platform (GCP) and BQ for the purposes of this demo, but if you're familiar with another public cloud or warehouse, you can substitute where applicable (i.e. `dbt-bigquery` -> `dbt-snowflake`).

## Getting Started

### Clone this repo
Clone the repo and open a terminal from the cloned directory

```bash
git clone https://github.com/chelseybeck/airflow-dbt-demo.git
```

This demo is still under development and updated regularly, so pull often (at least daily) to get the latest

```bash
git pull
```

### Prerequisites

- [Python 3.11+](https://www.python.org/downloads/)
- [Poetry](https://python-poetry.org/docs/)
- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [GCP account](https://cloud.google.com/solutions/smb)
- GCP Service Account (+ key) with the following permissions (allows Airflow to connect to BQ):
  - BigQuery Data Editor
  - BigQuery Data Viewer
  - Bigquery Job User

  The service account can be [created manually](https://cloud.google.com/iam/docs/service-accounts-create#creating) in the GCP UI or locally using the Terraform module in the `terraform` directory - [see details](/terraform/README.md)

## Python Environment Setup

We're using Poetry ([installation](https://python-poetry.org/docs/) is a pre-requisite)

1. Install dependencies

    ```bash
    poetry install
    ```

2. Open Poetry Shell
    Note: every new terminal should run in this virtual environment (unless using a customized venv). [Learn more](https://python-poetry.org/docs/basic-usage/#using-your-virtual-environment)

    ```bash
    poetry shell
    ```

## Set up Environment Variables

1. Copy the example `.env` file

    ```bash
    cp .env.example .env
    ```

2. Replace the file paths in `.env` with your system paths

## Initialize Airflow and Run DAG


1. Update Airflow configuration:

    - Find Airflow's Home directory
    
        ```bash
        airflow info
        ```

    - Update DAG directory
    
        Navigate to Airflow's home directory and open the `airflow.cfg` file. I.e.
    
        ```bash
        nano ~/airflow/airflow.cfg
        ```

    - Change the `dags_folder` path to the `airflow-dbt-demo` code repository and save. For example:

        ```
        dags_folder = /Users/username/airflow-dbt-demo/dags
        ```
    
    - Optional - remove DAG examples. When set to true, many examples are provided on the home page in the UI when Airflow is started
        ```
        load_examples = False
        ```

2. Initialize the database

    ```bash
    airflow db migrate
    ```

3. Create a user

    ```bash
    # create an admin user
    airflow users create \
    --username admin \
    --firstname Peter \
    --lastname Parker \
    --role Admin \
    --email spiderman@superhero.org
    ```

4. Add Google Cloud connection - allows connection to BQ - Airflow picks up credentials from your `.env` file

    ```bash
    airflow connections add 'google_cloud_default' \
    --conn-type 'google_cloud_platform' \
    --conn-extra "{\"extra__google_cloud_platform__project\": \"$GCP_PROJECT\"}"
    ```

4. Start the Airflow webserver:

    ```bash
    airflow webserver --port 8080
    ```

    Access the Airflow UI at `localhost:8080/home` & login
    [airflow home](http://0.0.0.0:8080/home)

5. Start the scheduler

    ```bash
    airflow scheduler
    ```

6. Run the DAGs from [Airflow's UI](http://0.0.0.0:8080/home):
  - Click on the DAG `spotify_ingestion_dag`
    - Loads Spotify data from a csv file into BigQuery
  - Click the 'play' button to trigger the DAG (upper right corner)

  - Click on the DAG `daily_dbt_dag`
    - Runs dbt jobs ([models directory](/analytics/models)) 
  - Click the 'play' button to trigger the DAG (upper right corner)
