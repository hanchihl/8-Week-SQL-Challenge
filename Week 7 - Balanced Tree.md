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

```

**Answer:**


2. What is the total quantity, revenue and discount for each segment?
```sql

```

**Answer:**


3. What is the top selling product for each segment?
```sql

```

**Answer:**


4. What is the total quantity, revenue and discount for each category?
```sql

```

**Answer:**


5. What is the top selling product for each category?
```sql

```

**Answer:**


6. What is the percentage split of revenue by product for each segment?
```sql

```

**Answer:**


7. What is the percentage split of revenue by segment for each category?
```sql

```

**Answer:**


8. What is the percentage split of total revenue by category?
```sql

```

**Answer:**


9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```sql

```

**Answer:**


10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```sql

```

**Answer:**


#### Reporting Challenge
Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.

Imagine that the Chief Financial Officer (which is also Danny) has asked for all of these questions at the end of every month.

He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).

Feel free to split up your final outputs into as many tables as you need - but be sure to explicitly reference which table outputs relate to which question for full marks :)

#### Bonus Challenge
Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.

Hint: you may want to consider using a recursive CTE to solve this problem!




