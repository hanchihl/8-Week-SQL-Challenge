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
from checkout_purchase

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

```sql
with 
t1 as 
	(select 
     		distinct visit_id as visit_purchase
     	from clique_bait.events
	where event_type = 3),
t2 as
    	(select 
     		e.visit_id,
        	e.page_id,
        	e.event_type,
       	 	t1.visit_purchase
	from clique_bait.events e  
    	left join t1
    		on t1.visit_purchase = e.visit_id),
product_summary as
	(select
   		product_category,
		p.page_name as product,
    		count(case when event_name = 'Page View' then 1 end) as total_views  ,
  		count(case when event_name = 'Add to Cart' then 1 end) as total_add_carts ,
    		count(case when event_name = 'Add to Cart' and visit_purchase is null then 1 end) as total_abandoned ,    
    		count (case when event_name = 'Add to Cart' and visit_purchase is not null then 1 end) as total_purchases
	from t2
	left join clique_bait.event_identifier ei
		on t2.event_type = ei.event_type
	left join clique_bait.page_hierarchy p
		on t2.page_id = p.page_id 
	group by 
   		p.page_name,
   		product_category
	having product_category <> 'null'    )
select * from product_summary
order by product_category desc
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/dfa7c25c-4ab2-49b8-8f85-59ec938da215)

**Summary for each product category:**

```sql
with 
t1 as 
	(select 
     		distinct visit_id as visit_purchase
     	from clique_bait.events
	where event_type = 3),
t2 as
    	(select 
     		e.visit_id,
        	e.page_id,
        	e.event_type,
       	 	t1.visit_purchase
	from clique_bait.events e  
    	left join t1
    		on t1.visit_purchase = e.visit_id)
select
   	product_category,
    	count(case when event_name = 'Page View' then 1 end) as total_views  ,
  	count(case when event_name = 'Add to Cart' then 1 end) as total_add_carts ,
    	count(case when event_name = 'Add to Cart' and visit_purchase is null then 1 end) as total_abandoned ,    
    	count (case when event_name = 'Add to Cart' and visit_purchase is not null then 1 end) as total_purchases
from t2
left join clique_bait.event_identifier ei
	on t2.event_type = ei.event_type
left join clique_bait.page_hierarchy p
	on t2.page_id = p.page_id 
group by 
   	product_category
having product_category <> 'null'    
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/5caf5423-419a-409b-bfcd-6eec75fdcecb)

Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?
- In this picture, we can see that Shellfish products are the best sellers. Oyster is the product with the most views, while Lobster is the product with the most 'Add to Cart' and purchase actions
  
![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/b84b43ea-2161-4e31-8f47-28ed7dba66e0)

2. Which product was most likely to be abandoned?
- Russian Caviar
  
3. Which product had the highest view to purchase percentage?
```sql
select 
	product,
    	round((total_purchases :: numeric /  total_views :: numeric) * 100,2) as percentage_view_to_purchase
from product_summary
order by percentage_view_to_purchase desc
limit 1
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/0245b0c1-0de0-496c-a497-b9dfccfda2a4)

4. What is the average conversion rate from view to cart add?
```sql
select 
    round((sum(total_add_carts :: numeric) /  sum(total_views :: numeric)) * 100,2) as percentage_view_to_add_cart
from product_summary
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/2a11073c-7895-4b76-ad25-1e23c3ad2134)

5. What is the average conversion rate from cart add to purchase?
```sql
select 
    round((sum(total_add_carts :: numeric) /  sum(total_purchases :: numeric)) * 100,2) as percentage_add_carts_to_purchase
from product_summary
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/faaf6a76-826f-4494-88f6-600da1b582fb)

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
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number).

```sql
select 
	visit_id,
    	user_id,
    	min(event_time) as earliest_visit_time,
	count(distinct e.page_id) as total_page_view ,
    	count(distinct case
        	when event_name = 'Add to Cart' and ph.product_category <> 'null' then ph.product_category end) as total_product_add_card,
	max( case
        	when event_name = 'Purchase' then 1 else 0 end) as purchase,
	max(c.campaign_name) as campaign_name,
    	sum(case
        	when ei.event_name = 'Ad Impression' then 1 else 0 end) as total_impression,
    	sum(case
        	when ei.event_name = 'Ad Click' then 1 else 0 end) as total_ad_click    
from clique_bait.events e
left join clique_bait.users u on e.cookie_id = u.cookie_id
left join clique_bait.event_identifier ei on e.event_type = ei.event_type
left join clique_bait.page_hierarchy ph on e.page_id = ph.page_id
left join clique_bait.campaign_identifier c on e.event_time between c.start_date and c.end_date
group by visit_id, user_id
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/62c30867-d568-40af-ba59-59a836b9f8c2)

#### 5. Insights

1. Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event


2. Does clicking on an impression lead to higher purchase rates?

```sql
select 
	round((count (case when total_ad_click = 1 and purchase = 1 then 1 end) ::float
    	/ count (case when total_ad_click = 1 then 1 end) * 100)::numeric, 2) as purchase_rate_ad_click,
	round((count (case when total_ad_click = 0 and purchase = 1 then 1 end) ::float
    	/ count (case when total_ad_click = 0 then 1 end) * 100)::numeric ,2) as purchase_rate_no_ad_click
from summary
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/7c827c61-0f42-42c8-9369-ab7d225d03e1)

##### Clicking on an impression lead to higher purchase rates

3. What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?

```sql
select 
	round((count (case when total_ad_click = 1 and purchase = 1 then 1 end) ::float
    	/ count (case when total_ad_click = 1 then 1 end) * 100)::numeric, 2) as purchase_rate_ad_click,
	round((count (case when total_impression = 1 and total_ad_click = 0 and purchase = 1 then 1 end) ::float
   	 / count (case when total_impression = 1 and total_ad_click = 0 then 1 end) * 100)::numeric ,2) as purchase_rate_no_ad_click,
	round((count (case when total_impression = 0 and purchase = 1 then 1 end) ::float
   	 / count (case when total_impression = 0 then 1 end) * 100)::numeric ,2) as purchase_rate_no_impression
from summary
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/37872334-e343-4f08-a3aa-ab7e81ca8544)

##### Impressions significantly influence customer behavior, leading to higher purchase rates. Specifically, customers who engage with ads after receiving impressions show a notably higher propensity to make purchases. This highlights the pivotal role of impressions in driving customer conversions. To capitalize on this, Clique Bait should prioritize increasing impressions for each customer and develop captivating ad content to further stimulate engagement. This strategic focus is likely to yield greater sales and overall success for the platform.

4. What metrics can you use to quantify the success or failure of each campaign compared to eachother?
