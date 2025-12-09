# SQL Project: Data Analysis for Zomato - A Food Delivery Company 
## overview 
This project Demonstrate my SQL problem_solving querry through analysis data for zomato , a popular food delivery company in India. the project involves setting up the database ,importing data, handling null values and solving verifying of buisness problems using complex queries. 
## Project Structure
- **Database Setup:** Create of the 'Zomato_project' database
- **Data Import:** inserting sample data throught mysqlconnector and create table
- **Data clining"** handle null values and remove false columns in data
- **Buisness problems:** Solving 20 specific buisness problems using sql queries

## lets see in below that 20 question and queries
### 1. calculate the total revenue of each restaurants 
``` sql
pd.read_sql_query(" select r.restaurant_id ,r.name,sum(o.total_amount) as total_revenue " \
"from orders o inner join restaurants r on o.restaurant_id = r.restaurant_id " \
"group by restaurant_id, r.name" \
" order by total_revenue desc;",engine)
```
### 2) give list of top 10 loyal customers by most order take place

```sql
pd.read_sql("select c.customer_id,c.name,count(o.order_id) as total_orders," \
"sum(o.total_amount) as total_revenue, avg(o.total_amount) as avg_order_value " \
"from orders o inner  join customers c on o.customer_id = c.customer_id " \
"group by c.customer_id,c.name" \
" order by total_orders desc limit 10;",engine) 
```

### 3) find peak time when most orders confirmed 
```sql
imp_hr = pd.read_sql("select hour(o.order_date) as order_hour ,count(order_id) as total_order_in_hr ,"\
 "avg(o.total_amount) as average_order_value "\
 "from orders o "\
 "group by order_hour order by total_order_in_hr desc ;",engine)
plt.figure(figsize=(10,6))
plt.title("Peak Hours ")
sns.barplot(data=imp_hr,x="order_hour",y="total_order_in_hr",hue="average_order_value")
plt.legend()
```

### 4) ranked restaurants by their revenue from all year include location 
```sql
pd.read_sql(" with RestaurantsRevenue as (select r.name as restaurant_name ,r.location,sum(o.total_amount) as total_revenue," \
"rank() over(partition by r.location order by sum(o.total_amount) desc) as ranking " \
"from orders o join restaurants r on o.restaurant_id = r.restaurant_id " \
" group by r.name,r.location order by total_revenue desc)" \
"select * from RestaurantsRevenue where ranking = 1 ",engine)
```
### 5) restaurants revenue ranking:
rank restaurants by their total revenue from the last year,including their name,total revenue, and rank eithin their city 
```sql
top_restaurants_in_2025 = pd.read_sql("with revenue_in_2025 as (select r.name as restaurant_name ,r.location," \
" sum(o.total_amount) as  total_revenue," \
" rank() over( partition by r.location order by sum(o.total_amount) desc) as ranking from " \
"orders o join restaurants r on o.restaurant_id  = r.restaurant_id " \
" where o.order_date like '%2024%'or'%2025%' "\
"group by r.name ,r.location order by sum(o.total_amount) desc)" \
"select * from revenue_in_2025 where ranking = 1",engine)
top_restaurants_in_2025
```
### 6) customer who order in 2024 but not in 2025
```sql
pd.read_sql_query(" select  distinct c.customer_id ,c.name,c.email,c.location " \
"from orders o join customers c on o.customer_id = c.customer_id" \
" where extract(year from o.order_date)= 2024" \
" and " \
"c.customer_id in ( select distinct c.customer_id from orders " \
"where extract(year from o.order_date)!= 2025)",engine)
```
### 7) churn customer:- who active in 6-month but not in 30 days 

