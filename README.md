# Assessment_MKyriakopoulou

**PART I**

**First I run some queries in order to check disctriptive numbers and get a better understanding of my data ** 

SELECT  city,
        cuisine_parent, 
        COUNT (DISTINCT order_id ) as unique_orders, 
        COUNT (DISTINCT shop_id) as unique_shops,
        ROUND (sum(basket)) as total_revenue


FROM `bi-2019-test.ad_hoc.orders_jan2021` 

WHERE city IN 
  
    (
        SELECT city
    FROM  `bi-2019-test.ad_hoc.orders_jan2021` 
    GROUP BY city
    HAVING  COUNT(DISTINCT order_id) >500  
    ORDER BY city 
    )


    GROUP BY city,
           cuisine_parent

ORDER BY city,
        cuisine_parent
        
        
        
 **same for figures by cuisine**
 
 SELECT  cuisine_parent, 
        COUNT (DISTINCT order_id ) as unique_orders, 
        COUNT (DISTINCT shop_id) as unique_shops,
        COUNT (DISTINCT user_id) as unique_users,
        ROUND (sum(basket)) as total_revenue


FROM `bi-2019-test.ad_hoc.orders_jan2021` 

WHERE city IN 
  
(
SELECT city
FROM  `bi-2019-test.ad_hoc.orders_jan2021` 
GROUP BY city
HAVING  COUNT(DISTINCT order_id) >500  
ORDER BY city
)


    GROUP BY cuisine_parent


ORDER BY cuisine_parent

        
        
**how many orders are placed in the "Breakfast" cuisine (Breakfast_Orders)? **
**how many users ordered from the "Breakfast" cuisine (Breakfast_Users)? **
       
 SELECT  city,
        cuisine_parent, 
        COUNT (DISTINCT order_id ) as Breakfast_Orders, 
        COUNT (DISTINCT shop_id) as Breakfst_Shops,
        COUNT (DISTINCT user_id) as Breakfast_Users,
        ROUND (sum(basket)) as total_revenue


FROM `bi-2019-test.ad_hoc.orders_jan2021` 

WHERE city IN 
  
(
SELECT city
FROM  `bi-2019-test.ad_hoc.orders_jan2021` 
GROUP BY city
HAVING  COUNT(DISTINCT order_id) >500  
ORDER BY city 
)

AND  cuisine_parent = "Breakfast"

GROUP BY city,cuisine_parent


ORDER BY city
  
        

**what is the Average Basket  of the Breakfast_Users that we defined before?**


SELECT city,
       ROUND(AVG(breakfast_users_revenue),0) as breakfast_user_average_basket
       

FROM 


        (
            SELECT  city,
                    user_id as breakfast_user_ids,
                    round(basket,0) as breakfast_users_revenue

        FROM  `bi-2019-test.ad_hoc.orders_jan2021`  a
        WHERE cuisine_parent ='Breakfast'

        
AND city IN 

        (
            SELECT city
        FROM  `bi-2019-test.ad_hoc.orders_jan2021` 
        GROUP BY city
        HAVING  COUNT(DISTINCT order_id) >500  
        ORDER BY city )
)
group by city

order by city




**PART II**

**I CREATE Quintiles for Order Value and Frequency just so I can  unerstand the data

SELECT b.brand,
        Sum(users_per_quintile) as quintile_users,
        b.Quintile,
        max(b.total_revenue) as max,
        min(b.total_revenue) as min

FROM 
         
         (
            Select a.brand,
                    COUNT (DISTINCT a.user_id) as users_per_quintile,
                    NTILE(5) over (ORDER BY a.total_revenue DESC) As Quintile,
                    a.total_revenue 
                
                    

            FROM    
                
                
                (
                    SELECT  brand,
                            user_id,
                            COUNT (DISTINCT order_id ) as total_orders,
                            ROUND (sum(basket)) as total_revenue 


                FROM `bi-2019-test.ad_hoc.orders_jan2021` 

                GROUP BY    brand, 
                            user_id
                Order BY total_orders DESC 

      

                ) as a


            Group By a.brand, a.total_revenue

            ORDER BY Quintile DESC 

         ) as b


         Group By b.brand, Quintile

         Order by Quintile
         
   ** SAME for USERS**

SELECT b.brand,
        Sum(users_per_quintile) as quintile_users,
        b.Quintile,
        max(b.total_revenue) as max,
        min(b.total_revenue) as min

FROM 
         
         (
            Select a.brand,
                    COUNT (DISTINCT a.user_id) as users_per_quintile,
                    NTILE(5) over (ORDER BY a.total_revenue DESC) As Quintile,
                    a.total_revenue 
                
                    

            FROM    
                
                
                (
                    SELECT  brand,
                            user_id,
                            COUNT (DISTINCT order_id ) as total_orders,
                            ROUND (sum(basket)) as total_revenue 


                FROM `bi-2019-test.ad_hoc.orders_jan2021` 

                GROUP BY    brand, 
                            user_id
                Order BY total_orders DESC 

      

                ) as a


            Group By a.brand, a.total_revenue

            ORDER BY Quintile DESC 

         ) as b


         Group By b.brand, Quintile

         Order by Quintile
    
    
    
  **We need to segment existing customers based on their frequency and order value. 
  
  **FOR ORDR VALUE***
  
  SELECT a.brand,
       Count (DISTINCT a.user_id) as total_users,
    CASE 
        WHEN a.total_revenue <=5 THEN "1.[0-5]€" 
        WHEN a.total_revenue >5 AND a.total_revenue <=10 THEN  "2.[6-10]€"
        WHEN a.total_revenue >10 AND a.total_revenue <=30 THEN  "3.[11-30]€"
        WHEN a.total_revenue >30 AND a.total_revenue <=50 THEN  "4.[31-50]€"
        WHEN a.total_revenue >50 THEN "5.[51+)€"
        END AS revenue_range

        FROM 
        

