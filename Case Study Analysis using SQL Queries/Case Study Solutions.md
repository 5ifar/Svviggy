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
- Group the result by `type` field to calculate the average price of food based on food type groups rounded to 2 decimal places.

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
  INNER JOIN menu m ON f.f_id = m.f_id
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

### 2b. What is the average price per food across restaurants?

**Steps:**
- Implement INNER JOIN to merge `food` and `menu` tables based on `f_id` fields.
- Group the result by `f_name` field to calculate the average price of food based on food groups across restaurants rounded to 2 decimal places.
- Order the result by `"Average Price"` in ascending order.

**Query:**
```sql
SELECT
  f_name AS "Food",
  ROUND(AVG(m.price), 2) AS "Average Price"
FROM food AS f
INNER JOIN menu AS m ON f.f_id = m.f_id
GROUP BY f_name
ORDER BY "Average Price";
```

**Answer:**
|Food|Average Price|
|-|-|
|Choco Lava cake|98.33|
|Rava Idli|120.00|
|Roti meal|140.00|
|Masala Dosa|180.00|
|Veg Manchurian|180.00|
|Rice Meal|213.33|
|Schezwan Noodles|220.00|
|Chicken Wings|230.00|
|Chicken Popcorn|300.00|
|Veg Pizza|400.00|
|Non-veg Pizza|450.00|

**Insight:**
- Across restaurants, Choco Lava cake has the lowest average price at $ 98.33, while Non-veg Pizza has the highest average price at $ 450.

---

### 2c. What is the average price of food for each restaurant?

**Steps:**
- Implement INNER JOIN to merge `restaurants` and `menu` tables based on `r_id` fields.
- Group the result by `r_name` field to calculate the average price of food based on restaurant groups rounded to 2 decimal places.
- Order the result by `"Average Price"` in ascending order.

**Query:**
```sql
SELECT
  r_name AS "Restaurant Name",
  ROUND(AVG(price), 2) AS "Average Price"
FROM menu AS m
INNER JOIN restaurants AS r ON m.r_id = r.r_id
GROUP BY r_name
ORDER BY "Average Price";
```

**Alternate Query: Implemented using CTEs**
```sql
WITH restaurant_details AS (
  SELECT * FROM restaurants AS r
  INNER JOIN menu AS m ON r.r_id = m.r_id
)
SELECT
  r_name AS "Restaurant Name",
  '$ '||ROUND(AVG(price),2) AS "Average Price"
FROM restaurant_details
GROUP BY r_name
ORDER BY r_name;
```

**Answer:**
|Restaurant Name|Average Price|
|-|-|
|box8|126.67|
|Dosa Plaza|176.67|
|kfc|215.00|
|China Town|216.67|
|dominos|316.67|

**Insight:**
- Dominos is the most expensive restaurant with an average price of $ 316.67, while Box8 is the least expensive restaurant with an average price of $ 126.67

---
