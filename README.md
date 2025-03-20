# SAP LLM ANALYST DEMO

1. Copy/Past and run all these in a Snowflake Worksheet. This will setup the git connection, fetch the repository
and will run the setup.sql script to setup the RAW layer. It will also creat the Notebook to be run

```sql

CREATE or replace DATABASE SAP_LLM_ANALYST2;

CREATE OR REPLACE API INTEGRATION API_GITHUB_REPO_SAP_LLM_ANALYST
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/ccarrero-sf/')
  ENABLED = TRUE;

CREATE OR REPLACE GIT REPOSITORY GITHUB_REPO_SAP_LLM_ANALYST
    api_integration = API_GITHUB_REPO_SAP_LLM_ANALYST
    origin = 'https://github.com/ccarrero-sf/sap_llm_analyst';

-- Make sure we get the latest files
ALTER GIT REPOSITORY GITHUB_REPO_SAP_LLM_ANALYST FETCH;

SELECT SYSTEM$BEHAVIOR_CHANGE_BUNDLE_STATUS('2024_08');
-- ENABLE LATEST PYTHON VERSIONS
SELECT SYSTEM$ENABLE_BEHAVIOR_CHANGE_BUNDLE('2024_08');
-- Check it is enabled
SELECT SYSTEM$BEHAVIOR_CHANGE_BUNDLE_STATUS('2024_08');

-- Create RAW layer automation
EXECUTE IMMEDIATE FROM @SAP_LLM_ANALYST2.PUBLIC.GITHUB_REPO_SAP_LLM_ANALYST/branches/main/setup.sql;

-- Copy the Notebook to be used

CREATE OR REPLACE NOTEBOOK SAP_PREP_GOLD
    FROM '@SAP_LLM_ANALYST2.PUBLIC.GITHUB_REPO_SAP_LLM_ANALYST/branches/main/' 
        MAIN_FILE = 'SAP_PREP_GOLD.ipynb' 
        QUERY_WAREHOUSE = COMPUTE_WH;

ALTER NOTEBOOK SAP_PREP_GOLD ADD LIVE VERSION FROM LAST;

```

2. Open the SAP_PREP_GOLD notebook and follow the instructions

3. Open Semantic Model Generator to create a semantic file to be used with Cortex Analyst