(
                    SELECT  brand,
                            user_id,
                            COUNT (DISTINCT order_id ) as total_orders,
                            ROUND (sum(basket)) as total_revenue 


                FROM `bi-2019-test.ad_hoc.orders_jan2021` 

                GROUP BY    brand, 
                            user_id
                Order BY total_orders DESC 

      

    
) as a

GROUP BY a.brand, revenue_range

ORDER BY revenue_range


**FOR FREQUENCY**

SELECT a.brand,
       Count (DISTINCT a.user_id) as total_users,
    CASE 
        WHEN a.total_orders =1 THEN "1.1 order" 
        WHEN a.total_orders >=2 AND a.total_orders <=5 THEN  "2.2-5 orders"
        WHEN a.total_orders >=6 AND a.total_orders <=10 THEN  "3.6-10 orders"
        WHEN a.total_orders >10 THEN "5.10+ orders"
        END AS order_freq

        FROM 
        

(
                    SELECT  brand,
                            user_id,
                            COUNT (DISTINCT order_id ) as total_orders,
                            ROUND (sum(basket)) as total_revenue 


                FROM `bi-2019-test.ad_hoc.orders_jan2021` 

                GROUP BY    brand, 
                            user_id
                Order BY total_orders DESC 

      

    
) as a

GROUP BY a.brand, order_freq

ORDER BY order_freq

**BOTH AT THE SAME TIME, FOR crosstabs**

SELECT a.brand,
       Count (DISTINCT a.user_id) as total_users,
    CASE 
        WHEN a.total_orders =1 THEN "1.1 order" 
        WHEN a.total_orders >=2 AND a.total_orders <=5 THEN  "2.2-5 orders"
        WHEN a.total_orders >=6 AND a.total_orders <=10 THEN  "3.6-10 orders"
        WHEN a.total_orders >10 THEN "5.10+ orders"
        END AS order_freq,
     CASE 
        WHEN a.total_revenue <=5 THEN "1.[0-5]€" 
        WHEN a.total_revenue >5 AND a.total_revenue <=10 THEN  "2.[6-10]€"
        WHEN a.total_revenue >10 AND a.total_revenue <=30 THEN  "3.[11-30]€"
        WHEN a.total_revenue >30 AND a.total_revenue <=50 THEN  "4.[31-50]€"
        WHEN a.total_revenue >50 THEN "5.[51+)€"
        END AS revenue_range

        FROM 
        

(
                    SELECT  brand,
                            user_id,
                            COUNT (DISTINCT order_id ) as total_orders,
                            ROUND (sum(basket)) as total_revenue 


                FROM `bi-2019-test.ad_hoc.orders_jan2021` 

                GROUP BY    brand, 
                            user_id
                Order BY total_orders DESC 

      

    
) as a

GROUP BY a.brand, order_freq, revenue_range

ORDER BY order_freq, revenue_range




**Which segment could be a valuable target group for a Marketing campaign about Coffee **

**ΤΗΕ code above - Crosstab - just for Breakfast Cuisine


SELECT a.brand,
       Count (DISTINCT a.user_id) as total_users,
    CASE 
        WHEN a.total_orders =1 THEN "1.1 order" 
        WHEN a.total_orders >=2 AND a.total_orders <=5 THEN  "2.2-5 orders"
        WHEN a.total_orders >=6 AND a.total_orders <=10 THEN  "3.6-10 orders"
        WHEN a.total_orders >10 THEN "5.10+ orders"
        END AS order_freq,
     CASE 
        WHEN a.total_revenue <=5 THEN "1.[0-5]€" 
        WHEN a.total_revenue >5 AND a.total_revenue <=10 THEN  "2.[6-10]€"
        WHEN a.total_revenue >10 AND a.total_revenue <=30 THEN  "3.[11-30]€"
        WHEN a.total_revenue >30 AND a.total_revenue <=50 THEN  "4.[31-50]€"
        WHEN a.total_revenue >50 THEN "5.[51+)€"
        END AS revenue_range

        FROM 
        

(
                    SELECT  brand,
                            user_id,
                            COUNT (DISTINCT order_id ) as total_orders,
                            ROUND (sum(basket)) as total_revenue 


                FROM `bi-2019-test.ad_hoc.orders_jan2021` 

                GROUP BY    brand, 
                            user_id
                Order BY total_orders DESC 

      WHERE cuisine_parent = "Breakfast"

    
) as a

GROUP BY a.brand, order_freq, revenue_range

ORDER BY order_freq, revenue_range
    

**Get time of date for Breakfast


select EXTRACT(HOUR FROM submit_dt) as breakfast_hour,
        count (Distinct order_id) as breakfast_orders,
        
from `bi-2019-test.ad_hoc.orders_jan2021`
WHERE cuisine_parent = "Breakfast"

Group by breakfast_hour
Order by breakfast_orders Desc
