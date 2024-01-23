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
- Use NOT IN operator to select user_id values that are not present among user_id values in the orders table.

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

