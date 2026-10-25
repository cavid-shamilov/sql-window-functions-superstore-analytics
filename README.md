# sql-window-functions-superstore-analytics
## Project Overview

This project demonstrates the practical application of advanced SQL window functions using the Superstore sales dataset in Microsoft SQL Server.

The repository contains 50 SQL practice tasks covering ranking, aggregate, offset, value, and percentile window functions. Each task focuses on a specific analytical question and explores how window functions can be used to compare records, rank sales, calculate group-level metrics, and analyze values across related rows without collapsing the underlying dataset.

### Key Areas Covered

- **Ranking and Distribution:** `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()`, `PERCENT_RANK()`, and `CUME_DIST()`
- **Window Aggregations:** `SUM()`, `AVG()`, `MIN()`, `MAX()`, and `COUNT()`
- **Running Totals and Moving Averages:** Cumulative sales and rolling calculations
- **Offset and Value Functions:** `LAG()`, `LEAD()`, `FIRST_VALUE()`, and `LAST_VALUE()`
- **Percentile Analysis:** `PERCENTILE_CONT()` and `PERCENTILE_DISC()`

### Dataset

The project uses a Superstore sales dataset containing order, customer, product, geographic, shipping, and sales information. Each row represents an order line, meaning a single order can appear in multiple rows when it contains multiple products or line items.

This distinction is important when calculating order counts, ranking sales records, and interpreting row-level analytical results.

### Project Objective

The objective is to strengthen advanced SQL skills through practical exercises, understand the differences between window functions, and apply them to common analytical scenarios involving sales performance, customer activity, product comparisons, and geographic segmentation.
Task 1: Write a query to assign a unique sequential integer to each order record in the table, ordered by Order_Date in ascending order. Display Row_ID, Order_ID, Order_Date, Sales, and the row number.

```sql
select row_number () over (order by Order_date asc) as Runk_Number,
Row_ID,
order_id,
Order_Date
from train
```
Task 2: Rank all sales records within each Category based on the Sales amount in descending order. Use DENSE_RANK() so that there are no gaps in the ranking sequence if sales amounts are tied. Include Category, Product_Name, Sales, and the calculated rank in the output

```sql
select dense_rank() over (partition by category order by sales desc) as Runk_Number,
Category,
Product_Name,
Sales
from train
```

Task 3: Identify the top 3 highest sales records for each Region. Display Region, Order_ID, Customer_Name, Sales, and their calculated rank.

```sql
with RankedSales as (
select 
Region,
Order_ID,
Customer_Name,
Sales,
dense_rank () over (partition by Region order by Sales desc) as Sales_Rank
from train
)
select
Region,
Order_ID,
Customer_Name,
Sales,
Sales_Rank
from RankedSales
where Sales_Rank <= 3
```

Task 4: Retrieve the earliest (first) order record for each customer based on Order_Date. Display Customer_ID, Customer_Name, Order_ID, Order_Date, and Sales.

```sql
with ranked_sales as(
select customer_id,
customer_name,
order_id,
order_date,
sales,
row_number () over (partition by customer_name order by order_date) as Sales_Rank
from train
)
select customer_id,
customer_name,
order_id,
order_date,
sales,
Sales_Rank
from ranked_sales
where Sales_Rank=1
```

Task 5: Assign an equal quartile group (from 1 to 4) to all orders based on their Sales amount in descending order, regardless of any category or region grouping. Display Order_ID, Sales, and the calculated quartile number.

```sql
select Order_ID,
Sales,
ntile(4) over ( order by sales desc) as Quartile_Group
from train
```

Task 6: Assign a percentile ranking (PERCENT_RANK()) to each order based on its Sales amount in ascending order, to see the relative standing of each sale. Display Order_ID, Sales, and the calculated percentile rank.

```sql
select Order_ID,
Sales,	
cast(PERCENT_RANK () over ( order by sales asc) as decimal(10,4)) 
from train
```

Task 7: Calculate the cumulative distribution (CUME_DIST()) of each order based on the Sales amount in ascending order, to see the relative position of a value within a group of values. Display Order_ID, Sales, and the calculated cumulative distribution value.

```sql
select Order_ID,
Sales,	
cast(cume_dist  () over ( order by sales asc) as decimal(10,4)) 
from train
```

Task 8: Find the top 2 highest sales records for each Category using ROW_NUMBER(). Display Category, Product_Name, Sales, and the row number.

