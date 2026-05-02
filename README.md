# Zepto Inventory — SQL Data Analysis

A hands-on SQL project where I analyzed Zepto's product inventory data to uncover pricing patterns, stock availability, and category-level business insights using PostgreSQL.

***

## About the Project

I've always wanted to work with real retail data rather than toy datasets. This project uses inventory data scraped from [Zepto](https://www.zeptonow.com/) (India's 10-minute grocery delivery platform) to practice the kind of SQL work that actually comes up in data analyst roles.

The dataset came from [Kaggle](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset/data?select=zepto_v2.csv) and is intentionally messy: zero-value prices, paise instead of rupees, duplicate product names across SKUs. Cleaning it up and making sense of it was the whole point.

***

## Dataset Overview

Each row is a SKU (Stock Keeping Unit). The same product can show up multiple times under different weights, pack sizes, or discount tiers, which is exactly how real e-commerce catalogs work.

| Column | Description |
|---|---|
| `sku_id` | Auto-generated primary key |
| `name` | Product name as listed on the app |
| `category` | Category (e.g., Snacks, Beverages, Fruits) |
| `mrp` | Maximum Retail Price (converted from paise to ₹) |
| `discountPercent` | Discount applied on MRP |
| `discountedSellingPrice` | Final price after discount (converted to ₹) |
| `availableQuantity` | Units currently in inventory |
| `weightInGms` | Product weight in grams |
| `outOfStock` | `TRUE` if unavailable |
| `quantity` | Pack units (or grams for loose items) |

***

## What's Inside the SQL File

### Table Setup

```sql
DROP TABLE IF EXISTS zepto;

CREATE TABLE zepto (
  sku_id SERIAL PRIMARY KEY,
  category VARCHAR(120),
  name VARCHAR(150) NOT NULL,
  mrp NUMERIC(8,2),
  discountPercent NUMERIC(5,2),
  availableQuantity INTEGER,
  discountedSellingPrice NUMERIC(8,2),
  weightInGms INTEGER,
  outOfStock BOOLEAN,
  quantity INTEGER
);
```

### Loading the Data

Use pgAdmin's Import tool, or run this directly:

```sql
\copy zepto(category, name, mrp, discountPercent, availableQuantity,
            discountedSellingPrice, weightInGms, outOfStock, quantity)
FROM 'data/zepto_v2.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', QUOTE '"', ENCODING 'UTF8');
```

> If you hit a UTF-8 encoding error, open the CSV in Excel and re-save it as **CSV UTF-8 (Comma delimited)**.

### Exploration

- Total row count and data preview
- NULL check across all columns
- All distinct product categories
- In-stock vs out-of-stock breakdown
- Products with multiple SKUs (same name, different variants)

### Cleaning

- Removed rows where `mrp = 0` or `discountedSellingPrice = 0`
- Converted both price columns from paise to rupees:

```sql
UPDATE zepto
SET mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;
```

### Business Queries

- Top 10 most discounted products
- Premium products (`mrp > ₹300`) that are currently out of stock, useful for restocking decisions
- Estimated revenue per category using price x available quantity
- High-MRP products (`> ₹500`) with low discounts (`< 10%`), flagging potential pricing gaps
- Top 5 categories by average discount
- Price-per-gram analysis for products over 100g
- Weight segmentation: `Low` (< 1000g), `Medium` (< 5000g), `Bulk` (>= 5000g)
- Total inventory weight by category

***

## How to Run It

1. Clone the repo and open the project folder
2. Set up a PostgreSQL database (pgAdmin works well)
3. Run `zepto_SQL_project.sql`, it handles table creation, cleaning, and all queries in order
4. Download `zepto_v2.csv` from Kaggle and import it using the method above

***

## Tools Used

- PostgreSQL
- pgAdmin
- Dataset: Kaggle (Zepto Inventory)


