# Chunking Strategy Evaluation Demo

## In this demo repo we will explore the following

### RETRIEVAL_SERVICE_SETUP.ipynb
 - Parse PDF documents containing Snowflake blog posts into markdown text and split text on paragraph seperator
 - Explore multiple chunking strategies
 - Index chunked text (with document titles and classifications appended) into vectors for retreival with Cortex Search Service

### SUMMIT_DEMO_AI_AGENT_OBSERVABILITY.ipynb
 - Test out Cortex Search Service and experiment with confidence score thresholds
 - Define web search agent to retrieve context from docs.snowflake.com when local results are not relevant
 - Set up class with various agents and instrument for Evaluation and Tracing in Snowflake AI Observability
 - Instantiate multiple versions of the class for A/B testing
 - Pass prompts to applications


### Snowflake AI Observability UI
- Review and compare each application side-by-side to better understand
  - Model Quality
  - Retrieval Quality
  - Latency
  - Cost
- Consider which application would be the best candidate for production based on insights


SETUP INSTRUCTIONS

```
-- Create DB, Schema, and Warehouse
CREATE DATABASE IF NOT EXISTS CHUNKING_EVAL_DEMO;
CREATE SCHEMA IF NOT EXISTS DATA;
CREATE WAREHOUSE IF NOT EXISTS CHUNKING_EVAL_WAREHOUSE WITH WAREHOUSE_SIZE='SMALL';

--Create Stage
CREATE STAGE IF NOT EXISTS DOCS ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE' ) DIRECTORY = ( ENABLE = TRUE);


-- Create an API integration with Github
CREATE OR REPLACE API INTEGRATION GIT_INTEGRATION_CHUNKING_EVAL_DEMO
   api_provider = git_https_api
   api_allowed_prefixes = ('https://github.com/sfc-gh-ebotwick')
   enabled = true
   comment='Git integration with Chunking Eval Demo Repo';

-- Create the integration with the Github demo repository
CREATE OR REPLACE GIT REPOSITORY CHUNKING_EVAL_DEMO_GIT_REPO
   ORIGIN = 'https://github.com/sfc-gh-ebotwick/chunking_strategy_eval_demo' 
   API_INTEGRATION = 'GIT_INTEGRATION_CHUNKING_EVAL_DEMO' 
   COMMENT = 'Github Repository ';

-- Fetch most recent files from Github repository
ALTER GIT REPOSITORY CHUNKING_EVAL_DEMO_GIT_REPO FETCH;

-- Copy Cortex Search Setup notebook into snowflake configure runtime settings
CREATE OR REPLACE NOTEBOOK CHUNKING_EVAL_DEMO.DATA.RETRIEVAL_SERVICE_SETUP
FROM '@CHUNKING_EVAL_DEMO.DATA.CHUNKING_EVAL_DEMO_GIT_REPO/branches/main/' 
MAIN_FILE = 'CORTEX_SEARCH_SETUP.ipynb' QUERY_WAREHOUSE = AI_OBS_WAREHOUSE
IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 3600;

-- Copy AI Obs notebook into snowflake configure runtime settings
CREATE OR REPLACE NOTEBOOK CHUNKING_EVAL_DEMO.DATA.RAG_EVAL
FROM '@CHUNKING_EVAL_DEMO.DATA.CHUNKING_EVAL_DEMO_GIT_REPO/branches/main/' 
MAIN_FILE = 'AI_AGENT_OBSERVABILITY.ipynb' QUERY_WAREHOUSE = AI_OBS_WAREHOUSE
IDLE_AUTO_SHUTDOWN_TIME_SECONDS = 3600;


```