```sql
with ranked_sales as (
select Category,
Product_Name,
sales,
row_number () over (partition by category order by sales desc) as Sales_Rank
from train
)
select Category,
product_name,
sales,
Sales_Rank
from ranked_sales 
where Sales_Rank<=2
```

Task 9: Assign a sequential ranking to all orders across the entire dataset based on Sales in descending order. If two orders have the exact same sales amount, they should receive the same rank, but the next rank should skip numbers accordingly (e.g., 1, 2, 2, 4). Display Order_ID, Sales, and the calculated rank.

```sql
select Order_ID,
sales,
rank() over (order by sales desc) 
from train
```

Task 10: Find the top 1 highest sales record for each combination of Region and Category. Display Region, Category, Product_Name, Sales, and the calculated rank

```sql
with RankedSales as (
select
Region,
Category,
Product_Name,
Sales,
row_number () over (partition by Region, Category order by Sales desc) as Sales_Rank
from train
)
select
Region,
Category,
Product_Name,
Sales,
Sales_Rank
from RankedSales
where Sales_Rank = 1
```

Task 11: Find the bottom 3 lowest sales records for each Region based on the Sales amount in ascending order. Display Region, Product_Name, Sales, and the calculated rank

```sql
with ranked_sales as (

select region,
Product_Name,
sales,
row_number () over (partition by region order by Sales asc) as Sales_Rank
from train
)
select region,
product_name,
sales,
Sales_Rank
from ranked_sales
where Sales_Rank<=3
```

Task 12: Rank all customers based on their total sales across all orders in descending order. Display Customer_ID, Customer_Name, total calculated sales, and their rank.

```sql
WITH RankedSales as (
select 
Customer_ID,
Customer_Name,
sum(Sales) as Total_Sales,
dense_rank () over (order by  sum(Sales) desc) as Sales_Rank
from train
group by Customer_ID, Customer_Name
)
select 
Customer_ID,
Customer_Name,
Total_Sales,
Sales_Rank
from RankedSales;
```

Task 13: Divide the orders within each Category into 2 equal halves (using NTILE(2)), ordered by Sales in descending order. Display Category, Product_Name, Sales, and the calculated group number.

```sql
with Ranked_Sales as (
select 
category,
product_name,
sales,
ntile(2)  over (partition by category order by sales desc) as Sales_Rank
from train

)
select 
category,
product_name,
sales,
Sales_Rank
from Ranked_Sales
```

Task 14: For each Region, find the orders that fall into the top 20% of sales by using PERCENT_RANK() greater than or equal to 0.80. Display orders where the percentile rank is >=0.80% within each region. Display Region, Product_Name, Sales, and the calculated percentile rank

```sql
with Ranked_Sales as (
select 
Region,
product_name,
sales,
cast(PERCENT_RANK () over (partition by region order by sales asc) as decimal(10,4)) as Percent_Rank
from train

)
select 
Region,
product_name,
sales,
Percent_Rank
from Ranked_Sales
where Percent_Rank >= 0.80
```

Task 15: For each Segment, assign a dense rank to all products based on their Sales in descending order, and also calculate the overall total sales per product using window aggregation. Display Segment, Product_Name, Sales, the dense rank, and the calculated segment total sales.

```sql
with Ranked_Sales as (
select 
Segment,
product_name,
sales,
sum(sales) over (partition by segment ) as Segment_Total_Sales,
dense_rank () over (partition by segment order by sales desc) as Sales_Rank
from train

)
select 
Segment,
product_name,
sales,
Segment_Total_Sales,
Sales_Rank
from Ranked_Sales
```

Task 16: Calculate the total sales for each row alongside the individual row details using an aggregate window function without partition (or across the whole table). Display Category, Product_Name, Sales, and the total overall sales across all rows.

```sql
select
category,
product_name,
sales,
sum(sales) over () as [Total Sales]
from train
```

Task 17: Calculate the total sales for each Category alongside each product's sales, so that every row shows its own product sales and the total sales of its respective category. Display Category, Product_Name, Sales, and the category total sales.

```sql
select
category,
product_name,
sales,
sum(sales) over (partition by category) as [Total Sales by Category]
from train
```

Task 18: Calculate the average sales for each Region alongside each order's sales. Display Region, Product_Name, Sales, and the average sales of the respective region.

```sql
select
Region,
product_name,
sales,
avg(sales) over (partition by region) as [Average Sales by Region]
from train 
```

