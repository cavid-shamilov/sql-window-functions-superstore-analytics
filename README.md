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

Task 7:

```sql
select Order_ID,
Sales,	
cast(cume_dist  () over ( order by sales asc) as decimal(10,4)) 
from train
```

Task 8: 

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

Task 11:

```sql

```

Task 12:

```sql

```

Task 13:

```sql

```

Task 14:

```sql

```

Task 15:

```sql

```

