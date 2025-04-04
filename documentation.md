<h1 align="center">📝 DATA CATALOG</h1>

---

**Author:** Ayesha Zubair  
**Last Updated:** 2025-04-04 

### Overview 
- **Project Title:** `Medallion Data Warehouse: End-to-End Analytics Pipeline`  
- **Source Dataset:** Provided by [@DataWithBaraa](https://www.youtube.com/@DataWithBaraa)  
- **Objective:** This document acts as the **data catalog and validation tracker**, detailing quality checks, fixes, and data structure across all three layers.  
  It supports a complete data pipeline based on the **Medallion Architecture** (_Bronze → Silver → Gold_), ensuring robust raw data ingestion, cleaning & transformation, and business-ready reporting. [📄 Data Architecture](data_architecture.pdf)

![Data Flow](data_flow.svg)

---
<h2 align="center"> 1️⃣ <u>Bronze Layer (Raw Data)</u></h2>

**Overview:** This section outlines various data quality checks performed on the **_Bronze Layer_** before transformation and loading into the **_Silver Layer_**. These checks ensure data consistency, accuracy, and standardization.
####  📥 Ingestion Process  
- **Data Source:** ERP / CRM 
- **File Format:** CSV  
- **Ingestion Method:** Batch ETL  
- **Tools Used:** PostgreSQL, VSCode

#### 🔍 Data Quality Checks - Bronze Layer 
- Issues identified: **13**
- Issues fixed: **13**

| Table Name                | Column Name            | Check Type                          | Issue Found                                    | Fix Needed                                      |
|---------------------------|------------------------|-------------------------------------|-----------------------------------------------|------------------------------------------------|
| **crm_cust_info**         | `cst_id`               | Nulls/Duplicates Check             | 6 records found  | Remove duplicates, fill missing values         |
|                           | `cst_firstname`        | Unwanted Spaces                    | 17 records found                 | Trim spaces                                   |
|                           | `cst_lastname`         | Unwanted Spaces                    | 22 records found                 | Trim spaces                                   |
|                           | `cst_marital_status`   | Data Standardization & Consistency | Null values found                           | Standardize values (S, M)                     |
|                           | `cst_gndr`             | Data Standardization & Consistency | Null values found                           | Standardize values (M, F)                     |
| **crm_prd_info**          | `prd_id`               | Nulls/Duplicates Check             | ✅ No issues found                             | ✅ No fix needed                                          |
|                           | `prd_nm`               | Unwanted Spaces                    | ✅ No issues found                             | ✅ No fix needed                                          |
|                           | `prd_cost`             | Null/Negative Values Check         | 2 records found            | Fill missing values, ensure no negative costs |
|                           | `prd_line`             | Data Standardization & Consistency | Null values found                           | Standardize values (M, R, S, T)               |
|                           | `prd_start_dt`, `prd_end_dt` | Invalid Date Order Check        | 200 records with end date < start date      | Correct invalid dates                        |
| **crm_sales_details**     | `sls_ord_num`          | Unwanted Spaces                    | ✅ No issues found                             | ✅ No fix needed                                          |
|                           | `sls_order_dt`         | Invalid Date Check                 | 19 records found         | Fix incorrect dates                          |
|                           | `sls_ship_dt`          | Invalid Date Check                 | ✅ No issues found                             | ✅ No fix needed                                          |
|                           | `sls_due_dt`           | Invalid Date Check                 | ✅ No issues found                             | ✅ No fix needed                                          |
|                           | `sls_sales`, `sls_price` | Sales Consistency Check            | 35 records with incorrect sales calculations | Ensure sales = quantity * price              |
|                           | `sls_quantity`         | Null/Negative Values Check         | ✅ No issues found                             | ✅ No fix needed                                          |
| **erp_cust_az12**        | `bdate`                | Out-of-Range Date Check            | 31 records with invalid birthdates          | Restrict birthdates between 1924 & today     |
|                           | `gen`                  | Data Standardization & Consistency | Inconsistent values & NULLs found           | Standardize gender values                    |
| **erp_loc_a101**         | `cntry`                | Data Standardization & Consistency | Inconsistent country values & NULLs found   | Standardize country names                    |
| **erp_px_cat_g1v2**      | `cat`, `subcat`, `maintenance` | Unwanted Spaces                 | ✅ No issues found                             | ✅ No fix needed                                          |
|                           | `maintenance`          | Data Standardization & Consistency | ✅ No issues found                                | ✅ No fix needed                                          |

---

<h2 align="center"> 2️⃣ <u>Silver Layer (Cleaned & Transformed)</u></h2>

**Overview:** This layer represents the cleaned and transformed dataset, where issues identified in the Bronze Layer (e.g., duplicates, missing values, inconsistencies) are addressed. The data is standardized, corrected, and reshaped into a more structured and reliable format — making it ready for final analysis and reporting in the Gold Layer.

#### 🔍 Data Quality Checks - Silver Layer
- Issues identified: **1**
- Issues fixed: **1**

| Table Name                         | Column Name       | Check Type                         | Issue Found | Fix Needed |
|-------------------------------------|----------------------|------------------------------------|-------------|------------|
| **crm_cust_info**, **erp_cust_az12** | `cst_id`            | Duplicates Check                  | ✅ No issue found          |✅ No fix needed         |
| **crm_cust_info**, **erp_cust_az12** | `cst_gndr`, `gen`   | Data Consistency | 4 records found | Investigate and standardize gender values (M, F) |
| **crm_prd_info**, **erp_px_cat_g1v2** | `prd_key`            | Duplicate Check                  | ✅ No issue found          | ✅ No fix needed        |

---
<h2 align="center"> 3️⃣ <u>Gold Layer (Business-Ready Data)</u></h2>

**Overview:** The Gold Layer represents the final, analysis-ready version of the data — fully optimized for reporting, dashboards, and business intelligence. It brings together cleaned and standardized data in a Star Schema format, made up of:

📊 Fact Tables: Capture measurable business activities (e.g., sales transactions).
🌍 Dimension Tables: Contain descriptive attributes like customer or product details.

#### 🔍 Data Quality Checks - Gold Layer
- Issues identified: **0**

| Table Name                         | Column Name       | Check Type                         | Issue Found | Fix Needed |
|-------------------------------------|----------------------|------------------------------------|-------------|------------|
| **gold.dim_customers**, **gold.dim_products**, **gold.fact_sales**  | `customer_key`, `product_key` | Referential Integrity Check        | ✅ No issue found          | ✅ No fix needed         |


### 🏆 Data Model for Analysis  
#### 1. **gold.dim_customers**
- **Purpose:** Stores customer details enriched with demographic and geographic data.

| Column Name      | Data Type     | Description                                                                                   |
|------------------|---------------|-----------------------------------------------------------------------------------------------|
| customer_key     | INT           | Surrogate key uniquely identifying each customer record in the dimension table.               |
| customer_id      | INT           | Unique numerical identifier assigned to each customer.                                        |
| customer_number  | VARCHAR(50)  | Alphanumeric identifier representing the customer, used for tracking and referencing.         |
| first_name       | VARCHAR(50)  | The customer's first name, as recorded in the system.                                         |
| last_name        | VARCHAR(50)  | The customer's last name or family name.                                                     |
| country          | VARCHAR(50)  | The country of residence for the customer (e.g., 'Australia').                               |
| marital_status   | VARCHAR(50)  | The marital status of the customer (e.g., 'Married', 'Single').                              |
| gender           | VARCHAR(50)  | The gender of the customer (e.g., 'Male', 'Female', 'n/a').                                  |
| birthdate        | DATE          | The date of birth of the customer, formatted as YYYY-MM-DD (e.g., 1971-10-06).               |
| create_date      | DATE          | The date and time when the customer record was created in the system|

#### 2. **gold.dim_products**
- **Purpose:** Provides information about the products and their attributes.

| Column Name         | Data Type     | Description                                                                                   |
|---------------------|---------------|-----------------------------------------------------------------------------------------------|
| product_key         | INT           | Surrogate key uniquely identifying each product record in the product dimension table.         |
| product_id          | INT           | A unique identifier assigned to the product for internal tracking and referencing.            |
| product_number      | VARCHAR(50)  | A structured alphanumeric code representing the product, often used for categorization or inventory. |
| product_name        | VARCHAR(50)  | Descriptive name of the product, including key details such as type, color, and size.         |
| category_id         | VARCHAR(50)  | A unique identifier for the product's category, linking to its high-level classification.     |
| category            | VARCHAR(50)  | The broader classification of the product (e.g., Bikes, Components) to group related items.  |
| subcategory         | VARCHAR(50)  | A more detailed classification of the product within the category, such as product type.      |
| maintenance_required| VARCHAR(50)  | Indicates whether the product requires maintenance (e.g., 'Yes', 'No').                       |
| cost                | INT           | The cost or base price of the product, measured in monetary units.                            |
| product_line        | VARCHAR(50)  | The specific product line or series to which the product belongs (e.g., Road, Mountain).      |
| start_date          | DATE          | The date when the product became available for sale or use, stored in|

#### 3. **gold.fact_sales**
- **Purpose:** Stores transactional sales data for analytical purposes.

| Column Name     | Data Type     | Description                                                                                   |
|-----------------|---------------|-----------------------------------------------------------------------------------------------|
| order_number    | VARCHAR(50)  | A unique alphanumeric identifier for each sales order (e.g., 'SO54496').                      |
| product_key     | INT           | Surrogate key linking the order to the product dimension table.                               |
| customer_key    | INT           | Surrogate key linking the order to the customer dimension table.                              |
| order_date      | DATE          | The date when the order was placed.                                                           |
| shipping_date   | DATE          | The date when the order was shipped to the customer.                                          |
| due_date        | DATE          | The date when the order payment was due.                                                      |
| sales_amount    | INT           | The total monetary value of the sale for the line item, in whole currency units (e.g., 25).   |
| quantity        | INT           | The number of units of the product ordered for the line item (e.g., 1).                       |
| price           | INT           | The price per unit of the product for the line item, in whole currency units (e.g., 25).      |

---

<h2> 💾 <u>Storage Efficiency by Layers</u></h2>

| **Layer**     | **Description**                     | **Storage Format**    | **Size**         |
|---------------|-------------------------------------|-----------------------|------------------|
| Bronze        | Raw unprocessed data                | CSV                   | 8120KB           |
| Silver        | Cleaned and standardized tables         | CSV                   | 9000KB           |
| Gold          | Virtual views (data not physically stored) | Views                 | 0 Bytes          |

---