Task 19: For each order, display Region, Category, Sub_Category, Product_Name, and Sales. Using aggregate window functions, calculate:

The total sales (SUM) for each Category within each Region as Region_Category_Total_Sales.

The average sales (AVG) for each Category across the entire dataset as Global_Category_Avg_Sales.

```sql
select region,
category,
sub_category,
product_name,
sales,
sum(sales) over(partition by region,category) as Region_Category_Total_Sales,
avg(sales) over(partition by category) as Global_Category_Avg_Sales
from train
```

Task 20: Calculate the running total (cumulative sum) of sales ordered by Sales in ascending order across the dataset. Display Product_Name, Sales, and the cumulative running total

```sql
select
Product_Name,
Sales,
sum(Sales) over  (order by Sales asc rows between unbounded preceding and current row) as Running_Total
from train
```

Task 21: Calculate the running total (cumulative sum) of sales within each Category, ordered by Sales in ascending order, using the proper row framing. Display Category, Product_Name, Sales, and the partitioned running total.

```sql
select category,
product_name,
sales,
sum(sales) over(partition by category order by sales asc rows between unbounded preceding and current row) as Running_Total
from train
```

Task 22: Calculate the moving or rolling average of Sales for each Category, ordering by Sales in ascending order, taking the current row and the preceding row (a 2-row moving window). Display Category, Product_Name, Sales, and the moving average.

```sql
select category,
product_name,
sales,
avg(sales) over(partition by category order by sales asc rows between 1  preceding and current row) as Moving_Average
from train
```

Task 23: For each Sub_Category, find both the minimum sales and maximum sales alongside each product's sales using aggregate window functions. Display Sub_Category, Product_Name, Sales, Min_Sales_In_SubCat, and Max_Sales_In_SubCat.

```sql
select Sub_Category,
product_name,
sales,
min(sales) over(partition by sub_category) as Min_Sales_In_SubCat,
max(sales) over(partition by sub_category) as Max_Sales_In_SubCat
from train
```

Task 24: Calculate the total count of orders (rows) for each Ship_Mode alongside each product's sales, using an aggregate window function. Display Ship_Mode, Product_Name, Sales, and the total count of orders in that respective ship mode.

```sql
select Ship_Mode,
product_name,
sales,
count(*) over(partition by ship_mode) as Total_Orders_By_Ship_Mode
from train
```

Task 25: Calculate each product's sales as a percentage of its respective Category's total sales. Display Category, Product_Name, Sales, and the percentage share

```sql
select category,
product_name,
sales,
sum(sales) over(partition by category),
cast(sales *100 / sum(sales) over(partition by category) as decimal(10,4))
from train
```

Task 26: Calculate the difference between each product's sales and the average sales of its respective Category. (Subtract the category average from the individual sales). Display Category, Product_Name, Sales, and the difference column.

```sql
select category,
product_name,
sales,
avg(sales) over(partition by category),
sales-avg(sales) over(partition by category) 
from train
```

Task 27: Find the maximum sales across the entire dataset and display it alongside every row, but also calculate how much each product's sales are below that overall maximum. Display Product_Name, Sales, and the difference from the global maximum

```sql
select category,
product_name,
sales,
max(sales) over() as Global_Max_Sales,
max(sales) over()-Sales as Diff_From_Global_Max
from train
```

Task 28: For each order, display Region, Segment, Product_Name, and Sales. Alongside these fields, calculate the average sales (AVG) for each Region and the total sales (SUM) for each Segment using aggregate window functions.

```sql
select Region,
segment,
Product_Name,
sales,
avg(sales) over(partition by region) as [Avg Sales in Region],
sum(sales) over(partition by segment) as [Total Sales in Segment]
from train
```

Task 29: For each order, display Ship_Mode, Segment, Product_Name, and Sales. Using an aggregate window function, calculate the total sales for each Ship_Mode

```sql
select Ship_Mode,
product_name,
sales,
sum(sales) over (partition by ship_mode) as Total_Sales_By_Ship_Mode
from train 
```

Task 30: For each order, display State, City, Product_Name, and Sales. Using aggregate window functions, calculate:

The total sales for each State as State_Total_Sales.

The total sales for each City as City_Total_Sales.

The percentage share of the city's sales relative to its state's total sales (rounded to 2 decimal places).

```sql
select state,city,
sum(sales) over(partition by state,city) as City_Total_Sales,
sum(sales) over(partition by state) as State_Total_Sales,
cast(round(sales*100.0/sum(sales)  over(partition by state), 2) as decimal(10,2)) as City_Share_Percentage
from train
```

