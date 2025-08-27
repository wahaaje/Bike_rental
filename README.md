# Bike rental shop - SQL Case study

##Introduction:

Emily is the shop owner, and she would like to gather data to help her grow the business. She has hired you as an SQL specialist to get the answers to her business questions such as How many bikes does the shop own by category? What was the rental revenue for each month? etc. The answers are hidden in the database. You just need to figure out how to get them out using SQL.

# Problem Statements
###Following are the business questions that Emily wants answers to. Use SQL to answer them. All the best.


### 1.  Emily would like to know how many bikes the shop owns by category. Can you get this for her? Display the category name and the number of bikes the shop owns in each category (call this column number_of_bikes ). Show only the categories where the number of bikes is greater than 2 .

select category, count(*) as number_of_bikes 
from bike
group by category
having count(*) > 2;


### 2.  Emily needs a list of customer names with the total number of memberships purchased by each. For each customer, display the customer's name and the count of memberships purchased (call this column membership_count ). Sort the results by membership_count , starting with the customer who has purchased the highest number of memberships.

SELECT c.name, count(m.id) as membership_count
FROM customer as c
RIGHT JOIN membership as m
on c.id = m.customer_id
group by c.name
order by membership_count DESC


### 3- Emily is working on a special offer for the winter months. Can you help her prepare a list of new rental prices?

### For each bike, display its ID, category, old price per hour (call this column old_price_per_hour ), discounted price per hour (call it new_price_per_hour ), old price per day (call it old_price_per_day ), and discounted price per day (call it new_price_per_day ).

### Electric bikes should have a 10% discount for hourly rentals and a 20% discount for daily rentals. Mountain bikes should have a 20% discount for hourly rentals and a 50% discount for daily rentals. All other bikes should have a 50% discount for all types of rentals. Round the new prices to 2 decimal digits.


SELECT id, 
category, 
price_per_hour as old_price_per_hour,
CASE WHEN category = 'electric' then round (price_per_hour - (price_per_hour*0.1) ,2)
		WHEN	category = 'mountain bike' then round(price_per_hour - (price_per_hour*0.2) ,2)
		else round(price_per_hour - (price_per_hour*0.5) ,2)
end as new_price_per_hour,
price_per_day as old_price_per_day,
case when category = 'electric' then round(price_per_day - (price_per_day*0.2) ,2)
	   when category = 'mountain bike' then round(price_per_day - (price_per_day*0.5) ,2)
       else round(price_per_day - (price_per_day*0.5) ,2)
  end as new_price_per_day
FROM bike;


### 4.  Emily is looking for counts of the rented bikes and of the available bikes in each category. Display the number of available bikes (call this column available_bikes_count ) and the number of rented bikes (call this column rented_bikes_count ) by bike category.

SELECT category,
COUNT(case when status ='available' then 1 end) as available_bikes_count,
COUNT(case when status ='rented' then 1 end) as rented_bikes_count
FROM bike
GROUP BY category;

### 5. Emily is preparing a sales report. She needs to know the total revenue from rentals by month, the total by year, and the all-time across all the years.Bike rental shop - SQL Case study 5. Display the total revenue from rentals for each month, the total for each year, and the total across all the years. Do not take memberships into account. There should be 3 columns: year , month , and revenue . Sort the results chronologically. Display the year total after all the month totals for the corresponding year. Show the all-time total as the last row.

select extract(year from start_timestamp) as year
, extract(month from start_timestamp) as month
, sum(total_paid) as revenue
from rental
group by year, month
union all
select extract(year from start_timestamp) as year
, null as month, sum(total_paid) as revenue
from rental
group by year
union all
select null as year, null as month, sum(total_paid) as revenue
from rental
order by year, month;

### 6.  Emily has asked you to get the total revenue from memberships for each combination of year, month, and membership type. Display the year, the month, the name of the membership type (call this column membership_type_name ), and the total revenue (call this column total_revenue ) for every combination of year, month, and membership type. Sort the results by year, month, and name of membership type.

SELECT extract(year from m.start_date) as year,
extract(month from m.start_date) as month,
mt.name as membership_type_name,
SUM(m.total_paid) as total_revenue
FROM membership as m
JOIN membership_type as mt
ON m.membership_type_id = mt.id
GROUP BY year, month, mt.name
ORDER BY year, month, mt.name

### 7.  Next, Emily would like data about memberships purchased in 2023, with subtotals and grand totals for all the different combinations of membership types and months. Display the total revenue from memberships purchased in 2023 for each combination of month and membership type. Generate subtotals and grand totals for all possible combinations. There should be 3 columns: membership_type_name , month , and total_revenue . Sort the results by membership type name alphabetically and then chronologically by month.

SELECT 
mt.name as membership_type_name, 
extract(month from m.start_date) as month,
SUM(m.total_paid) as revenue
FROM membership as m
JOIN membership_type as mt
on m.membership_type_id = mt.id
WHERE extract(year from m.start_date) = 2023
GROUP BY mt.name, month
ORDER BY mt.name, month

### 8. Emily wants to segment customers based on the number of rentals and see the count of customers in each segment. Use your SQL skills to get this! Categorize customers based on their rental history as follows: Customers who have had more than 10 rentals are categorized as 'more than 10' . Customers who have had 5 to 10 rentals (inclusive) are categorized as 'between 5 and 10' . Customers who have had fewer than 5 rentals should be categorized as 'fewer than 5' . Calculate the number of customers in each category. Display two columns: rental_count_category (the rental count category) and customer_count (the number of customers in each category)

WITH cte as
(
SELECT customer_id,
count(*) as total_rentals,
CASE WHEN COUNT(customer_id) > 10 THEN 'more than 10'
	WHEN COUNT(customer_id) BETWEEN 5 AND 10 THEN 'between 5 and 10'
    WHEN COUNT(customer_id) < 5 THEN 'fewer than 5' END AS category
FROM rental
GROUP BY customer_id
)
SELECT category as rental_count_category, count(*) as customer_count
FROM cte
GROUP BY category
order by customer_count;
