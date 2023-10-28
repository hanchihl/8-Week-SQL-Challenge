## Case Study #5 - Data Mart

### ------ Introduction ------
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.
In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.
Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.
The key business question he wants you to help him answer are the following:
- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?
  
### ------ Available Data ------
For this case study there is only a single table: data_mart.weekly_sales
The Entity Relationship Diagram is shown below with the data types made clear, please note that there is only this one table - hence why it looks a little bit lonely!

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/116a3710-584a-4693-9470-da6a792a068a)

#### Column Dictionary
The columns are pretty self-explanatory based on the column names but here are some further details about the dataset:
1. Data Mart has international operations using a **multi-region** strategy
2. Data Mart has both, a retail and online **platform** in the form of a Shopify store front to serve their customers
3. Customer **segment** and **customer_type** data relates to personal age and demographics information that is shared with Data Mart
4. **transactions** is the count of unique purchases made through Data Mart and sales is the actual dollar amount of purchases
Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a week_date value which represents the start of the sales week.
#### Example Rows
![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/d693f93b-3bb5-4949-928d-dee9ad80fc5c)

### ------ Case Study Questions ------
The following case study questions require some data cleaning steps before we start to unpack Danny’s key business questions in more depth.
**1. Data Cleansing Steps**

In a single query, perform the following operations and generate a new table in the **data_mart** schema named **clean_weekly_sales**:
- Convert the **week_date** to a DATE format
- Add a **week_number** as the second column for each **week_date** value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a **month_number** with the calendar month for each **week_date** value as the 3rd column
- Add a **calendar_year** column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called **age_band** after the original **segment** column using the following mapping on the number inside the **segment** value

|segment|	age_band|
|------|-----|
|1|	Young Adults|
|2|	Middle Aged|
|3 or 4|	Retirees|

- Add a new demographic column using the following mapping for the first letter in the **segment** values:

|segment|	age_band|
|------|-----|
|C|	Couples|
|F|	Couples|

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns

- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
```` sql
alter table data_mart.weekly_sales
alter column week_date type date using TO_DATE(week_date, 'DD/MM/YY'), --Chuyển cột date sang type date
alter column transactions type decimal, -- chuyển cột transactions và sales sang type decimal để tính cột avg_transaction sang số thập phân
alter column sales type decimal;

alter table data_mart.weekly_sales
add column week_number integer,
add column month_number integer,
add column calendar_year  integer,
add column age_band varchar(30),
add column demographic  varchar(30),
add column avg_transaction numeric(10,2) ;

update data_mart.weekly_sales
set week_number = extract(week from week_date),
 	month_number = extract(month from week_date),
 	calendar_year  = extract(year from week_date),
    age_band = case 
      	when segment = 'null' then 'unknown'
      	when right(segment,1) = '1' then 'Young Adults'
      	when right(segment,1) = '2' then 'Middle Aged'
        else 'Retirees'
        end,
    demographic = case
    	when segment = 'null' then 'unknown'
        when left(segment,1) = 'C' then 'Couples'
        else 'Families'
        end,
    avg_transaction = sales/ transactions ;
````
This is top rows from **weekly_sales**:

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/23e1c609-f9e8-47e7-906d-0bf749e7d767)
   
**2. Data Exploration**

1. What day of the week is used for each **week_date** value?
```` sql
select distinct to_char(week_date, 'Day')
from data_mart.weekly_sales;
````
![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/e883ca6c-5647-423a-8743-2d06b73cb78b)

- **Answer:** **Monday** is used for each week_date

2. What range of week numbers are missing from the dataset?
```` sql
with t as(
select generate_series(1, 53) as number)
select number
from t
left join data_mart.weekly_sales w
	on w.week_number = t.number
where w.week_number is null
````
- **Answer:** Missing numbers are in the range of 1-12 and 37-53
3. How many total transactions were there for each year in the dataset?
```` sql
select 
	calendar_year,
    	sum(transactions) as total_transaction
from data_mart.weekly_sales
group by calendar_year;    
````
**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/42009cce-c5c6-4704-adaa-2cfbe4a5543e)

4. What is the total sales for each region for each month?
```` sql
select 
	region,
    	calendar_year,
    	month_number,
    	sum(sales) as total_sales
from data_mart.weekly_sales
group by 
	region,
    	calendar_year,
    	month_number
order by 
	region asc,
	calendar_year asc,
	month_number asc;
````
5. What is the total count of transactions for each platform
```` sql
select 
	platform,
    	sum(transactions) as total_transaction
from data_mart.weekly_sales
group by platform;  
````

**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/9a1dd8ed-8b18-4590-83a1-5f131453b475)

