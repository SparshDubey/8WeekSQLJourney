USE dannys_diner;
#Q1) What is the total amount each customer spent at the restaurant?
SELECT
	s.customer_id,
	SUM(m.price) AS Total_Amount
FROM
	sales s
    	JOIN
        menu m ON s.product_id = m.product_id
GROUP BY(s.customer_id);

#Q2) How many days has each customer visited the restaurant?
SELECT
	s.customer_id, 
	COUNT(DISTINCT s.order_date) AS Visit_Count
FROM
	sales s
GROUP BY(s.customer_id);

#Q3)What was the first item from the menu purchased by each customer?
SELECT 
	s.customer_id, 
	MIN(s.order_date) AS First_OrderDate,
    m.product_name AS First_ProductOrdered
FROM
	sales s
		JOIN
        menu m ON s.product_id=m.product_id
GROUP BY s.customer_id;

#Q4) What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT 
	m.product_name,
    COUNT(s.product_id)
FROM
	sales s
		JOIN
        menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY COUNT(s.product_id) DESC
LIMIT 1;

#Q5) Which item was the most popular for each customer?

SELECT 
	a.customer_id,
    a.product_name
FROM
	(SELECT s.customer_id,m.product_name, RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id)DESC) AS r 
    FROM sales s JOIN menu m USING(product_id) 
    GROUP BY s.customer_id,m.product_name) a
WHERE r=1;

#Q6) Which item was purchased first by the customer after they became a member?
SELECT 
	me.customer_id,
    m.product_name,
    s.order_date,
    me.join_date
FROM
	members me 
		JOIN
        sales s ON s.customer_id=me.customer_id
        JOIN
        menu m ON s.product_id=m.product_id
WHERE s.order_date >= me.join_date
GROUP BY me.customer_id
ORDER BY me.customer_id,s.order_date;

#Q7) Which item was purchased just before the customer became a member?
SELECT 
	a.customer_id,
    a.product_name,
    a.order_date,
    a.join_date
FROM
	(SELECT me.customer_id,
			m.product_name,
            s.order_date,
            me.join_date,
            RANK() OVER (PARTITION BY me.customer_id ORDER BY s.order_date DESC) as r
	 FROM
		members me 
			JOIN
				sales s ON me.customer_id=s.customer_id
			JOIN
				menu m ON s.product_id=m.product_id
	 WHERE s.order_date<me.join_date) a
WHERE r=1;

#Q8) What is the total items and amount spent for each member before they became a member?

SELECT 
	me.customer_id as Customer_ID,
    COUNT(s.product_id) as Total_Items,
    SUM(m.price) as Total_Amt
FROM
	members me
		JOIN
        sales s ON me.customer_id=s.customer_id
		JOIN
        menu m ON m.product_id=s.product_id
WHERE 
	s.order_date<me.join_date
GROUP BY me.customer_id
ORDER BY s.customer_id;

#Q9) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

SELECT 
	s.customer_id,
    SUM(CASE 
		WHEN s.product_id='1' THEN m.price*20
        ELSE m.price*10 END) as Points
FROM
	sales s 
		JOIN 
        menu m ON m.product_id=s.product_id
GROUP BY s.customer_id;

#Q10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, 
#not just sushi - how many points do customer A and B have at the end of January?

SELECT 
	s.customer_id,
    SUM(CASE
			WHEN s.order_date<me.join_date OR s.order_date> DATE_ADD(me.join_date,INTERVAL 6 DAY) THEN
				CASE 
					WHEN s.product_id='1' THEN m.price*20
					ELSE m.price*10 
				END
            WHEN s.order_date>=me.join_date AND s.order_date<= DATE_ADD(me.join_date, INTERVAL 6 DAY) THEN       
                m.price*20 END) as Points
FROM
	sales s 
		JOIN 
        menu m ON m.product_id=s.product_id
        JOIN
			members me ON me.customer_id=s.customer_id
WHERE
	s.order_date<'2021-02-01'
GROUP BY s.customer_id
ORDER BY s.customer_id;

#BONUS QUESTION: JOIN ALL THE THINGS

SELECT
	s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
		WHEN s.customer_id=me.customer_id AND s.order_date>=me.join_date THEN 'Y'
        ELSE 'N'
	END as Is_Member
FROM
	sales s
		LEFT JOIN
        members me ON s.customer_id=me.customer_id
        JOIN
			menu m ON s.product_id=m.product_id;
    
    
#BONUS QUESTION: RANK ALL THE THINGS

WITH cte1 AS 
(SELECT 
	s.customer_id,
    s.order_date,
    m.product_name,
    m.price,
    CASE
		WHEN s.customer_id=me.customer_id AND s.order_date>=me.join_date THEN 'Y'
        ELSE 'N'
	END as Is_Member
FROM
	sales s
		LEFT JOIN
        members me ON s.customer_id=me.customer_id
        JOIN
			menu m ON s.product_id=m.product_id)
SELECT 
	c.*,
    CASE
		WHEN Is_member = 'Y' THEN
			RANK() OVER(PARTITION BY c.customer_id,c.Is_member ORDER BY c.order_date)
		ELSE
			NULL
	END AS Ranking
FROM
	cte1 c;
            


				



	
