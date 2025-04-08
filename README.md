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

**1️⃣ Tables Used:** The restaurant has recorded 3 key datasets includes: `sales`, `menu', `members`

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

