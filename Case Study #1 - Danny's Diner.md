# 🍜 Danny's Diner
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="image" width="400" height="400">

## Table Of Contents
- [Task](#task)
- [Tool Used](#tool-used)
- [Entity Relationship Image](#entity-relationship-image)

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

## Things to note moving forward
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





