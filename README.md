## Brazilian E-Commerce Public Dataset by Olist

This is my study project on descriptive analysis of the dataset from Kaggle 
#### Tools: SQL, BigQuery, Tableau

### About DATASET

This is a ecommerce public dataset of orders made at Olist Store, the largest department store in Brazilian marketplaces.
The dataset has information of 100k orders from 2016 to 2018. 
This is real commercial data and has been anonymised. [Link to the dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).

### Objectives of the EDA
#### Revenue Analysis
* Total revenue
* Number of orders over time
* What are the most popular product categories on Olist, Which product category generates highest revenue?
* What is the average order value (AOV) on Olist, and how does this vary by product category or payment method?
* Which product categories have the highest profit margins
* Number of orders by Pricing & sized by revenue generated

### Data schema
Logical model of the dataset
![schema](https://github.com/tata411/SQL_EDA_project/blob/9e60bb4c7faa72d99da2da1ada59beca25a9c494/pics/%D0%91%D0%B5%D0%B7%20%D0%BD%D0%B0%D0%B7%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F.png)

### Data preparation& cleaning

```
--Check duplicates and null values:
SELECT 
order_id,
COUNT(*) as count_id
 FROM `orders` 
GROUP BY order_id
HAVING count_id >1
 LIMIT 1000;

SELECT *
 FROM `orders` 
 WHERE order_id IS NULL OR
    customer_id IS NULL OR
    order_status is NULL;
```

```
--Check status of orders
SELECT DISTINCT(order_status)
FROM`orders` ;
```
![pic2](https://github.com/tata411/SQL_EDA_project/blob/9e60bb4c7faa72d99da2da1ada59beca25a9c494/pics/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-12-31%20%D0%B2%2008.41.21.png)

```
SELECT MIN(order_purchase_timestamp) as min_date,
MAX (order_purchase_timestamp) max_date
FROM orders]; 

```
![pic3](https://github.com/tata411/SQL_EDA_project/blob/9e60bb4c7faa72d99da2da1ada59beca25a9c494/pics/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-12-31%20%D0%B2%2009.00.46.png)

```
SELECT
DISTINCT LENGTH(order_id) as length_order_id
FROM orders; 
--32 symbols in order_id 

SELECT
DISTINCT LENGTH(customer_id) as l
FROM orders 
--32 symbols in customer_id 

```
### Revenue Analysis
#### What is the total revenue generated by Olist, and how has it changed over time?
```
SELECT
  ROUND(SUM(p.payment_value),2) AS total_revenue
FROM
  .payments AS p
JOIN
  .orders AS o
ON
  p.order_id=o.order_id
WHERE
  o.order_delivered_customer_date IS NOT NULL
  AND o.order_status <> 'canceled'
  --total revenue is 15421082.85

SELECT
  FORMAT_TIMESTAMP('%b %Y', o.order_purchase_timestamp) AS month,
  ROUND(SUM(p.payment_value),2) AS total_revenue
FROM
  .payments AS p
JOIN
  .orders AS o
ON
  p.order_id=o.order_id
WHERE
  o.order_delivered_customer_date IS NOT NULL
  AND o.order_status <> 'canceled'
GROUP BY
  1
ORDER BY
  1
```
![pic3](https://github.com/tata411/SQL_EDA_project/blob/b4b3063626957a64da2e7771787597691cf947dc/pics/total_revenue%20by%20month.png)

#### Number of orders over time
```
SELECT
  FORMAT_TIMESTAMP('%b %Y', o.order_purchase_timestamp) AS month,
  COUNT(o.order_id) AS num_of_order
FROM
  .orders AS o
WHERE
  o.order_delivered_customer_date IS NOT NULL
  AND o.order_status <> 'canceled'
GROUP BY
  1
ORDER BY
  1
```
![pic4](https://github.com/tata411/SQL_EDA_project/blob/73722ede75168db58f58b1a5009755e5ba326403/pics/order_per_month.png)

#### What are the most popular product categories on Olist, and how do their sales volumes compare to each other
```
SELECT
  p.product_category_name_english AS product_category,
  COUNT(o.order_id) AS num_of_orders,
  ROUND(SUM(pa.payment_value),2) AS total_revenue
FROM
  .payments AS pa
JOIN
  .orders AS o
ON
  o.order_id=pa.order_id
JOIN
 order_item AS oi
ON
  o.order_id=oi.order_id
JOIN
products AS p
ON
  oi.product_id=p.product_id
GROUP BY
  1
ORDER BY
  2 DESC
```
![pic6](https://github.com/tata411/SQL_EDA_project/blob/1913474617206157d2ffcce318f9a76d74f405dc/pics/top%20categories%20(1).png)
![pic7](https://github.com/tata411/SQL_EDA_project/blob/1913474617206157d2ffcce318f9a76d74f405dc/pics/top%20categories%20(2)%20(1).png)
#### What is the average order value (AOV) on Olist, and how does this vary by product category or payment method?
#### AOV = total revenue / total orders
```
SELECT
  ROUND(SUM(pa.payment_value)/COUNT(o.order_id),2) AS avg_order_value
FROM
  .payments AS pa
JOIN
  .orders AS o
ON
  o.order_id=pa.order_id
WHERE
  o.order_delivered_customer_date IS NOT NULL
  AND o.order_status <> 'canceled'
```
![pic7](https://github.com/tata411/SQL_EDA_project/blob/c92553faa04adaeca47187d131f706ff98658a51/pics/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-12-31%20%D0%B2%2009.29.09.png)

#### AOV by product category
```
SELECT
  p.product_category_name_english AS product_category,
  ROUND(SUM(pa.payment_value)/COUNT(o.order_id),2) AS avg_order_value
FROM
  .payments AS pa
JOIN
  .orders AS o
ON
  o.order_id=pa.order_id
JOIN
  .order_item AS oi
ON
  o.order_id=oi.order_id
JOIN
  .products AS p
ON
  oi.product_id=p.product_id
GROUP BY
  1
ORDER BY
  2 DESC
```
![pic7](https://github.com/tata411/SQL_EDA_project/blob/c92553faa04adaeca47187d131f706ff98658a51/pics/AVG.png)

#### Which product categories have the highest profit margins on Olist?
```
SELECT
  p.product_category_name_english AS product_category,
  ROUND(SUM(pa.payment_value),2) AS total_revenue,
  --общая выручка
  ROUND(SUM(oi.price)+SUM(oi.freight_value),2) AS total_cost,
  --общая себестоимость
  ROUND(SUM(pa.payment_value)-(SUM(oi.price)+SUM(oi.freight_value)),2) AS total_proft,
  -- общая прибыль
  ROUND((SUM(pa.payment_value)-(SUM(oi.price)+SUM(oi.freight_value)))/SUM(pa.payment_value)*100,0) profit_margin_percentage
FROM
  payments AS pa
JOIN
  orders AS o
ON
  o.order_id=pa.order_id
JOIN
 order_item AS oi
ON
  o.order_id=oi.order_id
JOIN
  products AS p
ON
  oi.product_id=p.product_id
GROUP BY
  1
ORDER BY
  5 DESC
```
![pic8](https://github.com/tata411/SQL_EDA_project/blob/c92553faa04adaeca47187d131f706ff98658a51/pics/profit_magins%20(1).png)
