# sql-window-functions-superstore-analytics
A comprehensive guide and practice repository for advanced SQL Window Functions (Ranking, Aggregate, and Offset) using the Superstore Sales dataset in MS SQL Server.

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

```

Task 18:

```sql
select
category,
product_name,
sales,
sum(sales) over (partition by category) as [Total Sales by Category]
from train
```

Task 19: Calculate the average sales for each Region alongside each order's sales. Display Region, Product_Name, Sales, and the average sales of the respective region.

```sql
select
Region,
product_name,
sales,
avg(sales) over (partition by region) as [Average Sales by Region]
from train
```

Task 20: Calculate the running total (cumulative sum) of sales ordered by Sales in ascending order across the dataset. Display Product_Name, Sales, and the cumulative running total

```sql

```

Task 21:

```sql

```

Task 22:

```sql

```

Task 23:

```sql

```

Task 24:

```sql

```

Task 25:

```sql

```

Task 26:

```sql

```

Task 27:

```sql

```

Task 28:

```sql

```

Task 29:

```sql

```

Task 30:
