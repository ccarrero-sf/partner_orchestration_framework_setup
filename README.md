# Partner Orchestration Framework Setup

This is the code to easily get Orchesrtration Framework up and running

Main and continuously update repository can be found here:
https://github.com/Snowflake-Labs/orchestration-framework

Run this code:

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
SELECT SYSTEM$ENABLE_BEHAVIOR_CHANGE_BUNDLE('2024_08');

-- enable cross-region inference if you wish to use a model which is not available in your region
ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION';

-- Create RAW layer automation
EXECUTE IMMEDIATE FROM @P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/setup.sql;

-- Copy the Notebook to be used

CREATE OR REPLACE STAGE IMPORTS;

COPY FILES
  INTO @IMPORTS
  FROM @P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/imports/
  PATTERN='.*zip';

CREATE OR REPLACE NOTEBOOK OF_GATEWAY_QUICKSTART
    FROM '@P_ORCHESTRATION_FRAMEWORK.PUBLIC.GITHUB_P_ORCHESTRATION_FRAMEWORK/branches/main/' 
        MAIN_FILE = 'QUICKSTART.ipynb' 
        QUERY_WAREHOUSE = COMPUTE_WH;

```

