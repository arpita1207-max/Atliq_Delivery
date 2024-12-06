```

# Atliq_Delivery
```

```
Problem Statement

AtliQ Mart is a growing FMCG manufacturer headquartered in Gujarat, India. It is currently operational in three cities Surat, Ahmedabad and Vadodara. They want to expand to other metros/Tier 1 cities in the next 2 years.

AtliQ Mart is currently facing a problem where a few key customers did not extend their annual contracts due to service issues. It is speculated that some of the essential products were either not delivered on time or not delivered in full over a continued period, which could have resulted in bad customer service. Management wants to fix this issue before expanding to other cities and requested their supply chain analytics team to track the ’On time’ and ‘In Full’ delivery service level for all the customers daily basis so that they can respond swiftly to these issues.

The Supply Chain team decided to use a standard approach to measure the service level in which they will measure ‘On-time delivery (OT) %’, ‘In-full delivery (IF) %’, and OnTime in full (OTIF) %’ of the customer orders daily basis against the target service level set for each customer.

```
```
Important Terms:-

ADD       :- Actual Delivery Date

IF %        :-Order in Full

OT %      :-Order On Time

OTIF %   :-Order On Time In Full

VOFR % :- Total Quantity Shipped/Total Quantity Ordered

LOTR %  :- Line Order On Time Rate

LIFR %  :- Line Order In Full Rate

LOTIFR %  :- Line Order On Time In Full Rate

```

```
Tables Used:-

select * from dim_customers;
select * from dim_date;
select * from dim_products;
select * from dim_targets_orders;
select * from fact_order_lines;
select * from fact_orders_aggregate;

```