---

Task 31: Display Category, Product_Name, and Sales. Alongside these, show the sales of the previous product within the same Category (ordered by Sales in ascending order) as Previous_Sales.

```sql
select Category,
Product_Name,
Sales,
lag(sales,1,0) over( partition by category order by sales asc) as Previous_Sales
from train
```

Task 32. Display Category, Product_Name, and Sales. Alongside these, show the sales of the next product within the same Category (ordered by Sales in descending order).

```sql
select Category,
Product_Name,
Sales,
lead(sales,1,0) over( partition by category order by sales desc) as Previous_Sales
from train
```

Task 33: Calculate the difference between each product's sales and the previous product's sales within the same Region (ordered by Sales in ascending order). Display Region, Product_Name, Sales, and the difference column.

```sql
select Region,
Product_Name,
Sales,
lag(sales,1,0) over( partition by region order by sales asc) as Previous_Sales,
Sales- lag(sales,1,0) over( partition by region order by sales asc) as Sales_Diff_From_Previous
from train
```

Task 34. Calculate the difference between the next product's sales and each product's sales within the same Region (ordered by Sales in descending order). Display Region, Product_Name, Sales, and the difference column.

```sql
select Region,
Product_Name,
Sales,
lead(sales,1,0) over( partition by region order by sales desc) as Next_Sales,
lead(sales,1,0) over( partition by region order by sales desc)-Sales as Sales_Diff_From_Next
from train
```

Task 35: Display Category, Product_Name, and Sales. Alongside these, show the lowest (first) sales value within each Category (ordered by Sales in ascending order).

```sql
select Category,
Product_Name,
Sales,
FIRST_VALUE (Sales) over( partition by category order by sales asc) as Lowest_Sales_In_Category
from train
```

Task 36: Function, display Sub_Category, Product_Name, and Sales. Alongside these, show the highest (last) sales value within each Sub_Category (ordered by Sales in ascending order).

```sql
select Sub_Category,
Product_Name,
Sales,
last_value (Sales) over( partition by sub_category order by sales asc ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as Highest_Sales_In_SubCategory
from train
```
Task 37: Function, compare each product's sales with the product sold immediately before it, categorized by Segment and ordered by Order_Date. Display Segment, Order_Date, Product_Name, Sales, and the previous sales amount as Prev_Segment_Sales

```sql
select Segment,
Order_Date,
Product_Name,
Sales,
lag(Sales,1,0) over( partition by segment order by order_date asc) as Prev_Segment_Sales
from train
```

Task 38. Compare each order's sales with the next order's sales within the same State, ordered by Order_Date and Sales. Display State, Order_Date, Product_Name, Sales, and the next sales amount

```sql
select state,
order_date,
product_name,
sales,
lead(sales,1,0) over(partition by state order by order_date,sales) as Next_State_Sales
from train
```

Task 39. Find the highest sales value within each City (ordered by Sales in descending order). Display City, Customer_Name, Product_Name, Sales, and the highest city sales

```sql
select city,
Customer_Name,
product_name,
sales,
FIRST_VALUE(sales) over(partition by city order by sales desc) as Max_Sales_in_City
from train
```
Task 40. For each order, display Region, Segment, Product_Name, and Sales. Using value/offset functions, calculate:

The sales of the previous product within the same Region and Segment (ordered by Sales in ascending order) as Prev_Sales.

The sales of the next product within the same Region and Segment (ordered by Sales in ascending order) as Next_Sales.

The difference between the next sales and the previous sales (Next_Sales - Prev_Sales) as Sales_Range_Span.

```sql
select Region,
segment,
Customer_Name,
product_name,
sales,
lag(sales,1,0) over(partition by region,segment order by sales asc) as Prev_Sales,
lead(sales,1,0) over(partition by region,segment order by sales asc) as Next_Sales,
lead(sales,1,0) over(partition by region,segment order by sales asc) -lag(sales,1,0) over(partition by region,segment order by sales asc) as Sales_Range_Span
from train
```

**Task 41. Display Ship_Mode, Sub_Category, Product_Name, and Sales. Alongside these, show the highest sales value within each Ship_Mode and Sub_Category combination (ordered by Sales in descending order) as Highest_Sales**

