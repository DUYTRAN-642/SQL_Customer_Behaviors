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

## ⚒️ Main Process

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

**4. Which item was purchased first by the customer after they became a member?**

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

**5. What is the total items and amount spent for each member before they became a member?**

* Filters sales to only those before the member’s join_date.
* Counts the number of product_id entries (i.e. total items bought).
* Sums the prices from the menu table to get total amount spent.
* Groups the results per customer.

```sql
select
  t1.customer_id
  ,count(t1.product_id) as total_items
  ,sum(t3.price) as total_amount
from dannys_diner.sales as t1
left join dannys_diner.members as t2
   on t1.customer_id = t2.customer_id
left join dannys_diner.menu as t3
   on t1.product_id = t3.product_id
where t1.order_date < t2.join_date
group by 1
order by 1
```

| customer_id | total_items | total_amount |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |

---

**6. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

* Joins sales and menu to get both price and product_name.
* For each sale, calculates:
* price * 20  if it's sushi
* price * 10 for all other items
* Sums these to get total points per customer.

```sql
with raw_data as(
select
  *
  ,(case when t1.product_id = 1 then 20* t2.price else 10*t2.price end) as points 
from dannys_diner.sales as t1
left join dannys_diner.menu as t2
on t1.product_id = t2.product_id
)

select 
customer_id
,sum(points) as total_points
from raw_data
group by 1
order by 1
```

| customer_id | total_points |
| ----------- | --- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

---

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

* Checks order date range to apply the first-week bonus.
* Applies sushi multiplier, first-week multiplier, or both.
* Sums all points earned up to January 31.

```sql
SELECT 
    s.customer_id,
    SUM(
        CASE
            -- First week AND sushi: 4x points
            WHEN s.order_date BETWEEN mbr.join_date AND mbr.join_date + INTERVAL '6 days'
                 AND mn.product_name = 'sushi'
            THEN mn.price * 10 * 4

            -- First week, not sushi: 2x points
            WHEN s.order_date BETWEEN mbr.join_date AND mbr.join_date + INTERVAL '6 days'
            THEN mn.price * 10 * 2

            -- After first week, sushi: 2x points
            WHEN mn.product_name = 'sushi'
            THEN mn.price * 10 * 2

            -- Everything else: normal points
            ELSE mn.price * 10
        END
    ) AS total_points
FROM dannys_diner.sales s
JOIN dannys_diner.menu mn ON s.product_id = mn.product_id
JOIN dannys_diner.members mbr ON s.customer_id = mbr.customer_id
  AND s.order_date <= '2021-01-31'
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 1020         |

---

## 🍽️ How It Helps Danny’s Diner

* Improve the loyalty program with targeted bonuses and smarter rewards.
* Personalize promotions based on customer preferences.
* Boost sales by focusing on high-performing items like sushi.
* Measure program success by tracking points vs. revenue.

## ✅ What We Learned

* Customers spend more and try new items after joining the loyalty program.
* The first-week double points and sushi bonus effectively boost engagement.
* We identified top customers, popular items, and spending patterns.


