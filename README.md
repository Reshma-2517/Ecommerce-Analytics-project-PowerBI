# Brazilian E-Commerce Analytics — Business Performance & Insights

## Project Overview

This project analyzes the Brazilian E-Commerce Public Dataset by Olist to understand how an online marketplace performs across sales, order fulfilment, delivery operations, customer experience, and purchasing behaviour.

The solution combines a relational MySQL analytics database with an interactive Power BI report. MySQL is used to store, validate, transform, and aggregate the source data into analysis-ready structures. Power BI connects to the prepared analytical data and presents the results through business-focused report pages.

The analysis covers the period represented in the source dataset: 2016–2018. It is designed to help stakeholders move beyond isolated transaction counts and understand the operational and commercial factors that influence marketplace performance.

## Business Context

An e-commerce marketplace must coordinate customers, sellers, products, payments, order processing, and delivery. These activities are connected: a delay in fulfilment can affect customer satisfaction; product and seller mix can influence sales performance; and purchasing patterns can vary across dates and regions.

Looking at these areas separately can hide important relationships. For example, a high order volume does not automatically indicate strong operational performance if a substantial share of orders is delayed or cancelled. Similarly, revenue growth is more useful when considered alongside product mix, freight charges, and customer experience.

This project brings the available transactional data together to provide a consolidated view of marketplace performance.

## Business Problem

The business needs a clear way to monitor sales performance, understand order and delivery outcomes, evaluate customer satisfaction, and identify purchasing trends. Without a consolidated analytical view, it is difficult to determine where performance is strong, where operational issues may exist, and which areas deserve further investigation.

The central business problem is the lack of an integrated, measurable view of commercial performance and the customer journey across the order lifecycle.

## Problem Statement

Develop a MySQL-based analytical data layer and a Power BI reporting solution that combine the Olist transactional datasets, calculate consistent business KPIs, and present actionable analysis of sales, delivery, customer satisfaction, and purchasing behaviour.

The solution should support data-quality validation, repeatable KPI calculation, and business interpretation rather than relying only on raw transaction tables.

## Project Objectives

- Consolidate the related Olist datasets into a structured MySQL database.
- Validate important data relationships and identify potential data-quality issues before analysis.
- Build reusable analytical views and KPI calculations for reporting.
- Measure order volume, order status, merchandise revenue, freight charges, and product sales.
- Evaluate delivery performance, including on-time and late delivery outcomes.
- Examine customer review scores and satisfaction patterns.
- Understand purchasing activity across months, weekdays, weekends, and hours.
- Identify meaningful trends and relationships that can inform operational and commercial decisions.
- Present the analysis in an interactive Power BI report.

## Dataset

