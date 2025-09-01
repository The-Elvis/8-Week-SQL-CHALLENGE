# 🍜 Danny's Diner
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="image" width="400" height="400">

## Table Of Contents
- [Task](#task)
- [Tool Used](#tool-used)
- [Entity Relationship Image](#entity-relationship-image)
- [Things to Note Moving Forward](#things-to-note-moving-forward)
- [Questions and Solution](#questions-and-solution)

**N.B:** All information regarding this case study was sourced from: [Here](https://8weeksqlchallenge.com/case-study-1/)

***

## Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

***

## Tool Used
- **SQL** (MySQL)

***

## Entity Relationship Image

<img src="https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png" alt="image2" width="400" height="400">

***

## Things to Note Moving Forward
In order to answer these questions and frankly most questions in this case study, we will be carrying out repeated joins making queries look long and repititive. To make this less repititive, it's advisable to create a temporary table with all the necessary joins, then query off your temp table. follow the steps below to create your temp table.

#### Steps:
- Use the **"JOIN"** funtion to merge the sales, members and menu table using the relationship described in the entity relationship image above.

````sql
SELECT *
FROM sales s 
LEFT JOIN members m 
ON s.customer_id = m.customer_id
JOIN menu n
ON s.product_id = n.product_id;
````

- Use the **"CREATE TEMPORARY TABLE table_name"** function to create a temp table, then add the same join query you have above to reproduce same result but this time, in a saved table(saved temporarily).
- Replace "table_name" with desired table name, in my case, I used "temp_t".

````sql
CREATE TEMPORARY TABLE temp_t
SELECT s.customer_id, order_date, s.product_id, product_name, price, join_date
FROM sales s 
LEFT JOIN members m 
ON s.customer_id = m.customer_id
JOIN menu n
ON s.product_id = n.product_id;
````

Now that we have a temp table with all the necessary joins, a simple query will answer most of the questions in this case study using our temp table. Moving forward, most queries will be from the temptable.

***

## Questions and Solution

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT Customer_id, SUM(price) AS Total_Money_Spent
FROM temp_t
GROUP BY customer_id 
ORDER BY 2 DESC;
````
**ANS:** Customer A spent $76, Customer B spent $74, and Customer C spent $36

**2. How many days has each customer visited the restaurant?**

````sql
SELECT Customer_id, COUNT(DISTINCT order_date) AS Total_Number_Of_Days_Visited
FROM temp_t
GROUP BY customer_id 
ORDER BY 2 DESC;
````
**ANS:** Customer A visited 6 days, B visited for 4 days while customer C only visited 2 days

**3. Study What was the first item from the menu purchased by each customer?**

This is best answered using Window Function(in this case Row_Number) inside of a CTE, then query off the CTE.

````sql
WITH first_order AS (
SELECT *,
ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) AS row_num
FROM temp_t
)
SELECT customer_id, product_name
FROM first_order
WHERE row_num = 1; -- answer
````
**ANS:** 
- A= Sushi 
- B= Curry
- C= Ramen

**4. What is the most purchased item on the menu?**

````sql
SELECT product_name, COUNT(product_name) Number_of_purchases
FROM temp_t
GROUP BY product_name
ORDER BY 2 DESC;
````
**ANS:** Ramen

**5. Which item was the most popular for each customer?**

Like question 3, this is best answered using Window Function(in this case Dense_Rank) inside of a CTE, then query off the CTE.

````sql
WITH popularity AS (
SELECT customer_id, product_name, COUNT(product_name) AS orders,
DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS rnk
FROM temp_t
GROUP BY customer_id, product_name
)
SELECT customer_id, product_name
FROM popularity
WHERE rnk = 1
````
**ANS:** Ramen for customer A, Ramen for C and the 3 items for customer B

**6. Which item was purchased first by the customer after they became a member?**

````sql
WITH item_after_join AS (
SELECT *,
ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) AS row_num
FROM temp_t
WHERE order_date >= join_date
)
SELECT customer_id, product_name
FROM item_after_join
WHERE row_num = 1;
````

**ANS:** 
- Customer A = Curry
- Customer B = Sushi

**7. Which item was purchased just before the customer became a member**

````sql
WITH item_after_join AS (
SELECT *,
ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date DESC) AS row_num
FROM temp_t
WHERE order_date < join_date
)
SELECT customer_id, product_name
FROM item_after_join
WHERE row_num = 1;
````
**ANS:** They (A and B) Both ordered Sushi before  they became a member

**8. What is the total Items and amount spent for each member before they became a member?**

````sql
SELECT customer_id, COUNT(product_name) AS total_items_BM, SUM(price) total_spent_BM
FROM temp_t
WHERE order_date < join_date
GROUP BY customer_id;
````
**ANS:** A bought 2 items, spent $25 while B bought 3 items, spent $40

**9. If each $1 spent equates 10 points and sushi  has 2x points multiplier, how many points would each customer have?**

````sql
SELECT customer_id, SUM(final_points) AS points_earned
FROM (SELECT customer_id, product_id, product_name, price, price * 10 AS base_points,
	CASE
		WHEN product_id = 1 THEN price * 10 * 2
		WHEN product_id != 1 THEN price * 10
	END AS final_points
	FROM temp_t 
		) AS pt_table
GROUP BY customer_id
ORDER BY 2 DESC;
````
**ANS:**
- B= 940 points
- A= 860 Points
- C= 360 points

**10. In the first week after a customer joins the program, they earn 2x points on all items, not just sushi, how many points do customer A and B have at the end of january?**

````sql
SELECT customer_id, SUM(final_points) AS points_earned
FROM (SELECT customer_id, product_id, product_name, price, price * 10 AS base_points,
CASE
	WHEN product_id = 1 THEN price * 10 * 2
	WHEN order_date >= join_date
	AND order_date < join_date + INTERVAL 7 DAY THEN price * 10 * 2
	ELSE price * 10
END AS final_points
FROM temp_t 
		) AS pt_table2
GROUP BY customer_id
ORDER BY 2 DESC;
````
**ANS:**
- A= 1,370 Points
- B= 940 points
- C= 360 points

**11. Bonus question create a table for customer_id, order_date, product_name, Price, and member Y/N**

I already have a temp table but let me create a table from scratch called JAT (Join All Things).

````sql
CREATE TABLE JAT
SELECT s.customer_id, order_date, product_name, price, 
CASE
	WHEN order_date >= join_date THEN 'Y'
	ELSE 'N'
END AS `member`
FROM sales s
JOIN menu n
ON s.product_id = n.product_id
LEFT JOIN members m
ON s.customer_id = m.customer_id
;


SELECT *
FROM JAT3;
````

**12. Bonus 2: Rank All The Things**

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

````sql
WITH customer_data AS (
SELECT s.customer_id, order_date, product_name, price, 
CASE
	WHEN order_date >= join_date THEN 'Y'
	ELSE 'N'
END AS member_status
FROM sales s
JOIN menu n
ON s.product_id = n.product_id
LEFT JOIN members m
ON s.customer_id = m.customer_id
)
SELECT *,
	CASE 
		WHEN member_status = 'N' THEN NULL
		ELSE DENSE_RANK() OVER (PARTITION BY customer_id, member_status ORDER BY order_date)
	END AS ranking
FROM customer_data;
````

- OR since we already have a table called JAT

````sql
SELECT *,
	CASE 
		WHEN `member` = 'N' THEN NULL
		ELSE DENSE_RANK() OVER (PARTITION BY customer_id, `member` ORDER BY order_date)
	END AS ranking
FROM JAT;
````





