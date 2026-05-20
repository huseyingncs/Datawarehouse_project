# Datawarehouse_project

## Modern Data Warehouse with Medallion Architecture in SQL Server

This project demonstrates the design and implementation of a modern data warehouse using **Microsoft SQL Server**, **T-SQL**, and the **Medallion Architecture** approach.

The main objective of this project is to build an end-to-end data warehouse pipeline that loads raw CRM and ERP data from CSV files, cleans and standardizes the data, integrates it into business-ready objects, and creates a final analytical model for reporting and decision-making.

---

## Project Overview

This project follows a structured data engineering workflow:

```text
Source CSV Files
        |
        v
Bronze Layer
Raw source data
        |
        v
Silver Layer
Cleaned and standardized data
        |
        v
Gold Layer
Business-ready analytical views
        |
        v
Analytics and Reporting
```

The project is designed to demonstrate practical skills in:

- Data warehouse design
- SQL Server development
- ETL pipeline implementation
- Data cleansing and transformation
- Data quality validation
- Star schema modelling
- Fact and dimension modelling
- Analytical data preparation

---

## Data Architecture

The project uses the **Medallion Architecture**, which consists of three layers:

### Bronze Layer

The Bronze Layer stores raw data loaded directly from the source CSV files.

In this layer:

- CRM and ERP source files are loaded into SQL Server.
- Data is stored in its original structure.
- Existing Bronze tables are truncated before loading.
- `BULK INSERT` is used to load CSV files.
- This layer supports traceability and debugging.

Bronze tables include:

```sql
bronze.crm_cust_info
bronze.crm_prd_info
bronze.crm_sales_details
bronze.erp_cust_az12
bronze.erp_loc_a101
bronze.erp_px_cat_g1v2
```

---

### Silver Layer

The Silver Layer contains cleaned, standardized, and transformed data from the Bronze Layer.

In this layer:

- Duplicate customer records are handled.
- Text values are trimmed and standardized.
- Gender and marital status values are converted into readable formats.
- Product keys are split into category and product components.
- Product line codes are mapped to meaningful values.
- Invalid or missing sales values are corrected.
- Date fields are converted into proper date formats.
- A technical column `dwh_create_date` is added to track load time.

Silver tables include:

```sql
silver.crm_cust_info
silver.crm_prd_info
silver.crm_sales_details
silver.erp_cust_az12
silver.erp_loc_a101
silver.erp_px_cat_g1v2
```

---

### Gold Layer

The Gold Layer contains the final business-ready analytical model.

In this project, the Gold Layer is implemented using SQL views. These views integrate data from the Silver Layer and prepare it for analytics and reporting.

Gold views include:

```sql
gold.dim_customers
gold.dim_products
gold.fact_sales
```

The Gold Layer follows a **star schema** design:

```text
              gold.dim_customers
                       |
                       |
gold.dim_products --- gold.fact_sales
```

---

## Data Sources

The project uses two source systems: **CRM** and **ERP**.

### CRM Source Files

CRM data contains customer, product, and sales transaction information.

```text
datasets/source_crm/cust_info.csv
datasets/source_crm/prd_info.csv
datasets/source_crm/sales_details.csv
```

### ERP Source Files

ERP data contains additional customer, location, and product category information.

```text
datasets/source_erp/cust_az12.csv
datasets/source_erp/loc_a101.csv
datasets/source_erp/px_cat_g1v2.csv
```

---

## Repository Structure

```text
Datawarehouse_project/
│
├── datasets/
│   ├── source_crm/
│   ├── source_erp/
│   └── placeholder
│
├── docs/
│   ├── Archtiecture.png
│   ├── data_catalog.md
│   ├── data_model.png
│   ├── naming_conventions.md
│   └── placeholer
│
├── scripts/
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   │
│   ├── gold/
│   │   └── ddl_gold.sql
│   │
│   ├── init_database.sql
│   └── placeholder
│
├── tests/
│   ├── quality_checks_gold.sql
│   ├── quality_checks_silver.sql
│   └── placeholder
│
├── LICENSE
└── README.md
```

---

## Scripts Description

### `scripts/init_database.sql`

Creates the `DataWarehouse` database and the three main schemas:

```sql
bronze
silver
gold
```

Important note: this script drops and recreates the `DataWarehouse` database if it already exists. It should be used carefully because existing data will be deleted.

---

### `scripts/bronze/ddl_bronze.sql`

Creates the Bronze Layer tables.

This script:

- Drops existing Bronze tables if they already exist.
- Recreates tables under the `bronze` schema.
- Defines structures for CRM and ERP raw data.

---

### `scripts/bronze/proc_load_bronze.sql`

