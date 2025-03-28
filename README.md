# Partner Orchestration Framework Setup

This is the code to easily get Orchesrtration Framework up and running within Snowflake. The first step creates a database called P_ORCHESTRATION_FRAMEWORK, uses Snowflake Git integration to clone this repository within that database, runs the setup.sql script that generate some data and create the services needed as example to run the Agent.

Main and continuously update repository for Orchestration Framework can be found here:
https://github.com/Snowflake-Labs/orchestration-framework

Kudos to Alejandro Herrera & Tyler White for creating this great framework!


## 1. Automatic Setup

Run this code to automatically create a database, get some data, install a notebook in Snowflake you can use to understand how the framework works and also one Streamlit in Snowflake App. In your Snowflake account, open a new Worksheet, copy & paste:

```sql

CREATE or replace DATABASE P_ORCHESTRATION_FRAMEWORK;

CREATE OR REPLACE API INTEGRATION API_P_ORCHESTRATION_FRAMEWORK
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/ccarrero-sf/')
  ENABLED = TRUE;

CREATE OR REPLACE GIT REPOSITORY GITHUB_P_ORCHESTRATION_FRAMEWORK
    api_integration = API_P_ORCHESTRATION_FRAMEWORK
    origin = 'https://github.com/ccarrero-sf/partner_orchestration_framework_setup';

-- Make sure we get the latest files
ALTER GIT REPOSITORY GITHUB_P_ORCHESTRATION_FRAMEWORK FETCH;

SELECT SYSTEM$BEHAVIOR_CHANGE_BUNDLE_STATUS('2024_08');
-- ENABLE LATEST PYTHON VERSIONS
--SELECT SYSTEM$ENABLE_BEHAVIOR_CHANGE_BUNDLE('2024_08');

-- enable cross-region inference if you wish to use a model which is not available in your region
ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION';

-- Create RAW layer automation
EXECUTE IMMEDIATE FROM @P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/setup.sql;

-- Copy the Notebook to be used

CREATE OR REPLACE STAGE IMPORTS
  DIRECTORY = (ENABLE = TRUE)
  ENCRYPTION = ( TYPE = 'SNOWFLAKE_SSE' );

COPY FILES
  INTO @IMPORTS
  FROM @P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/imports/
  PATTERN='.*zip';

CREATE OR REPLACE NOTEBOOK OF_GATEWAY_QUICKSTART
    FROM @P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/
        MAIN_FILE = 'QUICKSTART.ipynb' 
        QUERY_WAREHOUSE = COMPUTE_WH;

CREATE OR REPLACE STAGE STREAMLIT_STAGE
DIRECTORY = (ENABLE = true);

COPY FILES 
    INTO @STREAMLIT_STAGE
    FROM @P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/
    FILES =('app.py', 'environment.yml');

CREATE OR REPLACE STREAMLIT P_ORCHESTATOR_CHATBOT
    ROOT_LOCATION = '@STREAMLIT_STAGE'
    MAIN_FILE = 'app.py'
    TITLE = 'ORCHESTRATION FRAMEWORK CHATBOT'
    QUERY_WAREHOUSE = 'COMPUTE_WH'
    IMPORTS = '@P_ORCHESTRATION_FRAMEWORK.PUBLIC.IMPORTS/agent_gateway.zip';


```

Open the OF_GATEWAY_QUICKSTART Notebook, on the top, go to Packages, click on Stage Packages and paste:

```sql
@P_ORCHESTRATION_FRAMEWORK.PUBLIC.IMPORTS/agent_gateway.zip
```

Then click on Import. Run the Notebook step by step and notice how the Tools and Agent are created.


## 2. Install in your local setup

You can also make this work in your local environment or within Snowpark Container Services. You can also modify the source code to personalize your agent and adapt it to the tools you want to use. We recommend always take a look to the latest version of the official repository. This repository is frozen image for demo purposes.

1. Create a conda environment and activate it.

```code
conda create -n orchestration_framework python=3.11
conda activate orchestration_framework
```

2. Install the Orchestration Framework using pip

```code
pip install orchestration-framework
```

3. Install these other packages

```code
pip install streamlit
pip install dotenv
pip install trulens
pip install trulens-connectors-snowflake
```

4. If you want to test the Quickstart Notebook locally, [Download it from here](!https://github.com/ccarrero-sf/partner_orchestration_framework_setup/blob/main/QUICKSTART.ipynb)

You first have to create a .env file in the same folder where you have saved the Quickstart notebook. Replace it with your own values:

SNOWFLAKE_HOST='xxxxx.snowflakecomputing.com'
SNOWFLAKE_ACCOUNT='xxxxx'
SNOWFLAKE_USER='youruser'
SNOWFLAKE_ROLE='ACCOUNTADMIN'
SNOWFLAKE_PASSWORD='yourpassword'
SNOWFLAKE_DATABASE='P_ORCHESTRATION_FRAMEWORK'
SNOWFLAKE_SCHEMA='PUBLIC'
SNOWFLAKE_WAREHOUSE='COMPUTE_WH'

Run jupyter notebook and select the QUickstart notebook

```code
jupyter notebook
```

5. Download the [Streamlit App](!https://github.com/ccarrero-sf/partner_orchestration_framework_setup/blob/main/app.py) and Run it:

```code
streamlit run app.py
```








