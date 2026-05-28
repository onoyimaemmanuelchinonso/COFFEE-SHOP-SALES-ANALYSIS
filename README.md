# COFFEE-SHOP-SALES-ANALYSIS

## Table of Contents

- [Project Overview](project-overview)
- [Objectives](objectives)
- [Data Sources](data-sources)
- [Tools](tools)
- [Data Cleaning](data-cleaning)
- [Exporatory Data Analysis](exporatory-data-analysis)
- [Data Analysis](data-analysis)
- [Results/Findings](results/findings)
- [Recommendations](recommendations)
- [Conclusion](conclusion)
  
### Project Overview

Coffee shops operate in a fast-paced environment where every hour impacts revenue, every customer decision matters, and every location tells a different story. Understanding when customers visit, what they buy, and which locations perform best is not just helpful; it is essential for maximizing sales, reducing waste, and optimizing staff schedules. This project analyzes six months of coffee shop sales data from January to June 2023.

<img width="627" height="350" alt="coffee sales analysis dashboard" src="https://github.com/user-attachments/assets/006b13e8-91a2-4731-a166-102b2a8b53e1" />


### Objectives 

- Analyze monthly sales trends and month-over-month growth rates
- Track total orders and quantity sold across the six-month period
- Identify peak hours and slow periods using hourly sales data
- Compare weekday versus weekend sales performance
- Evaluate store location performance and identify top and bottom locations
- Determine best-selling product categories and individual products
- Uncover daily sales patterns for the strongest month (May)
- Provide actionable recommendations for staffing, promotions, inventory, and pricing

### Data Sources 

 Coffee Shop Sales: This table contains information about transactional sales records, including transaction date, transaction time, transaction quantity, store ID, store location, product ID, unit price, product category, product type and product detail.

 ### Tools

- SQL – For data extraction, cleaning, and writing analytical queries
- Power BI – For dashboard creation, data visualization

### Data Cleaning

- Checked for missing values across all columns; no significant nulls found
- Scanned for duplicate transaction IDs; no duplicates identified
- Verified data types: date, time, quantity, price, and text fields correctly formatted
- Standardized text fields like product category and store location for consistency
- Removed unnecessary columns including store_id and product_id from the data model

### Exploratory Data Analysis

EDA involves exploring the data to answer key questions, such as:

1. Calculate the total sales for each month respectively
2. Determine the month-on-month increase or decrease in sales
3. Calculate the total number of orders for each respective month
4. Determine the month-on-month increase or decrease in orders
5. Calculate the total number of Qty sold for each respective month
6. Determine the month-on-month increase or decrease in the total Qty sold
7. What are the total sales, total quantity sold and total orders for each day
8. Determine the total sales during weekdays and weekends
9. Sales trend over the period for the month of May
10. What are the total sales by store for the month of May
11. Calculate the total sales by product category
12. Determine total sales by product (Top 10)

### Data Analysis

1. Calculate the total sales for each month respectively
   
```sql
SELECT MONTH(transaction_date) AS Month_Number,
       DATENAME(MONTH, transaction_date) AS Month_Name,
       ROUND(SUM(unit_price * transaction_qty), 2) AS Total_Sales
FROM Coffee_Shop_Sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY Month_Number;
```

2. Determine the month-on-month increase or decrease in sales

```sql
SELECT MONTH(transaction_date) AS month,
       DATENAME(MONTH, transaction_date) AS month_name,
       ROUND(SUM(unit_price * transaction_qty), 2) AS total_sales,
       ROUND(
        (SUM(unit_price * transaction_qty) - LAG(SUM(unit_price * transaction_qty)) OVER (ORDER BY MONTH(transaction_date))) 
        / NULLIF(LAG(SUM(unit_price * transaction_qty)) OVER (ORDER BY MONTH(transaction_date)), 0) * 100, 
        2
    ) AS mom_increase_percentage
FROM Coffee_Shop_Sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date) 
ORDER BY month;
```

3. Calculate the total number of orders for each respective month

```sql
SELECT MONTH(transaction_date) AS Month_Number,
       DATENAME(MONTH, transaction_date) AS Month_Name,
       COUNT(transaction_id) AS Total_orders
FROM Coffee_Shop_Sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY Month_Number;
```

4. Determine the month-on-month increase or decrease in orders

```sql
SELECT
    MONTH(transaction_date) AS month,
    DATENAME(MONTH, transaction_date) AS month_name,
    COUNT(transaction_id) AS total_orders,
    CAST(
        (COUNT(transaction_id) - LAG(COUNT(transaction_id)) OVER (ORDER BY MONTH(transaction_date))) 
        * 100.0 / NULLIF(LAG(COUNT(transaction_id)) OVER (ORDER BY MONTH(transaction_date)), 0)
    AS DECIMAL(10, 2)) AS mom_increase_percentage
FROM Coffee_Shop_Sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY month;
```

5. Calculate the total number of Qty sold for each respective month

```sql
SELECT MONTH(transaction_date) AS Month_Number,
       DATENAME(MONTH, transaction_date) AS Month_Name,
       SUM(transaction_qty) AS total_qty
FROM Coffee_Shop_Sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY Month_Number;
```

6. Determine the month-on-month increase or decrease in the total Qty sold