```sql
pd.read_sql("with MaxDate as ( select max(order_date) as max_order_date from orders ),"\
"RecentOrders as ( select distinct o.customer_id from orders o cross join MaxDate Md "\
"where o.order_date >= date_sub(Md.max_order_date,interval 30 day) ),"\
"Retationalpool as ( select distinct o.customer_id from orders o cross join MaxDate Md "\
"where o.order_date >= date_sub(Md.max_order_date,interval 6 month) ),"\
"customerrecency as (select customer_id ,max(order_date) as last_order_date "\
"from orders group by customer_id ) "\
"select c.customer_id,c.name,cr.last_order_date,Md.max_order_date as analysis_date ,"\
"datediff(Md.max_order_date,cr.last_order_date ) as order_before_days_ago "\
"from "\
"Retationalpool rp left join recentorders ro on rp.customer_id = ro.customer_id "\
"inner join  "\
"customers c on rp.customer_id = c.customer_id "\
"inner join "\
"customerrecency cr on rp.customer_id = cr.customer_id "\
"cross join Maxdate md "\
"where ro.customer_id is null "\
"order by order_before_days_ago desc;",engine)
```
### 8) give list of restaurants that failed to deliver most of the time and how many time they failed
```sql
pd.read_sql(" select r.name ,o.status as order_status,count(o.status) as  no_of_order_cancelled " \
"from orders o inner join restaurants r on o.restaurant_id = r.restaurant_id inner join" \
"  deliveries d on o.order_id = d.order_id" \
"  where  o.status = 'Cancelled' group by r.name order by  no_of_order_cancelled desc ;",engine)
```
### 9) calculate rate comparison:
### calculate and compare the order cancellation rate for each restaurants the current year or the privious year
```sql
pd.read_sql(" with cancel_rate as  (select o.restaurant_id as restaurant_id,r.name as name,count(o.order_id) as total_orders,"\
"cast(sum(o.status = 'Cancelled')as signed) as not_deliverd_order " \
"from orders o join restaurants r on o.restaurant_id = r.restaurant_id " \
"where extract(year from o.order_date)= 2024 or extract(year from o.order_date)= 2025 "\
"group by r.restaurant_id , r.name  order by total_orders desc)" \
"select restaurant_id ,name,total_orders,not_deliverd_order," \
" round(not_deliverd_order/total_orders *100,2) as cancel_ratio from cancel_rate",engine)
```
### 10) riders average deliver time :
### determine each riders delivery time .
```sql
pd.read_sql(" with takenTime as (select p.delivery_person_id,o.order_id,p.name," \
" timediff(o.delivery_time,o.order_date) as take_time_to_delivered ,d.delivery_status" \
" from delivery_person p " \
"join deliveries d on p.delivery_person_id = d.delivery_person_id " \
"join orders o on d.order_id = o.order_id " \
" where d.delivery_status = 'Delivered') " \
" select name , sec_to_time(avg(take_time_to_delivered)) as avg_time_taken " \
" from takenTime group by name order by  avg_time_taken asc   ",engine)
```
### 11.monthly growth ratio 
```sql
pd.read_sql("with growth_ratio as ( select  o.restaurant_id as restaurant_id, " \
" date_format(o.order_date,'%m-%y') as month " \
", count(o.order_id) as current_month," \
" LAG(count(o.order_id),1) OVER(PARTITION BY o.restaurant_id ORDER BY" \
" date_format(o.order_date,'%m-%y') ) " \
" as privious_month_order FROM orders o " \
"join deliveries d ON o.order_id = d.order_id " \
"where d.delivery_status = 'Delivered'" \
" group by  o.restaurant_id , month )"
" SELECT restaurant_id,month,current_month,privious_month_order , "
"((current_month - privious_month_order)/privious_month_order)*100" \
" as growth_ratio from growth_ratio  order by restaurant_id,str_to_date(month,'%m,%y') desc;",engine)
```

