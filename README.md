use project;
-- view dataset --
select * from coffee;

-- renaming a column --
alter table coffee
rename column ï»¿transaction_id to transaction_id;

-- checking all column details --
describe coffee;

-- converting transaction_date to a date datatype from int --
update coffee
set transaction_date = str_to_date(transaction_date, '%m/%d/%Y');
alter table coffee
modify transaction_date date;

-- converting transaction_time to a time datatype from int --
alter table coffee
modify transaction_time time;

-- converting transaction_time to a float datatype from double --
alter table coffee
modify unit_price float;

-- checking for duplicates --
with cte as (select *,
row_number() over(partition by transaction_id, transaction_date, 
transaction_time, transaction_qty, store_id, store_location, product_id, 
unit_price, product_category, product_type, product_detail) as rn 
from coffee)
select * from cte where rn>1;

-- Revenue by month --
select round(sum(unit_price*transaction_qty)) as Revenue,
case when month(transaction_date) = 1 then 'January'
when month(transaction_date) = 2 then 'February'
when month(transaction_date) = 3 then 'March'
when month(transaction_date) = 4 then 'April'
when month(transaction_date) = 5 then 'May'
else 'June'
end as months
from coffee
group by months
order by 1 desc;

-- month-on-month percentage increase or decreace in revenue --
select month(transaction_date) monthly_transaction,
round(sum(unit_price*transaction_qty)) total_sale,
round((sum(unit_price * transaction_qty) - lag(sum(unit_price* transaction_qty), 1)
over())
 / lag(sum(unit_price*transaction_qty),1)
over()* 100, 1) as trans
from coffee
group by monthly_transaction
order by monthly_transaction;
 
--  month on month sales revenue--
select monthly, revenue, sum(revenue) over(order by monthly) as roll_revenue 
from(select month(transaction_date) monthly, round(sum(transaction_qty* unit_price)) revenue from coffee
group by month(transaction_date)) alias;

-- month on month difference in count of sales --
select case when month(transaction_date)= 1 then 'January'
when month(transaction_date)= 2 then 'February'
when month(transaction_date)= 3 then 'March'
when month(transaction_date)= 4 then 'April'
when month(transaction_date)= 5 then 'May'
else 'June'
end as month, count(transaction_qty) count_of_sales,
(count(transaction_qty) -lag(count(transaction_qty),1) over()) as sales_diff
from coffee
group by 1;

-- Total number of orders --
select case when month(transaction_date) = 1 then 'January'
when month(transaction_date) = 2 then 'February'
when month(transaction_date) = 3 then 'March'
when month(transaction_date) = 4 then 'April'
when month(transaction_date) = 5 then 'May'
else 'June'
end as months, count(transaction_qty) total_orders from coffee
group by 1
order by 2 desc;

-- Revenue by weekend/weekday for the month of may --
select round(sum(unit_price * transaction_qty)) as revenue , 
case when weekday(transaction_date) in (5,6) then 'weekends' else 'weekday'
end as days from coffee
where month(transaction_date)= 5 group by days;

-- Total number of orders by weekend/weekday for all months --
select count(transaction_qty) no_of_orders, month(transaction_date) months,
case when dayofweek(transaction_date) in (1,7) then 'weekends' else 'weekday' 
end as days 
from coffee
group by days, months
order by 1;

-- Revenue by store location --
select store_location, round(sum(unit_price * transaction_qty)) as revenue from coffee
where month(transaction_date) = 6
group by store_location;

-- daily average sales for the month of may --
select avg(sales) from (select transaction_date, sum(transaction_qty * unit_price) sales from coffee
where month(transaction_date)= 5
group by transaction_date) sales;

-- -- comparing daily sales with average sales, if daily sale is above or below average daily sales for month of may--
with cte as (select transaction_date, round(sum(transaction_qty * unit_price)) sales from coffee
where month(transaction_date)=5
group by transaction_date),
avg_sales as (select avg(sales) avg_sale from cte)
select transaction_date, sales, case when avg_sale > sales then 'Below avg'
when avg_sale = sales then  'Average'
else 'Above avg'
end as avg_sale
from cte, avg_sales;

-- Revenue by product category --
select product_category, round(sum(unit_price * transaction_qty)) as revenue from coffee
group by product_category
order by 2 desc;

-- Revenue by product category and product type --
select product_category, product_type, round(sum(unit_price * transaction_qty)) as revenue from coffee
group by product_category, product_type
order by 3 desc;

-- sales by hour of the day --
select hour(transaction_time) as clock, count(*)count_of_orders, sum(transaction_qty) order_qty,
round(sum(unit_price * transaction_qty))revenue,  case when weekday(transaction_date) in(5,6) then 'weekends'
else 'weekdays' end as day_of_week from coffee
where month(transaction_date) =5
group by clock, day_of_week
order by day_of_week,count_of_orders desc;

 -- Revenue and no of sales by day of week --
select round(sum(transaction_qty * unit_price)) as revenue, case when dayofweek(transaction_date) = 1 then 'Sunday'
 when dayofweek(transaction_date) = 2 then 'Monday'
  when dayofweek(transaction_date) = 3 then 'Tuesday'
   when dayofweek(transaction_date) = 4 then 'Wednesday'
    when dayofweek(transaction_date) = 5 then 'Thursday'
     when dayofweek(transaction_date) = 6 then 'Friday'
else 'Saturday'
  end as days_of_week, count(*) as count_of_order
from coffee
  where month(transaction_date) = 5
group by days_of_week
  order by 3 desc;
