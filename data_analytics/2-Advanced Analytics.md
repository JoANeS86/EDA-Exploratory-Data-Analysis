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
