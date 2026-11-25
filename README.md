# 🛒 Zepto E-commerce Inventory Analysis (SQL)

A real-world data analyst portfolio project built on an e-commerce inventory dataset scraped from [Zepto](https://www.zeptonow.com/), one of India's fastest-growing quick-commerce startups. The project simulates how analysts in e-commerce and retail work behind the scenes: from setting up a messy database, to exploring and cleaning the data, to answering business questions with SQL.

---

## 📌 Project Overview

The goal is to use SQL to:

- ✅ Set up a messy, real-world e-commerce inventory database
- ✅ Perform Exploratory Data Analysis (EDA) on product categories, availability, and pricing inconsistencies
- ✅ Clean the data by checking for nulls, removing invalid entries, and converting prices from paise to rupees
- ✅ Write business-driven queries that surface insights on pricing, inventory, stock availability, revenue, and more

**Tools used:** PostgreSQL, pgAdmin (or `psql`)

---

## 📁 Dataset Overview

The dataset comes from [Kaggle](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset/data?select=zepto_v2.csv) and was originally scraped from Zepto's official product listings. It mimics what you would typically find in a real e-commerce inventory system.

Each row represents a unique **SKU (Stock Keeping Unit)**. Product names can repeat because the same product may be listed multiple times with different package sizes, weights, discounts, or categories to improve visibility, which is exactly how real catalog data behaves.

### 🧾 Columns

| Column | Description |
|---|---|
| `sku_id` | Unique identifier for each product entry (synthetic primary key) |
| `name` | Product name as it appears on the app |
| `category` | Product category (Fruits, Snacks, Beverages, etc.) |
| `mrp` | Maximum Retail Price (originally in paise, converted to ₹) |
| `discountPercent` | Discount applied on MRP |
| `discountedSellingPrice` | Final price after discount (also converted to ₹) |
| `availableQuantity` | Units available in inventory |
| `weightInGms` | Product weight in grams |
| `outOfStock` | Boolean flag indicating stock availability |
| `quantity` | Number of units per package (mixed with grams for loose produce) |

---

## 🔄 Project Workflow

### 1. Database & Table Creation

Create the table with appropriate data types:

```sql
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

### 2. Data Import

Load the CSV using pgAdmin's import feature. If that isn't available, use `\copy` from `psql`:

```sql
\copy zepto(category,name,mrp,discountPercent,availableQuantity,
            discountedSellingPrice,weightInGms,outOfStock,quantity)
FROM 'data/zepto_v2.csv'
WITH (FORMAT csv, HEADER true, DELIMITER ',', QUOTE '"', ENCODING 'UTF8');
```

> **Note:** A UTF-8 encoding error came up during import. It was fixed by re-saving the CSV in **CSV UTF-8** format.

### 3. 🔍 Data Exploration

- Counted the total number of records
- Viewed a sample of the data to understand its structure
- Checked for null values across all columns
- Listed the distinct product categories
- Compared in-stock vs. out-of-stock product counts
- Detected product names appearing multiple times (different SKUs)

### 4. 🧹 Data Cleaning

- Identified and removed rows where MRP or discounted selling price was zero
- Converted `mrp` and `discountedSellingPrice` from paise to rupees for consistency and readability

### 5. 📊 Business Insights

| # | Question | Key SQL concepts |
|---|---|---|
| 1 | What are the top 10 best-value products by discount percentage? | `ORDER BY`, `LIMIT`, `DISTINCT` |
| 2 | Which high-MRP products are currently out of stock? | `WHERE`, filtering on boolean |
| 3 | What is the estimated revenue per category? | `SUM`, `GROUP BY` |
| 4 | Which expensive products (MRP > ₹500) have minimal discount (< 10%)? | Multi-condition filtering |
| 5 | Which 5 categories offer the highest average discount? | `AVG`, `ROUND`, `GROUP BY` |
| 6 | What is the price per gram for products 100g and above? | Calculated columns |
| 7 | How do products group into Low, Medium, and Bulk by weight? | `CASE WHEN` |
| 8 | What is the total inventory weight per category? | `SUM`, `GROUP BY` |

---

## 💻 Sample Queries

**Estimated revenue per category**

```sql
SELECT category,
       SUM(discountedSellingPrice * availableQuantity) AS total_revenue
FROM zepto
GROUP BY category
ORDER BY total_revenue DESC;
```

**Top 5 categories by average discount**

```sql
SELECT category,
       ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

**Grouping products by weight**

```sql
SELECT DISTINCT name, weightInGms,
  CASE WHEN weightInGms < 1000 THEN 'Low'
       WHEN weightInGms < 5000 THEN 'Medium'
       ELSE 'Bulk'
  END AS weight_category
FROM zepto;
```

The full script, including all exploration, cleaning, and analysis queries, is in [`zepto_analysis.sql`](zepto_analysis.sql).

---

## 🚀 How to Run

1. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   cd <your-repo>
   ```
2. Download `zepto_v2.csv` from [Kaggle](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset/data?select=zepto_v2.csv) and place it in the `data/` folder.
3. Create a PostgreSQL database and open it in pgAdmin or `psql`.
4. Run the table creation script, then import the CSV (see [Data Import](#2-data-import)).
5. Run the exploration, cleaning, and analysis queries in order.

> ⚠️ Run the paise-to-rupees `UPDATE` **only once**. Running it again will divide the prices by 100 a second time.

---

## 🧠 Skills Demonstrated

- Database and schema design with appropriate data types
- Importing and troubleshooting data (encoding issues)
- Exploratory data analysis with SQL
- Data cleaning and standardization
- Aggregations, grouping, and conditional logic
- Translating business questions into SQL

---

## 📂 Repository Structure

```
├── data/
│   └── zepto_v2.csv
├── zepto_analysis.sql
└── README.md
```

---

## 🙌 Acknowledgements

Dataset by [palvinder2006 on Kaggle](https://www.kaggle.com/datasets/palvinder2006/zepto-inventory-dataset). Data was originally scraped from Zepto's product listings and is used here for educational and portfolio purposes only.
