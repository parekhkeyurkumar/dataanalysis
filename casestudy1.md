# E-commerce Promotional Analysis Case Study

## Problem Statement

### Business Context
An e-commerce company launched a promotional campaign in a European market offering customers a free product when purchasing any item from a specific product category. The promotion ran for 4 days and targeted 11 different promotional SKUs.

### Campaign Structure
- **Duration:** 4-day promotional window
- **Geography:** European Market
- **Promotion Type:** Free item by order total
- **Trigger Products:** 11 specific SKUs in the promotional category
- **Free Product:** Promotional item code "FREEPRODUCT"
- **System Configuration:** Maximum awards limited to 1 per customer

### Issue Identified
Customer service reported that repeat customers were not receiving their free promotional items in subsequent orders during the same promotional period. Initial analysis suggested that customers received the free product only on their first qualifying order, but not on additional orders placed within the promotional timeframe.

### Business Impact
- Potential customer dissatisfaction
- Inconsistent promotional experience
- Risk of customer complaints and refund requests
- Need to verify if system limitation was causing the issue

### Analytical Challenge
Determine whether the promotional system's "Max Awards = 1" configuration was preventing customers from receiving free items on multiple qualifying orders during the promotional period.

### Required Analysis
1. Identify all customers who placed multiple qualifying orders during the promotional period
2. Verify promotional item distribution patterns across first vs. subsequent orders
3. Quantify the scope of affected customers and orders
4. Provide data-driven recommendations for future promotional configurations

### Technical Requirements
- SQL queries to extract and analyse transactional data
- Excel analysis for pattern identification and validation
- Cross-referencing customer orders with promotional item distribution

---

## Solution Approach

### Step 1: Data Extraction - Pulling Promotional Period Orders

The first step involved extracting all orders placed during the promotional period that contained either the qualifying promotional SKUs or the free product itself.

**Objective:** Extract comprehensive order data to identify patterns in promotional item distribution

**SQL Strategy:**
- Query both primary and secondary order databases for comprehensive coverage
- Filter for active orders only (excluding cancelled/inactive statuses)
- Include both promotional SKUs and free product codes
- Extract essential fields for analysis

```sql
-- Query 1: Primary Database
SELECT 
  market_code, order_date, customer_id, customer_name, order_id, 
  product_code, product_name, quantity, unit_price, order_total
FROM ecommerce.orders
WHERE order_status NOT IN ('CANCELLED','INACTIVE')
  AND order_type IN ('STANDARD','INTERNAL')
  AND market_code IN ('MARKET_CODE')
  AND product_code IN ('PROMO_SKU_1','PROMO_SKU_2','PROMO_SKU_3','PROMO_SKU_4',
                       'PROMO_SKU_5','PROMO_SKU_6','PROMO_SKU_7','PROMO_SKU_8',
                       'PROMO_SKU_9','PROMO_SKU_10','PROMO_SKU_11','FREEPRODUCT')
  AND order_date BETWEEN 'START_DATE' AND 'END_DATE'
ORDER BY order_id, product_code;

-- Query 2: Secondary Database
SELECT 
  market_code, order_date, customer_id, customer_name, order_id, 
  product_code, product_name, quantity, unit_price, order_total
FROM ecommerce_archive.orders
WHERE order_status NOT IN ('CANCELLED','INACTIVE')
  AND order_type IN ('STANDARD','INTERNAL')
  AND market_code IN ('MARKET_CODE')
  AND product_code IN ('PROMO_SKU_1','PROMO_SKU_2','PROMO_SKU_3','PROMO_SKU_4',
                       'PROMO_SKU_5','PROMO_SKU_6','PROMO_SKU_7','PROMO_SKU_8',
                       'PROMO_SKU_9','PROMO_SKU_10','PROMO_SKU_11','FREEPRODUCT')
  AND order_date BETWEEN 'START_DATE' AND 'END_DATE'
ORDER BY order_id, product_code;
```

**Key Query Components:**
- **Simple Filtering:** Direct WHERE clauses to identify relevant orders and products
- **Status Filtering:** Excluded orders with cancelled or inactive status
- **Comprehensive SKU List:** Included all 11 promotional SKUs plus the free product code
- **Date Range:** Limited to the 4-day promotional window

**Database Strategy Note:**
While a UNION operation could have combined results from both databases into a single output file, I chose to run separate queries for each database. This approach provided faster execution times and better performance, especially when dealing with large datasets across multiple database systems.

