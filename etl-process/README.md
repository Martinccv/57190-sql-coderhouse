This Docker Compose file orchestrates a multi-container environment with services for data processing, ETL (Extract, Transform, Load), and data visualization. Here's an overview of what each service does and how they interact with each other:

### Services Overview

1. **clean-csv**
   - **Purpose**: This service is responsible for processing and cleaning a CSV file.
   - **Build**: The service uses a Dockerfile located at `./python-ET/Dockerfile`.
   - **Environment Variables**: Specifies file paths for input and output data.
   - **Volumes**: Mounts the local `./data` directory to `/data` in the container.

2. **etl**
   - **Purpose**: This service runs a PostgreSQL database where the processed data will be loaded.
   - **Image**: Uses the latest PostgreSQL image.
   - **Environment Variables**: Sets up the PostgreSQL database with a user, password, and database name.
   - **Volumes**: 
     - Mounts the local `./data` directory to `/data` in the container.
     - Mounts a SQL initialization script to automatically set up the database schema.
   - **depends_on**: Waits for the `clean-csv` service to finish before starting.
   - **restart**: The service will restart on failure.

3. **mysql**
   - **Purpose**: This service runs a MySQL database.
   - **Image**: Uses the latest MySQL image.
   - **Environment Variables**: Sets up the MySQL database with a root password and database name.
   - **Volumes**: 
     - Mounts the local `./mysql_container` directory to `/mysql_container` in the container.
     - Mounts a SQL initialization script to automatically set up the database schema.
   - **depends_on**: Waits for the `clean-csv` service to finish before starting.
   - **restart**: The service will restart on failure.
   - **Ports**: Exposes port `3306` for MySQL.

4. **wait_until_ready**
   - **Purpose**: This service ensures that the PostgreSQL and MySQL databases are ready before proceeding.
   - **Build**: The service uses a Dockerfile located at `./wait_until_ready/Dockerfile`.
   - **depends_on**: Waits for both `etl` and `mysql` services to be ready.
   - **Volumes**: Mounts necessary scripts and logs.
   - **Command**: Runs a shell script (`/etl-waiter.sh`) to check the readiness of the databases.
   - **Environment Variables**: Provides database connection details for both PostgreSQL and MySQL.
   - **Healthcheck**: The service is marked healthy if the script exits successfully.

5. **metabase**
   - **Purpose**: This service runs Metabase, a data visualization tool.
   - **Image**: Uses the latest Metabase image.
   - **Ports**: Exposes port `3000` for the Metabase web interface.
   - **Environment Variables**: Provides connection details to the MySQL database.
   - **Healthcheck**: Verifies that the Metabase service is running and reachable.

### Process Flow Diagram

Here's a high-level representation of the process flow between these services:

```mermaid
graph LR
    A[clean-csv] -->|Generates cleaned CSV data| B[PostgreSQL (etl)]
    A -->|Generates cleaned CSV data| C[MySQL]
    B --> D[wait_until_ready]
    C --> D
    D --> E[Metabase]
```

### Summary

1. **clean-csv** service processes and cleans the CSV files, which are then used by both the PostgreSQL and MySQL databases.
2. **etl** (PostgreSQL) and **mysql** services load the cleaned data into their respective databases.
3. **wait_until_ready** ensures that both databases are fully initialized and ready before moving forward.
4. **metabase** provides a web interface for data visualization, connecting to the MySQL database.

This setup allows you to process data, load it into databases, and then visualize it using Metabase.