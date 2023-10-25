## ----- Introduction -----
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.
Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!
Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!
Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!
The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.
This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!
## ----- Available Data -----
The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get you familiar with their tables.

Entity Relationship Diagram
![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/cd2c81d5-bee1-47e0-9a36-48f479244aca)

# Case Study Questions
## A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?
````sql
select count(distinct region_id) as total_nodes from data_bank.customer_nodes;
````
### 2. What is the number of nodes per region?
````sql
select 
	region_name,
  count(distinct node_id) as total_node
from data_bank.customer_nodes n
left join data_bank.regions r
on n.region_id = r.region_id
group by region_name;
````
### 3. How many customers are allocated to each region?
````sql
select 
	region_name,
  count(distinct customer_id) as total_node
from data_bank.customer_nodes n
left join data_bank.regions r
on n.region_id = r.region_id
group by region_name;
````
### 4. How many days on average are customers reallocated to a different node?
````sql
with day_in_node as
(select 
	customer_id,
  node_id,
	sum(end_date - start_date) as days
from data_bank.customer_nodes
where end_date != '9999-12-31'
group by 
 	customer_id,
 	node_id)    
select round(avg(days)) as vag_days from day_in_node;    
 ````
### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
````sql
with day_in_node as
(select 
 	region_id, 
	customer_id,
    node_id,
	sum(end_date - start_date) as days
from data_bank.customer_nodes
where end_date != '9999-12-31'
group by 
 	region_id,
 	customer_id,
 	node_id)
select 
	percentile_cont(0.5) within group(order by days) as median,
    percentile_cont(0.8) within group(order by days) as per_80,
    percentile_cont(0.95) within group(order by days) as per_95
from  day_in_node;
````
## B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?
````sql
select
  txn_type,
  count (*),
  sum(txn_amount) as total_amount
from data_bank.customer_transactions
group by txn_type;
````
### 2. What is the average total historical deposit counts and amounts for all customers?
````sql
select round(avg(txn_amount),1), sum(txn_amount)
from data_bank.customer_transactions
where txn_type = 'deposit';
````
### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
````sql
with t as
(select 
	  extract ('month' from txn_date) as months,
    customer_id,
    sum(case when txn_type = 'deposit' then 1 else 0 end) as deposit,
    sum(case when txn_type != 'deposit' then 1 else 0 end) as other
from data_bank.customer_transactions
group by 
	customer_id,
    extract ('month' from txn_date))
select months, count (*) as total_customer
from t
where deposit > 0 and other > 0
group by months;
````
### 4. What is the closing balance for each customer at the end of the month?
khó quá chưa làm xong :(
````sql
  SELECT 
    customer_id, 
    (DATE_TRUNC('month', txn_date) + INTERVAL '1 MONTH - 1 DAY') AS closing_month, 
    SUM(CASE 
      WHEN txn_type = 'withdrawal' OR txn_type = 'purchase' THEN -txn_amount
      ELSE txn_amount END) AS transaction_balance
  FROM data_bank.customer_transactions
  GROUP BY 
    customer_id, txn_date 
  order by customer_id asc;
````
### 5. What is the percentage of customers who increase their closing balance by more than 5%?