### 12. customer_id total spending if above of average so gold or below of avg siver 
```sql
pd.read_sql(" select " \
"cx_category," \
"sum(total_orders) as total_orders," \
"sum(total_spent) as total_spent " \
"from "
" (select customer_id ,count(order_id) as total_orders,sum(total_amount) as total_spent ," \
"case when sum(total_amount)> (select avg(total_amount) from orders) then 'gold'" \
" else 'silver' " \
"end as cx_category " \
"from orders group by customer_id )" \
" as t1 group by cx_category ",engine)
```
### 13.rider monthly earning :
### calculated each riders total monthly earning assuming they earn 8% of the order_amount 
### each riders- orders- total amount -8%   
```sql
pd.read_sql("select p.delivery_person_id,p.name ," \
"date_format(o.order_date,'%m-%y') as month, sum(total_amount) as revenue , " \
" sum(o.total_amount*8/100) as riders_earning from " \
"delivery_person p join deliveries d on p.delivery_person_id = d.delivery_person_id join" \
" orders o on d.order_id = o.order_id " \
" group by p.delivery_person_id , p.name,month " \
"order by delivery_person_id,month,name asc  ",engine)
```
### 14. riders rating analysis : 
##### find the number of 5star ,4star ,3star rating each riders has 
#### riders receive this rating basded on delivery time 
#### if order deliverd less then 35min give 5 star
#### if they deliver under 45 then  give 4 star if they deliver under 55 min  give 3 or if they deliver after 75 give 2 star 
```sql
pd.read_sql(" select delivery_person_id,stars," \
" count(*) as total_stars from (select o.order_id ,d.delivery_id,d.delivery_person_id ," \
"timestampdiff(minute,o.order_date,o.delivery_time) as delivery_time_in_min ," \
" case when timestampdiff(minute,o.order_date,o.delivery_time) <= 35 then '5 star' " \
"when timestampdiff(minute,o.order_date,o.delivery_time) <= 45 then '4 Star' " \
"when timestampdiff(minute,o.order_date,o.delivery_time) <= 55 then '3 Star' " \
"when timestampdiff(minute,o.order_date,o.delivery_time) <= 75 then '2 Star' " \
"else '1 star' end as stars " \
"from deliveries d join orders o on d.order_id = o.order_id " \
"where d.delivery_status = 'Delivered' ) as t1  group by delivery_person_id,stars " \
"order by stars,total_stars desc ",engine)
```
### 15. order frequency by day  
### analyze order frequency per day of the weak and identify the peak day for each restaturant
```sql
pd.read_sql(" select * from (select restaurant_id ,dayname(order_date) as orders_date," \
" count(order_id) as counting_order" \
",rank() over(partition by restaurant_id order by count(order_id) desc) as ranking from orders" \
" group by restaurant_id, orders_date order by restaurant_id  ) as t1 where ranking = 1 ",engine)
```
### 16. calculate the total revenue generated by each customer ever all their orders
```sql
pd.read_sql("select c.customer_id,c.name,sum(o.total_amount) as total_revenue from orders o join " \
"customers c on o.customer_id = c.customer_id " \
"group by c.customer_id,c.name " \
"order by total_revenue desc ",engine)
```
### 17) identify sales trend 
### identify  the sales trend by comparing each month total sales to the privious month 
```sql
pd.read_sql("select extract(year from order_date) as year,extract(month from order_date) as month,sum(total_amount) as month_sale," \
" lag(sum(total_amount),1) over(order by extract(year from order_date),extract(month from order_date)) as privious_month_sale"\
" from orders group by year,month order by year,month asc ",engine)
```
### 18. rider efficiency: 
### evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest average
```sql
pd.read_sql(" with riders_time as (select  p.delivery_person_id,p.name,avg(timestampdiff(minute,o.order_date,o.delivery_time)) as delivery_time" \
" from orders o join deliveries d on o.order_id = d.order_id join delivery_person p on d.delivery_person_id	= p.delivery_person_id " \
"where d.delivery_status = 'Delivered' " \
" group by p.delivery_person_id,p.name ) " \
"select min(delivery_time) ,max(delivery_time) from riders_time ",engine)
```
### 19. find peak season 
```sql
pd.read_sql("with seasons as ( select restaurant_id, extract(month from order_date) as month ," \
"case when extract(month from order_date) between 3 and 6 then 'Summer'" \
" when extract(month from order_date) between 6 and 10 then 'rainy'" \
" else 'winter' end as season from orders) , " \
" seasons_orders as (select s.restaurant_id,season ,count(o.order_id) as no_of_orders from seasons s join " \
"orders o on s.restaurant_id = o.restaurant_id group by restaurant_id,season order by restaurant_id ,no_of_orders desc ) " \
"select season,sum(no_of_orders) as orders from seasons_orders  group by season order by  orders desc ",engine)

```
### 20. ranked each city based on the total revenue
```sql
pd.read_sql(" select * from (select r.location ,sum(o.total_amount) as total_revenue " \
" , rank() over( order by sum(o.total_amount) desc) as ranking "\
" from orders o join restaurants r on o.restaurant_id = r.restaurant_id  " \
" group by r.location order by location ) as t1 order by ranking",engine)
```
## Conclusion 
this project Highlishts my ability to handle complex sql queries and providing solutions to real work problem in the contex of food delivery service like Zomato.
## Notice 
all data is taken from kaggle.com 
