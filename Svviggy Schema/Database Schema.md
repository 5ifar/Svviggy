# 🧬 Svvigy Database Schema
The Svvigy DB Schema comprises of 7 Tables: Users, Restaurants, Delivery_Partner, Food, Menu, Orders & Order_Details.

---

## Table of Contents
- [Table Schema Design](#table-schema-design)
- [Entity Relationship Diagram](#entity-relationship-diagram)

---

## Table Schema Design

### 1. Users Table
This table contains details about the customer.

**Columns:**

1. `user_id` - indicates the User ID (Constraint: Primary Key)
2. `name` - indicates the User Name
3. `email` - indicates the User Email address
4. `password` - indicates the User Password

**Create Users Table Query:**

```sql
CREATE TABLE users (
  user_id INT,
  name VARCHAR(255),
  email VARCHAR(255),
  password VARCHAR
);
```

### 2. Restaurants Table
This table contains information about the restaurent and its cuisines.

**Columns:**

1. `r_id` - indicates the Restaurant ID (Constraint: Primary Key)
2. `r_name` - indicates the Restaurant Name
3. `cuisine` - indicates the Restaurant Cuisine options

**Create Restaurants Table Query:**

```sql
CREATE TABLE restaurants (
  r_id INT,
  r_name VARCHAR(255),
  cuisine VARCHAR(255)
);
```

### 3. Food Table
This table contains information about Names and details of food types.

**Columns:**

1. `f_id` - indicates the Food ID (Constraint: Primary Key)
2. `f_name` - indicates the Food Name
3. `type` - indicates the Food Type

**Create Food Table Query:**

```sql
CREATE TABLE food (
  f_id INT,
  f_name VARCHAR(255),
  type VARCHAR(255)
);
```

### 4. Menu Table
This table contains information of the prices of each item on the restaurant's menu.

**Columns:**

1. `menu_id` - indicates the Menu ID (Constraint: Primary Key)
2. `r_id` - indicates the Restaurant Id
3. `f_id` - indicates the Food Id
4. `price` - inidcates the Menu Price

**Create Menu Table Query:**

```sql
CREATE TABLE menu (
  menu_id INT,
  r_id INT,
  f_id INT,
  price INT
);
```

### 5. Orders Table
This table contains Details about the food, delivery details , order details and the order ratings.

**Columns:**

1. `order_id` - indicates the Order ID (Constraint: Primary Key)
2. `user_id` - indicates the User Id
3. `r_id` - indicates the Restaurant Id
4. `amount` - indicates the Order Amount
5. `date` - indicates the Order Date
6. `partner_id` - indicates the Delivery Partner Id
7. `delivery_time` - indicates the Order Delivery Time
8. `delivery_rating` - indicates the Order Delivery rating
9. `restaurant_rating` - indicates the Restaurant Rating

**Create Orders Table Query:**

```sql
CREATE TABLE orders (
  order_id int,
  user_id int,
  r_id int,
  amount int,
  date date, 
  partner_id int,
  delivery_time int,
  delivery_rating int,
  restaurant_rating int
);
```

### 6. Delivery_Partners Table
This table contains information about Delivery Person.

**Columns:**

1. `partner_id` - indicates the Partner ID (Constraint: Primary Key)
2. `partner_name` - indicates the Partner Name

**Create Delivery_Partners Table Query:**

```sql
CREATE TABLE delivery_partners (
  partner_id INT,
  partner_name VARCHAR(50)
);
```

### 7. Order_Details Table
This table contains details about each order that was created with the food details.

**Columns:**

1. `id` - indicates the Order_Details ID
2. `order_id` - indicates the Order ID
3. `f_id` - indicates the Food ID

**Create Order_Details Table Query:**

```sql
CREATE TABLE order_details (
  id INT,
  order_id INT,
  f_id INT
);
```

---

## Entity Relationship Diagram

