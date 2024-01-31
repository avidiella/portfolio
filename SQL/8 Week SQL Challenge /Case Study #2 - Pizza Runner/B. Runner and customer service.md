# Case study #2 - Pizza Runner 🍕

For this section, I have used the tables created during the data preparation step. Find them [here](https://github.com/avidiella/portfolio/blob/main/SQL/8%20Week%20SQL%20Challenge%20/Case%20Study%20%232%20-%20Pizza%20Runner/Data%20preparation.md).

## B. Runner and customer experience

**How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

```sql
SELECT
  to_char(registration_date, 'W') As week_period,
  COUNT(runner_id) AS signed_up_runners
FROM
  runners
GROUP BY
  week_period
ORDER BY
  week_period;
```

**Steps:**
* Extract the week number from `registration_date`.
* Count the number of runners.
* Group by week period.
* Order for more visual results.

**Output:**
| week_period | signed_up_runners |
| ----------- | ----------------- |
| 1           | 2                 |
| 2           | 1                 |
| 3           | 1                 |

In the first week, 2 runners signed up for Pizza Runner. In the second and third week, only 1 runner signed up.

**What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
SELECT
  runner_id,
  ROUND(AVG(DATE_PART('MINUTE', (pickup_time - order_time)))::numeric, 2) AS average_arrival_time
FROM
  clean_runner_orders
  INNER JOIN clean_customer_orders ON clean_runner_orders.order_id = clean_customer_orders.order_id
GROUP BY
  runner_id
ORDER BY
  runner_id;
```

**Steps:**
* Compute the time difference between the moment the order was made and when it was picked up.
* Extract the minutes from this difference.
* Compute the average of minutes and round to 2 decimals.
* Join `clean_runner_orders` and `clean_customer_orders` on `order_id`.
* Group by runner, so we get the average arrival time for each one.
* Order by runner for more visual results.

**Output:**

| runner_id | average_arrival_time |
| --------- | -------------------- |
| 1         | 15.33                |
| 2         | 23.40                |
| 3         | 10.00                |

For runner 1, it took roughly 15 minutes to arrive at Pizza Runner HQ. For runner 2, it took 23 minutes, approximately. Runner 3 arrived at the HQ in 10 minutes.

**Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql
WITH preparation_time AS(
  SELECT
    COUNT(pizza_id) AS total_pizzas,
    order_time,
    pickup_time,
    DATE_PART('MINUTE', (pickup_time - order_time)) AS preparing_time
  FROM
    clean_runner_orders
    INNER JOIN clean_customer_orders ON clean_runner_orders.order_id = clean_customer_orders.order_id
  WHERE
    cancellation IS NULL
  GROUP BY
    clean_runner_orders.order_id,
    pickup_time,
    order_time
)
SELECT
  total_pizzas,
  AVG(preparing_time) AS average_preparation_time
FROM
  preparation_time
GROUP BY
  total_pizzas
ORDER BY
  total_pizzas;
```

**Steps:**
* Create a CTE to extract the necessary data to compute the preparation time.
  * Count the total of pizzas that were prepared.
  * Extract `order_time`, `pickup_time` and the minutes of preparation substracting the `order_time` from `pickup_time`.
  * Select those cases where there was no cancellation and the pizzas were delivered.
* Create a query to get the number of total pizzas and the average of preparation time.
* Group by the number of total pizzas, so we get the average per amount of orders.
* Order for more visual results.

**Output:**

| total_pizzas | average_preparation_time |
| ------------ | ------------------------ |
| 1            | 12                       |
| 2            | 18                       |
| 3            | 29                       |

There is a relationship between the number of pizzas and how long the order takes to prepare. When only 1 pizza is ordered, the average preparation time is 12 minutes. When 2 pizzas are ordered, this time increases to 18 minutes. When 3 pizzas are ordered, the average preparation time is 29 minutes.

**What was the average distance travelled for each customer?**

```sql
SELECT
  customer_id,
  ROUND(AVG(distance), 2) AS average_distance
FROM 
  clean_customer_orders
  INNER JOIN clean_runner_orders ON clean_customer_orders.order_id = clean_runner_orders.order_id
GROUP BY
  customer_id
ORDER BY
  customer_id;
```

**Steps:**
* Compute the average distance and round to 2 decimals.
* Join `clean_runner_orders` and `clean_customer_orders` on `order_id`.
* Group by customer so we can check the distance per client.
* Order for more visual results.

**Output:**

| customer_id | average_distance |
| ----------- | ---------------- |
| 101         | 20.00            |
| 102         | 16.73            |
| 103         | 23.40            |
| 104         | 10.00            |
| 105         | 25.00            |

The average distance for each customer was:
* Customer 101: 20 km
* Customer 102: 16.73 km
* Customer 103: 23.40 km
* Customer 104: 10 km
* Customer 105: 25 km

**What was the difference between the longest and shortest delivery times for all orders?**

```sql
SELECT
  MIN(duration) AS shortest_delivery,
  MAX(duration) AS longest_delivery,
  MAX(duration) - MIN(duration) AS difference
FROM
  clean_runner_orders;
```

**Steps:**
* Select the minimum value in duration.
* Select the maximum value in duration.
* Substract the minimum value in duration to the maximum.

**Output:**

| shortest_delivery | longest_delivery | difference |
| ----------------- | ---------------- | ---------- |
| 10                | 40               | 30         |

The shortest delivery was 10 km, and the longest one, 40 km. The difference between the two is 30 km.

**What was the average speed for each runner for each delivery and do you notice any trend for these values?**

```sql
SELECT
  order_id,
  runner_id,
  distance AS distance_km,
  ROUND(duration/60, 2) AS duration_h,
  ROUND(distance/(duration/60), 2) AS speed_kmh
FROM
  clean_runner_orders
WHERE cancellation IS NULL
GROUP BY
  order_id,
  runner_id,
  duration,
  distance
ORDER BY
  order_id,
  runner_id;
```

**Steps:**
* Select variables `order_id`, `runner_id`, and `distance`.
* Compute the duration in hours by dividing by 60 all the values.
* Calculate the speed by dividing the distance in km by the duration in hours.
* Select cases where there was no cancellation.
* Group by different variables so we get the results per order.
* Order for more visual results.

**Output:**

| order_id | runner_id | distance_km | duration_h | speed_kmh |
| -------- | --------- | ----------- | ---------- | --------- |
| 1        | 1         | 20          | 0.53       | 37.50     |
| 2        | 1         | 20          | 0.45       | 44.44     |
| 3        | 1         | 13.4        | 0.33       | 40.20     |
| 4        | 2         | 23.4        | 0.67       | 35.10     |
| 5        | 3         | 10          | 0.25       | 40.00     |
| 7        | 2         | 25          | 0.42       | 60.00     |
| 8        | 2         | 23.4        | 0.25       | 93.60     |
| 10       | 1         | 10          | 0.17       | 60.00     |

The average speed for each order was:
* For order 1: 37.50 km/h
* For order 2: 44.44 km/h
* For order 3: 40.20 km/h
* For order 4: 35.10 km/h
* For order 5: 40.00 km/h
* For order 7: 60.00 km/h
* For order 8: 93.60 km/h
* For order 10: 60 km/h.

**What is the successful delivery percentage for each runner?**

```sql
SELECT
  runner_id,
  100 * COUNT(pickup_time)/COUNT(order_id) AS success_percent
FROM
  clean_runner_orders
GROUP BY
  runner_id
ORDER BY
  runner_id;
```

**Steps:**
* Compute the percentage by dividing the pickup time by the total of orders and multiply by 100.
* Group by runner so we get the percentage for each one.
* Order by runner for more visual results.

**Output:**

| runner_id | success_percent |
| --------- | --------------- |
| 1         | 100             |
| 2         | 75              |
| 3         | 50              |

Runner 1 has a successful delivery percentage of 100%. Runner 2 has 75% of successful deliveries and runner 3, 50%.

Next Section: [Ingredient Optimisation]()