```sql
select Ship_Mode,
Sub_Category,
Customer_Name,
product_name,
sales,
first_value(sales) over (partition by ship_mode,sub_category order by sales desc) as Highest_Sales
from train
```

**Task 42. Using the LAST_VALUE() function, display Segment, State, Product_Name, and Sales. Alongside these, show the lowest (last) sales value within each Segment and State combinatio**n.

```sql
select Segment,
State,
Product_Name,
sales,
last_value(sales) over (partition by segment,state order by sales desc ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as Lowest_Sales_In_State_Segment
from train
```

**Task 43. Using the LAG() function with an offset of 2, display Region, Category, Product_Name, and Sales. Alongside these, show the sales of the product from 2 rows before within the same Region and Category (ordered by Sales in ascending order) as Sales_2_Rows_Before.**

```sql
select Region,
Category,
Product_Name,
sales,
lag(sales,2,0) over (partition by region,category order by sales asc) as Sales_2_Rows_Before
from train
```

**Task 44. Using the LEAD() function with an offset of 2, display Region, Segment, Product_Name, and Sales. Alongside these, show the sales of the product from 2 rows ahead within the same Region and Segment (ordered by Sales in descending order) as Sales_2_Rows_Ahead.**

```sql
select Region,
Segment,
Product_Name,
sales,
lead(sales,2,0) over (partition by region,segment order by sales desc) as Sales_2_Rows_Ahead
from train
```
**Task 45. or each order, display Region, Sub_Category, Product_Name, and Sales. Using value/offset functions, calculate:
The first (lowest) sales value within the same Region and Sub_Category (ordered by Sales in ascending order) as First_Sales.
The last (highest) sales value within the same Region and Sub_Category (ordered by Sales in ascending order, using ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as Last_Sales.
The difference between the highest and lowest sales (Last_Sales - First_Sales) as SubCat_Sales_Range.**

```sql
select Region,
Sub_Category,
Product_Name,
sales,
first_value(sales) over (partition by region,sub_category order by sales asc) as First_Sales,
last_value(sales) over (partition by region,sub_category order by sales asc ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as Last_Sales,
last_value(sales) over (partition by region,sub_category order by sales asc ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)-first_value(sales) over (partition by region,sub_category order by sales asc) as [Last_Sales - First_Sales]
from train
```

**Task 46. Using the PERCENTILE_CONT() function, calculate the median (50th percentile, i.e., 0.5) of Sales for each Region. Display Region and name the calculated percentile column as Median_Sales_Cont.**

```sql
select region,
sales,
percentile_cont(0.50) within group (order by sales ) over (partition by region) as Median_Sales_Cont
from train
```

**Task 47. Using the PERCENTILE_DISC() function, calculate the 25th percentile (0.25) of Sales for each Category. Display Category, Product_Name, Sales, and the calculated discrete percentile column as Sales_25th_Disc.**

```sql
select Category,
Product_Name,
sales,
percentile_disc(0.25) within group (order by sales ) over (partition by category) as Sales_25th_Disc
from train
```

**Task 48. Calculate the 75th percentile (0.75) of Sales for each Segment using both PERCENTILE_CONT() and PERCENTILE_DISC(). Display Segment, Product_Name, Sales, PERCENTILE_CONT(0.75) as Sales_75th_Cont, and PERCENTILE_DISC(0.75) as Sales_75th_Disc in the same query to visually compare their outputs.**

```sql
select Segment,
Product_Name,
sales,
percentile_disc(0.75) within group (order by sales ) over (partition by segment) as Sales_75th_Disc,
percentile_cont(0.75) within group (order by sales ) over (partition by segment) as Sales_75th_Cont
from train
```
**Task 49. Using PERCENTILE_CONT(), calculate the median (0.50) sales for each combination of Region and Category. Display Region, Category, Product_Name, Sales, and the calculated median as Median_Sales_Region_Category.**

```sql
select Region,
Category,
Product_Name,
sales,
percentile_cont(0.50) within group (order by sales ) over (partition by region,category) as Median_Sales_Region_Category
from train
```

**Task 50. Using PERCENTILE_DISC(), calculate the 90th percentile (0.90) of Sales for each State (ordered by Sales). Display State, Customer_Name, Product_Name, Sales, and the 90th percentile value as Sales_90th_Percentile_Disc.**

```sql
select state,
Customer_Name,
Product_Name,
sales,
percentile_disc(0.90) within group (order by sales ) over (partition by state) as Sales_90th_Percentile_Disc
from train
```
