# Case Study #1 - Danny's Diner 🍜
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="Image" width="500" height="520">

## Table of contents
1. [Problem statement](#problem-statement)
2. [Case Study Questions](#case-study-questions)
3. [Bonus Questions](#bonus-questions)

Danny's Diner is a case study from the **8 Week SQL Challenge** by Dany Da. Detailed information and data related to this project are available [here](https://8weeksqlchallenge.com/case-study-1/).

## Problem statement
Danny wants to use data to answer some questions about his customers, their visiting patterns, the money they've spent and the most popular items in the menu to deliver a better experience to his customers.

You can find the structure of the databases provided by Danny [here](https://dbdiagram.io/d/Dannys-Diner-608d07e4b29a09603d12edbd?utm_source=dbdiagram_embed&utm_medium=bottom_open). 

## Case Study Questions

**1. What is the total amount each customer spent at the restaurant?**

 ```sql
SELECT
  customer_id,
  SUM(price) as total_spent
FROM
  sales
  INNER JOIN menu on sales.product_id = menu.product_id
GROUP BY
  customer_id
ORDER BY
  total_spent;
```

**Steps:**
* Sum the values of `prices` to calculate the total money spent.
* Join the tables `sales` and `menu` on `product_id`.
* Group by `customer`, so the sum of all the spent money is computed by client.
* Order by the value of `total_spent` for more visual results.

**Output:**
| customer_id | total_spent |
| ----------- | ----------- |
| C           | 36          |
| B           | 74          |
|A            |76           |

Customer A spent 76$, customer B spent 74$ and customer C spent 36$. The customer who has spent more money is customer A.

**2. How many days has each customer visited the restaurant?**

```sql
SELECT
  customer_id,
  COUNT(order_date) AS visit_count
FROM
  sales
GROUP BY
  customer_id
ORDER BY
  visit_count;
```
**Steps:**
* Count the `order_date` entries to calculate the number of visits.
* Group by `customer_id_ so the visits are counted for each customer.
* Order by the value of `visit_count` for more visual results.

**Output:**

| customer_id | visit_count |
| ----------- | ----------- |
| C           | 3           |
| B           | 6           |
| A           | 6           |

Customer A and customer B have visited the restaurant 6 times, while customer C has only visited it 3 times.

**3. What was the first item from the menu purchased by each customer?**
```sql
WITH sales_by_date AS(
  SELECT
    customer_id,
    order_date,
    product_name,
    DENSE_RANK() OVER (
    PARTITION BY customer_id
    ORDER BY
      order_date
      ) AS rank
  FROM
    sales
    INNER JOIN menu on sales.product_id = menu.product_id
  GROUP BY
    customer_id,
    order_date,
    product_name
)
SELECT
  customer_id,
  product_name
FROM
  sales_by_date
WHERE 
  rank = 1;
```
**Steps:**
* Create a Common Table Expression (CTE) to extract the orders by date.
    * Rank the order of dates by customer. 
    * Join `menu` and `sales` on `product_id`
    * Group by `customer_id`, `order_date` and `product_name`.
* Create a query to extract the oldest date from the CTE, which should be ranked as 1.

**Output:**
| customer_id | product_name |
| ----------- | -----------  |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

The first items purchased by A were curry and sushi. Customer had curry as their first purchase, and customer C ordered ramen.

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT 
  product_name,
  COUNT(product_name) AS times_purchased
FROM
  menu
  INNER JOIN sales on menu.product_id = sales.product_id
GROUP BY
  product_name
LIMIT
  1;
```
**Steps:**
* Count the entries of `product_name` to count the times it has been purchased.
* Join `sales` and `menu` on `product_id`.
* Group by `product_name`, so we get the count for each product.
* Show only 1 result on screen, which will be the most purchased.

**Output:**
| product_name | times_purchased |
| ------------- | -------------- |
| ramen         | 8              |

The most ordered item was ramen and it was purchased 8 times.

**5. Which item was the most popular for each customer?**
```sql
WITH purchased_food AS(
  SELECT
    customer_id, 
    product_name, 
    COUNT(product_name) AS times_purchased, 
    DENSE_RANK() OVER (
      PARTITION BY customer_id 
      ORDER BY 
        COUNT(product_name) DESC
    ) AS rank 
  FROM 
    sales 
    INNER JOIN menu on sales.product_id = menu.product_id 
  GROUP BY 
    customer_id, 
    product_name 
  ORDER BY 
    customer_id
) 
SELECT 
  customer_id, 
  product_name, 
  times_purchased 
FROM 
  purchased_food 
WHERE 
  rank = 1;
```
**Steps:**
* Create a CTE to count the times each product was purchased.
    * Count each case of `product_name` on the table to count the number of times each item was purchased.
    * Rank those counts by customer.
    * Join menu and sales on `product_id`.
    * Group by `customer_id` and `product_id` and order by customer.
* Create a query to extract the `times_purchased` counts ranked as 1, which should be the highest values.

**Output:**
| customer_id | product_name | times_purchased |
| ----------- | -----------  | --------------- |
| A           | ramen        | 3               |
| B           | sushi        | 2               |
| B           | curry        | 2               |
| B           | ramen        | 2               |
| C           | ramen        | 3               |

The most popular item for customer A was ramen, which they purchased three times. Customer B was fond of all the dishes, as they ordered two times each. For customer C, the most popular item was ramen.

**6. Which item was purchased first by the customer after they became a member?**
```sql
WITH ranked_purchases AS (
  SELECT 
    sales.customer_id,
    product_name,
    ROW_NUMBER() OVER(
    PARTITION BY sales.customer_id
    ORDER BY order_date) AS row
  FROM
    menu
    INNER JOIN sales ON menu.product_id = sales.product_id
    INNER JOIN members ON sales.customer_id = members.customer_id
  WHERE
    order_date >= join_date
)
SELECT
  customer_id,
  product_name
FROM
  ranked_purchases
WHERE
  row = 1;
```
**Steps:**
* Create a CTE to enumerate each row.
    * Add row numbers ordered by `order_date` for each customer.
    * Join `menu` and `sales` on `product_id`.
    * Join `members` and `sales` on `customer_id`.
    * Select dates equal to or after the date the customer became a member.
* Create a query to extract the first products purchased by each customer, which should be on `row` = 1.

**Output:**
| customer_id | product_name |
| ----------- | -----------  |
| A           | curry        |
| B           | sushi        |

The first item customer A ordered as a member was curry. Customer B ordered sushi.

**7. Which item was purchased just before the customer became a member?**
```sql
WITH ranked_purchases AS (
  SELECT 
    sales.customer_id,
    product_name,
    ROW_NUMBER() OVER(
    PARTITION BY sales.customer_id
    ORDER BY order_date DESC) AS rank
  FROM
    menu
    INNER JOIN sales ON menu.product_id = sales.product_id
    INNER JOIN members ON sales.customer_id = members.customer_id
  WHERE
    order_date < join_date
)
SELECT
  customer_id,
  product_name
FROM
  ranked_purchases
WHERE
  rank = 1;
```

**Steps:**
* Create a CTE to enumerate each row.
    * Add row numbers ordered by `order_date` for each customer.
    * Join `menu` and `sales` on `product_id`.
    * Join `members` and `sales` on `customer_id`.
    * Select dates before the date the customer became a member.
* Create a query to extract the last product purchased by each customer before they had a membership, which should be on `row` = 1.

**Output:**
| customer_id | product_name |
| ----------- | -----------  |
| A           | sushi        |
| B           | sushi        |

Before becoming members, both customer A and B ordered sushi.

# 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT
  sales.customer_id,
  COUNT(sales.customer_id) AS order_amount,
  SUM(price) AS amount_spent
FROM
  menu
  INNER JOIN sales ON menu.product_id = sales.product_id
  INNER JOIN members ON sales.customer_id = members.customer_id
WHERE
  order_date < join_date
GROUP BY
  sales.customer_id;
```
**Steps:**
* Count the `customer_id` entries to obtain the amount of orders per customer.
* Sum the prices to compute the money spent.
* Join sales and menu on `product_id`.
* Join members and sales on `customer_id`.
* Select cases before the `join_date`.
* Group by customer.

**Output:**
| customer_id | product_name | amount_spent |
| ----------- | -----------  | ------------ |
| B           | 3            | 40           |
| A           | 2            | 25           |

The customer that has made more orders is customer B. They are also the customer that has spent more money at the restaurant.

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
WITH point_counter AS(
  SELECT
    product_id,
  CASE
    WHEN product_id = 1 THEN (price * 10) * 2
    ELSE price * 10
  END AS points
  FROM
    menu
)
SELECT
  customer_id,
  SUM(points) AS point_count
FROM 
  sales
  INNER JOIN point_counter ON sales.product_id = point_counter.product_id
GROUP BY
  customer_id
ORDER BY
  point_count DESC;
```
**Steps:**
* Create a CTE to compute the points gained by each customer.
    * Create a case for sushi, so $1 equates to 20 points.
    * In other cases (curry and ramen), 1 dollar equates to 10 points.
* Create a query to sum the total points gained by each customer.
* Join sales with the CTE `point_counter` to link the results with the customer IDs.
* Group by customer and order by `point_count` for more visual results.

**Output:**
| customer_id | point_count |
| ----------- | ----------- |
| B           | 940         |
| A           | 860         |
| C           | 360         |

With this point system, customer B would have 940 points, customer A would have 860 and customer C would have 360 points.

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH point_calculator AS (
  SELECT
    sales.customer_id,
    price,
    CASE
      WHEN order_date >= join_date AND order_date <= join_date + 7 THEN (price * 10) * 2
      ELSE CASE
        WHEN menu.product_id = 1 THEN (price * 10) * 2
        ELSE price * 10
        END
      END AS points,
      EXTRACT(MONTH FROM order_date) AS month
  FROM
    sales
    INNER JOIN menu ON sales.product_id = menu.product_id
    INNER JOIN members ON sales.customer_id = members.customer_id
)
SELECT
  customer_id,
  SUM(points) AS january_points
FROM
  point_calculator
WHERE
  month = 1
GROUP BY
  customer_id
ORDER BY
  january_points DESC;
```
**Steps:**
* Create a CTE to compute the points gained by each customer.
    * Create a case for orders in the first week of membership (between the `join_date` and the 7 following days).
    * In other cases create a new case for sushi so $1 equates to 20 points.
    * In the rest of cases (curry and ramen after first week of membership) $1 equates to 10 points.
    * Extract the month from the `order_date` column.
    * Join menu and sales on `product_id`.
    * Join members on `customer_id`.
* Create a query to sum the total points gained by each customer.
* Select purchases on January (month = 1).
* Group by customer and order by points gained in January for more visual results.

**Output:**
| customer_id | january_points |
| ----------- | -------------- |
| A           | 1370           |
| B           | 940            |

In this case, customer A would have 1370 points in total, and customer B, would sum 940 points.

# Bonus Questions

**Join All The Things**

We need to create a table so Danny and his team can use it to quickly derive insights without needing to join the underlying tables using SQL. We need to recreate this output:

| customer_id |	order_date	| product_name | price | member |
| ----------- | ----------- | ------------ | ----- | ------ |
| A           | 2021-01-01	 | curry	       | 15    | N      |
| A           |	2021-01-01  | sushi	       | 10    | N      |
| A           |	2021-01-07  | curry	       | 15    | Y      |
| A	          | 2021-01-10  | ramen        | 12    | Y      |
| A	          | 2021-01-11  | ramen	       | 12    | Y      |
| A	          | 2021-01-11  | ramen        | 12    | Y      |
| B           | 2021-01-01  | curry        | 15    | N      |
| B	          | 2021-01-02 	| curry	       | 15    | N      |
| B	          |2021-01-04	  | sushi	       | 10    | N      |
| B	          |2021-01-11	  | sushi	       | 10    | Y      |
| B	          |2021-01-16	  | ramen	       | 12    | Y      |
| B	          |2021-02-01   | ramen	       | 12    | Y      |
| C	          |2021-01-01	  | ramen	       | 12    | N      |
| C	          |2021-01-01	  | ramen	       | 12    | N      |
| C	          |2021-01-07 	 | ramen	       | 12    | N      |

Here is the code to do it:
```sql
SELECT
  sales.customer_id,
  order_date,
  product_name,
  price,
  CASE
    WHEN order_date >= join_date THEN 'Y'
    WHEN order_date < join_date THEN 'N'
    ELSE 'N'
  END AS member
FROM
  sales
  INNER JOIN menu ON sales.product_id = menu.product_id
  LEFT JOIN members ON sales.customer_id = members.customer_id
ORDER BY
  customer_id,
  order_date,
  price;
```
**Steps:**
* Select the variables we would like to display.
* Create a case in order to create the column `member` with the values "Y" and "N".
* Join all the tables.
* Order by `customer_id`, `order_date` and `price`.

**Rank All The Things**

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program. Let's recreate the following table:

| customer_id |	order_date	| product_name | price | member | ranking |
| ----------- | ----------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01	| curry	       | 15    | N      | NULL    |
| A           |	2021-01-01  | sushi	       | 10    | N      | NULL    |
| A           |	2021-01-07  | curry	       | 15    | Y      | 1       |
| A	          | 2021-01-10  | ramen        | 12    | Y      | 2       |
| A	          | 2021-01-11  | ramen	       | 12    | Y      | 3       |
| A	          | 2021-01-11  | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01  | curry        | 15    | N      | NULL    |
| B	          | 2021-01-02	| curry	       | 15    | N      | NULL    |
| B	          |2021-01-04	| sushi	       | 10    | N      | NULL    |
| B	          |2021-01-11	| sushi	       | 10    | Y      | 1       |
| B	          |2021-01-16	| ramen	       | 12    | Y      | 2       |
| B	          |2021-02-01	| ramen	       | 12    | Y      | 3       |
| C	          |2021-01-01	| ramen	       | 12    | N      | NULL    |
| C	          |2021-01-01	| ramen	       | 12    | N      | NULL    |
| C	          |2021-01-07	| ramen	       | 12    | N      | NULL    |

Here is the code to do it:

```sql
WITH customer_table AS(
  SELECT
    sales.customer_id,
    order_date,
    product_name,
    price,
    CASE
      WHEN order_date >= join_date THEN 'Y'
      WHEN order_date < join_date THEN 'N'
      ELSE 'N'
    END AS member
  FROM
    sales
    INNER JOIN menu ON sales.product_id = menu.product_id
    LEFT JOIN members ON sales.customer_id = members.customer_id
)
SELECT
  *,
  CASE
    WHEN member = 'N' THEN NULL
    ELSE RANK() OVER (
    PARTITION BY customer_id
    ORDER BY order_date)
  END AS ranking
FROM
  customer_table
ORDER BY
  customer_id,
  order_date,
  ranking,
  price;
```
**Steps:**
* Create a CTE to recreate the previous table.
    * Create a case in order to create the column `member` with the values "Y" and "N".
    * Join all the tables.
* Create a query to select all the variables included in the CTE.
* Create a case to fill the column `member` as "NULL" for dates where the customer didn't have a membership.
* In other cases, create a rank by `order_date` for each customer.
* Order by `customer_id`, `order_date`, `ranking` and `price`.