**Source:** [Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

The dataset contains approximately 100,000 orders from the Olist marketplace, covering 2016 to 2018. It is divided into multiple CSV files because each file represents a different business entity or transaction area. The files are related through identifiers such as `order_id`, `customer_id`, `product_id`, and `seller_id`.

The dataset is real commercial data that has been anonymised. The source describes the dataset as information from orders made through multiple marketplaces in Brazil. See the Kaggle dataset page for its data card, schema, version information, and licence.

### Source CSV files

| CSV file | Business information | Important analytical use |
|---|---|---|
| `olist_customers_dataset.csv` | Customer identifiers and customer location | Customer-level analysis and geographic segmentation |
| `olist_orders_dataset.csv` | Order status and order lifecycle timestamps | Order volume, status, delivery duration, and delivery performance |
| `olist_order_items_dataset.csv` | Products, sellers, item prices, freight values, and shipping limits | Merchandise value, freight, item volume, seller and product analysis |
| `olist_order_reviews_dataset.csv` | Review scores, comments, and review timestamps | Customer satisfaction analysis |
| `olist_products_dataset.csv` | Product categories and physical product attributes | Product and category analysis |
| `olist_sellers_dataset.csv` | Seller identifiers and seller location | Seller-level and regional analysis |
| `product_category_name_translation.csv` | Portuguese-to-English product category mapping | Readable product-category reporting |

### Important dataset interpretation notes

- An order can contain multiple order items. Therefore, the number of rows in the order-items dataset is not expected to equal the number of orders.
- An order can have multiple payment records. Payment rows should not be joined to order items without controlling for row multiplication.
- Customer records include both `customer_id` and `customer_unique_id`. The source explains that an order is associated with a customer ID, while the unique customer ID can be used to identify repeat purchases across orders.
- Delivery timestamps can be null for orders that have not reached the relevant delivery milestone. Such records should not automatically be treated as zero-day deliveries.
- Review data is not guaranteed to contain a review for every order.
- Monetary values in the source represent Brazilian currency (BRL). They should not be labelled as INR or USD without an explicit currency conversion.
- The source dataset is historical and should not be interpreted as a live feed of current Olist operations.

## Technology Stack

| Technology | Role in the project |
|---|---|
| MySQL 8.0 or later | Relational database, data storage, SQL transformations, validation, KPI calculations, analytical views, and snapshot/alert logic |
| SQL | Data definition, data loading, joins, aggregations, analytical queries, stored procedures, and validation |
| Power BI Desktop | Data modelling, measures, report visuals, filters, and business presentation |
| Kaggle | Source and download location for the Olist public dataset |

## Solution Architecture

The project follows a database-to-reporting workflow:

1. **Source acquisition:** Download the Olist CSV files from Kaggle.
2. **Relational storage:** Create the `olist_analytics` MySQL database and its source tables.
3. **Data loading:** Import the CSV files into the corresponding MySQL tables.
4. **Data validation:** Run integrity checks to assess duplicates, missing relationships, invalid values, and unexpected records.
5. **SQL transformation:** Join and aggregate the source tables while controlling for one-to-many relationships.
6. **Analytical layer:** Use reusable views and KPI calculations to provide consistent reporting datasets.
7. **Power BI connection:** Connect Power BI to the MySQL database and use the prepared analytical data.
8. **Reporting:** Explore sales, delivery, customer satisfaction, and purchasing behaviour through the report pages.

The intended analytical database is named `olist_analytics`.

## MySQL Database and Data Import

This section explains how to obtain the source data, prepare MySQL, and load the CSV files into the database. The data can be downloaded directly from Kaggle; it does not need to be manually copied from another database.

### 1. Download the Olist dataset

1. Open the [Olist dataset on Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).
2. Sign in to Kaggle if prompted.
3. Select **Download** to download the dataset archive.
4. Extract the downloaded ZIP archive.
5. Confirm that the extracted CSV files include the source files listed in the Dataset section.
6. Keep the CSV files accessible from the computer on which MySQL Workbench or the MySQL client is running.


### 2. Create the MySQL database

Open MySQL Workbench and connect to the local MySQL Server. Run:

```sql
CREATE DATABASE IF NOT EXISTS olist_analytics;
USE olist_analytics;
```

The source tables must exist before CSV data is loaded. Run the project's table-creation SQL first. The table definitions should match the source CSV headers and data types. In particular:

- Identifiers should use compatible character types and lengths.
- Dates and timestamps should use appropriate `DATE` or `DATETIME` types.
- Price, freight, and payment values should use `DECIMAL`, not floating-point types, for reliable monetary calculations.
- Review scores and instalment counts should use numeric types.
- Fields that are legitimately missing in the source should permit `NULL`.

Do not load a CSV into a table whose column order or data types do not match the file.

### 3. Enable local CSV loading

`LOAD DATA LOCAL INFILE` reads the file from the client computer and transfers its contents to the MySQL server. Both the server and the client must permit local loading.

Check the server setting:

```sql
SHOW VARIABLES LIKE 'local_infile';
```

If local loading is disabled, enable it for the current server session with an account that has the required privileges:

```sql
SET GLOBAL local_infile = 1;
```

If the command is not permitted, use a MySQL administrator account or enable `local_infile` in the MySQL server configuration and restart the server. In MySQL Workbench, also ensure that the connection/client allows the use of `LOAD DATA LOCAL INFILE`. Reconnect after changing client or server settings if required.

`LOCAL` is important: the CSV path is resolved on the computer running the client, not necessarily on the database server. MySQL documents the syntax and security behaviour in the [LOAD DATA statement reference](https://dev.mysql.com/doc/refman/8.0/en/load-data.html).

### 4. Import the CSV files

The following examples show the standard import pattern for Olist CSV files. Update each file path to the actual location on your computer. The example uses forward slashes in Windows paths because they are accepted by MySQL.

Run each statement after its target table has been created. The target column list must match the actual table definition. If the project schema uses different column names, map the CSV fields to the corresponding target columns in the correct order.

```sql
USE olist_analytics;

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_customers_dataset.csv'
INTO TABLE olist_customers_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(customer_id, customer_unique_id, customer_zip_code_prefix,
 customer_city, customer_state);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_orders_dataset.csv'
INTO TABLE olist_orders_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(order_id, customer_id, order_status, order_purchase_timestamp,
 order_approved_at, order_delivered_carrier_date,
 order_delivered_customer_date, order_estimated_delivery_date);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_order_items_dataset.csv'
INTO TABLE olist_order_items_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(order_id, order_item_id, product_id, seller_id,
 shipping_limit_date, price, freight_value);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_order_payments_dataset.csv'
INTO TABLE olist_order_payments_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(order_id, payment_sequential, payment_type,
 payment_installments, payment_value);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_order_reviews_dataset.csv'
INTO TABLE olist_order_reviews_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(review_id, order_id, review_score, review_comment_title,
 review_comment_message, review_creation_date,
 review_answer_timestamp);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_products_dataset.csv'
INTO TABLE olist_products_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(product_id, product_category_name,
 product_name_length, product_description_length,
 product_photos_qty, product_weight_g,
 product_length_cm, product_height_cm, product_width_cm);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_sellers_dataset.csv'
INTO TABLE olist_sellers_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(seller_id, seller_zip_code_prefix, seller_city, seller_state);

LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/product_category_name_translation.csv'
INTO TABLE product_category_name_translation
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(product_category_name, product_category_name_english);
```

If geolocation is included in the database, import `olist_geolocation_dataset.csv` into its matching table using the file's header order:

```sql
LOAD DATA LOCAL INFILE 'D:/olist_project_analysis/olist_geolocation_dataset.csv'
INTO TABLE olist_geolocation_dataset
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES
(geolocation_zip_code_prefix, geolocation_lat,
 geolocation_lng, geolocation_city, geolocation_state);
```

**Line-ending note:** The examples use Windows CRLF line endings (`'\r\n'`). If the import reports malformed rows or places multiple records into one row, check the CSV line endings and change the `LINES TERMINATED BY` clause to `'\n'` when appropriate.

**Timestamp and empty-value note:** Confirm that blank CSV fields are handled as `NULL` where the target columns permit nulls. If timestamp columns are stored as text during staging, convert them to `DATETIME` in a controlled transformation step. Do not silently replace missing delivery timestamps with arbitrary dates.

**Column-mapping note:** The statements above show the standard Olist source column order. If the SQL table definitions use a different order, always provide an explicit column list that maps each CSV field to the correct target column. Do not assume that `LOAD DATA` will infer the mapping from the header row; `IGNORE 1 LINES` skips the header, it does not map columns by header name.

### 5. Validate that the data was imported

After loading, check that each table contains records:

```sql
SELECT 'customers' AS table_name, COUNT(*) AS row_count
FROM olist_customers_dataset
UNION ALL
SELECT 'orders', COUNT(*) FROM olist_orders_dataset
UNION ALL
SELECT 'order_items', COUNT(*) FROM olist_order_items_dataset
UNION ALL
SELECT 'payments', COUNT(*) FROM olist_order_payments_dataset
UNION ALL
SELECT 'reviews', COUNT(*) FROM olist_order_reviews_dataset
UNION ALL
SELECT 'products', COUNT(*) FROM olist_products_dataset
UNION ALL
SELECT 'sellers', COUNT(*) FROM olist_sellers_dataset
UNION ALL
SELECT 'category_translation', COUNT(*) FROM product_category_name_translation;
```

If geolocation was imported, check it separately:

```sql
SELECT COUNT(*) AS geolocation_rows
FROM olist_geolocation_dataset;
```

The published dataset contains approximately 100,000 orders, but row counts differ across tables because each table represents a different entity or event. For example, order items and payments can contain multiple rows per order. Compare counts with the downloaded dataset version rather than expecting every table to have the same number of rows.

### 6. Run data-quality and relationship checks

Before building analytical outputs, validate the key relationships and business rules. Examples include:

```sql
-- Duplicate order identifiers in the orders table
SELECT order_id, COUNT(*) AS occurrences
FROM olist_orders_dataset
GROUP BY order_id
HAVING COUNT(*) > 1;

-- Order items that do not have a matching order
SELECT oi.order_id
FROM olist_order_items_dataset AS oi
LEFT JOIN olist_orders_dataset AS o
    ON oi.order_id = o.order_id
WHERE o.order_id IS NULL;

-- Payments that do not have a matching order
SELECT p.order_id
FROM olist_order_payments_dataset AS p
LEFT JOIN olist_orders_dataset AS o
    ON p.order_id = o.order_id
WHERE o.order_id IS NULL;

-- Review scores outside the expected 1–5 range
SELECT *
FROM olist_order_reviews_dataset
WHERE review_score NOT BETWEEN 1 AND 5;

-- Negative item prices or freight values
SELECT *
FROM olist_order_items_dataset
WHERE price < 0 OR freight_value < 0;

-- Non-positive payment values
SELECT *
FROM olist_order_payments_dataset
WHERE payment_value <= 0;
```

These are examples of validation checks, not a claim that every check must return zero rows in every dataset version. Investigate exceptions before deciding whether to exclude, correct, or retain them. Preserve legitimate null values where they represent an incomplete order lifecycle.

### 7. Execute the analytical SQL layer

Once the raw data is loaded and validated, execute the project's SQL modules in the order required by their dependencies. The analytical layer includes reusable views and KPI outputs used to avoid repeating complex joins in every report query.

The project has used analytical objects including:

- `vw_order_item_summary`
- `vw_order_payment_summary`
- `vw_order_review_summary`
- `vw_order_summary`
- `vw_order_item_detail`
- `vw_category_performance`
- `vw_kpi_order_metrics`
- `vw_kpi_results`
- `vw_kpi_dashboard`

These objects provide consolidated order-level, item-level, category-level, and KPI-oriented datasets. Their definitions in the SQL project are the source of truth for the exact calculation logic.

The SQL work also includes validation, performance review, business analysis, alerting, and KPI snapshot functionality. In particular, the snapshot process stores KPI results at a point in time so that captured results can be reviewed and compared. The alerting process records rule-based findings in a business-alert table. These database features are separate from the Power BI visuals; they support monitoring and auditability in the SQL layer.

## Power BI Report

The Power BI report is organised around business questions rather than the physical CSV files. Its report pages cover the project context and the major analytical areas.

### Report pages and purpose

| Report page | Purpose |
|---|---|
| Cover Page | Introduces the Brazilian e-commerce business performance and insights report. |
| Business Problem | Explains the business context and the need for consolidated performance analysis. |
| Problem Statement | Describes the analytical solution being developed. |
| Introduction | Provides context for the dataset and the reporting exercise. |
| Order & Delivery Performance | Examines order volume and status, delivered orders, late orders, on-time delivery rate, and delivery trends. |
| Sales Performance | Analyses merchandise revenue, freight charges, items sold, product-category performance, and monthly revenue trends. |
| Customer Experience & Satisfaction | Examines review volume, average review score, five-star review rate, review-score distribution, and satisfaction patterns across Brazilian states and product categories. |
| Purchasing Behavior & Trends | Explores monthly purchasing trends, peak-day activity, average daily orders, active purchase days, and purchasing activity by day and hour. |
| Insights | Brings the analytical findings together for business interpretation. |
| Recommendations | Translates the findings into possible business actions. |
| Conclusion | Summarises the overall analytical outcome. |
| Thank You | Closing page. |

The report includes KPI cards, trend charts, category comparisons, distribution visuals, geographic comparisons, and time-based analysis. Interactive report elements such as filters and slicers allow the displayed results to be explored by the available dimensions.

### Main analytical areas

#### 1. Order and delivery performance

This section evaluates the order lifecycle and fulfilment outcomes. It includes measures and views such as:

- Total orders.
- Order status distribution.
- Delivered orders.
- Late orders.
- On-time delivery rate.
- Monthly on-time delivery trends.
- Late orders by customer state.

The purpose is to distinguish demand from fulfilment performance. Order volume describes activity, while delivery measures indicate how effectively orders progress through the fulfilment process.

#### 2. Sales performance and revenue

This section examines commercial activity through measures such as:

- Merchandise revenue.
- Freight charges.
- Items sold.
- Total orders.
- Monthly revenue trend.
- Product-category performance.

Merchandise revenue represents the value of products sold based on item prices. Freight charges represent the shipping amounts associated with order items. These should be interpreted separately because freight is not the same as merchandise value. When comparing totals, use the exact definitions in the SQL views and measures to avoid double counting.

#### 3. Customer experience and satisfaction

This section evaluates customer feedback using review data. It includes:

- Total reviews.
- Average review score.
- Five-star review rate.
- Review-score distribution.
- Customer satisfaction score.
- Satisfaction comparisons across states and product categories.
- Month-over-month satisfaction movement.
- Repeated-customer rate, where supported by the customer identifiers and measure definition.

Review scores range from 1 to 5 in the source dataset. They provide a customer-feedback signal, but they do not explain the reason for satisfaction or dissatisfaction on their own. They are most useful when interpreted alongside delivery performance, product categories, and order outcomes.

#### 4. Purchasing behaviour and trends

This section studies when customers place orders and how activity changes over time. It includes:

- Peak-day orders.
- Average daily orders.
- Active purchase days.
- Average orders per active day.
- Purchasing activity by day and hour.
- Weekday and weekend comparisons.
- Monthly purchasing patterns.

These views help identify periods of higher activity and support planning for staffing, promotions, and operational capacity. A peak in order count should be interpreted as demand concentration, not automatically as a peak in profit.

## KPI Definitions and Interpretation

The following definitions describe the business meaning of the key measures used in the project. The exact implementation, filters, and aggregation level should be taken from the SQL view or Power BI measure used in the report.

| KPI / measure | Business meaning | Interpretation |
|---|---|---|
| Total Orders | Count of distinct orders | Indicates the volume of marketplace transactions |
| Delivered Orders | Orders whose status indicates delivery completion | Measures completed fulfilment volume |
| Canceled Orders | Orders marked as canceled | Helps monitor order loss before fulfilment |
| Cancellation Rate | Canceled orders divided by the relevant total-order base | Indicates the share of orders that were cancelled |
| Merchandise Revenue | Sum of item prices at the order-item level | Measures product value sold, excluding freight |
| Freight Charges | Sum of order-item freight values | Measures shipping charges associated with items |
| Average Order Value | Average merchandise value per order, according to the implemented definition | Indicates the typical merchandise value of an order |
| Average Delivery Days | Average elapsed time between the selected order and delivery timestamps | Measures delivery duration for eligible orders |
| On-Time Delivery Rate | Share of eligible delivered orders delivered by the estimated date | Measures delivery reliability |
| Average Review Score | Average customer review score | Provides an overall customer-feedback indicator |
| Total Payment Value | Sum of payment values at the payment-record level | Measures recorded payment value; reconcile it carefully against order and item totals |
| Items Sold | Count of order-item records or item quantity, depending on the implemented measure | Measures product units/line items represented in the source |
| Five-Star Review Rate | Share of valid reviews with a score of 5 | Indicates the proportion of top-rated reviews |
| Repeated Customers Rate | Share of customers identified as repeat purchasers using `customer_unique_id` | Indicates customer repeat-purchase behaviour |

**Important:** KPIs can change depending on the denominator, date field, order-status filter, and aggregation level. For example, an on-time delivery rate should normally be calculated only for orders with the required delivery timestamps. A payment total should be aggregated at payment grain, while merchandise revenue should be aggregated at order-item grain. Joining both tables directly can multiply values if the join creates multiple rows per order.

## Data Modelling and Analytical Considerations

The Olist data is relational and contains one-to-many relationships. Correct analytical modelling is essential to avoid inflated counts and sums.

- `orders` is the central order-level entity.
- `customers` provides customer and geographic attributes.
- `order_items` contains item-level details and can have multiple records for one order.
- `order_payments` can contain multiple payment records for one order.
- `order_reviews` contains review information associated with orders.
- `products` provides product attributes and category information.
- `sellers` provides seller information.
- `product_category_name_translation` maps category names to English.
- `geolocation` provides postal-code geographic information when included.

### Avoiding join multiplication

A direct join from orders to order items and payments can create more than one row for a single order. For example, an order with three item rows and two payment rows can produce six joined rows if both child tables are joined before aggregation. Summing item price or payment value from that multiplied result would overstate the totals.

The project addresses this analytical risk through separate summary views and controlled aggregation. Order items, payments, and reviews should be aggregated to the intended grain before combining them into an order-level reporting dataset.

### Date and null handling

Order lifecycle timestamps represent different milestones: purchase, approval, carrier handover, customer delivery, and estimated delivery. They are not interchangeable. Delivery duration and on-time performance must use the appropriate pair of timestamps.

Null timestamps may indicate that an order did not reach a milestone. For example, an undelivered or cancelled order may not have a customer-delivery timestamp. Such records should be excluded from delivery-duration calculations where appropriate, rather than treated as zero days.

### Currency

The Olist dataset uses Brazilian real (BRL). Merchandise and freight amounts should retain their source currency in the report. Any conversion to another currency requires an explicitly selected exchange rate and conversion date.

## Business Insights and Decision Support

The report is intended to help stakeholders investigate questions such as:

- How is order volume changing over time?
- What proportion of orders reaches delivery successfully?
- Where are late deliveries more concentrated?
- How does merchandise revenue change month by month?
- Which product categories contribute meaningfully to sales?
- How do freight charges compare with merchandise value?
- What does the distribution of review scores indicate about customer experience?
- Do satisfaction patterns vary by state or product category?
- When are customers most active during the day and week?
- Are there signs of repeat purchasing behaviour?

The answers should be interpreted together. For example, a region with strong order demand but weaker delivery performance may require an operational investigation. A product category with strong sales but weaker review scores may warrant a closer look at product quality, seller fulfilment, or delivery experience. These are investigation paths, not proof of causation.

## Recommendations Framework

The analysis can support the following types of business action:

1. **Improve delivery reliability:** Investigate late-delivery concentrations by state, time period, and order lifecycle stage. Review carrier handover and fulfilment bottlenecks before selecting corrective actions.
2. **Monitor commercial performance:** Track merchandise revenue and freight separately so that changes in product sales are not confused with changes in shipping charges.
3. **Review category-level performance:** Compare category sales with customer feedback to identify categories that may need product, seller, or fulfilment review.
4. **Use customer feedback operationally:** Investigate low review scores alongside delivery outcomes and category information to identify recurring customer pain points.
5. **Plan for demand patterns:** Use day- and hour-level purchasing activity to inform operational staffing, campaign timing, and fulfilment capacity.
6. **Encourage repeat purchasing:** Monitor repeat-customer behaviour using the unique customer identifier and evaluate retention initiatives over time.
7. **Maintain KPI consistency:** Use centralised SQL views and documented definitions so that reports and monitoring processes use consistent calculations.
8. **Strengthen monitoring and auditability:** Use rule-based alerts and timestamped KPI snapshots to retain evidence of detected exceptions and historical KPI states.

Recommendations should be prioritised only after validating the underlying metric, reviewing the affected segment, and checking whether the observed pattern is persistent.

## SQL Monitoring, Alerts, and KPI Snapshots

In addition to analytical reporting, the SQL framework includes monitoring and auditability capabilities.

### Business alerts

The alerting layer records rule-based findings in a `business_alerts` table. The table design includes fields for the alert identifier, run identifier, rule code, severity, entity type, entity key, alert message, observed value, threshold value, and detection timestamp.

This structure makes it possible to retain what rule was triggered, which entity was affected, what value was observed, what threshold was applied, and when the finding was detected. Alerts are records created by executing the relevant SQL logic; they should not be assumed to run continuously in the background unless an external scheduler or MySQL event schedule has been configured and enabled.

### KPI snapshots

The snapshot layer stores KPI values at a particular calculation time. The `sp_capture_kpi_snapshot()` procedure is used to capture measures such as total orders, delivered orders, cancelled orders, cancellation rate, merchandise value, freight value, average order value, average delivery days, on-time delivery rate, average review score, and total payment value.

Snapshots provide a historical record of calculated KPI states. They are useful for auditability and comparison, but a snapshot is not automatically a time-series trend unless snapshots are captured at multiple meaningful intervals and compared consistently.

## How to Open and Refresh the Power BI Report

1. Install or open Power BI Desktop.
2. Open `ecommerce_analytics.pbix`.
3. If Power BI reports a missing MySQL connector, install the supported MySQL database connector/driver for Power BI and restart Power BI Desktop.
4. Confirm that the MySQL Server is running and that the `olist_analytics` database is available.
5. If prompted, select the appropriate MySQL server and database connection.
6. Provide the database credentials required by the local MySQL instance.
7. If the report uses a saved server name or database connection that differs from the current machine, update the data source settings in Power BI.
8. Select **Refresh** to retrieve the latest data from the connected database.
9. If the report uses Import mode, refresh loads the latest source data into the Power BI model. If it uses DirectQuery, visuals query the source according to the model's DirectQuery configuration.

A Power BI refresh will only succeed if the database is reachable, the required tables or views exist, credentials are valid, and the connector is installed. Refreshing the PBIX does not itself download the Kaggle CSV files or import them into MySQL; those are separate data preparation steps.

## Reproducing the Project from the Source Data

To reproduce the analysis on another computer:

1. Download the Olist dataset from Kaggle.
2. Extract the CSV files.
3. Install MySQL Server 8.0 or later and MySQL Workbench.
4. Create the `olist_analytics` database.
5. Execute the project's SQL table definitions.
6. Enable local CSV loading in both the MySQL server and client.
7. Import the CSV files into their matching source tables.
8. Run the data-quality and relationship checks.
9. Execute the analytical SQL modules in dependency order to create the views, procedures, and monitoring objects.
10. Install the MySQL connector required by Power BI.
11. Open `ecommerce_analytics.pbix` in Power BI Desktop.
12. Update the database connection if the local server or database details differ.
13. Refresh the report and review the report pages.

## Limitations and Considerations

- The dataset represents historical activity from 2016–2018 and does not represent current marketplace performance.
- The data is anonymised and may not contain all operational context needed to explain a business outcome.
- Review scores are subjective customer feedback and should not be treated as a complete measure of customer experience.
- Delivery analysis depends on the availability and validity of the relevant timestamps.
- Revenue, freight, and payment measures have different grains and must be aggregated carefully.
- The dataset supports observational analysis. It can reveal patterns and associations, but it does not by itself establish causation.
- The accuracy of the report depends on the correctness of the database schema, imported data, SQL transformations, relationships, and Power BI measures.
- The import statements assume the standard Olist CSV headers and a compatible target-table schema. Confirm the table definitions and file line endings before executing them.

## Dataset Licence and Attribution

The dataset is provided by Olist through Kaggle. The Kaggle dataset page lists the licence as **CC BY-NC-SA 4.0**. Review the licence terms before redistributing the dataset or derivative materials, particularly for commercial use.

- Dataset: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- Dataset acknowledgement: Olist and the dataset collaborators.
- MySQL loading reference: [MySQL 8.0 — LOAD DATA Statement](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)

## Conclusion

This project demonstrates an end-to-end analytical workflow using a real-world relational e-commerce dataset. MySQL provides the structured data foundation, validation logic, reusable analytical views, KPI calculations, alert records, and historical snapshots. Power BI presents the resulting information through a report focused on order and delivery performance, sales, customer satisfaction, and purchasing behaviour.

The overall goal is to convert disconnected transactional records into a consistent business-performance view that supports monitoring, investigation, and better-informed operational and commercial decisions.