6. What is the percentage of sales for Retail vs Shopify for each month?
```` sql
select
	calendar_year,
    	month_number,
    	round(sum(case when platform ='Retail' then sales else 0 end) / sum(sales)*100,2) as retail_percent,
	round(sum(case when platform ='Shopify' then sales else 0 end) / sum(sales)*100,2) as shopify_percent
from data_mart.weekly_sales
group by 
	month_number,
    	calendar_year
order by
	calendar_year asc,
	month_number asc
````
7. What is the percentage of sales by demographic for each year in the dataset?
```` sql
select
    	calendar_year,
    	round(sum(case when demographic ='Couples' then sales else 0 end) / sum(sales)*100,2) as couples_percent,
	round(sum(case when demographic ='Families' then sales else 0 end) / sum(sales)*100,2) as families_percent,
    	round(sum(case when demographic ='unknown' then sales else 0 end) / sum(sales)*100,2) as unknown_percent
from data_mart.weekly_sales
group by 
    	calendar_year
order by
	calendar_year asc;
````
**Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/88f93a1a-a0b0-4788-ba9e-60d08e54e204)


8. Which **age_band** and **demographic** values contribute the most to Retail sales?
```` sql
select 
	age_band, 
	demographic,
	sum(sales) as total_sales
from data_mart.weekly_sales
where 
 	and platform = 'Retail'
group by     
	age_band, 
	demographic
order by total_sales desc
limit 1;
````

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/59610575-d8d0-44e1-a4fb-3d64aae55ba7)

- **Answer:** Unknown age_band  and demographic has the most to Retail sales

9. Can we use the **avg_transaction** column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
We can't use average transaction size for each year for Retail vs Shopify. Right calculate = sum Sales/sum Transactions
```` sql
with t as
(
select 
	platform,
  	calendar_year,
  	avg(transactions) as avg_trans_group,
    avg(avg_transaction) as avg_trans
from data_mart.weekly_sales  
group by     
	platform, 
  	calendar_year;
````

 **Answer:**

![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/72c7b5e1-b7db-4453-a8dd-e01823046a25)

**3. Before & After Analysis**

This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.
Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.
We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before
Using this analysis approach - answer the following questions:
1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
```` sql
with t as(
select 
	sum(case when week_date between '2020-06-15' and '2020-06-14'::date + interval '4 weeks' then sales else 0 end) as sales_after,
	sum(case when week_date between '2020-06-15'::date - interval '4 weeks' and '2020-06-14' then sales else 0 end) as sales_before    
from data_mart.weekly_sales)
select 
	sales_before,
    sales_after,
    sales_after - sales_before as changes,
    round((sales_after - sales_before)/sales_before*100,2) as percent_changes
from t;
````

 **Answer:**
 
 ![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/f7e1b7a3-8dc8-4da8-9e76-6416b4860f11)

2. What about the entire 12 weeks before and after?
```` sql
with t as(
select 
	sum(case when week_date between '2020-06-15' and '2020-06-14'::date + interval '12 weeks' then sales else 0 end) as sales_after,
	sum(case when week_date between '2020-06-15'::date - interval '12 weeks' and '2020-06-14' then sales else 0 end) as sales_before    
from data_mart.weekly_sales)
select 
	sales_before,
    sales_after,
    sales_after - sales_before as changes,
    round((sales_after - sales_before)/sales_before*100,2) as percent_changes
from t;
````

 **Answer:**

 ![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/132faee2-c5a0-4bb2-a7be-c0d50c61fa3e)

3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
   
I just show periods before and after 4 weeks in 2018 and 2019 because these codes are the same
```` sql
with t as(
select 
	sum(case when week_date between '2019-06-15' and '2019-06-14'::date + interval '4 weeks' then sales else 0 end) as sales_after_19,
	sum(case when week_date between '2019-06-15'::date - interval '4 weeks' and '2019-06-14' then sales else 0 end) as sales_before_19,
	sum(case when week_date between '2018-06-15' and '2018-06-14'::date + interval '4 weeks' then sales else 0 end) as sales_after_18,
	sum(case when week_date between '2018-06-15'::date - interval '4 weeks' and '2018-06-14' then sales else 0 end) as sales_before_18   
from data_mart.weekly_sales)
select 
    sales_after_19 - sales_before_19 as changes_19,
    round((sales_after_19 - sales_before_19)/sales_before_19*100,2) as percent_changes_19,
    sales_after_18 - sales_before_18 as changes_18,
    round((sales_after_18 - sales_before_18)/sales_before_18*100,2) as percent_changes_18   
from t;
````

 **Answer:**

 ![image](https://github.com/hanchihl/8-Week-SQL-Challenge/assets/89310493/46df0403-8673-4db2-a910-96ee78d42730)

- customer_type
  
Do you have any further recommendations for Danny’s team at Data Mart or any interesting insights based off this analysis?



