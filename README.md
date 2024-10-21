# posit-snowflake-yyz
Posit + Snowflake Native App Enablement Session -- Toronto

## Getting Set Up

1. Run the following in your Snowflake account to ingest the data we'll use for these demos.
    ```sql
    CREATE DATABASE IF NOT EXISTS LENDING_CLUB;
    GRANT USAGE ON DATABASE LENDING_CLUB TO ROLE SYSADMIN;
    GRANT USAGE ON FUTURE SCHEMAS IN DATABASE LENDING_CLUB TO ROLE SYSADMIN;
    USE DATABASE LENDING_CLUB;
    
    CREATE SCHEMA IF NOT EXISTS ML;
    GRANT ALL PRIVILEGES ON FUTURE TABLES IN SCHEMA ML TO ROLE SYSADMIN;
    USE SCHEMA ML;

    CREATE STAGE LOAN_DATA
        URL = 's3://posit-snowflake-mlops';

    CREATE OR REPLACE FILE FORMAT FORMAT_LENDING_CLUB_PARQUET
        TYPE = PARQUET
        NULL_IF = ('NULL', 'null');

    CREATE OR REPLACE TABLE LOAN_DATA
        USING TEMPLATE (
            SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
            FROM TABLE(
                INFER_SCHEMA(
                    LOCATION => '@LENDING_CLUB.ML.LOAN_DATA/loan_data.parquet',
                    FILE_FORMAT => 'LENDING_CLUB.ML.FORMAT_LENDING_CLUB_PARQUET'
                )
            )
        );

    COPY INTO LOAN_DATA
        FROM '@LENDING_CLUB.ML.LOAN_DATA/loan_data.parquet'
        FILE_FORMAT = (FORMAT_NAME = LENDING_CLUB.ML.FORMAT_LENDING_CLUB_PARQUET)
        MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
    ```
2. Grab a Posit Workbench License (there's a link in my slides!)
3. Install the Posit Workbench Native App using the steps [here](https://docs.posit.co/ide/server-pro/2024.08.0/integration/snowflake/native-app/install.html).
4. Make sure your default Snowflake role is set to SYSADMIN (why? [Snowflake won't allow you to sign in as ACCOUNTADMIN via OAuth](https://docs.snowflake.com/en/user-guide/oauth-custom#blocking-specific-roles-from-using-the-integration))
5. Head into one of the demo asset folders and try it out!
