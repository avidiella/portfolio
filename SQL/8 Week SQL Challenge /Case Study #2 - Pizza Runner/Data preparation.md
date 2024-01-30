# Case Study #2 - Pizza Runner 🍕

## Data preparation

### Table `customer_orders`

The table `customer_orders` looks like this:

![image](https://github.com/avidiella/portfolio/assets/143961739/daf19a15-e35d-43a9-a225-7bc9bf06fbd6)

The issues we should adress are:
* Inconsistency in the format of data entry in the column exclusions. There are codes for the ingredients, which are numbers, and text strings.
* Inconsistency in the format of data entry in the column extras. We have the codes for the ingredients, which are numbers, cells with null values, and text strings.

```sql
CREATE TEMPORARY TABLE clean_customer_orders AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE
    WHEN exclusions = 'null' OR exclusions = '' THEN null
    ELSE exclusions
  END AS exclusions,
  CASE
    WHEN extras = 'null' OR extras = '' THEN null
    ELSE extras
  END AS extras,
  order_time
FROM
  customer_orders;
```

**Steps:**
* Create a temporary table for our clean data.
* Select the columns that don't need any cleaning.
* For column `exclusions` create a case to fill empty cells and cells filled with the string "null" with null values.
* For column `extras` create a case to fill empty cells and cells filled with the string "null" with null values.

With our code, the table now looks like this:

![image](https://github.com/avidiella/portfolio/assets/143961739/fc596494-9fde-4aae-8872-03b43c317e39)


### Table `runner_orders`

The table `runner_orders` looks like this:

![image](https://github.com/avidiella/portfolio/assets/143961739/64eda08e-f1b4-4880-b3d2-f1ee56719131)

The issues to address are:
* Inconsistency in the format of data entry in column pickup_time. There are dates and "null" strings.
* The data type of the column pickup_time is `character varying`, but should be `timestamp`.
* Inconsistency in the format of data entry in column distance. There are distances with a numeric value followed by the measurement unit; there are other cases where there is a space between the value and the unit, integers, floats and strings.
* The data type of the column distance is `character varying`, but it should be `float`.
* Inconsistency in the format of data entry in column duration. There are times with a numeric value followed by the time unit, numeric values and time units separated by a space, and "null" strings.
* The data type of the column duration is `character varying`, but it should be `integer`.
* Inconsistency in the format of data entry in column cancellation. There are empty cells, null values and different strings.

```sql
CREATE TEMPORARY TABLE clean_runner_orders AS
SELECT
  order_id,
  runner_id,
  CASE
    WHEN pickup_time = 'null' THEN null
    ELSE pickup_time
  END AS pickup_time,
  CASE
    WHEN distance = 'null' THEN null
    ELSE TRIM('km' FROM distance)
  END AS distance,
  CASE
    WHEN duration = 'null' THEN null
    ELSE SUBSTRING(duration, 1, 2)
  END AS duration,
  CASE
    WHEN cancellation = 'null' OR cancellation = '' THEN null
    ELSE cancellation
  END AS cancellation
FROM
  runner_orders;

ALTER TABLE
  clean_runner_orders
  ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::timestamp without time zone,
  ALTER COLUMN distance TYPE float USING distance::double precision,
  ALTER COLUMN duration TYPE int USING duration::integer;
```

**Steps:**
* Create a temporary table for our clean data.
* Select the columns that don't need any cleaning.
* For column pickup_time, create a case to fill cells filled with the string "null" with null values.
* For column distance, create a case to fill cells with string "null" with null values, else trim the measurement unit from the numeric value.
* For column duration, create a case to fill cells with string "null" or empty cells with null values, else extract the numeric value of the duration.
* For column cancellation, create a case to fill empty cells and cells filled with the string "null" with null values.
* Modify the columns in the temporary table to have the correct data type.

With our code, the table now looks like this:

![image](https://github.com/avidiella/portfolio/assets/143961739/7efc11d4-bb97-480c-846d-b4839a6b42c4)


Next section: [Pizza Metrics]()
