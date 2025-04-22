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












