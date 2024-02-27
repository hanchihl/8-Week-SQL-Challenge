## Case Study #6 - Clique Bait
### --------- Introduction ---------
Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!

In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

### --------- Available Data ---------
For this case study there is a total of 5 datasets which you will need to combine to solve all of the questions.

#### Users
Customers who visit the Clique Bait website are tagged via their cookie_id.

<img src="https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/27056fe2-643f-4f51-848e-2811b14ab84b" width=400 height=400>

#### Events
Customer visits are logged in this events table at a cookie_id level and the event_type and page_id values can be used to join onto relevant satellite tables to obtain further information about each event.

The sequence_number is used to order the events within each visit.

<img src="https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/df6eff30-1ee6-419f-85dd-b513c1425e59" width=600 height=450>

#### Event Identifier
The event_identifier table shows the types of events which are captured by Clique Bait’s digital data systems.

<img src="https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/2f8421c0-1846-430f-8a20-cf85a332d8f9" width=250 height=300>

#### Campaign Identifier
This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.

<img src="https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/b25be6f7-82d3-4963-92b2-96614468715a" width=500 height=230>

#### Page Hierarchy
This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.

<img src="https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/dddbb258-5009-4da7-92f5-38565e9cec1f" width=400 height=400>

### Case Study Questions
#### 1. Enterprise Relationship Diagram
[Link Diagram for dataset](https://dbdiagram.io/d/Clique-Bait_Week_6-653f5ea2ffbf5169f0b5ac9b)

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/a67bbdbc-1a37-492c-bfec-2eef5b51d7fe)

#### 2. Digital Analysis
Using the available datasets - answer the following questions using a single query for each one:
1. How many users are there?
```sql
select count(distinct user_id) as total_user
from clique_bait.users
````
**Answers:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/19e91e28-4feb-4ff7-80ab-daf5fa272eca)

2. How many cookies does each user have on average?
```sql
with t as 
(select user_id, count(*)
from clique_bait.users
group by user_id)
select round(avg(count),0) as avg_cookies from t;
````
**Answers:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/8be27b5c-c22e-4975-bdf7-00b59e840076)

3. What is the unique number of visits by all users per month?
```sql
select 
	extract(month from event_time) as months,
    count(distinct visit_id) as unique_visits
from clique_bait.events
group by months;
````
**Answers:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/5359a8cb-447b-424e-b296-26866e9d89b0)

4. What is the number of events for each event type?
```sql
select 
	event_type,
	count(*) as number_event
from clique_bait.events
group by event_type
order by event_type asc;
````
**Answers:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/770b048f-8655-4807-b4f0-834cfae7d39f)

5. What is the percentage of visits which have a purchase event?
```sql
select 
	100 * count (distinct visit_id) / 
    ( select count (distinct visit_id)
    	from clique_bait.events) as percentage
from clique_bait.events
where event_type = 3
````
**Answers:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/2daef755-bb45-475e-8b27-b00db0813d45)

6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
with checkout_purchase as (
select 
  visit_id,
  max(case when event_type = 1 and page_id = 12 then 1 else 0 end) as checkout,
  max(case when event_type = 3 then 1 else 0 end) as purchase
from clique_bait.events
group by visit_id)

select 
  round(100 * (1-(sum(purchase)::numeric/sum(checkout))),2) as percentage_checkout_view_with_no_purchase
from checkout_purchase"
![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/e57c033c-be87-4f97-93d4-33f302d87bdb)

````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/9f7d12d7-7f1b-4735-9d26-aeb64717ee65)

7. What are the top 3 pages by number of views?
```sql
select 
	event_name, count (*) as number_of_view
from clique_bait.events e
left join clique_bait.event_identifier ei
	on e.event_type = ei.event_type
group by event_name
order by number_of_view desc
limit 3;
````
**Answers:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/955edff6-8e5f-4cbe-9363-a7a989c3c2bf)

8. What is the number of views and cart adds for each product category?
```sql
select 
	p.product_category,
    count(
      	case when event_name = 'Page View' then 1 end) as total_views,
    count(
      	case when event_name = 'Add to Cart' then 1 end) as total_add_carts       
from clique_bait.events e
left join clique_bait.event_identifier ei
	on e.event_type = ei.event_type
left join clique_bait.page_hierarchy p
	on e.page_id = p.page_id    
group by p.product_category; 
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/f90daf97-31e3-4f5d-a6f6-931c95421cef)

9. What are the top 3 products by purchases?
```sql

````

#### 3. Product Funnel Analysis
Using a single SQL query - create a new output table which has the following details:

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?
2. Which product was most likely to be abandoned?
3. Which product had the highest view to purchase percentage?
4. What is the average conversion rate from view to cart add?
5. What is the average conversion rate from cart add to purchase?

```sql
t1 as
(
  select 
	p.page_name as product,
    count(
      	case when event_name = 'Page View' then 1 end) as total_views     
	from clique_bait.events e
	left join clique_bait.event_identifier ei
		on e.event_type = ei.event_type
	left join clique_bait.page_hierarchy p
		on e.page_id = p.page_id    
	group by product),
t2 as    
(
  select 
	p.page_name as product,
    count(
      	case when event_name = 'Add to Cart' then 1 end) as total_add_carts   
	from clique_bait.events e
	left join clique_bait.event_identifier ei
		on e.event_type = ei.event_type
	left join clique_bait.page_hierarchy p
		on e.page_id = p.page_id    
	group by product)
select 
	t1.product, 
    total_views,
    total_add_carts
from t1
left join t2
	on t1.product = t2.product
````

#### 4. Campaigns Analysis
Generate a table that has 1 single row for every unique **visit_id** record and has the following columns:

- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

- Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
- Does clicking on an impression lead to higher purchase rates?
- What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
- What metrics can you use to quantify the success or failure of each campaign compared to eachother?