```sql
SELECT
    MONTH(transaction_date) AS month,
    DATENAME(MONTH, transaction_date) AS month_name,
    SUM(transaction_qty) AS total_quantity_sold,
    CAST(
        (SUM(transaction_qty) - LAG(SUM(transaction_qty)) OVER (ORDER BY MONTH(transaction_date))) 
        * 100.0 / NULLIF(LAG(SUM(transaction_qty)) OVER (ORDER BY MONTH(transaction_date)), 0)
    AS DECIMAL(10, 2)) AS mom_increase_percentage
FROM Coffee_Shop_Sales
GROUP BY MONTH(transaction_date), DATENAME(MONTH, transaction_date)
ORDER BY month;
```

7. What are the total sales, total quantity sold and total orders for each day

```sql
SELECT
    transaction_date,
    ROUND(SUM(unit_price * transaction_qty), 2) AS total_sales,
    SUM(transaction_qty) AS total_quantity_sold,
    COUNT(transaction_id) AS total_orders
FROM Coffee_Shop_Sales
GROUP BY transaction_date
ORDER BY transaction_date;
```

8. Determine the total sales during weekdays and weekends

```sql
SELECT
    CASE WHEN DATENAME(WEEKDAY, transaction_date) IN ('Saturday', 'Sunday') THEN 'Weekends'
        ELSE 'Weekdays'
    END AS day_type,
    ROUND(SUM(unit_price * transaction_qty), 2) AS total_sales
FROM Coffee_Shop_Sales
WHERE MONTH(transaction_date) = 5
GROUP BY CASE WHEN DATENAME(WEEKDAY, transaction_date) IN ('Saturday', 'Sunday') THEN 'Weekends'
        ELSE 'Weekdays'
    END;
```

9. Sales trend over the period for the month of May

```sql
SELECT ROUND(AVG(total_sales), 2) AS average_sales
FROM (
    SELECT 
        SUM(unit_price * transaction_qty) AS total_sales
    FROM coffee_shop_sales
	WHERE MONTH(transaction_date) = 5  -- Filter for May
    GROUP BY transaction_date
) AS internal_query;
```

10. What are the total sales by store for the month of May

```sql
SELECT 
	store_location,
	ROUND(SUM(unit_price * transaction_qty), 2) as Total_Sales
FROM coffee_shop_sales
WHERE
	MONTH(transaction_date) =5 -- may month
GROUP BY store_location
ORDER BY Total_Sales DESC; 
```

11. Calculate the total sales by product category

```sql
SELECT 
	product_category,
	ROUND(SUM(unit_price * transaction_qty),2) as Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5 
GROUP BY product_category
ORDER BY Total_Sales DESC
```

12. Determine total sales by product (Top 10)

```sql
SELECT TOP 10 product_type,
	   ROUND(SUM(unit_price * transaction_qty),1) as Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5 
GROUP BY product_type
ORDER BY Total_Sales DESC;
```

### Results/Findings

#### Sales Performance

- Total sales grew from $81,678 in January to $166,486 in June
- February was the only month with negative growth at -6.77%
- May had the strongest growth at 31.77%, followed by March at 29.8%
- Sales more than doubled over the six-month period

#### Order Trends

- Orders increased from 17,314 in January to 35,352 in June
- Order growth pattern matched sales growth closely
- May had the highest order growth at 32.33%

#### Quantity Sold

- Quantity sold grew from 24,870 in January to 50,942 in June
- Growth patterns aligned with sales and orders

#### Daily Performance

- Daily sales averaged $5,056 in May
- Peak day was May 8 at $5,604
- Slowest day was May 6 at $4,205

#### Hourly Sales (May)

- Peak hours: 7 AM to 10 AM
- 10 AM recorded the highest sales at $19,639
- Sales dropped sharply after 11 AM

#### Weekday vs Weekend (May)

- Weekdays drove 72% of total sales
- Weekends accounted for only 28%

#### Store Location (May)

- Hell's Kitchen: $52,599
- Astoria: $52,429
- Lower Manhattan: $51,700
- All three locations performed nearly equally

#### Product Category (May)

- Coffee: $60,363 (top category)
- Tea: $44,540
- Bakery: $18,566
- Drinking Chocolate: $16,320

#### Top 10 Products (May)

- Barista Espresso: $20,424
- Brewed Chai tea: $17,427
- Hot chocolate: $16,320
- Gourmet brewed coffee: $15,559
- Brewed herbal tea: $10,930

### Recommendations

Staff are fully present during morning peak hours from 7 AM to 10 AM on weekdays. Launch afternoon promotions to boost slow periods after 11 AM. Introduce weekend-specific promotions to close the gap with weekdays. Invest in top categories: Coffee, Tea, and Bakery. Expand coffee offerings with seasonal varieties. Bundle coffee with bakery items to increase average order value. Study Hell's Kitchen's success and apply lessons to other locations. Consider small price increases on top-selling items. Align inventory with peak hours and best-selling products.

### Conclusion

This coffee shop's sales analysis reveals a business with strong momentum from January to June. Sales, orders, and quantity sold all more than doubled over the period. The morning rush from 7 AM to 10 AM drives the majority of daily revenue, while afternoons and weekends present clear growth opportunities. Coffee dominates all product categories, and Hell's Kitchen leads store locations. By implementing the recommendations around staffing, promotions, product strategy, and inventory management, the coffee shop can sustain growth and turn a strong six months into a long-term success story.