**Output:** Combined dataset from both databases containing all order details for analysis in subsequent steps.

### Step 2: Data Processing and Cleaning

**Raw Data Characteristics:**
After executing the SQL queries, the combined dataset contained approximately 12,000 rows exported to Excel format. The data structure showed significant redundancy due to the line-item level detail:

- **Repeated Fields:** Customer names, order IDs, order totals, order dates, and market codes appeared multiple times
- **Variable Fields:** Only product codes and individual line item details differed between rows
- **Data Granularity:** Each row represented a single product line within an order

**Sample Data Structure:**
```
order_id | customer_name | product_code  | order_total
---------|---------------|---------------|------------
12345    | Customer A    | REGULAR_001   | 150
12345    | Customer A    | PROMO_SKU_6   | 150
12345    | Customer A    | FREEPRODUCT   | 150
12346    | Customer B    | REGULAR_002   | 200
12346    | Customer B    | PROMO_SKU_3   | 200
```

**Data Cleaning Process:**

1. **Text Formatting Cleanup:**
   - Used Excel Find & Replace to remove double spaces ('  ') and replace with no space ('')
   - Ensured consistent formatting across all text fields

2. **Data Preparation for Analysis:**
   - Identified that customer information, order totals, and dates were duplicated across multiple rows
   - Recognised need to group data by order ID, name to create distinct order-level records
   - Prepared data for subsequent filtering and analysis steps

**Key Observation:**
The line-item level data required aggregation to perform customer-level analysis, as each order appeared as multiple rows (one per product purchased). This structure was essential for identifying which orders contained promotional SKUs versus free products.

### Step 3: Identifying Repeat Customers Using Power Query

**Objective:** Identify customers who placed multiple orders during the promotional period to test the hypothesis about promotional limitations.

**Power Query Analysis:**
After cleaning the data, the next critical step was to identify which customers placed multiple orders during the 4-day promotional window. This required aggregating the line-item data by customer to see the complete order pattern.

**Power Query Process:**

1. **Group Data by Customer Name:**
   - Used Power Query's "Group By" function to aggregate all rows by customer name
   - This consolidated multiple line items per order and multiple orders per customer

2. **Create Custom Column for Order Aggregation:**
   ```powerquery
   Order Number
   Text.Combine(List.Transform(List.Distinct([Name][C5]), Text.From), ",")
   ```

3. **Analysis Logic:**
   - The formula extracts all unique order IDs for each customer
   - `List.Distinct([Name][C5])` gets unique order numbers for each grouped customer
   - `Text.Combine()` joins multiple order numbers with commas
   - `List.Transform(..., Text.From)` ensures proper text formatting

**Results:**
The Power Query analysis revealed customers with multiple orders during the promotional period:

**Sample Output:**
```
Customer Name           | Order Numbers
------------------------|---------------------------
Customer A              | 12345,12346,12347
Customer B              | 12348
Customer C              | 12349,12350
Customer D              | 12351,12352,12353,12354
```

**Key Findings:**
- Multiple customers placed 2-4 orders during the promotional period
- This confirmed the need to analyse promotional item distribution across sequential orders
- Provided the foundation for testing the "Max Awards = 1" hypothesis

**Strategic Importance:**
This step was crucial because it isolated the specific customer cohort that could be affected by the promotional limitation. Only customers with multiple orders during the promotional period were relevant to testing whether the system's "Max Awards = 1" setting was preventing promotional items in subsequent orders.

### Step 4: Order-Level Analysis and Free Product Detection

**Objective:** Create a distinct order-level dataset to analyse promotional item distribution across individual orders.

**Challenge Identified:**
While Step 3 identified customers with multiple orders, we needed to analyze each individual order to determine which specific orders received the free product. Since the same customer could have multiple order numbers during the promotional period, we needed order-level granularity.

**Power Query Transformation:**

1. **Group by Order Number:**
   - Changed the grouping strategy from customer name to order number
   - Each order became a unique record regardless of how many line items it contained

2. **Create Consolidated Item Codes Column:**
   ```powerquery
   Item Codes
   Text.Combine(List.Transform(List.Distinct([Order Number][C6]), Text.From), ",")
   ```
   
   **Logic Explanation:**
   - `List.Distinct([Order Number][C6])` extracts all unique product codes within each order
   - `Text.Combine()` joins all product codes with commas
   - Result: Single string containing all products purchased in that order

3. **Add Supporting Columns:**
   - **Order Date:** First occurrence of order date for each order number
   - **Customer Name:** Customer associated with each order
   - **Total Invoice Amount:** Distinct total amount for each order

