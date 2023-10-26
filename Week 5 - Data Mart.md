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

### ------ Case Study Questions ------
The following case study questions require some data cleaning steps before we start to unpack Danny’s key business questions in more depth.
**1. Data Cleansing Steps**
In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:
Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
  
|segment|	age_band|
|------|-----|
|1|	Young Adults|
|2|	Middle Aged|
|3 or 4|	Retirees|