Creates the stored procedure:

```sql
bronze.load_bronze
```

This procedure:

- Truncates Bronze tables.
- Loads CSV files using `BULK INSERT`.
- Loads CRM and ERP source data into the Bronze Layer.
- Prints load duration for each table.
- Includes error handling with `TRY...CATCH`.

Example execution:

```sql
EXEC bronze.load_bronze;
```

---

### `scripts/silver/ddl_silver.sql`

Creates the Silver Layer tables.

This script:

- Drops existing Silver tables if they already exist.
- Recreates cleaned table structures under the `silver` schema.
- Adds `dwh_create_date` as a technical load timestamp column.

---

### `scripts/silver/proc_load_silver.sql`

Creates the stored procedure:

```sql
silver.load_silver
```

This procedure transforms data from Bronze to Silver.

Main transformations include:

- Removing duplicate customer records
- Trimming text columns
- Standardizing gender values
- Standardizing marital status values
- Cleaning product keys
- Creating category IDs
- Mapping product line codes
- Cleaning and converting dates
- Correcting sales amount and price inconsistencies
- Standardizing country values

Example execution:

```sql
EXEC silver.load_silver;
```

---

### `scripts/gold/ddl_gold.sql`

Creates Gold Layer analytical views.

This script creates:

```sql
gold.dim_customers
gold.dim_products
gold.fact_sales
```

These views integrate Silver Layer tables into a business-ready star schema.

---

## Gold Layer Data Model

### `gold.dim_customers`

This dimension view stores customer information enriched with demographic and location data.

Main columns:

```text
customer_key
customer_id
customer_number
first_name
last_name
country
marital_status
gender
birthdate
create_date
```

Purpose:

- Provides a business-friendly customer dimension.
- Combines CRM customer data with ERP demographic and location data.
- Uses a generated surrogate key called `customer_key`.

---

### `gold.dim_products`

This dimension view stores product information enriched with category data.

Main columns:

```text
product_key
product_id
product_number
product_name
category_id
category
subcategory
maintenance
cost
product_line
start_date
```

Purpose:

- Provides a business-friendly product dimension.
- Combines CRM product data with ERP product category data.
- Keeps only current product records by filtering records where `prd_end_dt IS NULL`.

---

### `gold.fact_sales`

This fact view stores sales transaction information.

Main columns:

```text
order_number
product_key
customer_key
order_date
shipping_date
due_date
sales_amount
quantity
price
```

Purpose:

- Stores measurable sales events.
- Connects sales transactions to customer and product dimensions.
- Supports sales, customer, and product analytics.

---

## ETL Process

### Step 1: Create Database and Schemas

Run:

```sql
scripts/init_database.sql
```

This creates:

```text
DataWarehouse
bronze schema
silver schema
gold schema
```

---

### Step 2: Create Bronze Tables

Run:

```sql
scripts/bronze/ddl_bronze.sql
```

---

### Step 3: Load Bronze Data

Run:

```sql
scripts/bronze/proc_load_bronze.sql
```

Then execute:

```sql
EXEC bronze.load_bronze;
```

This loads the CSV files from the CRM and ERP source folders into Bronze tables.

---

### Step 4: Create Silver Tables

Run:

```sql
scripts/silver/ddl_silver.sql
```

---

### Step 5: Load Silver Data

Run:

```sql
scripts/silver/proc_load_silver.sql
```

Then execute:

```sql
EXEC silver.load_silver;
```

This cleans, standardizes, and transforms the Bronze data into Silver tables.

---

### Step 6: Create Gold Views

Run:

```sql
scripts/gold/ddl_gold.sql
```

This creates the final analytical views:

```sql
gold.dim_customers
gold.dim_products
gold.fact_sales
```

---

### Step 7: Run Quality Checks

Run the test scripts:

```sql
tests/quality_checks_silver.sql
tests/quality_checks_gold.sql
```

These scripts validate data quality, consistency, and relationships across Silver and Gold layers.

---

## Data Quality Checks

The project includes SQL-based quality checks for the Silver and Gold layers.

### Silver Layer Checks

The Silver quality checks focus on:

- Duplicate records
- Null primary keys
- Unwanted spaces in text columns
- Standardized categorical values
- Invalid dates
- Incorrect sales calculations
- Data consistency after transformation

### Gold Layer Checks

The Gold quality checks focus on:

- Uniqueness of surrogate keys
- Referential integrity between fact and dimension views
- Relationship validation between sales, customers, and products

---

## Naming Conventions

The project follows consistent naming conventions documented in:

```text
docs/naming_conventions.md
```

Main rules:

