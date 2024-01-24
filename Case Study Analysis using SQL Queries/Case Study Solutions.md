# 🥡 Svviggy Case Study
<img src="https://logolook.net/wp-content/uploads/2023/04/Swiggy-Emblem.png" width="20%" height="20%">

---

## Table of Contents
- [Introduction](#introduction)
- [Business Task](#business-task)
- [Case Study Questions and Solutions](#case-study-questions-and-solutions)

---

## Introduction

--To be added--

---

## Business Task

--To be added--

---

## Case Study Questions and Solutions
*The following queries have been executed using PostgreSQL v15 on pgAdmin 4 GUI.*

### 1. How many customers have not placed any orders?

**Steps:**
- Use `NOT IN` operator to select `user_id` values that are not present among `user_id` values in the `orders` table.

**Query:**
```sql
SELECT
  user_id AS "User Id",
  name AS "Customer Name"
FROM users
WHERE user_id NOT IN (SELECT DISTINCT user_id FROM orders);
```

**Answer:**
|User Id|Customer Name|
|-|-|
|6|Anupama|
|7|Rishabh|

**Insight:**
- Anupama & Rishabh have not placed any orders.

---

### 2A. What is the average price of each food type?

**Steps:**
- Implement INNER JOIN to merge `food` and `menu` tables based on `f_id` fields.
- Group the result by `type` field to calculate the average price based on food type based groups rounded to 2 decimal places.

**Query:**
```sql
SELECT
  type AS "Food Type",
  ROUND(AVG(price), 2) AS "Average Price"
FROM food AS f
INNER JOIN menu AS m ON f.f_id = m.f_id
GROUP BY type;
```

**Alternate Query: Implemented using CTEs**
```sql
WITH food_details AS(
  SELECT * FROM food f
  INNER JOIN menu m
  ON f.f_id = m.f_id
)
SELECT
  type,
  ROUND(AVG(price),2) AS "Average Price",
  MEDIAN(price) AS "Median Price",
  STATS_MODE(price) AS "Mode Price"
FROM food_details
GROUP BY type
ORDER BY "Average Price" DESC;
```

**Answer:**
|Food Type|Average Price|
|-|-|
|Non-veg|326.67|
|Veg|181.25|

**Insight:**
- Average price of Non-Veg and Veg food is $ 326.67 and $ 181.25 respectively.

---

