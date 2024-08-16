## Case Study #7 - Balanced Tree Clothing Co.
### --------- Introduction ---------
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

### --------- Available Data ---------
For this case study there is a total of 4 datasets for this case study - however you will only need to utilise 2 main tables to solve all of the regular questions, and the additional 2 tables are used only for the bonus challenge question!

#### Product Details
balanced_tree.product_details includes all information about the entire range that Balanced Clothing sells in their store.

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/4326552b-8cb7-4452-b23d-052e0474e0ab)

#### Product Sales
balanced_tree.sales contains product level information for all the transactions made for Balanced Tree including quantity, price, percentage discount, member status, a transaction ID and also the transaction timestamp.

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/eb5ff7c8-9a76-4f5b-8c15-f45fdde186b3)

#### Product Hierarcy & Product Price
Thes tables are used only for the bonus question where we will use them to recreate the balanced_tree.product_details table.

*balanced_tree.product_hierarchy*

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/72ed9751-6f83-4a62-91d3-d7fcf9f31aa8)

*balanced_tree.product_prices*

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/2c692484-5bf6-4684-a210-0f4fdc615ff5)

### --------- Case Study Questions ---------
The following questions can be considered key business questions and metrics that the Balanced Tree team requires for their monthly reports.

Each question can be answered using a single query - but as you are writing the SQL to solve each individual problem, keep in mind how you would generate all of these metrics in a single SQL script which the Balanced Tree team can run each month.

#### High Level Sales Analysis
1. What was the total quantity sold for all products?
```sql
select sum(qty) as total_quantity_sale
from balanced_tree.sales 
```

**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/c491f51a-02ac-48a9-b4cd-5bc285513152)

2. What is the total generated revenue for all products before discounts?
3. What was the total discount amount for all products?
```sql
select 
	sum(qty*price) as revenue_before_discount,
	sum(qty*discount) as total_discount
from balanced_tree.sales 
```

**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/8cdd1a6a-5c96-4d14-8980-527e10b2c11d)

#### Transaction Analysis
1. How many unique transactions were there?
```sql
select count(distinct txn_id) as total_unique_trasaction
from balanced_tree.sales
```

**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/1b13e322-78d6-4c5b-aa94-81d4e681adae)

2. What is the average unique products purchased in each transaction?
```sql
with t as

(select count(distinct prod_id) as unique_product
from balanced_tree.sales 
group by txn_id)

select round(avg(unique_product),2) as avg_unique_prodct_per_transaction
from t;
```

**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/79e74b5e-15b8-4a41-9f0a-ed69ff7b66ff)

3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
with t as
(select 
	txn_id,
    	sum(qty *price*(100-discount)/100) as revenue
from balanced_tree.sales 
group by txn_id)
select 
	percentile_cont(0.25) within group (order by revenue) as p25th_percentile_value,
	percentile_cont(0.5) within group (order by revenue) as p50th_percentile_value,
	percentile_cont(0.75) within group (order by revenue) as p75th_percentile_value    
from t
```

**Answer:**

![image](https://github.com/user-attachments/assets/e56c5a0b-e135-4e54-993f-b8c53b1270e0)

4. What is the average discount value per transaction?
```sql
with t as
(select 
	txn_id,
    	sum(qty *price*discount/100) as discount
from balanced_tree.sales 
group by txn_id)
select 
	round(avg(discount),2) as avg_discount
from t
```

**Answer:**

![image](https://github.com/user-attachments/assets/2ad599a1-d04e-4776-ac48-2c294b7f6abc)

5. What is the percentage split of all transactions for members vs non-members?
```sql
select 
	round( count(case when member = 't' then 1 end) *100.0 / count (*),2) as percentage_of_member,
	round( count(case when member = 'f' then 1 end) *100.0 / count (*),2) as percentage_of_nonmember 
from balanced_tree.sales 
```

**Answer:**

![image](https://github.com/user-attachments/assets/e8b960ef-a10f-4e77-9e4e-fa5e71609e67)

6. What is the average revenue for member transactions and non-member transactions?
```sql
select 
    round(
        SUM(CASE WHEN member = 't' THEN qty * price * (100.0 - discount) END) / 
      	count (CASE WHEN member = 't' THEN 1 END))  as avg_revenue_of_member,
    round(
        SUM(CASE WHEN member = 'f' THEN qty * price * (100.0 - discount) END) / 
      	count (CASE WHEN member = 't' THEN 1 END))  as avg_revenue_of_nonmember
FROM 
    balanced_tree.sales;
```

**Answer:**

![image](https://github.com/user-attachments/assets/a79a046f-4669-4799-a000-57d10cb63e26)

#### Product Analysis
1. What are the top 3 products by total revenue before discount?
```sql
select 
	product_name ,
	sum(s.qty * s.price) as revenue 
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by product_name
order by revenue desc
limit 3
```

**Answer:**

![image](https://github.com/user-attachments/assets/f3330e49-5db6-4bb7-88d3-39d27d74f4f3)

2. What is the total quantity, revenue and discount for each segment?
```sql
select 
	segment_name,
    	sum(s.qty) as total_quantity,
	sum(s.qty * s.price * (100 - s.discount) /100) as revenue,
    	sum(s.qty * s.price * s.discount/100) as discount
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by segment_name
```

**Answer:**

![image](https://github.com/user-attachments/assets/dd82f604-6946-4a3b-b36b-41539cc8b3fa)

3. What is the top selling product for each segment?
```sql
select 
	distinct on (segment_name)
    	segment_name,
	product_name,
	sum(s.qty * s.price * (100 - s.discount) /100) as revenue
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by 
	segment_name,
    	product_name
