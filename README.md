SQL Project: Meesho Sales Analysis
🔹 Project Overview

This project analyzes Meesho’s sales dataset using MySQL 8.4.
The goal is to extract business insights such as regional performance, customer profitability, discount patterns, and loss-making cases.

We designed the database, cleaned the data, and applied basic to advanced SQL queries including aggregations, window functions, and analytical queries.

🔹 Database Schema

Database: Messho_sales
Table: sales

Column Name	Data Type	Description
Customer_id	VARCHAR(20)	Unique Customer ID
Customer_Name	VARCHAR(50)	Customer Name
Region	VARCHAR(50)	Customer Region (East/West/North/South)
Sales_Amount	DECIMAL(10,2)	Sales in INR
Discount	INT	Discount %
Profit_Percent	INT	Profit %
Loss	INT	Loss Amount
🔹 Data Cleaning

Handled NULL values with NULLIF().

Converted Customer_id to VARCHAR (to support alphanumeric IDs).

Allowed Sales_Amount to accept NULL.

ALTER TABLE sales MODIFY Customer_id VARCHAR(20);
ALTER TABLE sales MODIFY Sales_Amount DECIMAL(10,2) NULL;

🔹 Business Questions & SQL Queries
1️⃣ Total Sales Amount
select sum(Sales_amount) as total_sales from sales;


Insight: Overall revenue generated.

2️⃣ Average Discount
select avg(discount) as Avg_Discount from sales;


Insight: Helps identify pricing strategy effectiveness.

3️⃣ Count of NULL Sales
select count(*) as Null_sales from sales where Sales_amount IS NULL;


Insight: Data quality check.

4️⃣ Customers with Sales > 1000
select Customer_Name, Sales_amount 
from sales 
where Sales_Amount > 1000;

5️⃣ Region-wise Sales
select Region, sum(sales_amount) as total_sales 
from sales 
group by Region;


Insight: Identify top-performing region.

6️⃣ Top 3 Customers by Sales
select Customer_Name, Sales_amount 
from sales 
order by Sales_amount desc 
limit 3;

7️⃣ Profit Calculation (in INR)
select Customer_Name, Sales_amount, Profit_Percent,
(Sales_amount*Profit_Percent/100) as Profit_in_Rs
from sales;

8️⃣ Highest Profit Customer in Each Region
SELECT Customer_Name, Region, Sales_amount, Profit_Percent,
(coalesce(Sales_amount,0)*coalesce(Profit_Percent,0)/100) as Profit_in_Rs
from(
  SELECT Customer_Name, Region, Sales_amount, Profit_Percent,
  rank() over(partition by Region order by(coalesce(Sales_amount,0)*coalesce(Profit_Percent,0)/100) desc) as rnk
  from sales
) t 
where rnk = 1;


Insight: Finds most profitable customer per region.

9️⃣ Loss-Making Customers
select Customer_Name, Sales_amount, Profit_Percent,
(Sales_amount*Profit_Percent/100) as Profit_In_Rs
from sales
where (Sales_amount*Profit_Percent/100)<0;

🔟 Top 5% Customers by Sales
select Customer_Name, Sales_amount 
from(
  select Customer_Name, Sales_amount,
  cume_dist() over(order by Sales_amount desc) as Sales_Percentile
  from sales
) t 
where Sales_Percentile <= 0.05;


Insight: Identifies VIP / high-value customers.

1️⃣1️⃣ Region-wise Sales Contribution (%)
select Region,
       round(sum(Sales_amount) * 100.0 / (select sum(Sales_amount) from sales), 2) as Contribution_Percent
from sales
group by Region
order by Contribution_Percent desc;


Insight: Contribution of each region in total sales.

1️⃣2️⃣ Duplicate Customers Across Regions
select Customer_Name, count(distinct Region) as Region_count
from sales
group by Customer_Name
having Region_Count > 1;


Insight: Detects customers operating in multiple regions.

1️⃣3️⃣ Above Average Customers (Region-wise)
select Customer_Name, Region, Sales_amount
from sales s
where Sales_amount > (
    select avg(sales_amount) 
    from sales 
    where Region = s.Region
);


Insight: Finds top performers above region’s average.

🔹 Key Business Insights

The South Region contributes the highest % of sales.

Some customers show negative profit → loss cases.

Top 5% customers drive majority of revenue → Pareto principle in action.

Discounts have a mixed impact (correlation analysis needed in PostgreSQL).

A few customers operate in multiple regions → potential high-value clients.

🔹 Recommendations

Focus marketing on South & North regions for maximum ROI.

Investigate loss-making customers/products and redesign pricing.

Create loyalty programs for top 5% customers.

Improve data entry process to reduce NULL sales records.

Study discount vs profit correlation to optimize offers.
