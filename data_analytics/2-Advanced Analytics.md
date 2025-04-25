# Advanced Analytics

## Answer Business Questions

Use of advanced techniques to solve real business questions.

## Steps for Advanced Analytics

### 1. Trends

Analyze how a measure evolves over time, to track trends and identify seasonality in the data.

-- <ins>Changes over years</ins>: A high-level overview insights that helps with strategic decision-making.

    SELECT
  	  YEAR(order_date) OrderYear,
  	  SUM(sales_amount) AS TotalSales,
  	  COUNT(DISTINCT customer_key) AS TotalCustomers,
  	  SUM(quantity) AS TotalQuantity
    FROM Gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY YEAR(order_date)
    ORDER BY YEAR(order_date)

<p align="center">
<img src="https://github.com/user-attachments/assets/e3a0673a-6300-4793-98ea-8c79584ba78d" />
</p>

-- <ins>Changes over months</ins>: Detailed insight to discover seasonality in the data.

    SELECT
      MONTH(order_date) OrderYear,
      SUM(sales_amount) AS TotalSales,
      COUNT(DISTINCT customer_key) AS TotalCustomers,
      SUM(quantity) AS TotalQuantity
    FROM Gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY MONTH(order_date)
    ORDER BY MONTH(order_date)

<p align="center">
<img src="https://github.com/user-attachments/assets/364f65d8-b4ba-4ee3-9d44-c2058032865e" />
</p>

-- I can also use years and months combined for more detailed insights.

    SELECT
      YEAR(order_date) OrderYear,
      MONTH(order_date) OrderYear,
      SUM(sales_amount) AS TotalSales,
      COUNT(DISTINCT customer_key) AS TotalCustomers,
      SUM(quantity) AS TotalQuantity
    FROM Gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY YEAR(order_date), MONTH(order_date)
    ORDER BY YEAR(order_date), MONTH(order_date)

<p align="center">
<img src="https://github.com/user-attachments/assets/17ff1e22-4206-489b-85b6-44695db24738" />
</p>


### 2. Cumulative Analysis

Aggregate the data progressively over time, to understand whether the business is growing or declining.

For example, I can calculate the total sales per month and the running total of sales over time (***in this case, I've partitioned the data by year,
otherwise the data would've used the default window frame, where the data is aggregated between the unbounded preceding and current row***).

    SELECT
        order_date,
        total_sales,
        SUM(total_sales) OVER (PARTITION BY YEAR(order_date) ORDER BY order_date) AS running_total_sales
    FROM
    (
    SELECT
        DATETRUNC(MONTH, order_date) AS order_date,
        SUM(sales_amount) AS total_sales
    FROM Gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(MONTH, order_date)
    ) t

<p align="center">
<img src="https://github.com/user-attachments/assets/8a0d9368-34c7-4739-90a1-5440265310ad" />
</p>

I could also get the running total sales per year (in that case, I'd delete the "PARTITION BY YEAR" since it wouldn't make any sense):

    SELECT
        order_date,
        total_sales,
        SUM(total_sales) OVER (ORDER BY order_date) AS running_total_sales
    FROM
    (
    SELECT
        DATETRUNC(YEAR, order_date) AS order_date,
        SUM(sales_amount) AS total_sales
    FROM Gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(YEAR, order_date)
    ) A

<p align="center">
<img src="https://github.com/user-attachments/assets/d8626497-97ed-4e4f-b490-971c30e28b7d" />
</p>

Or the moving average:

    SELECT
        order_date,
        total_sales,
        SUM(total_sales) OVER (ORDER BY order_date) AS running_total_sales,
        AVG(avg_price) OVER (ORDER BY order_date) AS moving_average
    FROM
    (
    SELECT
        DATETRUNC(YEAR, order_date) AS order_date,
        SUM(sales_amount) AS total_sales,
        AVG(price) AS avg_price
    FROM Gold.fact_sales
    WHERE order_date IS NOT NULL
    GROUP BY DATETRUNC(YEAR, order_date)
    ) A

<p align="center">
<img src="https://github.com/user-attachments/assets/12b20162-6378-40ac-be51-8cdc55d4f3b7" />
</p>

### 3. Performance Analysis

The process of comparing the current value with a target value, to measure the success and compare the performance.

For example, I could analyze the yearly performance of products by comparing their sales to both the average sales performance of the product and
the previous year's sales:

    WITH sales_per_year AS
    (
    SELECT
        YEAR(s.order_date) order_year,
        p.product_name,
        SUM(s.sales_amount) current_sales	
    FROM Gold.fact_sales s LEFT JOIN Gold.dim_products p
    ON s.product_key = p.product_key
    WHERE s.order_date IS NOT NULL
    GROUP BY YEAR(s.order_date), p.product_name
    )
    SELECT
        order_year,
        product_name,
        current_sales,
        AVG(current_sales) OVER(PARTITION BY product_name) average_sales,
        current_sales - AVG(current_sales) OVER(PARTITION BY product_name) AS diff_avg,
        CASE WHEN current_sales - AVG(current_sales) OVER(PARTITION BY product_name) > 0 THEN 'Above Average'
             WHEN current_sales - AVG(current_sales) OVER(PARTITION BY product_name) < 0 THEN 'Below Average'
             ELSE 'Avg'
        END average_change,
        -- Year-Over-Year Analysis
        LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) py_sales,
        current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) diff_py,
        CASE WHEN current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) > 0 THEN 'Increase'
             WHEN current_sales - LAG(current_sales) OVER(PARTITION BY product_name ORDER BY order_year) < 0 THEN 'Decrease'
             ELSE 'No Change'
        END py_change
    FROM sales_per_year

