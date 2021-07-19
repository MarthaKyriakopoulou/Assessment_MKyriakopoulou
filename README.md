# Assessment_MKyriakopoulou

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
        
**how many orders are placed in the "Breakfast" cuisine (Breakfast_Orders)? **
       
       
        
        