- Use lowercase letters.
- Use snake_case.
- Use English names.
- Avoid SQL reserved words.
- Use source prefixes for Bronze and Silver objects.
- Use `dim_` prefix for dimension views.
- Use `fact_` prefix for fact views.

Examples:

```sql
bronze.crm_cust_info
silver.crm_sales_details
gold.dim_customers
gold.fact_sales
```

---

## Documentation

Project documentation is stored in the `docs` folder.

```text
docs/Archtiecture.png
docs/data_catalog.md
docs/data_model.png
docs/naming_conventions.md
```

### `data_catalog.md`

Documents the Gold Layer objects and their columns.

### `data_model.png`

Shows the final data model and star schema.

### `naming_conventions.md`

Explains naming standards used across schemas, tables, views, columns, and stored procedures.

---

## Tools and Technologies

- Microsoft SQL Server
- SQL Server Management Studio
- T-SQL
- Stored Procedures
- SQL Views
- BULK INSERT
- Medallion Architecture
- Star Schema
- GitHub

---

## How to Run the Project

### 1. Clone the Repository

```bash
git clone https://github.com/huseyingncs/Datawarehouse_project.git
```

### 2. Open SQL Server Management Studio

Open SSMS and connect to your local SQL Server instance.

### 3. Run Scripts in Order

Run the following scripts in this order:

```text
1. scripts/init_database.sql
2. scripts/bronze/ddl_bronze.sql
3. scripts/bronze/proc_load_bronze.sql
4. EXEC bronze.load_bronze;
5. scripts/silver/ddl_silver.sql
6. scripts/silver/proc_load_silver.sql
7. EXEC silver.load_silver;
8. scripts/gold/ddl_gold.sql
9. tests/quality_checks_silver.sql
10. tests/quality_checks_gold.sql
```

---

## Important Path Note

The Bronze loading procedure uses local CSV file paths inside `proc_load_bronze.sql`.

Example:

```sql
C:\sql\dwh_project\datasets\source_crm\cust_info.csv
```

Before running the Bronze load procedure, make sure the file paths in `proc_load_bronze.sql` match the location of the dataset folder on your computer.

---

## Example Analytical Queries

### Total Sales

```sql
SELECT
    SUM(sales_amount) AS total_sales
FROM gold.fact_sales;
```

### Sales by Product Category

```sql
SELECT
    p.category,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
LEFT JOIN gold.dim_products p
    ON f.product_key = p.product_key
GROUP BY p.category
ORDER BY total_sales DESC;
```

### Sales by Country

```sql
SELECT
    c.country,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
LEFT JOIN gold.dim_customers c
    ON f.customer_key = c.customer_key
GROUP BY c.country
ORDER BY total_sales DESC;
```

### Monthly Sales Trend

```sql
SELECT
    YEAR(order_date) AS sales_year,
    MONTH(order_date) AS sales_month,
    SUM(sales_amount) AS total_sales
FROM gold.fact_sales
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY sales_year, sales_month;
```

---

## Business Questions This Project Can Answer

This data warehouse can support analysis such as:

- Which products generate the highest revenue?
- Which product categories perform best?
- Which customers generate the most sales?
- Which countries contribute most to revenue?
- How do sales change over time?
- What is the relationship between quantity, price, and sales amount?
- Which customers and products are linked to each sales transaction?

---

## Project Outcome

The final result is a SQL Server data warehouse that transforms raw CRM and ERP CSV files into a clean and business-ready analytical model.

The project delivers:

- A Bronze Layer for raw source data
- A Silver Layer for cleaned and standardized data
- A Gold Layer for analytical views
- A star schema with customer and product dimensions
- A sales fact view for reporting
- SQL quality checks for validation
- Documentation for naming conventions and the data catalog

---

## Key Skills Demonstrated

This project demonstrates practical data engineering skills, including:

- SQL Server database development
- T-SQL scripting
- Data warehouse architecture
- ETL pipeline development
- Stored procedure creation
- CSV data loading with `BULK INSERT`
- Data cleansing and transformation
- Data quality testing
- Star schema design
- Fact and dimension modelling
- Analytical SQL queries
- Technical documentation

---

## About Me

I am **Huseyin Gencaslan**, an MSc Data Science student with a background in Industrial Engineering. I am interested in data engineering, data science, analytics, and building end-to-end data solutions.

This project was created to demonstrate my practical skills in SQL Server, ETL development, data warehousing, dimensional modelling, and analytics.

---

## Contact

- GitHub: `huseyingncs`
- LinkedIn: `huseyingencaslan`
- Email: `hgencaslan663@gmail.com`

---

## License

This project is licensed under the MIT License.