<p align="center">
<img src="https://github.com/user-attachments/assets/0bce2444-b38e-499d-a44f-adccc72b39a7" />
</p>

Regarding the Year-Over-Year Analysis section, I could check instead the Month-Over-Month scenario by just changing function YEAR for MONTH.

YOY is good for long term trends analysis, while MOM is short term trend analysis (focus on the seasonality of the data).

### 4. Part-To-Whole Analysis

Analyze how an individual part is performing compared to the overall, allowing us to understand which category has the greatest impact on the
business.

For example, identify the categories that contribute the most to overall sales:

    WITH category_sales AS
    (
    SELECT
        p.category,
        SUM(s.sales_amount) total_sales
    FROM Gold.fact_sales s LEFT JOIN Gold.dim_products p
    ON s.product_key = p.product_key
    GROUP BY p.category
    )
    
    SELECT
        category,
        total_sales,
        SUM(total_sales) OVER() overall_sales,
        CONCAT(ROUND((CAST(total_sales AS FLOAT) / SUM(total_sales) OVER()) * 100, 2), '%') percentage_of_total
    FROM category_sales
    ORDER BY total_sales DESC

<p align="center">
<img src="https://github.com/user-attachments/assets/87732301-d549-49b7-8bc2-238aac5505ed" />
</p>

### 5. Data Segmentation

Group the data based on a specific range, helping to understand the correlation between two measures.

To do this I'll use measures, turning one of them into a dimension (categorization: I take one measure, and based on the range of this measure I am
building a new category), and then aggregating the other one based on the new category.

For example, segment products into cost ranges and count how many products fall into each segment:

    WITH product_segments AS (
    SELECT
        product_name,
        cost,
        CASE WHEN cost < 100 THEN 'Below 100'
             WHEN cost BETWEEN 100 AND 500 THEN '100-500'
             WHEN cost BETWEEN 500 AND 1000 THEN '500-1000'
             ELSE 'Above 1000'
        END cost_range
    FROM Gold.dim_products)
    
    SELECT
        cost_range,
        COUNT(product_name) total_products
    FROM product_segments
    GROUP BY cost_range
    ORDER BY total_products DESC

<p align="center">
<img src="https://github.com/user-attachments/assets/cf48c770-1a7b-4a58-948f-752da9f3624d" />
</p>

Another example:

Group customers into three segments based on their spending behaviour:

    -VIP: At least 12 months of history and spending more than 5000€.
    -Regular: At least 12 months of history but spending 5000€ or less.
    -New: Lifespan less than 12 months.

And find the total number of customers by each group.

    WITH customer_spending AS (
    SELECT
        c.customer_key,
        SUM(s.sales_amount) AS total_spending,
        MIN(order_date) AS first_order,
        MAX(order_date) AS last_order,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) lifespan
    FROM Gold.fact_sales s LEFT JOIN Gold.dim_customers c
    ON s.customer_key = c.customer_key
    GROUP BY c.customer_key)
    
    SELECT
        customer_segment,
        COUNT(customer_key) total_customers
    FROM (
    SELECT
        customer_key,
        CASE WHEN lifespan >= 12 AND total_spending > 5000 THEN 'VIP'
             WHEN lifespan >= 12 AND total_spending <= 5000 THEN 'Regular'
             ELSE 'New'
        END customer_segment
    FROM customer_spending) A
    GROUP BY customer_segment
    ORDER BY total_customers DESC

<p align="center">
<img src="https://github.com/user-attachments/assets/73a665cf-72cc-4011-9d3e-3642079dcbc7" />
</p>

