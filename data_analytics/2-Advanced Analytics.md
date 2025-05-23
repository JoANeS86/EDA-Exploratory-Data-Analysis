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

### 6. Build Customer Report

Customer Report

Purpose: This report consolidates key customer metrics and behaviors.

Highlights:

    1. Gathers essential fields such as names, ages, and transaction details.
    2. Segments customers into categories (VIP, Regular, New) and age groups.
    3. Aggregates customer-level metrics:

        - Total Orders
        - Total Sales
        - Total Quantity Purchased
        - Total Products
        - Lifespan (in months)

    4. Calculates valuable KPIs:

        - Recency (months since last order)
        - Average Order Value
        - Average Monthly Spend

SQL Code:

    CREATE VIEW Gold.report_customers AS
    WITH base_query AS (
    /*-------------------------------------------------------------------------
    1/ Base Query: Retrieves core columns from tables.
    -------------------------------------------------------------------------*/
    SELECT
        s.order_number,
        s.product_key,
        s.order_date,
        s.sales_amount,
        s.quantity,
        c.customer_key,
        c.customer_number,
        CONCAT(c.first_name, ' ', c.last_name) customer_name,
        DATEDIFF(YEAR, birthdate, GETDATE()) age
    FROM Gold.fact_sales s LEFT JOIN Gold.dim_customers c
    ON s.customer_key = c.customer_key
    WHERE order_date IS NOT NULL)
    
    , customer_aggregation AS (
    /*-------------------------------------------------------------------------
    2/ Customer Aggregations: Summarizes key metrics at the customer level.
    -------------------------------------------------------------------------*/
    SELECT
        customer_key,
        customer_number,
        customer_name,
        age,
        COUNT(DISTINCT order_number) AS total_orders,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        COUNT(product_key) AS total_products,
        MAX(order_date) AS last_order_date,
        DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan
    FROM base_query
    GROUP BY customer_key, customer_number, customer_name, age)
    
    SELECT
        customer_key,
        customer_number,
        customer_name,
        age,
        CASE WHEN age < 20 THEN 'Under 20'
             WHEN age BETWEEN 20 AND 29 THEN '20-29'
             WHEN age BETWEEN 30 AND 39 THEN '30-39'
             WHEN age BETWEEN 40 AND 49 THEN '40-49'
             ELSE '50 and above'
        END age_group,
        CASE WHEN lifespan >= 12 AND total_sales > 5000 THEN 'VIP'
             WHEN lifespan >= 12 AND total_sales <= 5000 THEN 'Regular'
             ELSE 'New'
        END customer_segment,
        last_order_date,
        DATEDIFF(MONTH, last_order_date, GETDATE()) AS recency,
        total_orders,
        total_sales,
        total_quantity,
        total_products,
        lifespan,
        CASE WHEN total_orders = 0 THEN 0
             ELSE total_sales / total_orders
        END AS avg_order_value,
        CASE WHEN lifespan = 0 THEN total_sales
             ELSE total_sales / lifespan
        END AS avg_monthly_spend
    FROM customer_aggregation

Now I can use this View to run different analysis on it, like reviewing the total number of customers and total sales per age or customer segment:

    SELECT
        age_group,
        COUNT(customer_number) AS total_customers,
        SUM(total_sales) AS total_sales
    FROM Gold.report_customers
    GROUP BY age_group
    
    
    SELECT
        customer_segment,
        COUNT(customer_number) AS total_customers,
        SUM(total_sales) AS total_sales
    FROM Gold.report_customers
    GROUP BY customer_segment


<p align="center">
<img src="https://github.com/user-attachments/assets/6dbb30bb-7b66-46c7-81f3-934f9a51b05f" />
</p>

### 7. Build Product Report

Product Report

Purpose: This report consolidates key product metrics and behaviors.

Highlights:

    1. Gathers essential fields such as product name, category, subcategory and cost.
    2. Segments products by revenue to identify High-Performers, Mid-Range, or Low-Performers.
    3. Aggregates product-level metrics:

        - Total Orders
        - Total Sales
        - Total Quantity Sold
        - Total Customers (unique)
        - Lifespan (in months)

    4. Calculates valuable KPIs:

        - Recency (months since last sales)
        - Average Order Revenue (AOR)
        - Average Monthly Revenue

SQL Code:

    CREATE VIEW Gold.report_products AS
        WITH base_query AS (
        /*-------------------------------------------------------------------------
        1/ Base Query: Retrieves core columns from tables.
        -------------------------------------------------------------------------*/
        SELECT
            s.order_number,
            s.order_date,
            s.customer_key,
            s.sales_amount,
            s.quantity,
            p.product_key,
            p.product_name,
            p.category,
            p.subcategory,
            p.cost
        FROM Gold.fact_sales s LEFT JOIN Gold.dim_products p
        ON s.product_key = p.product_key
        WHERE order_date IS NOT NULL)
        
        , product_aggregation AS (
        /*-------------------------------------------------------------------------
        2/ Product Aggregations: Summarizes key metrics at the product level.
        -------------------------------------------------------------------------*/
        SELECT
            product_key,
            product_name,
            category,
            subcategory,
            cost,
            DATEDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
            MAX(order_date) AS last_sale_date,
            COUNT(DISTINCT order_number) AS total_orders,
            COUNT(DISTINCT customer_key) AS total_customers,
            SUM(sales_amount) AS total_sales,
            SUM(quantity) AS total_quantity,
            ROUND(AVG(CAST(sales_amount AS FLOAT) / NULLIF(quantity, 0)), 1) AS avg_selling_price
        FROM base_query
        GROUP BY product_key, product_name, category, subcategory, cost)
        /*-------------------------------------------------------------------------
        3/ Final Query: Combines all product results into one output.
        -------------------------------------------------------------------------*/ 
        SELECT
            product_key,
            product_name,
            category,
            subcategory,
            cost,
            last_sale_date,
            DATEDIFF(MONTH, last_sale_date, GETDATE()) AS recency_in_months,
            CASE WHEN total_sales < 50000 THEN 'High-Performer'
                 WHEN total_sales < 50000 THEN 'Mid-Range'
                 ELSE 'Low Performer'
            END product_segment,
            lifespan,
            total_orders,
            total_sales,
            total_quantity,
            total_customers,
            avg_selling_price,
            CASE WHEN total_orders = 0 THEN 0
                 ELSE total_sales / total_orders
            END AS avg_order_revenue,
            CASE WHEN lifespan = 0 THEN total_sales
                 ELSE total_sales / lifespan
            END AS avg_monthly_revenue
        FROM product_aggregation

        
