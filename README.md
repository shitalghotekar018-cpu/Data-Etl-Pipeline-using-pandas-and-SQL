MySQL Customer ETL Pipeline

A simple ETL (Extract, Transform, Load) pipeline built with Python, Pandas, and MySQL. The project extracts customer data from a MySQL table, cleans and transforms the data, loads it into a cleaned table, and generates multiple SQL-based report tables.

Project Overview

This project performs the following steps:

Extract customer data from MySQL.
Transform the data using Pandas.
Load the cleaned data into a new MySQL table.
Run SQL queries for customer analysis.
Store each query result in a separate MySQL report table.
Technologies Used
Python 3
MySQL
MySQL Connector/Python
Pandas
SQL
Project Structure
customer-etl/
│
├── etl_pipeline.py
├── README.md
└── requirements.txt

Database Configuration

The script connects to MySQL using the following configuration:

connection_config = {
    "host": "localhost",
    "user": "root",
    "password": "shital22",
    "connection_timeout": 5,
    "use_pure": True
}


Security Note: Do not store your real MySQL password directly in source code. For a real project, use environment variables or a .env file.

The script automatically creates the following database if it does not already exist:

customer

Source Table

Before running the ETL pipeline, the following table must exist:

mall_customers


The expected columns are:

Column	Description
CustomerID	Unique customer ID
Genre	Customer gender
Age	Customer age
Annual_Income_(k$)	Annual income in thousands of dollars
Spending_Score	Spending score from 1–100

The script checks whether mall_customers exists before continuing.

ETL Process
1. Extract

The pipeline connects to MySQL and creates the customer database if necessary.

It then reads the source table using Pandas:

query = "SELECT * FROM mall_customers"
df = pd.read_sql(query, connection)


The original data is displayed before transformation.

2. Transform

The following data-cleaning operations are performed.

Check missing values
df.isnull().sum()

Remove duplicate rows
df = df.drop_duplicates()

Clean Genre values

Extra spaces are removed:

df["Genre"] = df["Genre"].astype(str).str.strip()


Gender values are converted to uppercase:

df["Genre"] = df["Genre"].str.upper()


For example:

Male   → MALE
Female → FEMALE

Validate Age

Only customers between 1 and 100 years old are retained:

df = df[(df["Age"] > 0) & (df["Age"] <= 100)]

Validate Annual Income

Negative income values are removed:

df = df[df["Annual_Income_(k$)"] >= 0]

Validate Spending Score

Only spending scores between 0 and 100 are retained:

df = df[
    (df["Spending_Score"] >= 0) &
    (df["Spending_Score"] <= 100)
]

3. Load

The cleaned data is loaded into the following table:

mall_customers_cleaned


The table structure is:

CREATE TABLE mall_customers_cleaned (
    CustomerID INT PRIMARY KEY,
    Genre VARCHAR(20),
    Age INT,
    Annual_Income INT,
    Spending_Score INT
);


Before loading new data, the existing table data is cleared:

TRUNCATE TABLE mall_customers_cleaned;


This ensures that each pipeline execution loads the latest cleaned dataset without creating duplicate records.

4. SQL Reports

The pipeline creates separate report tables for different analyses.

All Customers

Table:

all_customers


Query:

SELECT *
FROM mall_customers_cleaned;

Customers Above 30

Table:

customers_above_30


Query:

SELECT *
FROM mall_customers_cleaned
WHERE Age > 30;


This report identifies customers older than 30.

High-Spending Customers

Table:

high_spending_customers


Query:

SELECT *
FROM mall_customers_cleaned
WHERE Spending_Score > 70;


This identifies customers with a spending score greater than 70.

Average Customer Age

Table:

average_age


Query:

SELECT AVG(Age) AS Average_Age
FROM mall_customers_cleaned;


This calculates the average age of customers.

Genre-Wise Customer Count

Table:

genre_wise_customer_count


Query:

SELECT Genre, COUNT(*) AS Total_Customers
FROM mall_customers_cleaned
GROUP BY Genre;


This shows the number of customers in each genre.

Top 5 Highest-Income Customers

Table:

top_5_highest_income_customers


Query:

SELECT *
FROM mall_customers_cleaned
ORDER BY Annual_Income DESC
LIMIT 5;


This returns the five customers with the highest annual income.

Generated Database Tables

After successful execution, the customer database contains the following tables:

mall_customers
mall_customers_cleaned
all_customers
customers_above_30
high_spending_customers
average_age
genre_wise_customer_count
top_5_highest_income_customers

Installation
1. Install Python

Make sure Python 3 is installed.

Check your Python version:

python --version

2. Install Dependencies

Install the required Python packages:

pip install mysql-connector-python pandas


Alternatively, create a requirements.txt file:

mysql-connector-python
pandas


Then install:

pip install -r requirements.txt

MySQL Setup

Make sure MySQL Server is installed and running.

Log in to MySQL:

mysql -u root -p


The Python script automatically creates the customer database.

However, the source table mall_customers must already be imported.

Example:

USE customer;

SHOW TABLES;

SELECT *
FROM mall_customers
LIMIT 5;

Running the Pipeline

Save the Python code as:

etl_pipeline.py


Then run:

python etl_pipeline.py


A successful execution will display messages similar to:

Connecting to MySQL...
MySQL Connection Successful!

Original Data:
...

Checking Missing Values:
...

Cleaned Data:
...

Total Rows After Cleaning: ...

Data Successfully Loaded into mall_customers_cleaned!

All SQL query results were stored in separate tables.

ETL Pipeline Completed Successfully!

Data Flow
                 MySQL
                   │
                   ▼
          mall_customers
                   │
                   │ EXTRACT
                   ▼
              Pandas
                   │
                   │ TRANSFORM
                   ▼
          Data Cleaning
                   │
                   │ LOAD
                   ▼
       mall_customers_cleaned
                   │
                   ▼
             SQL Analysis
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
   Customers   High Spend   Average Age
      >30
        │
        ├───────────────┐
        ▼               ▼
 Genre-wise Count   Top 5 Income

Key Features
Automatic MySQL database creation
Source-table validation
Missing-value inspection
Duplicate removal
Text cleaning
Data validation
Pandas-based transformation
MySQL data loading
SQL analytical queries
Separate report tables
Repeatable ETL execution
Important Security Recommendation

The current script contains the MySQL password directly in the Python file:

"password": "shital22"


For production or GitHub projects, avoid committing passwords.

A better approach is to use environment variables:

import os

connection_config = {
    "host": os.getenv("MYSQL_HOST", "localhost"),
    "user": os.getenv("MYSQL_USER", "root"),
    "password": os.getenv("MYSQL_PASSWORD"),
    "connection_timeout": 5,
    "use_pure": True
}


Then set the password in your environment instead of putting it in the source code.

Learning Objectives

This project demonstrates practical knowledge of:

ETL pipeline development
Python
Pandas
MySQL
SQL queries
Data cleaning
Data validation
Database table creation
Data insertion
Basic data analytics
Automating data-processing workflows
Future Improvements

Possible improvements include:

Add logging instead of print() statements.
Move database credentials to environment variables.
Use batch inserts with executemany() for better performance.
Add automated data-quality checks.
Add error handling around SQL operations.
Export reports to CSV or Excel.
Create visualizations using Matplotlib or Seaborn.
Schedule the ETL pipeline using a task scheduler.
Add unit tests.
Add a dashboard using Power BI or Tableau.
Author

Customer ETL Pipeline Project

Built using Python, Pandas, and MySQL.
