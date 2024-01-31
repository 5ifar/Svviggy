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
- Use `NOT IN` operator to select `user_id` values in the `users` table that are not present among `user_id` values in the `orders` table.

**Query:**
```sql
SELECT
  user_id AS "User Id",
  name AS "Customer Name"
FROM users
WHERE user_id NOT IN (SELECT DISTINCT user_id FROM orders);
```

**Output:**
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

**Output:**
|Food Type|Average Price|
|-|-|
|Non-veg|326.67|
|Veg|181.25|

**Insight:**
- Average price of Non-Veg and Veg food is $ 326.67 and $ 181.25 respectively.

---

### 2B. What is the average price per food across restaurants?

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

**Output:**
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

### 2C. What is the average price of food for each restaurant?

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

**Output:**
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

### 3A. Find the top restaurant in terms of the number of orders for the month of June.

**Steps:**
- Implement LEFT JOIN to merge `restaurants` and `orders` tables based on `r_id` fields to ensure all `r_id` fields are considered from `orders` table even if they are missing in the `restaurants` lookup table.
- Filter the results for the month of June using either the EXTRACT or TO_CHAR functions.
- Group the result by `r_name` field to calculate the total count of `order_id` fields based on restaurant groups.
- Order the result by `"Order Count"` in descending order and LIMIT results to the top 1.

**Query:**
```sql
SELECT
  r_name AS “Restaurant Name”,
  COUNT(order_id) AS “Order Count”
FROM orders AS o
LEFT JOIN restaurants AS r ON o.r_id = r.r_id
WHERE EXTRACT(MONTH FROM date) = 6
GROUP BY r_name
ORDER BY “Order Count” DESC
LIMIT 1;
```

**Alternate Query:**

Instead of EXTRACT Function, `TRIM(TO_CHAR(date, 'Month')) = 'June'` can also be used. The reason we use TRIM function is because TO_CHAR() takes the trailing spaces. So TRIM is used to remove any unecessary space.

**Output:**
|Restaurant Name|Order Count|
|-|-|
|kfc|3|

**Insight:**
- Highest number of orders were made in the month of June from KFC.

---

### 3B. Find the top restaurant in terms of the number of orders for all months.

**Steps:**
- to be added

**Query: Implemented using CTEs**
```sql
WITH res AS (
  SELECT
    TO_CHAR(o.date, 'Month') AS order_month, 
    r.r_name, COUNT(*) AS order_count
  FROM orders o
  INNER JOIN restaurants r ON o.r_id = r.r_id
  GROUP BY EXTRACT(month FROM o.date), TO_CHAR(o.date, 'Month'), r.r_name
)
SELECT
  order_month AS "Order Month",
  r_name AS "Restaurant Name",
  order_count AS "Total Orders"
FROM (
  SELECT
    order_month, r_name, order_count,
    RANK() OVER (PARTITION BY order_month ORDER BY order_count DESC) AS res_rank
  FROM res
) AS subquery
WHERE res_rank = 1
ORDER BY EXTRACT(month FROM TO_DATE(order_month, 'Month'));
```

**Alternate Query: Less Optimized but Simpler**
```sql
SELECT
  r_name AS Restaurant_Name,
  EXTRACT(MONTH FROM date) AS Order_Month,
  COUNT(order_id) AS Order_Count
FROM orders AS o
LEFT JOIN restaurants AS r ON o.r_id = r.r_id 
GROUP BY EXTRACT(MONTH FROM date), r_name
ORDER BY Order_Month ASC, Order_Count DESC;
```

**Output:**
|Order Month|Restaurant Name|Total Orders|
|-|-|-|
|May|Dosa Plaza|3|
|June|kfc|3|
|July|kfc|3|

**Insight:**
- Top restaurant in terms of the number of orders was Dosa Plaza for May and KFC for June and July.

---

### 4. Find restaurants with monthly revenue greater than 500 in the month of June.

**Steps:**
- Implement LEFT JOIN to merge `restaurants` and `orders` tables based on `r_id` fields to ensure all `r_id` fields are considered from `orders` table even if they are missing in the `restaurants` lookup table.
- Filter the results for the month of June using either the EXTRACT or TO_CHAR functions.
- Group the result by `r_name` field to calculate the sum of `amount` fields based on restaurant groups.
- Filter the aggregated results based on if the sum of `amount` fields for a restaurant is greater than 500.

**Query:**
```sql
SELECT
  r_name AS "Restaurant Name",
  SUM(amount) AS "Monthly Revenue"
FROM orders AS o
LEFT JOIN restaurants AS r ON o.r_id = r.r_id
WHERE EXTRACT(MONTH FROM date) = 6
GROUP BY r_name
HAVING SUM(amount) > 500
```

**Output:**
|Restaurant Name|Monthly Revenue|
|-|-|
|dominos|950|
|kfc|990|

**Insight:**
- Dominos & KFC had monthly revenues greater than 500 in June.

---

### 5. Show all orders with order details for a particular customer (user_id is 4) in a date range (1st June 2022 to 1st August 2022).

**Steps:**
- Implement INNER JOIN to merge `orders` table with `users`, `restaurants`, `order_details` and `food` tables based on `user_id`, `r_id`, `order_id` and `f_id` fields respectively.
- Filter the results for `user_id` 4 and `date` range between '01-06-22' and '01-08-22'.

**Query:**
```sql
SELECT
  o.order_id AS "Order Id",
  r_name AS "Restaurant Name",
  f_name AS "Food"
FROM orders AS o
INNER JOIN users AS u ON o.user_id = u.user_id
INNER JOIN restaurants AS r ON o.r_id = r.r_id
INNER JOIN order_details AS od ON o.order_id = od.order_id
INNER JOIN food AS f ON od.f_id = f.f_id
WHERE o.user_id = 4 AND date between to_date('01-06-22','DD-MM-YY') and to_date('01-08-22','DD-MM-YY');
```

**Note:**
- `to_date` operator allows us to enter values in our preferred date format, if not used we need to mandatorily enter in `YYYY-MM-DD` format as followed in the `orders` table.

**Output:**
|Order Id|Restaurant Name|Food|
|-|-|-|
|1018|Dosa Plaza|Schezwan Noodles|
|1018|Dosa Plaza|Veg Manchurian|
|1019|China Town|Schezwan Noodles|
|1019|China Town|Veg Manchurian|
|1020|China Town|Schezwan Noodles|
|1020|China Town|Veg Manchurian|

**Insight:**
- Customer with user_id 4 ordered Schezwan Noodles and Veg Manchurian from Dosa Plaza and China Town between 1st June 2022 and 1st Aug 2022.

---
