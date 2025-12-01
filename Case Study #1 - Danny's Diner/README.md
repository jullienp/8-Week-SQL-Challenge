# Case Study #1 - Danny's Diner

For the full description for this case study, please visit: [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)

## Table of Contents

## Introduction

Danny is a lover of Japanese food so he has decided to open a restaurant which sells his 3 favourite foods: sushi, curry and ramen. 

### Problem Statement

Danny wants to use the data to find out more about his customers and answer a few questions about their visiting patterns, how much money they’ve spent and which items on the menu are their favourites. He plans to use these insights to expand the existing customer loyalty program.

## Datasets

Danny has shared 3 key datasets for this case study.

- Table 1: `sales`

<details>

| customer_id | order_date | product_id |
| --- | --- | --- |
| A | 2021-01-01 | 1 |

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

- Table 2: `menu`

<details>

| product_id | product_name | price |
|------------|--------------|-------|
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

</details>

- Table 3: `members`

<details>

| customer_id | join_date   |
|-------------|-------------|
| A           | 2021-01-07  |
| B           | 2021-01-09  |

</details>

## Case Study Questions and Solutions

1. What is the total amount each customer spent at the restaurant?

Solution:

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

- `JOIN` to combine tables `sales` and `menu` together and get the price of the items each customer purchased
- `SUM` to calculate the total amount each customer has spent
- `GROUP BY` each customer

Answer:

|customer_id | total_amount_spent |
|---|---|
| A | 76 |
| B | 74 |
| C | 36 |

Total amount spent by each customer:
- Customer A - 76
- Customer B - 74
- Customer C - 36

2. How many days has each customer visited the restaurant?

3. What was the first item from the menu purchased by each customer?

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

5. Which item was the most popular for each customer?

6. Which item was purchased first by the customer after they became a member?

7. Which item was purchased just before the customer became a member?

8. What is the total items and amount spent for each member before they became a member?

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

## Reflection