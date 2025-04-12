# Exploratory Data Analysis (EDA)

## Understand Data

First thing to do when analyzing data is identifying Dimensions and Measures:

<p align="center">
<img src="https://github.com/user-attachments/assets/c74770d7-e928-40af-80bb-ec5f795e2cab" />
</p>

The reason why I must identify Dimensions and Measures is because when I'm analyzing data, I'll end up grouping the data by Dimensions (Countries,
Products, Categories...) and also, I'll be answering questions ("How much?", "How many?"...) for what I'll need the Measures.

## Steps for EDA

### 1. Database Exploration

To explore the structure of the database, I could review in depth the Object Explorer, but I also could use the following SQL queries (or variations of them):

  -- Explore all objects in the database

      SELECT * FROM INFORMATION_SCHEMA.TABLES

  -- Explore all columns in the database

      SELECT * FROM INFORMATION_SCHEMA.COLUMNS

### 2. Dimensions Exploration

Use DISTINCT keyword to identify unique values in each Dimension, then I'll know how data could be grouped or segemented (which will be useful for later analysis).

### 3. Date Exploration

Identify the earliest and latest dates (boundaries) to have more understanding of the timespan of our business, which will help me later by making different type of complex analysis.

### 4. Measures Exploration

Calculate the key metrics of the business (Big Number), by using aggregate functions in SQL for any measure in the dataset.

For example:

  -- Find the Total Sales.  
  
  -- Find how many items are sold.
  
  -- Find the Average Selling Price.
  
  -- Find the Total Number of Orders.
  
  -- Find the Total Number of Products.
  
  -- Find the Total Number of Customers.
  
  -- Find the Total Number of Customers that have placed an order.

Then I could put together all those metrics to generate a report containing the key metrics of the business (Big Picture of the Business):

      SELECT 'Total Sales' AS measure_name, SUM(sales_amount) AS measure_value FROM Gold.fact_sales
      UNION ALL
      SELECT 'Total Quantity', SUM(quantity) FROM Gold.fact_sales
      UNION ALL
      SELECT 'Average Price', AVG(price) FROM Gold.fact_sales
      UNION ALL
      SELECT 'Total Nr. Orders', COUNT(DISTINCT order_number) FROM Gold.fact_sales
      UNION ALL
      SELECT 'Total Nr. Products', COUNT(product_name) FROM Gold.dim_products
      UNION ALL
      SELECT 'Total Nr. Customers', COUNT(customer_key) FROM Gold.dim_customers

<p align="center">
<img src="https://github.com/user-attachments/assets/21bbce20-aaed-4f36-8795-951932a9e001" />
</p>

### 5. Magnitude

Compare the measure values by categories, to get more insights about the business and better understand the differences between these categories.

### 6. Ranking Analysis

Order the values of dimensions by measure (Top N / Bottom N).



























