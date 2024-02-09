# <img src="https://logolook.net/wp-content/uploads/2023/04/Swiggy-Emblem.png" width="7%" height="7%"> Svviggy Case Study

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

###### *NOTE: While I understand the importance of SQL query optimization, the column name aliases used in the queries below are long and certainly not optimized. This has been done in favour of readability of the outcome for an unassuming business end user and should not be treated as ignorance.*

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

### 6. Which restaurant has the highest number of repeat customers?

**Steps:**
- Create a subquery to group `orders` table data by `r_id` and `user_id` fields and filter results for groups with more than 1 count of `order_id` field based on restaurant and user pair groups. Alias the subquery as `rcr`.
- Implement INNER JOIN to merge `restaurants` table with the `rcr` subquery based on `r_id` field.
- Group the results by `r_id` and `r_name` fields and filter the top 1 row based on descending sort by `COUNT(user_id)`.
- Alias the `COUNT(user_id)` column of the filtered row as “Total Repeat Customers” for the restaurant.

**Query:**
```sql
SELECT
  rcr.r_id,
  r_name AS "Restaurant Name",
  COUNT (user_id) AS "Total Repeat Customers"
FROM
(SELECT
  r_id, user_id,
  COUNT(order_id) AS visits
FROM orders
GROUP BY r_id, user_id
HAVING COUNT(order_id) > 1) AS rcr
INNER JOIN restaurants AS r ON rcr.r_id = r.r_id
GROUP BY rcr.r_id, r_name
ORDER BY COUNT (user_id) DESC
LIMIT 1;
-- rcr alias denotes Repeat Customer Restaurants
```

**Alternate Query: Implemented using CTEs**
```sql
WITH repeated_cust AS (
SELECT
  r.r_name, o.user_id,
  count(order_id) as order_count
FROM restaurants AS r
INNER JOIN orders AS o ON r.r_id = o.r_id
GROUP BY r.r_name, o.user_id
HAVING COUNT(order_id) > 1
),
loyal_cust AS (
SELECT
  r_name AS "Restaurant Name",
  COUNT(user_id) AS "Total Repeat Customers"
FROM repeated_cust
GROUP BY r_name
ORDER BY COUNT(user_id) DESC
LIMIT 1
)
SELECT * FROM loyal_cust;
```

**Output:**
|r_id|Restaurant Name|Total Repeat Customers|
|-|-|-|
|2|kfc|2|

**Insight:**
- KFC is the restaurant with the highest count of 2 repeat customers.

---

### 7. Find the most loyal customer for all restaurants.

**Steps:**
- Create a subquery to group `orders` table data by `r_id` and `user_id` fields and filter results for groups with more than 1 count of `order_id` field based on restaurant and user pair groups. Alias the subquery as `rcr`.
- Implement INNER JOIN to merge `restaurants` table with the `rcr` subquery based on `r_id` field.
- Implement INNER JOIN to merge `users` table with the `rcr` subquery based on `user_id` field.
- Group the results by `r_id`, `r_name` and `name` fields and order the result by `r_id` field.

**Query:**
```sql
SELECT
  rcr.r_id,
  r_name AS "Restaurant Name",
  name AS "Loyal Customer"
FROM 
(SELECT
  r_id, user_id,
  COUNT(order_id) AS visits FROM orders
GROUP BY r_id, user_id
HAVING COUNT(order_id) > 1) AS rcr
INNER JOIN restaurants AS r ON rcr.r_id = r.r_id
INNER JOIN users AS u ON u.user_id = rcr.user_id
GROUP BY rcr.r_id, r_name, name
ORDER BY rcr.r_id;
-- rcr alias denotes Repeat Customer Restaurants
```

**Output:**
|r_id|Restaurant Name|Loyal Customers|
|-|-|-|
|1|dominos|Neha|
|2|kfc|Neha|
|2|kfc|Vartika|
|3|box8|Nitish|
|4|Dosa Plaza|Ankit|
|5|China Town|Ankit|

**Insight:**
- KFC is the only restaurant with 2 Most Loyal Customers.

---

### 8A. Calculate Month-over-Month revenue growth of Svviggy.

