# Case Study #2 - Pizza Runner 🍕

For this section, I have used the tables created during the data preparation step. Find them [here](https://github.com/avidiella/portfolio/blob/main/SQL/8%20Week%20SQL%20Challenge%20/Case%20Study%20%232%20-%20Pizza%20Runner/Data%20preparation.md).

## A. Pizza Metrics

**How many pizzas were ordered?**

```sql
SELECT
  COUNT(order_id) AS total_pizzas_ordered
FROM
  clean_customer_orders;
```
**Steps:**
* Count the number of `order_id`.

**Output:**

| total_pizzas_ordered |
| -------------------- |
| 14                   |

A total of 14 pizzas were ordered.

**How many unique customer orders were made?**

```sql
SELECT
  COUNT(DISTINCT(order_id)) AS unique_orders
FROM
  clean_customer_orders;
```

**Steps:**
* Count the disctinct entries of `order_id`.

**Output:**

| unique_orders |
| ------------- |
| 10            |

A total of 10 unique orders were made.

**How many successful orders were delivered by each runner?**

```sql
SELECT
  runner_id,
  COUNT(order_id) AS successful_delivery
FROM
  clean_runner_orders
WHERE
  cancellation IS NULL
GROUP BY
  runner_id;
```

**Steps:**
* Count each entry of `order_id`.
* Select those cases where there is no cancellation.

**Output:**

| runner_id | successful_delivery |
| --------- | ------------------- |
| 1         | 4                   |
| 2         | 3                   |
| 3         | 1                   |

Runner 1 made 4 successful deliveries, runner 2 made 3 successful deliveries and runner 3 made 1 successful delivery.

**How many of each type of pizza was delivered?**

```sql
SELECT
  pizza_name,
  COUNT(pizza_names.pizza_id) AS delivered_pizza
FROM
  clean_customer_orders
  INNER JOIN pizza_names ON clean_customer_orders.pizza_id = pizza_names.pizza_id
  INNER JOIN clean_runner_orders ON clean_customer_orders.order_id = clean_runner_orders.order_id
WHERE
  cancellation IS NULL
GROUP BY
  pizza_names.pizza_name;
```

**Steps:**
* Count each entry of `order_id`.
* Join `clean_customer_orders` with `pizza_names` on `pizza_id`.
* Join `clean_runner_orders` with `clean_customer_orders` on `order_id`
* Select those cases where there is no cancellation.
* Group by pizza name.

**Output:**

| pizza_name | delivered_pizza |
| ---------- | --------------- |
| Vegetarian | 3               |
| Meatlovers | 9               |

The runners have delivered 3 Vegetarians and 9 Meatlovers.

**How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT
  customer_id,
  pizza_name,
  COUNT(pizza_name) AS total_orders
FROM
  clean_customer_orders
  INNER JOIN pizza_names ON clean_customer_orders.pizza_id = pizza_names.pizza_id
GROUP BY
  customer_id, pizza_name
ORDER BY
  customer_id;
```

**Steps:**
* Select variables and count the pizza names.
* Join `clean_customer_orders` with `pizza_names` ON `pizza_id`.
* Group by customer and pizza type.
* Order by customer ID for more visual results.

**Output:**

| customer_id | pizza_name | total_orders |
| ----------- | ---------- | ------------ |
| 101         | Meatlovers | 2            |
| 101         | Vegetarian | 1            |
| 102         | Meatlovers | 2            |
| 102         | Vegetarian | 1            |
| 103         | Meatlovers | 3            |
| 103         | Vegetarian | 1            |
| 104         | Meatlovers | 3            |
| 105         | Vegetarian | 1            |

Customer 101 and 102 ordered 2 Meatlovers and 1 Vegetarian each, customer 103 ordered 3 Meatlovers and 1 Vegetarian, customer 104 ordered 3 Meatlovers, and customer 105 ordered 1 Vegetarian.

**What was the maximum number of pizzas delivered in a single order?**

```sql
SELECT
  COUNT(order_id) AS order_count
FROM
  clean_customer_orders
GROUP BY
  order_id
ORDER BY
  order_count DESC
LIMIT
  1;
```

**Steps:**
* Count entries of `order_id`.
* Group by order ID.
* Order them by count in descending order.
* Show the fist one of them (the maximum).

**Output:**

| order_count |
| ----------- |
| 3           |

The maximum number of pizzas delivered in a single order is 3.

**For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

```sql
SELECT 
  customer_id, 
  SUM(
    CASE WHEN exclusions IS NOT NULL
    OR extras IS NOT NULL THEN 1 ELSE 0 END
  ) AS changes, 
  SUM(
    CASE WHEN exclusions IS NULL 
    AND extras IS NULL THEN 1 ELSE 0 END
  ) AS no_change 
FROM 
  clean_customer_orders 
  INNER JOIN clean_runner_orders ON clean_customer_orders.order_id = clean_runner_orders.order_id 
WHERE 
  cancellation IS NULL
GROUP BY 
  customer_id 
ORDER BY 
  customer_id;
```

**Steps:**
* Create a case to sum the total cases where there are exclusions or extras.
* Create a case to sum the total cases where there are no exclusions and extras.
* Join `clean_customer_orders` with `clean_runner_orders` on `order_id`.

**Output:**

| customer_id | changes | no_change |
| ----------- | ------- | --------- |
| 101         | 0       | 2         |
| 102         | 0       | 3         |
| 103         | 3       | 0         |
| 104         | 2       | 1         |
| 105         | 1       | 0         |

* Customer 101 ordered 2 pizzas with no changes.
* Customer 102 ordered 3 pizzas with no changes.
* Customer 103 ordered 3 pizzas with changes.
* Customer 104 ordered 2 pizzas with changes and 1 with no changes.
* Customer 105 ordered 1 pizza with changes.

**How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT
  COUNT(clean_customer_orders.order_id) AS exclusions_and_extras
FROM
  clean_customer_orders
  INNER JOIN clean_runner_orders ON clean_customer_orders.order_id = clean_runner_orders.order_id
WHERE
  cancellation IS NULL
  AND
  exclusions IS NOT NULL
  AND
  extras IS NOT NULL;
```

**Steps:**
* Count the total of orders.
* Join `clean_customer_orders` with `clean_runner_orders` on `order_id`.
* Select cases where:
  * There were no cancellations.
  * The customer asked for ingredient exclusions.
  * The customer asked for extras on their pizza.

**Output:**

| exclusions_and_extras |
| --------------------- |
| 1                     |

Only 1 pizza had both exclusions and extras.

**What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
  DATE_PART('hour', order_time) AS hour_of_day,
  COUNT(order_id) AS num_orders
FROM
  clean_customer_orders
GROUP BY
  hour_of_day
ORDER BY
  hour_of_day;
```

**Steps:**
* Extract the hours from `order_time`.
* Count the number of orders.
* Group and order by hour.

**Output:**

| hour_of_day | num_orders |
| ----------- | ---------- |
| 11          | 1          |
| 13          | 3          |
| 18          | 3          |
| 19          | 1          |
| 21          | 3          |
| 23          | 3          |

* There was 1 order at 11h.
* There were 3 at 13h.
* There were 3 orders at 18h.
* There was 1 order at 19h.
* There were 3 orders at 21h.
* There were 3 orders at 23h.

**What was the volume of orders for each day of the week?**

```sql
SELECT 
  to_char(order_time, 'Day') AS weekday,
  COUNT(order_id) AS num_orders
FROM
  clean_customer_orders
GROUP BY
  weekday
ORDER BY
  weekday;
```

**Steps:**
* Extract the weekday name from `order_time`.
* Count the number of orders.
* Group and order by weekday.

**Output:**

|  weekday  | num_orders |
| --------- | ---------- |
| Friday    | 1          |
| Saturday  | 5          |
| Thursday  | 3          |
| Wednesday | 5          |

* There was 1 order on Friday.
* There were 5 orders on Saturday.
* There were 3 orders on Thursday.
* There were 5 orders on Wednesday.

Next section: [Runner and Customer Experience](https://github.com/avidiella/portfolio/blob/main/SQL/8%20Week%20SQL%20Challenge%20/Case%20Study%20%232%20-%20Pizza%20Runner/B.%20Runner%20and%20customer%20service.md)
