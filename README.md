# Supply Chain Analysis
 
 
![supplychain](https://user-images.githubusercontent.com/87641079/197320730-031bcb8b-d77a-484c-af7f-cb3df8f5aa84.png)




## WEEK 1 - Danny's Dinner
	
This challenge needs us to determine customer movements with 10 different questions. You may find my approaches to these challenges below. But first let's summarize the challenge description. You can find the link to the challenge [here](https://8weeksqlchallenge.com/case-study-1/).
	
	
## Challenge Description
	
We have 3 tables from a restaurant database: sales, menu and members. Each table includes specific information (see diagram below). Our mission is to assist the restaurant about customers visiting patterns, how much they have spent and their favourite items.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

	
<img src = 'https://miro.medium.com/max/595/1*fEmZXjnIof5BHL_sLGDVUg.png' >
<p>

	
You may find the .sql file that creates the tables in the repository (MySQL).

## Challanges and Solutions

You may find my solutions in MySQL and my intuition on them with some comments.  


**Q1) Count of Top 50 Products which has maximum order**

Code:
	
``` sql
SELECT 	 
	  product_id, 
    SUM(unit_quantity) AS total_orders
FROM orderlist
GROUP BY product_id
ORDER BY total_orders DESC
LIMIT 50;
```
**Output**

![1](https://user-images.githubusercontent.com/87641079/196190917-cb5463a5-7972-4da2-89d7-831ea3c06012.png)


**Q2) Sum of quantity ordered by each customer**
	
Code:
	
	
``` sql
SELECT 
	  customer, 
    SUM(unit_quantity) AS total_quantity
FROM orderlist
GROUP BY  customer
ORDER BY total_quantity DESC;
```
**Output**

![2](https://user-images.githubusercontent.com/87641079/196192433-6ed9f5df-ef0e-401d-b722-21058ec9d3f5.png)

**Q3) Origin & destination port of each customer**
	
Code: 
	
```sql
SELECT 
	DISTINCT(customer),
    origin_port,
    destination_port,
    plant_code
FROM orderlist
ORDER BY customer;
```
**Output**

![3](https://user-images.githubusercontent.com/87641079/196192522-c4ef5125-2c82-42d7-a099-34f56b33e4f9.png)

**Q4) List of Customers with respective product ids where the early delivery is more then 3**
	
Code:  
	
```sql
SELECT
	  customer,
    order_id,
    product_id,
    ship_ahead_day_count
FROM orderlist
WHERE ship_ahead_day_count > 3 
ORDER BY order_id;
```
**Output**

![4](https://user-images.githubusercontent.com/87641079/196192581-c87b0645-8f1d-4ff9-8481-efd149f5eb2d.png)

**Q5) List the product with total orders & quantity and average weight**
	
Code: 

```sql
SELECT
	  product_id,
    COUNT(product_id) AS total_orders,
    SUM(unit_quantity) AS total_units,
    ROUND(AVG(weight),0) AS avg_weight
FROM orderlist
GROUP BY product_id
ORDER BY avg_weight DESC;
```
**Output**

![5](https://user-images.githubusercontent.com/87641079/196192635-c3947b86-aa47-4eec-9516-53d977d76944.png)

**Q6) List of ports and plants used for customers**
	
Code: 

```sql
SELECT
	  DISTINCT(o.customer),
    pl.port,
    pl.plant_code
FROM orderlist o
JOIN plantports pl
ON o.plant_code = pl.plant_code
ORDER BY o.customer;
```
**Output**

![6](https://user-images.githubusercontent.com/87641079/196192686-ab25fb94-5e8d-4536-91c3-ff4fe874f79a.png)

**Q7) List the plants which are operating at under & over capacity**
	
Code: 

````sql
SELECT o.plant_code,
	  CASE 
    WHEN o.unit_quantity > cap.daily_capacity THEN "over capacity"
    ELSE "under capacity"
    END AS capacity
FROM orderlist o
JOIN whcapacities cap
ON o.plant_code = cap.plant_id
ORDER BY capacity;
````
**Output**

![7](https://user-images.githubusercontent.com/87641079/196192738-b1c1ed21-5baf-4048-942e-521b0557c74d.png)

**Q8) Count of plants which are operating at over-capacity**

Code: 

````sql
SELECT
	a.plant_code, 
	count(a.order_id),
	b.daily_capacity, 
	(b.daily_capacity - count(a.order_id)) as diff
FROM orderlist as a
INNER JOIN whcapacities as b
ON a.plant_code = b.plant_ID
GROUP BY a.plant_code,b.daily_capacity;


SELECT 	count(capacity)
FROM 	(SELECT *,
	        Case 
	          When diff < 0 Then 'over-capacity'
		  Else 'under-capacity'
		  END capacity
	 FROM capacity
	 ORDER BY diff) as sub
WHERE capacity = 'over-capacity';
````
**Output**

![8](https://user-images.githubusercontent.com/87641079/196192776-f59841af-b161-41af-86ea-01a3bbaf21e3.png)
	
**Q9) Count of plants which are operating at under-capacity**
	
Code:
	
````sql
SELECT
	a.plant_code, 
	count(a.order_id),
	b.daily_capacity, 
	(b.daily_capacity - count(a.order_id)) as diff
FROM orderlist as a
INNER JOIN whcapacities as b
ON a.plant_code = b.plant_ID
GROUP BY a.plant_code,b.daily_capacity;


SELECT 	count(capacity)
FROM 	(SELECT *,
	        Case 
	          When diff < 0 Then 'over-capacity'
		  Else 'under-capacity'
		  END capacity
	 FROM capacity
	 ORDER BY diff) as sub
WHERE capacity = 'under-capacity';
````

**Output**

![9](https://user-images.githubusercontent.com/87641079/196192873-f3ac7195-71ce-4950-8640-169b55966d99.png)


**Question-10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

Code:  

````sql
SELECT customer_id, SUM(total_points)
FROM 
(WITH points AS
(
SELECT s.customer_id, 
       (s.order_date - mem.join_date) AS first_week,
       m.price,
       m.product_name,
       s.order_date
FROM sales s
JOIN menu m 
ON s.product_id = m.product_id
JOIN members AS mem
ON mem.customer_id = s.customer_id
)
SELECT customer_id,
       order_date,
       CASE 
       WHEN first_week BETWEEN 0 AND 7 THEN price * 20
       WHEN (first_week > 7 OR first_week < 0) AND product_name = 'sushi' THEN price * 20
       WHEN (first_week > 7 OR first_week < 0) AND product_name != 'sushi' THEN price * 10
       END AS total_points
FROM points
WHERE EXTRACT(MONTH FROM order_date) = 1
) as t
GROUP BY customer_id;
````
**Output**

![10](https://user-images.githubusercontent.com/87641079/196192929-04f5358e-884c-462f-aa3d-c5c6ca46cacf.png)
						       
