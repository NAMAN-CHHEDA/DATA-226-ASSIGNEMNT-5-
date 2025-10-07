# DATA-226-ASSIGNEMNT-5-Stock Prices ETL using Airflow and Snowflake

This repository contains the Airflow DAG **`assignment5_stock_prices.py`**, which automates the extraction of stock data from the **Alpha Vantage API** and loads it into a **Snowflake** data warehouse.  
The DAG demonstrates ETL (Extract, Transform, Load) automation, transaction management, and idempotent data loading.

---

## 🧩 Overview of the Pipeline

**DAG ID:** `assignment5`  
**Tags:** `ETL`, `stocks`

### Tasks
| Task Name | Description |
|------------|--------------|
| **ensure_objects** | Creates the target Snowflake schema (`RAW`) and table (`ASSIGNMENT_5`) if they don’t exist. |
| **extract_prices** | Fetches stock market data for a given symbol using the Alpha Vantage API. The last 90 days of daily OHLCV (Open, High, Low, Close, Volume) data are extracted. |
| **load_prices_idempotent** | Loads the extracted data into Snowflake. Uses a temporary staging table and a `MERGE` statement to update or insert records (idempotent loading). |

**Task Dependency:**


---

## ⚙️ Setup & Requirements

### 1. Install dependencies
In your Airflow environment:

pip install apache-airflow snowflake-connector-python requests

2. Airflow Connection (Snowflake)

Create a Snowflake connection in Airflow Admin → Connections
Conn Id: snowflake_conn
Conn Type: Snowflake
Extra (JSON):

{
  "account": "sfedu02-lvb17920",
  "database": "USER_DB_GECKO",
  "warehouse": "GECKO_QUERY_WH",
  "role": "SYSADMIN"
}

3. Airflow Variables

Set the following variables under Admin → Variables:

Variable	Example Value	Description
snowflake_userid	your_username	Snowflake login name
snowflake_password	your_password	Snowflake password
snowflake_account	sfedu02-lvb17920	Account locator
snowflake_database	USER_DB_GECKO	Target database
snowflake_warehouse	GECKO_QUERY_WH	Warehouse for compute
role	SYSADMIN	Snowflake role
vantage_api_key	your_alpha_vantage_key	Alpha Vantage API key
stock_symbol	GOOG	Stock ticker symbol
📄 Table Structure

RAW.ASSIGNMENT_5

Column	Type	Description
SYMBOL	VARCHAR(10)	Stock ticker symbol
DATE	DATE	Trading day
OPEN	FLOAT	Opening price
CLOSE	FLOAT	Closing price
HIGH	FLOAT	Highest price of the day
LOW	FLOAT	Lowest price of the day
VOLUME	INTEGER	Number of shares traded
🔁 How the DAG Works

ensure_objects creates schema/table if missing.

extract_prices calls Alpha Vantage API → JSON response parsed into a Python list of dicts.

load_prices_idempotent creates a temporary staging table and MERGEs data into the target Snowflake table.
This approach prevents duplicates and ensures the load is idempotent.

✅ Execution Instructions

Copy the DAG file to your Airflow dags/ directory:

dags/assignment5_stock_prices.py


Start Airflow Scheduler and Webserver:

airflow scheduler
airflow webserver


Trigger the DAG assignment5 from the Airflow UI.

📊 Expected Output

After running successfully, you’ll have an updated table:

SELECT * FROM RAW.ASSIGNMENT_5 ORDER BY SYMBOL, DATE DESC;