4. **Create Free Product Detection Column:**
   ```excel
   =IF(ISNUMBER(SEARCH("FREEPRODUCT",[@[Item Codes]])),"Yes","No")
   ```
   
   **Detection Logic:**
   - `SEARCH("FREEPRODUCT",[@[Item Codes]])` looks for the free product code within the comma-separated list
   - `ISNUMBER()` returns TRUE if the search finds the free product code
   - `IF()` converts the result to "Yes" or "No" for easy analysis

**Sample Transformed Data:**
```
Order Number | Customer Name | Order Date | Item Codes                                    | Total Amount | Free Product Added?
-------------|---------------|------------|-----------------------------------------------|--------------|-------------------
12345        | Customer A    | 2025-05-23 | REGULAR_001,PROMO_SKU_6,FREEPRODUCT           | 150          | Yes
12346        | Customer A    | 2025-05-24 | REGULAR_002,PROMO_SKU_3                       | 200          | No
12347        | Customer B    | 2025-05-23 | REGULAR_003,PROMO_SKU_1,FREEPRODUCT           | 175          | Yes
12348        | Customer B    | 2025-05-25 | REGULAR_004,PROMO_SKU_2                       | 180          | No
```

**Key Outcomes:**
- **Distinct Order Records:** Each row now represented one complete order
- **Consolidated Product View:** All items purchased in a single order visible in one field
- **Binary Free Product Indicator:** Clear "Yes/No" identification of orders with promotional items
- **Analysis-Ready Dataset:** Prepared for sequential order analysis by customer

This transformation was critical for the next phase: analysing the chronological order of purchases by repeat customers to validate the "Max Awards = 1" hypothesis.

### Step 5: Data Restructuring for VLOOKUP Analysis

**Challenge:** The Step 3 output contained comma-separated order numbers in single cells, making it tricky to perform clean VLOOKUP operations between the customer list and order-level data from Step 4.

**Data Structure Problem:**
```
Customer Name | Order Numbers (Comma-separated)
--------------|--------------------------------
Customer_A    | 78234567,78345678,78456789,78567890
Customer_B    | 78123456,78234567,78345678,78456789
```

**Goal:** Transform this into individual rows for each order number while preserving customer associations:
```
Customer Name | Order Number
--------------|-------------
Customer_A    | 78234567
Customer_A    | 78345678
Customer_A    | 78456789
Customer_A    | 78567890
```

**Transformation Options Considered:**

1. **Manual Approach:** 
   - Use "Text to Columns" with comma delimiter
   - Copy and transpose data
   - **Drawback:** Time-intensive and error-prone for large datasets

2. **TOCOL Function:** 
   - Modern Excel function for array manipulation
   - **Drawback:** Still requires manual setup and configuration

3. **Power Query Solution (Chosen):**
   - Automated and repeatable process
   - Handles data type conversions automatically
   - Scalable for larger datasets

**Power Query Implementation:**

1. **Data Selection:**
   - Selected the customer name and comma-separated order numbers columns
   - Loaded into Power Query Editor

2. **Column Transformation:**
   - Split the order numbers column by comma delimiter
   - This created multiple columns (Order1, Order2, Order3, etc.)

3. **Unpivot Transformation:**
   - Selected all the newly created order columns
   - Went to Transform → Unpivot Columns
   - This converted columns back to rows with proper customer associations

4. **Cleanup:**
   - Deleted the "Attribute" column (contained Order1, Order2, etc. labels)
   - Renamed "Value" column to "Order Number"
   - Closed and loaded the transformed data

**Result:**
A clean, normalised table with one row per customer-order combination, perfectly structured for VLOOKUP operations against the Step 4 order-level analysis table.

**Why This Approach Was Superior:**
- **Automation:** No manual copy-paste operations
- **Accuracy:** Eliminated human error in data transformation
- **Scalability:** Could handle hundreds of customers with varying order quantities
- **Repeatability:** Process could be refreshed automatically if source data changed

This restructuring enabled seamless integration between customer identification (Step 3) and order-level promotional analysis (Step 4) through efficient VLOOKUP operations.

### Step 6: Final Analysis - VLOOKUP Integration and Hypothesis Validation

**Objective:** Combine the restructured customer data with order-level analysis to validate whether repeat customers received free products only in their first orders.

**VLOOKUP Implementation:**
After restructuring the data in Step 5, we now had a clean table with individual customer-order combinations that could be seamlessly integrated with the order-level analysis from Step 4.

