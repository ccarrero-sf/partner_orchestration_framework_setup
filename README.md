# Partner Orchestration Framework Setup

This is the code to easily get Orchesrtration Framework up and running

Main and continuously update repository can be found here:
https://github.com/Snowflake-Labs/orchestration-framework

Run this code to automatically create a database, get some data and install a notebook within Streamlit in Snowflake:

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

Open the OF_GATEWAY_QUICKSTART, on the top, go to Packages, click on Stage Packages and paste:

```sql
@P_ORCHESTRATION_FRAMEWORK.PUBLIC.IMPORTS/agent_gateway.zip
```

Then click on Import

## Install in your local setup

Follow these instructions to install it in your laptop for greater flexibility:

1. Create a conda environment and activate it.

```code
conda create -n orchestration_framework python=3.11
conda activate orchestration_framework
```

2. Install the Orchestration Framework

```code
pip install orchestration-framework
```




