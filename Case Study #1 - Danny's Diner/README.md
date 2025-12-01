# Case Study #1 - Danny's Diner

For the full description for this case study, please visit: [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)



## Table of Contents

[Introduction](#introduction)

[Case Study Question and Solutions](#case-study-questions-and-solutions)



## Introduction

Danny is a lover of Japanese food so he has decided to open a restaurant which sells his 3 favourite foods: sushi, curry and ramen. 

### Problem Statement

Danny wants to use the data to find out more about his customers and answer a few questions about their visiting patterns, how much money they’ve spent and which items on the menu are their favourites. He plans to use these insights to expand the existing customer loyalty program.

### Datasets

Danny has shared 3 key datasets for this case study.

<details>

<summary>Table 1: sales</summary>

| customer_id | order_date   | product_id |
|-------------|--------------|------------|
| A           | 2021-01-01   | 1          |
| A           | 2021-01-01   | 2          |
| A           | 2021-01-07   | 2          |
| A           | 2021-01-10   | 3          |
| A           | 2021-01-11   | 3          |
| A           | 2021-01-11   | 3          |
| B           | 2021-01-01   | 2          |
| B           | 2021-01-02   | 2          |
| B           | 2021-01-04   | 1          |
| B           | 2021-01-11   | 1          |
| B           | 2021-01-16   | 3          |
| B           | 2021-02-01   | 3          |
| C           | 2021-01-01   | 3          |
| C           | 2021-01-01   | 3          |
| C           | 2021-01-07   | 3          |

</details>

<details>

<summary>Table 2: menu</summary>

| product_id | product_name | price |
|------------|--------------|-------|
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

</details>

<details>

<summary>Table 3: members</summary>

| customer_id | join_date   |
|-------------|-------------|
| A           | 2021-01-07  |
| B           | 2021-01-09  |

</details>

***

## Case Study Questions and Solutions

**1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT
	s.customer_id,
    SUM(m.price) AS total_amount_spent
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m 
	ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

Result:

|customer_id | total_amount_spent |
|---|---|
| A | 76 |
| B | 74 |
| C | 36 |

- Customer A spent the most amount with £76
- Customer B spend £74
- Customer C spent the least amount with £36.

***

**2. How many days has each customer visited the restaurant?**

```sql
SELECT
	s.customer_id,
    COUNT(DISTINCT s.order_date) AS days_visited
FROM dannys_diner.sales AS s
GROUP BY s.customer_id;
```

Result:

| customer_id | days_visited |
| --- | --- |
| A | 4 |
| B | 6 |
| C | 2 |

- Customer A has been to the restaurant 4 times.
- Customer B has been to the restaurant 6 times, which is the most amount out of all customers.
- Customer C has been to the restaurant 2 times, which is the least amount out of all customers.

***

**3. What was the first item from the menu purchased by each customer?**

```sql
WITH sales_order AS(
  SELECT *, ROW_NUMBER () OVER (PARTITION BY s.customer_id ORDER BY order_date) AS order_of_sale
  FROM dannys_diner.sales AS s
  JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
)
SELECT customer_id, product_name AS first_item_purchased
FROM sales_order
WHERE order_of_sale = 1;
```

Result:

| customer_id | first_item_purchased |
| --- | --- |
| A | curry |
| B | curry |
| C | ramen |

- From my solution, Customer A first purchased curry from the restaurant. 
    - Looking at the `sales` table, Customer A first visited the restaurant on `2021-01-01` and ordered two items: sushi and curry.
    - My solution may not be accurate as Customer A did order 2 items as their first order. The data does not provide timestamps for each order so I do not know exactly which item was ordered first by a customer when ordered on the same day.
    - I used `ROW_NUMBER` in my solution as if Danny was to change the datatype for `order_date` to include a timestamp, my solution can determine which item was ordered first.
- Customer B first purchased curry from the restaurant.
- Customer C first purchased ramen from the restaurant.

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT 
	m.product_name,
	COUNT(s.product_id) AS times_ordered
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.menu AS m 
	ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY COUNT(s.product_id) DESC
LIMIT 1;
```

Result:

| product_name | times_ordered |
| --- | --- |
| ramen | 8 |

- Ramen is the most purchased item on the menu and was purchased 8 times by all customers.

***

**5. Which item was the most popular for each customer?**

<details>

<summary>First attempt</summary>

```sql
WITH sales_count AS(
  SELECT 
  	s.customer_id, 
  	m.product_name, 
  	COUNT(s.product_id) AS product_count,
  	ROW_NUMBER () OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS product_count_rank
  FROM dannys_diner.sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name AS most_popular_item
FROM sales_count
WHERE product_count_rank = 1;
```

Result:

| customer_id | most_popular_item |
| --- | --- |
| A | ramen |
| B | ramen |
| C | ramen |

- Customer A's favourite item is ramen.
- Customer B's favourite item is ramen.
- Customer C's favourite item is ramen.

</details>

- From my first attempt, I used a CTE to calculate how many times each item was ordered by each customer and ranked them using `ROW_NUMBER` based on the `COUNT` of each item ordered. 
- I compared my answer to other solutions and realised that I didn't look at the `COUNT` for each customer and item properly and missed that Customer B ordered each item the same amount of times.
- I improved my solution to use `DENSE_RANK` to handle when a customer has more than 1 favourite item.

```sql
WITH sales_count AS(
  SELECT 
  	s.customer_id, 
  	m.product_name, 
  	COUNT(s.product_id) AS product_count,
  	DENSE_RANK () OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS product_count_rank
  FROM dannys_diner.sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
  GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name AS most_popular_item, product_count
FROM sales_count
WHERE product_count_rank = 1;
```

Result:
| customer_id | most_popular_item | product_count |
| --- | --- | --- |
| A | ramen | 3 |
| B | curry | 2 |
| B | ramen | 2 |
| B | sushi | 2 |
| C | ramen | 3 |

- Customer A's most popular item is ramen and has ordered it 3 times.
- Customer B has ordered each item the same amount of times, ordering curry, ramen and sushi 2 times each.
- Customer C's most popular item is ramen and has ordered it 3 times.

***

**6. Which item was purchased first by the customer after they became a member?**
```sql
WITH sales_order AS(
  SELECT 
    s.customer_id, 
    m.product_name, 
    ROW_NUMBER () OVER (PARTITION BY s.customer_id ORDER BY order_date) AS order_of_sale
  FROM dannys_diner.sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
  JOIN members AS mem on s.customer_id = mem.customer_id
  WHERE s.order_date > join_date
)

SELECT customer_id, product_name AS first_item_purchased
FROM sales_order 
WHERE order_of_sale = 1;
```

Result:

| customer_id | first_item_purchased |
| --- | --- |
| A | ramen |
| B | sushi |

- Customer A became a member on `2021-01-07`. Their first purchase after becoming a member was ramen.
- Customer B became amember on `2021-01-09`. Their first purchase after becoming a member was sushi.

***

**7. Which item was purchased just before the customer became a member?**
```sql
WITH sales_order AS(
  SELECT 
    s.customer_id, 
    m.product_name, 
    ROW_NUMBER () OVER (PARTITION BY s.customer_id ORDER BY order_date DESC) AS order_of_sale
  FROM dannys_diner.sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
  JOIN members AS mem on s.customer_id = mem.customer_id
  WHERE s.order_date < join_date
)

SELECT customer_id, product_name AS item_purchased_before_membership
FROM sales_order 
WHERE order_of_sale = 1;
```

Result:

| customer_id | item_purchased_before_membership |
| --- | --- |
| A | sushi |
| B | sushi |

- Before becoming a member, Customer A and Customer B ordered sushi.

***

**8. What is the total items and amount spent for each member before they became a member?**
```sql
WITH before_member AS(
  SELECT s.customer_id, m.product_name, m.price
  FROM dannys_diner.sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
  JOIN members AS mem on s.customer_id = mem.customer_id
  WHERE s.order_date < join_date
)

SELECT 
  customer_id, 
  COUNT(*) AS total_items_purchased_before_membership, 
  sum(price) AS total_amount_spent_before_membership
FROM before_member
GROUP BY customer_id;
```

Result:

| customer_id | total_items_purchased_before_membership | total_amount_spent_before_membership |
| --- | --- | --- |
| B | 3 | 40 |
| A | 2 | 25 |

- Before Customer A became a member, they ordered 2 items and spent £25.
- Before Customer B became a member, thy ordered 3 items and spent £40.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
WITH menu_points AS(
  SELECT
  	*,
  	CASE
  		WHEN m.product_id = 1 THEN m.price*20
  		WHEN m.product_id != 1 THEN m.price*10
  	END AS points
  FROM dannys_diner.menu AS m
)

SELECT s.customer_id, SUM(points) as points
FROM menu_points
JOIN sales AS s ON s.product_id = menu_points.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

Result:
| customer_id | points |
| --- | --- |
| A | 860 |
| B | 940 |
| C | 360 |

- Customer A would have a total of 860 points.
- Customer B would have a total of 940 points, which is the most amount of points out of all customers.
- Customer C would have a total of 360 points, which is the least amount of points out of all customers.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
WITH member_sales AS(
  SELECT 
  	s.customer_id, 
  	mem.join_date, 
  	s.order_date, 
  	s.product_id, 
  	m.product_name, 
  	m.price,
  	CASE
  		WHEN s.order_date <= mem.join_date + 6 THEN m.price*20
  		WHEN s.product_id = 1 THEN m.price*20
  		WHEN s.product_id != 1 THEN m.price*10
  	END AS points
  FROM dannys_diner.sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
  JOIN members AS mem on s.customer_id = mem.customer_id
  WHERE s.order_date >= mem.join_date AND s.order_date < '2021-01-31'
  ORDER BY s.order_date
)

SELECT customer_id, SUM(points) AS points
FROM member_sales
GROUP BY customer_id;
```

Result:

| customer_id | points |
| --- | --- |
| B | 320 |
| A | 1020 |

- Customer A will have 1020 points.
- Customer B will have 320 points.

***

**Bonus Questions**

**Join All The Things**

```sql
SELECT
	s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
    	WHEN mem.join_date IS NOT NULL AND s.order_date >= mem.join_date THEN 'Y'
        ELSE 'N'
    END AS member
FROM dannys_diner.sales AS s
LEFT JOIN menu AS m ON s.product_id = m.product_id
FULL JOIN members AS mem on s.customer_id = mem.customer_id
ORDER BY s.customer_id, s.order_date;
```

Result:

| customer_id | order_date  | product_name | price | member |
|-------------|-------------|--------------|-------|--------|
| A           | 2021-01-01 | curry         | 15    | N      | 
| A           | 2021-01-01 | sushi         | 10    | N      | 
| A           | 2021-01-07 | curry         | 15    | Y      | 
| A           | 2021-01-10 | ramen         | 12    | Y      | 
| A           | 2021-01-11 | ramen         | 12    | Y      | 
| A           | 2021-01-11 | ramen         | 12    | Y      | 
| B           | 2021-01-01 | curry         | 15    | N      | 
| B           | 2021-01-02 | curry         | 15    | N      | 
| B           | 2021-01-04 | sushi         | 10    | N      | 
| B           | 2021-01-11 | sushi         | 10    | Y      | 
| B           | 2021-01-16 | ramen         | 12    | Y      |
| B           | 2021-02-01 | ramen         | 12    | Y      | 
| C           | 2021-01-01 | ramen         | 12    | N      | 
| C           | 2021-01-01 | ramen         | 12    | N      | 
| C           | 2021-01-07 | ramen         | 12    | N      | 

- With this table, Danny can see if the customer was a member when the order was taken.

***

**Rank All The Things**

```sql
WITH sales_members AS(
  SELECT
  	s.customer_id,
  	s.order_date,
  	m.product_name,
  	m.price,
  	CASE
  		WHEN mem.join_date IS NOT NULL AND s.order_date >= mem.join_date THEN 'Y'
  		ELSE 'N'
  	END AS member
  FROM dannys_diner.sales AS s
  LEFT JOIN menu AS m ON s.product_id = m.product_id
  FULL JOIN members AS mem on s.customer_id = mem.customer_id
)
SELECT *,
	CASE
    	WHEN member = 'N' THEN NULL
        ELSE RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
    END AS ranking
FROM sales_members;
```

Result:

| customer_id | order_date  | product_name | price | member | ranking |
|-------------|-------------|---------------|-------|--------|---------|
| A           | 2021-01-01 | curry         | 15    | N      | null    |
| A           | 2021-01-01 | sushi         | 10    | N      | null    |
| A           | 2021-01-07 | curry         | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen         | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen         | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen         | 12    | Y      | 3       |
| B           | 2021-01-01 | curry         | 15    | N      | null    |
| B           | 2021-01-02 | curry         | 15    | N      | null    |
| B           | 2021-01-04 | sushi         | 10    | N      | null    |
| B           | 2021-01-11 | sushi         | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen         | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen         | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen         | 12    | N      | null    |
| C           | 2021-01-01 | ramen         | 12    | N      | null    |
| C           | 2021-01-07 | ramen         | 12    | N      | null    |

- With this table, Danny can see when the customer is a member when the order was taken and the ranking groups when orders were taken on the same day. 

***