**Steps:**
- Create a subquery to group `orders` table data by `Month` and calculate `SUM(amount)` as Current Month Revenue of Svviggy. Alias the subquery as `Monthly Revenue Subquery`.
- Create a subquery with `Monthly Revenue Subquery` nested inside it to calculate the Previous Month Revenue using the `LAG` operator to select the previous 1 month revenue when ordered by `Month`. Alias the subquery as `Month-over-Month Revenue Subquery`.
- Based on Current Month Revenue and Previous Month Revenue values calculate the growth percentage compared to Previous Month Revenue. Round the result to 2 decimals. Alias as `Growth%`.

**Query:**
```sql
SELECT
  Month, "Current Revenue", "Previous Revenue",
  ROUND(("Current Revenue" - "Previous Revenue") / "Previous Revenue"::numeric * 100, 2) AS "Growth%"
FROM
  (SELECT
    Month, "Current Revenue",
    LAG("Current Revenue", 1) OVER (ORDER BY Month ASC) AS "Previous Revenue"
  FROM
    (SELECT
      EXTRACT(MONTH FROM date) AS Month,
      SUM(amount) AS "Current Revenue"
    FROM orders
    GROUP BY Month
    ORDER BY Month) AS "Monthly Revenue Subquery"
  ) AS "Month-over-Month Revenue Subquery";
```

**Alternate Query: Implemented using CTEs**
```sql
WITH sales AS (
  SELECT
  EXTRACT(MONTH FROM date) AS month,
  SUM(amount) AS revenue
  FROM orders
  GROUP BY EXTRACT(MONTH FROM date)
  ORDER BY EXTRACT(MONTH FROM date) ASC
)
SELECT month, revenue, prev_revenue, (revenue - prev_revenue) / prev_revenue::numeric * 100 AS growth
FROM (
  SELECT month, revenue, LAG(revenue, 1) OVER (ORDER BY month ASC) AS prev_revenue
  FROM sales
) AS monthly_revenue;
```

**Output:**
|Month|Current Revenue|Previous Revenue|Growth%|
|-|-|-|-|
|5|2425|[null]|[null]|		
|6|3220|2425|32.78|
|7|4845|3220|50.47|

**Insight:**
- Svviggy's Month-Over-Month Revenue Growth Rate seems to be increasing steadily with 32.78% in June and 50.47% in July.

---

### 8B. Calculate Month-over-Month revenue growth of a specific restaurant.

**Steps:**
- Follow the same steps as above Q.8, additionally add the filter for `r_id` field value of the restaurant required in the WHERE clause of `Monthly Revenue Subquery`.

---

### 9. Month-over-Month revenue growth of each restaurant collated.

**Steps:**
- Beyond my current competency, to be attempted soon.

---

### 10. Find the top 3 most ordered foods.

**Steps:**
- Implement INNER JOIN to merge `order_details` table with the `food` table based on `f_id` field.
- Group the results by `f_name` field and calculte the total count of `order_id` fields based on food group and order them by the count in descending order.
- Use the FETCH operator to get only the top 3 rows.

**Query:**
```sql
SELECT
  f_name AS "Food",
  COUNT(order_id) AS "Order Count"
FROM order_details AS od
INNER JOIN food AS f ON f.f_id = od.f_id
GROUP BY f_name
ORDER BY "Order Count" DESC
FETCH FIRST 3 ROWS ONLY;
```

**Alternate Query: Using DENSE_RANK**
```sql
SELECT
  f_name,
  order_count
FROM (
  SELECT
    f_name, COUNT(order_id) AS order_count, 
    DENSE_RANK () OVER(ORDER BY COUNT(order_id) DESC) AS rank
  FROM order_details AS od
  INNER JOIN food AS f ON f.f_id = od.f_id
  GROUP BY f_name) AS "Rank Subquery"
WHERE rank <= 3;
```

**Output:**
|Food|Order Count|
|-|-|
|Choco Lava cake|13|
|Chicken Wings|8|
|Non-veg Pizza|5|

**Insight:**
- Choco Lava Cake, Chicken Wings & Non-veg Pizza are the Top 3 most ordered foods.

---

### 11. Find the favourite food for each customer and its order frequency. 

**Steps:**
- to be added

