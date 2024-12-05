```

# Atliq_Delivery
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

```

# Total Products Sold

select count( l.product_id) as total_products_sold
 from fact_order_lines l inner join fact_orders_aggregate a
on a.order_id=l.order_id
where a.order_placement_date=l.order_placement_date;
```

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



```
# Category Summary

```

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



```
#Insights

1. There are a total of 31729 orders

2. There are a total of 57096 Products Sold

3. In Full % 52.28% and In Full Target % 76.51%

4. On Time % 59.03% and On Time Target % 86.09%

5. On Time In Full  % 29.02% and OTIF Target % 65.91%

```






