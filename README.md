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

To explore the structure of the database, I could review in depth the Object Explorer, but I also could use the following SQL queries:

  -- Explore all objects in the database

      SELECT * FROM INFORMATION_SCHEMA.TABLES

  -- Explore all columns in the database

      SELECT * FROM INFORMATION_SCHEMA.COLUMNS