**Query:**
```sql
WITH food_info AS (
 SELECT
  o.user_id, od.f_id,
  count(*) as frequency
 FROM orders AS o
 INNER JOIN order_details AS od ON od.order_id = o.order_id
 GROUP BY o.user_id, od.f_id 
 ORDER BY o.user_id ASC
)
SELECT
  t1.user_id, u.name,
  f.f_name AS "Food",
  frequency AS "Order Frequency"
FROM food_info AS t1
INNER JOIN users AS u ON u.user_id = t1.user_id
INNER JOIN food AS f ON f.f_id = t1.f_id
WHERE t1.frequency = 
  (SELECT MAX(frequency) 
   FROM food_info AS t2
   WHERE t2.user_id = t1.user_id)
ORDER BY frequency DESC;
```

**Output:**
|user_id|name|Food|Order Frequency|
|-|-|-|-|
|1|Nitish|Choco Lava cake|5|
|5|Neha|Choco Lava cake|5|
|2|Khushboo|Choco Lava cake|3|
|3|Vartika|Chicken Wings|3|
|4|Ankit|Schezwan Noodles|3|
|4|Ankit|Veg Manchurian|3|

**Insight:**
- Choco Lava cake is the favourite food for 3 customers with the highest order frequency of 5 for Nitish and Neha.

---

### 12. What is the overall revenue generated by the Svviggy during a specific time period?

**Steps:**
- Filter the `orders` table between 1st May and 1st June 2022.
- Calculate the sum of the `amount` column values as `Total Revenue`.

**Query:**
```sql
SELECT
  SUM(amount) as “Total Revenue”
FROM orders
WHERE date BETWEEN to_date('01-05-22','DD-MM-YY') AND to_date('01-06-22','DD-MM-YY');
```

**Note:**
- `to_date` operator allows us to enter values in our preferred date format, if not used we need to mandatorily enter in `YYYY-MM-DD` format as followed in the `orders` table.

**Output:**
|Total Revenue|
|-|
|2425|

**Insight:**
- Svviggy generated a total revenue of 2425 from 1st May to 1st June 2022.

---

### 13. What is the average order value per user?

**Steps:**
- Implement INNER JOIN to merge `orders` table with the `users` table based on `user_id` field.
- Group the results by `name` field and calculte the average of `amount` field based on user group. Round the `AVG(amount)` value to 2 decimal places. Order the outcome by `AVG(amount)` in descending order.

**Query:**
```sql
SELECT
  name,
  ROUND(AVG(amount), 2) as "Avg Order Value"
FROM orders AS o
INNER JOIN users AS u ON o.user_id = u.user_id
GROUP BY name
ORDER BY "Avg Order Value" DESC;
```

**Output:**
|Name|Avg Order Value|
|-|-|
|Neha|607.00|
|Khushboo|534.00|
|Ankit|360.00|
|Nitish|333.00|
|Vartika|264.00|

**Insight:**
- Neha has the highest average order value.

---

### 14. What is the average delivery time for each restaurant, and how does it affect customer satisfaction?

**Steps:**
- Implement INNER JOIN to merge `orders` table with the `restaurants` table based on `r_id` field.
- Group the results by `r_name` field and calculte the average of `delivery_time` and `delivery_rating` fields based on restaurant group. Round the values to 2 decimal places. Order the outcome by `"Average Delivery Rating"` in descending order.

**Query:**
```sql
SELECT
  r_name AS "Restaurant Name",
  ROUND(AVG(delivery_time), 2) as "Average Delivery Time",
  ROUND(AVG(delivery_rating), 2) as "Average Delivery Rating"
FROM orders AS o
INNER JOIN restaurants AS r ON o.r_id = r.r_id
GROUP BY r_name
ORDER BY "Average Delivery Rating" DESC;
```

**Output:**
|Restaurant Name|Average Delivery Time|Average Delivery Rating|
|-|-|-|
|dominos|24.40|4.00|
|kfc|42.50|3.25|
|box8|40.50|3.00|
|Dosa Plaza|39.00|2.80|
|China Town|54.33|2.67|

**Insight:**
- While the lowest average delivery time restaurant gets the highest rating and the highest average delivery time restaurant gets the lowest rating, the other restaurants (KFC & Dosa Plaza) dont follow the same correlation. While they can be considered as weakly correlated they do not imply causation.