```
#Executive Screen:-

```
```

#Total Orders

select count(distinct a.order_id) as total_orders
 from fact_order_lines l inner join fact_orders_aggregate a
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date;
```
![Total Orders](https://github.com/user-attachments/assets/2c3babf6-aedc-44ea-b041-1c767d76f543)



```

# Total Products Sold

select count( l.product_id) as total_products_sold
 from fact_order_lines l inner join fact_orders_aggregate a
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date;
```

![Total Products Sold](https://github.com/user-attachments/assets/673aa70a-4eb0-49a7-b723-7b8731feaf7c)



```

# In Full% and InFull Target
select * from dim_targets_orders;
select round(sum(in_full)/sum(total)*100,2) as in_full_percent,round(avg(infull_target),2) as in_full_target
from (
select a.customer_id,count( distinct case when a.in_full=1 then a.order_id end) as in_full,
count( distinct a.order_id) as total
 from fact_order_lines l inner join fact_orders_aggregate a
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
group by a.customer_id)t inner join dim_targets_orders o
on o.customer_id=t.customer_id;
```

![in full percent](https://github.com/user-attachments/assets/b44a768c-2d8f-4648-9d5b-41c247e286cf)



```

# On Time% and On Time Target
select round(sum(on_time)/sum(total)*100,2) as on_time_percent,round(avg(ontime_target),2) as on_time_target
from (
select a.customer_id,count( distinct case when a.on_time=1 then a.order_id end) as on_time,
count( distinct a.order_id) as total
 from fact_order_lines l inner join fact_orders_aggregate a
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
group by a.customer_id)t inner join dim_targets_orders o
on o.customer_id=t.customer_id;
```

![on time](https://github.com/user-attachments/assets/1b3eb73c-d093-4455-9bd3-d1afc72f1b93)



```

# Otif% and Otif Target
select round(sum(otif)/sum(total)*100,2) as otif_percent,round(avg(otif_target),2) as otif_target
from (
select a.customer_id,count( distinct case when a.otif=1 then a.order_id end) as otif,
count( distinct a.order_id) as total
 from fact_order_lines l inner join fact_orders_aggregate a
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
group by a.customer_id)t inner join dim_targets_orders o
on o.customer_id=t.customer_id;
```

![otif](https://github.com/user-attachments/assets/3373ce49-2a3c-49fe-9da1-61b921cf2f3e)



```

# In Full,InFull Target ,On Time ,OnTime Target,Otif,Otif target by  city
select city,round((in_full)/(total)*100,2) in_full_percent,
round((on_time)/(total)*100,2) on_time_percent,
round((on_time_in_full)/(total)*100,2) otif_percent,
infull_target_percent,
ontime_target_percent,
otif_target_percent
 from  (
select c.city,count(distinct case when a.in_full=1 then a.order_id end) as in_full,
count(  distinct case when a.on_time=1 then  a.order_id end) as on_time,
count( distinct case when a.otif=1 then a.order_id end) as on_time_in_full,
count( distinct l.order_id) as total,
round(avg(infull_target),2) as infull_target_percent,
round(avg(ontime_target),2) as ontime_target_percent,
round(avg(otif_target),2) as otif_target_percent
from fact_order_lines l inner join fact_orders_aggregate a inner join dim_targets_orders t
inner join dim_customers c
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and t.customer_id=a.customer_id 
and c.customer_id=t.customer_id
group by c.city) t;
```

![by city](https://github.com/user-attachments/assets/84b3345e-9cfa-412b-ad62-d1fdf66d334a)




```

#In Full ,On Time and OTIF by quarter
select Quarter,round((in_full/total)*100,2) as in_full_percent,
round((on_time/total)*100,2) as on_time_percent,
round((on_time_in_full/total)*100,2) as otif_percent
from 
(select quarter(d.date) as Quarter,
count(  distinct case when a.in_full=1 then  a.order_id end) as in_full,
count(  distinct case when a.on_time=1 then  a.order_id end) as on_time,
count( distinct case when a.otif=1 then a.order_id end) as on_time_in_full,
count( distinct l.order_id) as total from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_date d
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and d.date=l.order_placement_date
group by quarter(d.date))t;
```
![quarter orders](https://github.com/user-attachments/assets/86ac1910-774d-49fb-9a31-29ddfe9d0fc1)



```

#In Full ,On Time and OTIF by month
select Month,round((in_full/total)*100,2) as in_full_percent,
round((on_time/total)*100,2) as on_time_percent,
round((on_time_in_full/total)*100,2) as otif_percent
from 
(select Month(d.date) as Month,
count(  distinct case when a.in_full=1 then  a.order_id end) as in_full,
count(  distinct case when a.on_time=1 then  a.order_id end) as on_time,
count( distinct case when a.otif=1 then a.order_id end) as on_time_in_full,
count( distinct l.order_id) as total from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_date d
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and d.date=l.order_placement_date
group by month(d.date))t;
```

![month order](https://github.com/user-attachments/assets/922bcd5b-0439-486a-a972-0724721d3241)



```

#In Full ,On Time and OTIF by week no
select Month,Week_No,round((in_full/total)*100,2) as in_full_percent,
round((on_time/total)*100,2) as on_time_percent,
round((on_time_in_full/total)*100,2) as otif_percent
from 
(select Month(d.date) as Month,week_no,
count(  distinct case when a.in_full=1 then  a.order_id end) as in_full,
count(  distinct case when a.on_time=1 then  a.order_id end) as on_time,
count( distinct case when a.otif=1 then a.order_id end) as on_time_in_full,
count( distinct l.order_id) as total from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_date d
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and d.date=l.order_placement_date
group by month(d.date),d.week_no)t;
```

![week order](https://github.com/user-attachments/assets/1f1f3c1b-bcb4-4cab-9bf4-f0c8d3ff7065)



```

# Unsastified Customers by infull

select * from (
select customer_name,in_full ,dense_rank() over(order by in_full asc) as in_full_rnk
#on_time,rank() over(order by on_time asc) as on_time_rnk,
#otif,rank() over(order by otif asc) as otif_rnk
from (
SELECT 
    a.customer_id,
	count( CASE WHEN a.in_full=1 THEN a.order_id END) AS in_full
FROM fact_orders_aggregate a
INNER JOIN fact_order_lines l 
    ON a.order_id = l.order_id
    AND a.order_placement_date = l.order_placement_date
    AND a.customer_id = l.customer_id
group by a.customer_id) t inner join  dim_customers c
on c.customer_id=t.customer_id) t where in_full_rnk=1 ;
```

![in full rnk](https://github.com/user-attachments/assets/eb3d8025-56ee-46da-bed4-f24487e29a96)




```

# Unsastified Customers by ontime

select * from (
select customer_name,
on_time,dense_rank() over(order by on_time asc) as on_time_rnk
from (
SELECT 
    a.customer_id,
	count( CASE WHEN a.on_time=1 THEN a.order_id END) AS on_time
FROM fact_orders_aggregate a
INNER JOIN fact_order_lines l 
    ON a.order_id = l.order_id
    AND a.order_placement_date = l.order_placement_date
    AND a.customer_id = l.customer_id
group by a.customer_id) t inner join  dim_customers c
on c.customer_id=t.customer_id) t where on_time_rnk=1 ;
```

![ontime rnk](https://github.com/user-attachments/assets/fc697a12-f6dd-4674-a910-c42d530224b8)




```

# Unsastified Customers by otif

select * from (
select customer_name,
otif,rank() over(order by otif asc) as otif_rnk
from (
SELECT 
    a.customer_id,
	count( CASE WHEN a.otif=1 THEN a.order_id END) AS otif
FROM fact_orders_aggregate a
INNER JOIN fact_order_lines l 
    ON a.order_id = l.order_id
    AND a.order_placement_date = l.order_placement_date
    AND a.customer_id = l.customer_id
group by a.customer_id) t inner join  dim_customers c
on c.customer_id=t.customer_id )t where otif_rnk= 1 ;
```

![otif rnk](https://github.com/user-attachments/assets/5441e591-50bc-4a60-b785-79d6d949077f)





```
# Category Summary

```
```

# Line In full percent
select
round(count(   case when l.in_full=1 then  a.order_id end) /count(  l.order_id)*100,2) as lifr_percent from 
fact_orders_aggregate a inner join fact_order_lines l
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id;
```

![lifr_percent](https://github.com/user-attachments/assets/aa953a2a-999a-4e28-81cb-554506fda91d)

```

# Line On time percent
select
round(count(   case when l.on_time=1 then  a.order_id end) /count(  l.order_id)*100,2) as lotr_percent from 
fact_orders_aggregate a inner join fact_order_lines l
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id;
```

![lotr_percent](https://github.com/user-attachments/assets/5c6a76d9-898d-424c-ad53-129266fb4162)

```

# Line On time In full percent
select
round(count(   case when l.on_time_in_full=1 then  a.order_id end) /count(  l.order_id)*100,2) as lotif_percent from 
fact_orders_aggregate a inner join fact_order_lines l
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id;
```

![lotif_percent](https://github.com/user-attachments/assets/15606d40-b287-4211-8251-d9e6ca81dbc6)

```

#vofr percent
select
round(sum(l.delivery_qty) /sum(l.order_qty)*100,2) as vofr_percent from 
fact_orders_aggregate a inner join fact_order_lines l
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id;
```
![vofr](https://github.com/user-attachments/assets/b1a4d77b-946b-4e9d-abbf-dc69e53adc9b)

```

#lifr,lotr,lotif by quarter

select quarter,round(((line_in_full)/(total_prod_sold))*100,2) as lifr_percent,
round(((line_on_time)/(total_prod_sold))*100,2) as lotr_percent,
round(((line_otif)/(total_prod_sold))*100,2) as lotif_percent
from 
(select quarter(c.date) as quarter,
count(   case when l.in_full=1 then  a.order_id end) as line_in_full,
count(   case when l.on_time=1 then  a.order_id end) as line_on_time,
count(  case when l.on_time_in_full=1 then a.order_id end) as line_otif,
count(  l.order_id) as total_prod_sold from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_date c
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and c.date=l.order_placement_date
group by quarter(c.date)) t;
```

![ifr by quar](https://github.com/user-attachments/assets/ef1e0a0d-2bdc-403f-ad68-b3830b885606)



```

#lifr,lotr,lotif by month

select month,round(((line_in_full)/(total_prod_sold))*100,2) as lifr_percent,
round(((line_on_time)/(total_prod_sold))*100,2) as lotr_percent,
round(((line_otif)/(total_prod_sold))*100,2) as lotif_percent
from 
(select monthname(c.date) as month,
count(   case when l.in_full=1 then  a.order_id end) as line_in_full,
count(   case when l.on_time=1 then  a.order_id end) as line_on_time,
count(  case when l.on_time_in_full=1 then a.order_id end) as line_otif,
count(  l.order_id) as total_prod_sold from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_date c
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and c.date=l.order_placement_date
group by monthname(c.date)) t;
```

![prod line month](https://github.com/user-attachments/assets/1bba9562-b060-4d45-8061-729b449fa6aa)



```

#lifr,lotr,lotif by week_no

select week_no,round(((line_in_full)/(total_prod_sold))*100,2) as lifr_percent,
round(((line_on_time)/(total_prod_sold))*100,2) as lotr_percent,
round(((line_otif)/(total_prod_sold))*100,2) as lotif_percent
from 
(select week_no,
count(   case when l.in_full=1 then  a.order_id end) as line_in_full,
count(   case when l.on_time=1 then  a.order_id end) as line_on_time,
count(  case when l.on_time_in_full=1 then a.order_id end) as line_otif,
count(  l.order_id) as total_prod_sold from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_date c
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and c.date=l.order_placement_date
group by c.week_no) t;
```

![prod line week ](https://github.com/user-attachments/assets/111a034b-9fb8-4340-84fa-b1e47a43db73)



```
lifr,lofr,lotif by category
select category,round(((line_in_full)/(total_prod_sold))*100,2) as lifr_percent,
round(((line_on_time)/(total_prod_sold))*100,2) as lotr_percent,
round(((line_otif)/(total_prod_sold))*100,2) as lotif_percent
from 
(select c.category,
count(   case when l.in_full=1 then  a.order_id end) as line_in_full,
count(   case when l.on_time=1 then  a.order_id end) as line_on_time,
count(  case when l.on_time_in_full=1 then a.order_id end) as line_otif,
count(  l.order_id) as total_prod_sold from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_products c
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and c.product_id=l.product_id
group by c.category) t;
```

![categ line](https://github.com/user-attachments/assets/d4b6ed9a-f5d3-4426-9ca2-39df210d5ee3)




```

#Detailed Screen

```
```

#lifr,lotr,lotif by product_name,category 
select product_name,category,round(((line_in_full)/(total_prod_sold))*100,2) as lifr_percent,
round(((line_on_time)/(total_prod_sold))*100,2) as lotr_percent,
round(((line_otif)/(total_prod_sold))*100,2) as lotif_percent,
round((vofr*100),2) as vofr_percent,
total_prod_sold
from 
(select c.product_name,c.category,
count(   case when l.in_full=1 then  a.order_id end) as line_in_full,
count(   case when l.on_time=1 then  a.order_id end) as line_on_time,
count(  case when l.on_time_in_full=1 then a.order_id end) as line_otif,
sum(delivery_qty)/sum(order_qty) as vofr,
count(  l.order_id) as total_prod_sold from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_products c
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and c.product_id=l.product_id
group by c.product_name,c.category) t
order by total_prod_sold desc;
```

![prod and category by line](https://github.com/user-attachments/assets/39564101-2cc6-414e-b07e-4584ebbff6f8)



```

# infull,ontime,otifby customer name
select customer_name,round(((in_full)/(total_orders))*100,2) as in_full_percent,
round(((on_time)/(total_orders))*100,2) as on_time_percent,
round(((otif)/(total_orders))*100,2) as otif_percent,
round(((in_full)/(total_orders))*100,2)-round((infull_target),2) as diff_infull_percent,
round(((on_time)/(total_orders))*100,2)-round((ontime_target),2) as diff_ontime_percent,
round(((otif)/(total_orders))*100,2)-round((otif_target),2) as diff_otif_percent,
total_orders
from 
(select c.customer_name,
count(  distinct case when a.in_full=1 then  a.order_id end) as in_full,
count( distinct  case when a.on_time=1 then  a.order_id end) as on_time,
count( distinct case when a.otif=1 then a.order_id end) as otif,
avg(infull_target) as infull_target,
avg(ontime_target)  as ontime_target,
avg(otif_target) as otif_target,
count( distinct l.order_id) as total_orders from 
fact_orders_aggregate a inner join fact_order_lines l inner join dim_customers c inner join dim_targets_orders t
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date
and a.customer_id=l.customer_id
and t.customer_id=l.customer_id
and c.customer_id=l.customer_id
group by c.customer_name) t
order by total_orders desc;
```
![customer by line](https://github.com/user-attachments/assets/365a30ea-83fb-4e56-8e66-11b2b3bb46f4)





```
#Insights

1. There are a total of 31729 orders

2. There are a total of 57096 Products Sold

3. In Full % 52.28% and In Full Target % 76.51%

4. On Time % 59.03% and On Time Target % 86.09%

5. On Time In Full  % 29.02% and OTIF Target % 65.91%

6. Line In Full % 65.96%.

7. Line On Time % 71.12%.

8. Line On Time In Full % 47.95%.

9. VOFR % 96.59%

10. IF % and OTIF% is highest in Q3(From Month July to Sep) 
   Quarter	Line On Time % 	Line On Time %	Line On Time In Full %
     1	         65.79	                71.45	        48.00
     2	         65.93	                70.94	        47.76
     3	         66.09	                71.22	        48.22


11.Most Unsatisified Customer by IF% is Elite Mart.Most Unsatisified Customer by OT% is Lotus Mart,Acclaimed Stores.Most Unsatisified Customer by OTIF% is Acclaimed Stores.



12. 12.1 Lotus Mart customer has placed highest order 3550 and their is a  delay in between the
       Actual IF%  53.35 and and IF% target 75.38.It missed in mark by 22.03.
       Actual OT%  28.11 and and OT% target 77.3.It missed in mark by 49.19.
       Actual OTIF%  16.34 and and OTIF% target 57.97.It missed in mark by 41.63.

  12.2 Acclaimed Stores customer has placed highest order 3510 and their is a  delay in between the
       Actual IF%  52.36 and and IF% target 75.41.It missed in mark by -23.05.
       Actual OT%  29.43 and and OT% target 76.35.It missed in mark by -46.92.
       Actual OTIF%  15.47 and and OTIF% target 57.74.It missed in mark by -42.27.


13. Food Items has the highest percentage by IF%,OT% and OTIF%.
Category	    lifr_percent	    lotr_percent        lotif_percent
Dairy	                   65.95	            70.81	47.83
Food	                   66.43	            72.07	48.84
beverages	           65.54	            71.40	47.55


14. Top 3 Products by Total Product Sold.VOFR %(delivery_qty/order_qty) is aroud 96% for below three products.
Product 	        Category	lifr_percent	lotr_percent	lotif_percent	vofr_percent	total_prod_sold
AM Butter 500	        Dairy	                65.19	           70.39	46.39	      96.46	3272
AM Ghee 150	        Dairy	                66.72	           69.87	47.93	      96.69	3209
AM Ghee 250	        Dairy	                65.25	           69.59	46.94	      96.53	3200






```