**VLOOKUP Formula Used:**
```excel
=VLOOKUP([@[Order Number]],'Distinct Orders'!A:G,5,FALSE)
```

**Integration Process:**
1. **Primary Table:** Customer names with individual order numbers (from Step 5)
2. **Lookup Table:** Order-level analysis with promotional item detection (from Step 4)
3. **VLOOKUP Columns Added:**
   - Order Date
   - Item Codes (comma-separated list of all products)
   - Total Invoice Amount
   - Free Product Added (Yes/No indicator)

**Sample Final Analysis Table:**
```
Customer Name | Order Number | Order Date | Item Codes                             | Total Amount | Free Product Added?
--------------|--------------|------------|----------------------------------------|--------------|-------------------
Customer_A    | 78234567     | 2025-05-23 | REGULAR_001,PROMO_SKU_6,FREEPRODUCT    | 218          | Yes
Customer_A    | 78345678     | 2025-05-24 | REGULAR_002,PROMO_SKU_3                | 100          | No
Customer_A    | 78456789     | 2025-05-25 | REGULAR_003,PROMO_SKU_2                | 227          | No
Customer_A    | 78567890     | 2025-05-26 | REGULAR_004,PROMO_SKU_1                | 210.9        | No
Customer_B    | 78123456     | 2025-05-23 | REGULAR_005,PROMO_SKU_4,FREEPRODUCT    | 102          | Yes
Customer_B    | 78234567     | 2025-05-23 | REGULAR_006,PROMO_SKU_5                | 107.5        | No
```

**Critical Findings:**

✅ **Hypothesis CONFIRMED:** The analysis revealed a clear pattern across all repeat customers:
- **First Qualifying Order:** Customers received the free product when purchasing promotional SKUs
- **Subsequent Orders:** Despite purchasing qualifying promotional SKUs, customers did NOT receive free products in later orders
- **System Limitation Validated:** The "Max Awards = 1" setting was indeed preventing promotional items from being added to consecutive qualifying orders

**Business Impact Quantified:**
- Multiple customers affected by the promotional limitation
- Consistent pattern across all repeat purchasers during the promotional period
- Clear evidence that the promotional system configuration was causing customer experience issues

**Conclusion:**
The comprehensive analysis successfully proved that the promotional system's "Max Awards = 1" configuration was preventing customers from receiving free products in multiple qualifying orders during the same promotional period. This finding provided the data-driven evidence needed to adjust promotional settings for future campaigns and potentially address affected customer orders retroactively.

### Step 7: Solution Implementation and Testing

**System Configuration Update:**
Based on the validated findings, immediate action was taken to prevent future occurrences of this promotional limitation issue.

**Backend System Adjustment:**
- **Previous Setting:** Max Awards = 1 (limiting customers to one promotional item per promotional period)
- **Updated Setting:** Max Awards = 5 (allowing customers multiple promotional items during the same campaign)
- **Rationale:** The new limit accommodates realistic customer purchasing behavior while maintaining promotional budget control

**Testing Protocol:**
1. **Environment:** RC (Release Candidate) testing environment
2. **Test Scenario:** Simulated multiple qualifying orders from the same customer account
3. **Validation Points:**
   - First order with promotional SKU → Free product added ✅
   - Second order with promotional SKU → Free product added ✅
   - Third order with promotional SKU → Free product added ✅
   - Continued testing up to the new limit

**Test Results:**
- **Success:** Multiple orders from the same customer now correctly receive promotional items
- **System Behavior:** Promotional logic functions as intended across sequential orders
- **Customer Experience:** Consistent promotional benefit delivery regardless of order frequency

**Business Impact:**
- **Customer Retention:** Prevented potential customer dissatisfaction from inconsistent promotional experiences
- **Future Prevention:** Established proper promotional limits that accommodate realistic customer behavior
- **Process Improvement:** Created data-driven methodology for promotional configuration validation

**Long-term Value:**
This analysis and solution implementation established a framework for:
- Proactive promotional system monitoring
- Data-driven promotional parameter optimization
- Customer experience consistency across multi-order scenarios

The successful resolution demonstrates how comprehensive data analysis can identify system limitations, validate hypotheses, and drive actionable business improvements that directly impact customer satisfaction and retention.

---

*This case study demonstrates complex e-commerce promotional analysis combining SQL data extraction, Excel modeling, and business logic validation to resolve customer experience issues.*

