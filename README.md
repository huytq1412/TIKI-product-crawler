# Tiki Product Crawler 

A high-volume data extraction tool for Tiki products, optimized for speed and stability.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![Status](https://img.shields.io/badge/Status-Completed-success)

## Features

* High Speed: Uses Multithreading (`concurrent.futures`) to handle 30 concurrent requests, exponentially increasing speed compared to sequential processing.
* Batch Processing: The system uses Stream Processing via the chunksize parameter in Pandas.
* Auto-Restart Mechanism: 
  * Uses subprocess to automatically restart the process after 10 seconds in case of a crash or network loss via the monitoring script `run.py`.
  * Auto-retries up to 5 times upon failure.
* Resume Mechanism: If interrupted, the next run will exclude successfully crawled IDs and resume at the stopped batch (no need to crawl from the beginning). Automatically skips completed batches. 
* Clean Data:
    * Automatically removes HTML tags.
    * Standardizes line breaks (`\n`).
* Anti-Blocking: Uses `fake-useragent` to rotate identities, avoiding being blocked by the server due to too many requests from a single source.
* Load: Loads into PostgreSQL using Upsert (Update if exists, Insert if new) and Transaction (Rollback on error) mechanisms to ensure data integrity.

Project2 Structure

```
Project2/
├── config/
│   ├── __init__.py
│   ├── config.py                   # Reads and configures database connection
│   └── database.ini                # Database connection info (ignored by git)
├── data/
│   ├── raw/                        # Contains 'products-0-200000.csv' file
│   │   └── products-0-200000.csv   # Input file containing Tiki product IDs 
│   └── processed/                  # Directory containing output product files 
│       ├── jsonfile/               # Directory for successfully crawled product output files (ignored by git)
│       └── errorfile/              # Directory for output files containing failed product IDs (ignored by git)
├── etl/
│   ├── extract/ 
│   │   ├── __init__.py
│   │   └── get_product.py          # Crawls detailed data for each product and cleans description
│   └── load
│       ├── __init__.py
│       └── load_data.py            # Creates tables and loads data into database
├── src/
│   ├── __init__.py
│   ├── add_error.py                # Handles writing data to files
│   ├── get_successed_product.py    # Retrieves the list of all successfully crawled IDs
│   ├── retry_error_product.py      # Re-crawls data for failed IDs from the client side
│   ├── main.py                     # Reads input CSV, multi-thread crawls, processes data, and pushes to output files
│   └── run.py                      # Runs the entire project with auto-restart support
├── .env                            # Environment variables (ignored by git)
│                                   Includes variables for PostgreSQL connection,
│                                       DATA_PATH (path to save csv file),
│                                       JSON_FILE_PATH (path to export json files),
│                                       ERROR_FILE_PATH (path to export files containing failed product IDs)
│                                       DATABASE_CONN_FILE (path to save database connection info)
├── .gitignore                      # Files excluded when pushing to git
├── requirements.txt                # Required libraries
└── README.md                       # Documentation
```

## Setup & Configuration
1. System Requirements
Python 3.10 or higher.

2. Install Libraries
Virtual environment (venv) recommended:

```
# Create virtual environment
python -m venv .venv

# Activate (Linux/Mac)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install libraries
pip install -r requirements.txt
```
3. Environment Variable Configuration (.env)
* Tạo file .env tại thư mục gốc và điền thông tin tương ứng:
```
# Data paths
DATA_PATH = "~/UNIGAP/Project2/data"
JSON_FILE_PATH = "~/UNIGAP/Project2/jsonfile"
ERROR_FILE_PATH = "~/UNIGAP/Project2/errorfile"
DATABASE_CONN_FILE = "~/UNIGAP/Project2/config/database.ini"
```

4. Database Connection Configuration. Create a database.ini file and fill in the corresponding info
```
# Data paths
[postgresql]
host=localhost
port=5432
database=yourDB
user=youruser
password=yourpassword
```

## Usage Guide
Run the monitoring script `run.py` (Recommended). The program will automatically restart if a crash occurs.

```
python src/run.py
```

## Output File Format (JSON)
Each JSON file contains about 1,000 products with the structure:

```
[
    {
        "id": 1391347,
        "name": "Bộ Xếp Hình...",
        "url_key": "bo-xep-hinh-tia-sang",
        "price": 211200,
        "description": "Thông tin chi tiết...\n- Chất liệu an toàn.",
        "images": [...]
    }
```