order by 
	segment_name,
	revenue desc
```

**Answer:**

![image](https://github.com/user-attachments/assets/732dd55f-924b-418b-a14d-55ebf634f049)

4. What is the total quantity, revenue and discount for each category?
```sql
select 
	category_name,
    	sum(s.qty) as total_quantity,
	sum(s.qty * s.price * (100 - s.discount) /100) as revenue,
    	sum(s.qty * s.price * s.discount/100) as discount
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by category_name
```

**Answer:**

![image](https://github.com/user-attachments/assets/f5b002be-6846-48ed-972e-3d1d0891253d)

5. What is the top selling product for each category?
```sql
select 
	distinct on (category_name)
    	category_name,
	product_name,
	sum(s.qty * s.price * (100 - s.discount) /100) as revenue
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by 
	category_name,
    	product_name
order by 
	category_name,
	revenue desc
```

**Answer:**

![image](https://github.com/user-attachments/assets/9f45f135-43c6-4cfc-a5fd-3b766b512c55)

6. What is the percentage split of revenue by product for each segment?
```sql
with t as
(select  
	segment_name, 
    	product_name,
    	sum(s.qty * s.price * (100 - s.discount) /100) as revenue
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by segment_name, product_name)
select 
	segment_name, 
    	product_name,
    	round(revenue * 100/ sum(revenue) over (partition by segment_name),2) as segment_percentage
from t
```

**Answer:**

![image](https://github.com/user-attachments/assets/1a995603-afc3-4502-8672-b023b290b819)

7. What is the percentage split of revenue by segment for each category?
```sql
with t as
(select  
	category_name, 
    	segment_name,
    	sum(s.qty * s.price * (100 - s.discount) /100) as revenue
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by category_name, segment_name)
select 
	category_name, 
    	segment_name,
    	round(revenue * 100/ sum(revenue) over (partition by category_name),2) as category_percentage
from t
```

**Answer:**

![image](https://github.com/user-attachments/assets/d8c8752b-2be6-491e-84da-c39c8bfb5d81)

8. What is the percentage split of total revenue by category?
```sql
select 
	category_name, 
  	round(revenue / sum(revenue) over()  * 100,2) as percentage
from(
select  
		category_name, 
    	sum(s.qty * s.price * (100 - s.discount) /100) as revenue
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by category_name) a
```

**Answer:**

![image](https://github.com/user-attachments/assets/8cea7918-c351-4545-97db-b2f5047bc460)

9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```sql
with t1 as
(select 
 	product_name,
 	count(distinct txn_id) as total_transaction
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id
group by product_name),
t2 as
(select count(distinct txn_id) as all_transaction
 from balanced_tree.sales)

select
 	product_name,
    total_transaction,
    round(total_transaction/all_transaction ::numeric,2) AS penetration
from t1, t2
```

**Answer:**

![image](https://github.com/user-attachments/assets/812d2930-c22f-45ca-b04a-e4edff4c8575)

10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```sql
with t1 as
(select product_name as prod_id, txn_id
from balanced_tree.sales s
left join balanced_tree.product_details d
on s.prod_id = d.product_id),
 
product_combinations as (
    select 
        s1.txn_id,
        least(s1.prod_id, s2.prod_id, s3.prod_id) as prod1,
        greatest(s1.prod_id, s2.prod_id, s3.prod_id) as prod3,
        case 
            when s1.prod_id <> least(s1.prod_id, s2.prod_id, s3.prod_id) and s1.prod_id <> greatest(s1.prod_id, s2.prod_id, s3.prod_id) then s1.prod_id
            when s2.prod_id <> least(s1.prod_id, s2.prod_id, s3.prod_id) and s2.prod_id <> greatest(s1.prod_id, s2.prod_id, s3.prod_id) then s2.prod_id
            else s3.prod_id
        end as prod2
    from 
        t1 s1
    join 
        t1 s2 on s1.txn_id = s2.txn_id and s1.prod_id < s2.prod_id
    join 
        t1 s3 on s2.txn_id = s3.txn_id and s2.prod_id < s3.prod_id
)
select 
    prod1, prod2, prod3, 
    count(*) as combination_count
from 
    product_combinations pc
group by 
        prod1, prod2, prod3
order by 
    combination_count desc
limit 1
```

**Answer:**

![image](https://github.com/user-attachments/assets/b8e18aae-46ff-4b28-94f7-c17ae27de8fd)

#### Reporting Challenge
Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

#### Bonus Challenge
Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.

Hint: you may want to consider using a recursive CTE to solve this problem!

```sql
with t as
(select 
	p1.id as style_id,
    p1.parent_id as segment_id,
    p1.level_text as style_name,
    p2.parent_id as category_id,
    p2.level_text as segment_name
from balanced_tree.product_hierarchy p1
left join balanced_tree.product_hierarchy p2
on p2.id = p1.parent_id)

select 
	p.product_id,
    p.price,
    concat (t.style_name, '-' ,p3.level_text )as product_name,
	t.category_id,
    t.segment_id,
    t.style_id,
    p3.level_text as category_name,
    t.segment_name,
    t.style_name   
from t
left join  balanced_tree.product_hierarchy p3
on t.category_id = p3.id 
left join balanced_tree.product_prices p
on p.id = t.style_id
where t.category_id is not null
```

**Answer:**

![image](https://github.com/user-attachments/assets/6d016543-19c9-4c39-9a63-44fea37908df)

