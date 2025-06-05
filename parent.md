# dataanalysis
Data Analysis

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
- SQL queries to extract and analyze transactional data
- Excel analysis for pattern identification and validation
- Cross-referencing customer orders with promotional item distribution

---

*This case study demonstrates complex e-commerce promotional analysis combining SQL data extraction, Excel modeling, and business logic validation to resolve customer experience issues.*
