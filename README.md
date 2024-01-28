# 🍕 Case Study #2 Pizza Runner
<p align="center">
<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">
</p>

## 📚 Table of Contents
- [Business Task](#business-task)
- [Dataset](#dataset)
- [Data Cleaning](#data-cleaning)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-2/). 

***

## Business Task
Did you know that a whopping 115 million kilograms of pizza are consumed daily worldwide? (Well according to Wikipedia anyway…) Inspired by the concept of "80s Retro Styling and Pizza Is The Future!" on Instagram, Danny launched Pizza Runner. To secure funding for his Pizza Empire expansion, he "Uberized" pizza delivery, recruiting runners and developing a mobile app with funds from maxing out his credit card.

***

## Dataset

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

The dataset includes:

- `runners`: This table displays the registration date for each new runner.

- `customer_orders`: Captures customer pizza orders, with one row per pizza in the order. The pizza_id corresponds to the ordered pizza type, while exclusions are ingredient_id values to be removed, and extras are ingredient_id values to be added.

- `runner_orders`: After orders are received, they are assigned to a runner. Some orders may be canceled. Pickup_time is the timestamp when the runner picks up pizzas from Pizza Runner headquarters. Distance and duration fields indicate the travel required for delivery.

- `pizza_names`: Pizza Runner offers only two pizzas: Meat Lovers or Vegetarian.

- `pizza_recipes`: Each pizza_id has a standard set of toppings used in the pizza recipe.

- `pizza_toppings`: This table lists all topping_name values with their corresponding topping_id value.

***

## Data Cleaning

### Table: customer_orders

Looking at the `customer_orders` table below:
- In the `exclusions` column, there are missing/blank spaces ' ' and null values. 
- In the `extras` column, there are missing/blank spaces ' ' and null values.

<img width="1063" alt="image" src="https://user-images.githubusercontent.com/81607668/129472388-86e60221-7107-4751-983f-4ab9d9ce75f0.png">

To clean the table:
- Create a temporary table with all the columns
- Remove null values in the `exclusions` and `extras` columns and replace them with blank space ' '.

````sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;
`````

The clean `customers_orders_temp` table now looks like the below and we will use this table to run all our queries.

<img width="1058" alt="image" src="https://user-images.githubusercontent.com/81607668/129472551-fe3d90a0-1e8b-4f32-a2a7-2ecd3ac469ef.png">

***

## Table: runner_orders

Looking at the `runner_orders` table below:
- In the `exclusions` column, there are missing/ blank spaces ' ' and null values. 
- In the `extras` column, there are missing/ blank spaces ' ' and null values

<img width="1037" alt="image" src="https://user-images.githubusercontent.com/81607668/129472585-badae450-52d2-442e-9d50-e4d0d8fce83a.png">

To clean the table:
- In `pickup_time` column, remove nulls and replace with blank space ' '.
- In `distance` column, remove "km" and nulls and replace them with blank space ' '.
- In `duration` column, remove "minutes", "minute" and nulls and replace them with blank space ' '.
- In `cancellation` column, remove NULL and null and and replace with blank space ' '.

````sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN pickup_time LIKE 'null' THEN ' '
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN distance LIKE 'null' THEN ' '
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN duration LIKE 'null' THEN ' '
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN cancellation IS NULL or cancellation LIKE 'null' THEN ' '
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;
````

Then, we alter the `pickup_time`, `distance` and `duration` columns to the correct data type.

````sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time DATETIME,
ALTER COLUMN distance FLOAT,
ALTER COLUMN duration INT;
````

The clean `runner_orders_temp` table now looks like the one below and we will use this table to run all our queries.

<img width="915" alt="image" src="https://user-images.githubusercontent.com/81607668/129472778-6403381d-6e30-4884-a011-737b1eff7379.png">

***

## Question and Solution

### A. Pizza Metrics

#### 1. How many pizzas were ordered?

````sql
SELECT COUNT(*) AS number_of_ordered_pizzas
FROM pizza_runner.customer_orders
````

**Answer**
| number_of_ordered_pizzas | 
| ------------------------ | 
| 14                       | 

#

#### 2. How many unique customer orders were made?

````sql
SELECT COUNT(DISTINCT order_id) AS unique_customer_order
FROM customer_orders_temp
````

 **Answer**
| unique_customer_order | 
| --------------------- | 
| 10                    | 

#

#### 3. How many successful orders were delivered by each runner?

````sql
SELECT runner_id,
       COUNT(DISTINCT order_id) AS count
FROM customer_orders_temp
WHERE distance <> 0
GROUP BY runner_id
````

**Answer**
| runner\_id | count |
| ---------- | ----- |
| 1          | 4     |
| 2          | 3     |
| 3          | 1     |

#

#### 4. How many of each type of pizza was delivered?

````sql
SELECT t2.pizza_name,
       COUNT(t1.pizza_id) AS number_of_order
FROM pizza_runner.customer_orders AS t1
JOIN pizza_runner.pizza_names AS t2
  ON t1.pizza_id = t2.pizza_id
JOIN pizza_runner.runner_orders AS t3
  ON t1.order_id = t3.order_id
WHERE t3.distance <> 0
GROUP BY t2.pizza_name
````

**Answer**
| pizza\_name | number\_of\_order |
| ----------- | ----------------- |
| Meatlovers  | 9                 |
| Vegetarian  | 3                 |

#

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?

````sql
SELECT t1.customer_id,
       t2.pizza_name,
       COUNT(t1.pizza_id) AS number_of_order
FROM pizza_runner.customer_orders AS t1
JOIN pizza_runner.pizza_names AS t2
  ON t1.pizza_id = t2.pizza_id
GROUP BY 1, 2
ORDER BY 1, 2
 ````
 
 **Answer**
 | customer\_id | pizza\_name | number\_of\_order |
| ------------ | ----------- | ------------------ |
| 101          | Meatlovers  | 2                  |
| 101          | Vegetarian  | 1                  |
| 102          | Meatlovers  | 2                  |
| 102          | Vegetarian  | 1                  |
| 103          | Meatlovers  | 3                  |
| 103          | Vegetarian  | 1                  |
| 104          | Meatlovers  | 3                  |
| 105          | Vegetarian  | 1                  |

#

#### 6. What was the maximum number of pizzas delivered in a single order?

````sql
WITH cte AS(
  	  SELECT order_id,
                 COUNT(pizza_id) AS number_of_pizzas,
                 DENSE_RANK() OVER(ORDER BY COUNT(pizza_id) DESC) AS rank
          FROM pizza_runner.customer_orders
          WHERE EXISTS (SELECT 1
                        FROM pizza_runner.runner_orders
                        WHERE pizza_runner.customer_orders.order_id = pizza_runner.runner_orders.order_id
                        AND distance <> 0
                      )
         GROUP BY order_id
)

SELECT number_of_pizzas
FROM cte
WHERE rank = 1
````

**Answer**
| number_of_pizzas | 
| ---------------- | 
| 3                | 

#

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT c.customer_id,
       SUM(
         CASE WHEN c.exclusions <> ' ' OR c.extras <> ' ' THEN 1
         ELSE 0 END) AS at_least_1_change,
       SUM(
         CASE WHEN c.exclusions = ' ' AND c.extras = ' ' THEN 1 
         ELSE 0 END) AS no_change
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE r.distance <> 0
GROUP BY c.customer_id
ORDER BY c.customer_id;
````

**Answer**
| customer_id | at_least_1_change | no_change | 
| ----------- | ----------------- | --------- |
| 101         | 0                 | 2         |
| 102         | 0                 | 3         |
| 103         | 3                 | 0         |
| 104         | 2                 | 1         |
| 105         | 1                 | 0         |

#

#### 8.  How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT SUM(
         CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1
         ELSE 0 END) AS pizza_count_w_exclusions_extras
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE r.distance >= 1 
  AND exclusions <> ' ' 
  AND extras <> ' ';
````

**Answer**
| count	       | 
| ------------ | 
| 1            | 

#

#### 9.  What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT 
  DATE_PART('hour', order_time) AS hour,
  COUNT(order_id) AS count
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 1
````

**Answer**
| hour | count |
| ---- | ----- |
| 11   | 1     |
| 12   | 2     |
| 13   | 3     |
| 18   | 3     |
| 19   | 1     |
| 21   | 3     |
| 23   | 1     |

#

### 10.  What was the volume of orders for each day of the week?

````sql
SELECT TO_CHAR(order_time,'Day') as day_of_week,
       COUNT(*)
FROM pizza_runner.customer_orders
GROUP BY 1
ORDER BY 2
````

**Answer**
| day\_of\_week | count |
| ------------- | ----- |
| Sunday        | 1     |
| Saturday      | 3     |
| Monday        | 5     |
| Friday        | 5     |

***

### B. Runner and Customer Experience

#### 1. How many runners signed up for each 1 week period? (i.e. week starts `2021-01-01`)
````sql
SELECT (DATE_TRUNC('week', registration_date - INTERVAL '4 DAY') + INTERVAL '4 DAY'
         ) :: DATE AS start_of_week,
       COUNT(DISTINCT runner_id)
FROM pizza_runner.runners
GROUP BY 1
````

**Answer**
| start\_of\_week| count |
| ---------------| ----- |
| 2021-01-01     | 2     |
| 2021-01-08     | 1     |
| 2021-01-15     | 1     |

#

#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

````sql
WITH cte AS(
           SELECT DISTINCT t1.order_id,
                  t2.runner_id,
                  (EXTRACT(EPOCH FROM t2.pickup_time :: TIMESTAMP) -
                   EXTRACT(EPOCH FROM t1.order_time)
                  ) / 60 as difference_in_minutes
           FROM pizza_runner.customer_orders AS t1
           JOIN pizza_runner.runner_orders AS t2
             ON t1.order_id = t2.order_id
          WHERE t2.pickup_time <> 0
)

SELECT runner_id,
       CEILING(AVG(difference_in_minutes) :: NUMERIC) AS average_time_in_minutes
FROM cte
GROUP BY 1
ORDER BY 1
````

**Answer**
| runner\_id | average\_time\_in\_minutes |
| ---------- | -------------------------- |
| 1          | 14                         |
| 2          | 20                         |
| 3          | 10                         |

#

#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

````sql
WITH prep_time_cte AS
(
  SELECT c.order_id, 
    	 COUNT(c.order_id) AS pizza_order, 
    	 c.order_time, 
    	 r.pickup_time, 
    	 DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS prep_time_minutes
  FROM pizza_runner.customer_orders AS c
  JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
  WHERE r.distance != 0
  GROUP BY c.order_id, c.order_time, r.pickup_time
)

SELECT pizza_order, 
       AVG(prep_time_minutes) AS avg_prep_time_minutes
FROM prep_time_cte
WHERE prep_time_minutes > 1
GROUP BY pizza_order;
````

**Answer**
| pizza_order | avg_prep_time_minutes | 
| ----------- | --------------------- | 
| 1           | 12                    | 
| 2           | 16                    | 
| 3           | 30                    | 

#

#### 4.  What was the average distance travelled for each customer?

````sql
SELECT 
  c.customer_id, 
  ROUND(AVG(r.distance), 1) AS avg_distance
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE r.duration != 0
GROUP BY c.customer_id;
````

**Answer**
| customer\_id | avg\_distance |
| ------------ | ------------- |
| 101          | 20.0          |
| 102          | 16.7          |
| 103          | 23.4          |
| 104          | 10.0          |
| 105          | 25.0          |

#

### 5.  What was the difference between the longest and shortest delivery times for all orders?

````sql
WITH cte AS(
          SELECT (UNNEST(REGEXP_MATCH(duration, '^[0-9]{2}')) :: NUMERIC) AS new_duration
          FROM pizza_runner.runner_orders
          WHERE distance <> 'null'
)

SELECT MAX(new_duration)-MIN(new_duration) AS difference_between_max_and_min
FROM cte
````

**Answer**
| difference_between_max_and_min | 
| ------------------------------ | 
| 30                             | 

#

#### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
WITH cte AS(
          SELECT DISTINCT t1.runner_id,
                 t1.order_id,
                 t2.customer_id,
                 DATE_PART('hour', pickup_time :: TIMESTAMP) AS PU_hour,
                 (UNNEST(REGEXP_MATCH(distance, '^[0-9,.]+')) :: NUMERIC) AS new_distance,
                 (UNNEST(REGEXP_MATCH(duration, '^[0-9]{2}')) :: NUMERIC) AS new_duration
          FROM pizza_runner.runner_orders AS t1
          JOIN pizza_runner.customer_orders AS t2
            ON t1.order_id = t2.order_id
         WHERE distance <> 'null'
)

SELECT runner_id,
       order_id,
       customer_id,
       PU_hour,
       CEILING(new_distance / (new_duration / 60)) as average_speed
FROM cte
ORDER BY 1, 5
````

**Answer**
| runner\_id | order\_id | customer\_id | pu\_hour | average\_speed |
| ---------- | --------- | ------------ | -------- | -------------- |
| 1          | 1         | 101          | 18       | 38             |
| 1          | 3         | 102          | 0        | 40             |
| 1          | 2         | 101          | 19       | 44             |
| 1          | 10        | 104          | 18       | 60             |
| 2          | 4         | 103          | 13       | 35             |
| 2          | 7         | 105          | 21       | 60             |
| 2          | 8         | 102          | 0        | 94             |
| 3          | 5         | 104          | 21       | 40             |

#

### 7. What is the successful delivery percentage for each runner?

````sql
SELECT runner_id,
       CEILING(100 * (SUM(
                         CASE WHEN distance != 'null' THEN 1
                              ELSE 0 END
                          ) / SUM(COUNT(*)) OVER(PARTITION BY runner_id)
                      )
               ) AS success_rate
FROM pizza_runner.runner_orders
GROUP BY 1
ORDER BY 1
````

**Answer**
| runner\_id | success\_rate |
| ---------- | ------------- |
| 1          | 100           |
| 2          | 75            |
| 3          | 50            |

***

### C. Ingredient Optimisation

#### 1. What are the standard ingredients for each pizza?

````sql
WITH new_pizza_recipes AS (
  		        SELECT pizza_id,
    			       REGEXP_SPLIT_TO_TABLE(toppings, ',\s') :: INTEGER AS topping_id
  			FROM pizza_runner.pizza_recipes
)

SELECT t1.pizza_id,
       STRING_AGG(t2.topping_name, ', ') as ingredients
FROM new_pizza_recipes AS t1
JOIN pizza_runner.pizza_toppings AS t2
  ON t1.topping_id = t2.topping_id
GROUP BY 1
ORDER BY 1
````

**Answer**
| pizza\_id | ingredients                                                           |
| --------- | --------------------------------------------------------------------- |
| 1         | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 2         | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

#

#### 2. What was the most commonly added extra?

````sql
WITH cte AS(
          SELECT REGEXP_SPLIT_TO_TABLE(extras, ',\s') AS new_extras
          FROM pizza_runner.customer_orders
          WHERE extras IS NOT NULL
            AND extras <> 'null'
)

SELECT t2.topping_name,
       COUNT(*) AS frequency
FROM cte AS t1
JOIN pizza_runner.pizza_toppings AS t2
  ON t1.new_extras::INTEGER = t2.topping_id
WHERE new_extras <> ''
GROUP BY 1
ORDER BY 2 DESC, 1 
````

**Answer**
| topping\_name | frequency |
| ------------- | --------- |
| Bacon         | 4         |
| Cheese        | 1         |
| Chicken       | 1         |

#

#### 3. What was the most common exclusion?

````sql
WITH cte AS(
           SELECT REGEXP_SPLIT_TO_TABLE(exclusions, ',\s') AS new_exclusions
           FROM pizza_runner.customer_orders
           WHERE exclusions <> 'null'
)

SELECT t2.topping_name,
       COUNT(*) AS frequency
FROM cte AS t1
JOIN pizza_runner.pizza_toppings AS t2
  ON t1.new_exclusions :: INTEGER = t2.topping_id
WHERE new_exclusions <> ''
GROUP BY 1
ORDER BY 2 DESC, 1
````

**Answer**
| topping\_name | frequency |
| ------------- | --------- |
| Cheese        | 4         |
| BBQ Sauce     | 1         |
| Mushrooms     | 1         |