---

### 15. What is the average rating for each restaurant and delivery partner?

**Steps:**
1. - Implement INNER JOIN to merge `orders` table with the `restaurants` table based on `r_id` field.
   - Group the results by `r_name` field and calculte the average of `restaurant_rating` field based on restaurant group. Round the value to 2 decimal places. Order the outcome by `"Average Restaurant Rating"` in descending order.
2. - Implement INNER JOIN to merge `orders` table with the `delivery_partners` table based on `partner_id` field.
   - Group the results by `partner_name` field and calculte the average of `delivery_rating` field based on partner group. Round the value to 2 decimal places. Order the outcome by `"Average Delivery Rating"` in descending order.

**Queries:**
```sql
SELECT
  r_name AS "Restaurant Name",
  ROUND(AVG(restaurant_rating), 2) AS "Average Restaurant Rating"
FROM orders AS o
INNER JOIN restaurants AS r ON o.r_id = r.r_id
GROUP BY r_name
ORDER BY "Average Restaurant Rating" DESC;

SELECT
  partner_name AS "Delivery Partner",
  ROUND(AVG(delivery_rating), 2) AS "Average Delivery Rating"
FROM orders AS o
INNER JOIN delivery_partners AS dp ON o.partner_id = dp.partner_id
GROUP BY partner_name
ORDER BY "Average Delivery Rating" DESC;
```

**Outputs:**
|Restaurant Name|Average Restaurant Rating|
|-|-|
|box8|4.67|
|Dosa Plaza|3.67|
|China Town|3.67|
|kfc|2.20|
|dominos|1.67|

|Delivery Partner|Average Delivery Rating|
|-|-|
|Lokesh|4.00|
|Gyandeep|3.50|
|Kartik|3.00|
|Amit|3.00|
|Suresh|2.86|

**Insights:**
- Box8 has the highest average restaurant rating of 4.67 and Dominos has the lowest average restaurant rating of 1.67.
- Lokesh has the highest average delivery rating of 4.00 and Suresh has the lowest average delivery rating of 2.86.

---

### 16. How do the ratings for restaurants and delivery partners correlate with customer retention?

**Steps:**
1. For `group_1` CTE:
   - Implement INNER JOIN to merge `orders` table with the `restaurants` table based on `r_id` field.
   - Group the results by `r_name` field and calculte the average of `restaurant_rating` field and distinct count of `user_id` field based on restaurant group.
2. For `group_2` CTE:
   - Implement INNER JOIN to merge `orders` table with the `delivery_partners` table based on `partner_id` field.
   - Group the results by `partner_name` field and calculte the average of `delivery_rating` field and distinct count of `user_id` field based on partner group.
3. Joining both CTEs:
   -  Implement INNER JOIN to merge `group_1` CTE with the `group_2` CTE based on `unique_users` field.
   -  Implement the `CORR` operator to calculate the coefficient of correlation between `avg_restaurant_rating` and `avg_delivery_rating` fields.

**Queries:**
```sql
WITH group_1 AS (
  SELECT
    r.r_name,
    COUNT(DISTINCT o.user_id) AS unique_users,
    AVG(o.restaurant_rating) AS avg_restaurant_rating
  FROM orders AS o
  INNER JOIN restaurants AS r ON o.r_id = r.r_id
  GROUP BY r.r_name
),
group_2 AS (
  SELECT
    dp.partner_name,
    COUNT(DISTINCT o.user_id) AS unique_users,
    AVG(o.delivery_rating) AS avg_delivery_rating
  FROM orders AS o
  INNER JOIN delivery_partners AS dp ON o.partner_id = dp.partner_id
  GROUP BY dp.partner_name
)
SELECT
  CORR(AVG_RESTAURANT_RATING, AVG_DELIVERY_RATING) AS "Coefficient of Correlation"
FROM group_1 AS g1
INNER JOIN group_2 AS g2 ON g1.UNIQUE_USERS = g2.UNIQUE_USERS;
```

**Output:**
|Coefficient of Correlation|
|-|
|0.7074762526640902|

**Insight:**
- The average of restaurant rating and average of delivery rating has a coefficient of Correlation of approximately 0.7, indicating a strong positive correlation.

---

