### Expert-Level SQL Questions

#### Table Structure

Assume we have the following table structure for a `sales` table:

**sales**

| sale_id | sale_date  | product_id | customer_id | quantity | price | discount | total_amount |
|---------|------------|------------|-------------|----------|-------|----------|--------------|
| 1       | 2023-01-01 | 101        | 201         | 2        | 50.00 | 5.00     | 95.00        |
| 2       | 2023-01-02 | 102        | 202         | 1        | 100.00| 0.00     | 100.00       |
| 3       | 2023-01-03 | 101        | 203         | 3        | 50.00 | 15.00    | 135.00       |
| 4       | 2023-01-04 | 103        | 204         | 1        | 200.00| 20.00    | 180.00       |
| 5       | 2023-01-05 | 104        | 205         | 4        | 25.00 | 10.00    | 90.00        |
| 6       | 2023-01-06 | 102        | 206         | 2        | 100.00| 5.00     | 195.00       |
| 7       | 2023-01-07 | 105        | 207         | 5        | 20.00 | 0.00     | 100.00       |
| 8       | 2023-01-08 | 101        | 208         | 1        | 50.00 | 2.00     | 48.00        |
| 9       | 2023-01-09 | 103        | 209         | 2        | 200.00| 15.00    | 385.00       |
| 10      | 2023-01-10 | 104        | 210         | 3        | 25.00 | 7.00     | 68.00        |

---

### 1. Calculate the cumulative sales amount for each product by date, considering the discounts applied.

**Answer:**
```sql
SELECT 
    sale_date, 
    product_id, 
    total_amount, 
    SUM(total_amount) OVER (PARTITION BY product_id ORDER BY sale_date) AS cumulative_sales_amount
FROM 
    sales
ORDER BY 
    product_id, sale_date;
```
---

### 2. Find the top 3 products with the highest total sales amount.

**Answer:**
```sql
SELECT 
    product_id, 
    SUM(total_amount) AS total_sales_amount
FROM 
    sales
GROUP BY 
    product_id
ORDER BY 
    total_sales_amount DESC
LIMIT 3;
```
---

### 3. Calculate the month-over-month sales growth for each product.

**Answer:**
```sql
SELECT 
    product_id, 
    DATE_TRUNC('month', sale_date) AS month, 
    SUM(total_amount) AS monthly_sales,
    LAG(SUM(total_amount)) OVER (PARTITION BY product_id ORDER BY DATE_TRUNC('month', sale_date)) AS previous_month_sales,
    (SUM(total_amount) - LAG(SUM(total_amount)) OVER (PARTITION BY product_id ORDER BY DATE_TRUNC('month', sale_date))) / NULLIF(LAG(SUM(total_amount)) OVER (PARTITION BY product_id ORDER BY DATE_TRUNC('month', sale_date)), 0) * 100 AS month_over_month_growth
FROM 
    sales
GROUP BY 
    product_id, DATE_TRUNC('month', sale_date)
ORDER BY 
    product_id, month;
```
---

### 4. Identify the top 5 customers who have spent the most on products.

**Answer:**
```sql
SELECT 
    customer_id, 
    SUM(total_amount) AS total_spent
FROM 
    sales
GROUP BY 
    customer_id
ORDER BY 
    total_spent DESC
LIMIT 5;
```
---

### 5. Calculate the average discount percentage applied for each product.

**Answer:**
```sql
SELECT 
    product_id, 
    AVG(discount / (price * quantity) * 100) AS average_discount_percentage
FROM 
    sales
GROUP BY 
    product_id;
```
---

### 6. Find the product with the highest average sale amount per transaction.

**Answer:**
```sql
SELECT 
    product_id, 
    AVG(total_amount) AS average_sale_amount
FROM 
    sales
GROUP BY 
    product_id
ORDER BY 
    average_sale_amount DESC
LIMIT 1;
```
---

### 7. Calculate the total revenue generated by each product category (assuming categories are available in a separate table).

**Answer:**
```sql
-- Assuming we have a categories table
-- categories
-- | category_id | category_name |
-- |-------------|---------------|
-- | 1           | Electronics   |
-- | 2           | Clothing      |
-- | 3           | Groceries     |

-- And a product_category table
-- product_category
-- | product_id | category_id |
-- |------------|-------------|
-- | 101        | 1           |
-- | 102        | 1           |
-- | 103        | 2           |
-- | 104        | 3           |
-- | 105        | 3           |

SELECT 
    c.category_name, 
    SUM(s.total_amount) AS total_revenue
FROM 
    sales s
JOIN 
    product_category pc ON s.product_id = pc.product_id
JOIN 
    categories c ON pc.category_id = c.category_id
GROUP BY 
    c.category_name;
```
---

### 8. Determine the average number of products sold per transaction for each product.

**Answer:**
```sql
SELECT 
    product_id, 
    AVG(quantity) AS average_quantity_sold
FROM 
    sales
GROUP BY 
    product_id;
```
---

### 9. Find the date with the highest total sales amount across all products.

**Answer:**
```sql
SELECT 
    sale_date, 
    SUM(total_amount) AS daily_sales_amount
FROM 
    sales
GROUP BY 
    sale_date
ORDER BY 
    daily_sales_amount DESC
LIMIT 1;
```
---

### 10. Calculate the percentage of total revenue contributed by each product.

**Answer:**
```sql
WITH total_revenue AS (
    SELECT 
        SUM(total_amount) AS total_sales
    FROM 
        sales
)
SELECT 
    product_id, 
    SUM(total_amount) AS product_sales,
    (SUM(total_amount) / total_sales.total_sales) * 100 AS revenue_percentage
FROM 
    sales, total_revenue
GROUP BY 
    product_id, total_sales.total_sales
ORDER BY 
    revenue_percentage DESC;
```
---