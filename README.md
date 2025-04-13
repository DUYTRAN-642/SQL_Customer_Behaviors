# SQL_Customer_Behaviors
![image](https://github.com/user-attachments/assets/58362dbd-e3f7-45d3-af42-5f20e1467c34)

📊 Project Title: Danny's Diner Restaurant

Author: Duy Tran

Date: 2025-04 -06

Tools Used: Postgre SQL

## 📌 Background & Overview

### 📖 Objectives

   * A cute little restaurant 'Danny's Diner' that sells 3 favourite foods: sushi, curry and ramen. They has captured the operation data and need help to run business.
   * This project help them to identify its customers' visiting patterns, how much money they’ve spent and also which menu items are their favourite
   * The project's output also tends to help the restaurant owner to deliver a better and more personalised experience for his loyal customers and decide whether he should expand the existing customer loyalty program

### 👤 Who is this project for?

   * The owner of the restaurant, his name is Danny, the project would give him the data insights and lead to effective business decisions
   * The shareholsers and investors who join this venture, the project help them evaluate the potentials of their investment to this business

## 📂 Dataset Description & Data Structure

### 📌 Data Source:

A case study obtained from 8 Week SQL Challenge, link: https://8weeksqlchallenge.com/case-study-1/

### 📊 Data Structure & Relationships

**1️⃣ Tables Used:** The restaurant has recorded 3 key datasets includes: `sales`, `menu' , `members`

![image](https://github.com/user-attachments/assets/e133b7ab-2334-4cfb-acbb-fa5f42d0677f)

**2️⃣ Data Snapshot**

<details>
<summary>👉🏻 Fact Table: sales</summary>
<br>

| customer_id | order_date | product_id |
|-------------|------------|------------|
| A           | 2021-01-01 | 1          |
| A           | 2021-01-01 | 2          |
| A           | 2021-01-07 | 2          |
| A           | 2021-01-10 | 3          |
| A           | 2021-01-11 | 3          |
</details>

<details>
<summary>👉🏻 Dim Table 1: menu</summary>
<br>

| customer_id | order_date | product_id |
|-------------|------------|------------|
| A           | 2021-01-01 | 1          |
| A           | 2021-01-01 | 2          |
| A           | 2021-01-07 | 2          |
| A           | 2021-01-10 | 3          |
| A           | 2021-01-11 | 3          |
</details>

<details>
<summary>👉🏻 Dim Table 2: members</summary>
<br>

| product_id | product_name | price |
|------------|--------------|-------|
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |
</details>

⚒️ Main Process

**1. What is the total amount each customer spent at the restaurant?**

The purpose of this query is to extract valuable financial data related to customer purchases and product prices, which can be leveraged for various business and marketing decisions

```sql
SELECT
 t1.customer_id
 ,sum(t2.price) as total_amount
FROM dannys_diner.sales as t1
left join dannys_diner.menu as t2
on t1.product_id = t2.product_id
group by 1
order by 1;
```

| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

**2. How many days has each customer visited the restaurant?**

The result can be queried from table of `sales`. Because one customer can have more than 1 order per day, the  syntax ` count distinct` should be deployed to find number of days each csutomer visited the restaurant.

```sql  
    SELECT
      	customer_id,
        count(distinct order_date) as days_count
    FROM dannys_diner.sales
    group by 1
    ;
```

| customer_id | days_count |
| ----------- | ---------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

**3. What was the first item from the menu purchased by each customer?**

This query had `RANKING` syntax to identify the earliest item purchased, but there were customers bought more than 1 item at first date, so the syntax of `STRING_AGG` in Postgre SQL was used to set the aggregate the outputs.

```sql
with raw_table as(
SELECT
     *
    ,rank() over(partition by t1.customer_id order by t1.order_date) as ranking
from dannys_diner.sales as t1
left join dannys_diner.menu as t2
on t1.product_id = t2.product_id
  )
  
select 
     customer_id
    ,string_agg(product_name,', ') as first_items
from raw_table
where ranking = 1
group by 1
order by 1
```

| customer_id | first_items  |
| ----------- | ------------ |
| A           | curry, sushi |
| B           | curry        |
| C           | ramen, ramen |

---

**Which item was purchased first by the customer after they became a member?**

* Filters sales to after the customer’s join_date.
* Finds the first order_date after membership.
* Gets all products bought on that date, and joins with menu to get product_name.
* Uses STRING_AGG to handle multiple items bought on the same date.

```sql
WITH sales_after_join AS (
    SELECT 
        s.customer_id,
        s.product_id,
        s.order_date
    FROM dannys_diner.sales s
    JOIN dannys_diner.members m 
        ON s.customer_id = m.customer_id
    WHERE s.order_date > m.join_date
),
first_order_per_customer AS (
    SELECT 
        customer_id,
        MIN(order_date) AS first_order_date
    FROM sales_after_join
    GROUP BY customer_id
)
SELECT 
    saj.customer_id,
    STRING_AGG(m.product_name, ', ') AS first_items_after_join,
    saj.order_date
FROM sales_after_join saj
JOIN first_order_per_customer fop 
    ON saj.customer_id = fop.customer_id 
   AND saj.order_date = fop.first_order_date
JOIN dannys_diner.menu m 
    ON saj.product_id = m.product_id
GROUP BY saj.customer_id, saj.order_date
ORDER BY saj.customer_id;
```

| customer_id | first_items_after_join | order_date |
| ----------- | ---------------------- | ---------- |
| A           | ramen                  | 2021-01-10 |
| B           | sushi                  | 2021-01-11 |

---
