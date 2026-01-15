Week5
Developers week 5
Week 5: Advanced Data Manipulation with Pandas
This week focuses on mastering data grouping, aggregation, filtering, and cleaning using Pandas. You will learn to handle datetime and text data, merge multiple datasets, and create pivot tables for insightful summarization.
Hands-On Practice:
•	Calculate monthly sales totals
•	Filter data with multiple conditions
•	Clean and transform text and date columns
•	Merge customer and sales data
•	Summarize data using pivot tables
Project:
Analyse customer purchasing patterns, identify top customers, and build a sales performance dashboard.

Day 1: Data Loading & Exploration
 Objectives
•	Load datasets
•	Understand structure
•	Identify missing values
import pandas as pd
sales_df = pd.read_csv(r"D:\vi\developer_arena\week5\sales_data.csv")
customers_df = pd.read_csv(r"D:\vi\developer_arena\week5\customer_churn.csv")
sales_df.head(), customers_df.head()
 
# Dataset structure
sales_df.info()
customers_df.info()
 


# Check missing values
sales_df.isnull().sum()
customers_df.isnull().sum()
 

Day 2: Data Cleaning & Preparation
 Objectives
•	Handle missing values
•	Convert date columns
•	Create calculated columns

# Convert Date column
sales_df['Date'] = pd.to_datetime(sales_df['Date'], errors='coerce')

# Remove rows with invalid dates
sales_df.dropna(subset=['Date'], inplace=True)

# Ensure numeric columns
sales_df['Quantity'] = pd.to_numeric(sales_df['Quantity'], errors='coerce')
sales_df['Price'] = pd.to_numeric(sales_df['Price'], errors='coerce')
sales_df['Total_Sales'] = pd.to_numeric(sales_df['Total_Sales'], errors='coerce')

# Fill missing numeric values
sales_df.fillna(0, inplace=True)

# Extract date parts
sales_df['Year'] = sales_df['Date'].dt.year
sales_df['Month'] = sales_df['Date'].dt.month


Day 3: Customer Analysis
Objectives
•	Identify top customers
•	Fill missing values 
•	Create churn flag


# Convert TotalCharges to numeric
customers_df['TotalCharges'] = pd.to_numeric(customers_df['TotalCharges'], errors='coerce')

# Fill missing values
customers_df['TotalCharges'].fillna(customers_df['TotalCharges'].median(), inplace=True)

# Create churn flag
customers_df['ChurnFlag'] = customers_df['Churn'].map({'Yes': 1, 'No': 0})

customers_df.rename(columns={'CustomerID': 'Customer_ID'}, inplace=True)

Day 4: Detailed Customer Analysis
•	Identify top customers
•	Calculate customer lifetime value (CLV)
•	Analyze regional distribution
1.	pd.merge(sales_df, customers_df, on='Customer_ID') → Combines sales and customer data using the Customer_ID column.
2.	how='left' → Keeps all rows from sales_df, adding matching customer info from customers_df.
3.	Result (merged_df) → A single DataFrame with sales details and corresponding customer information.

merged_df = pd.merge(
    sales_df,
    customers_df,
    on='Customer_ID',
    how='left'
)
merged_df.head()
 




Total Revenue
1.	merged_df['Total_Sales'].sum() → Calculates the total revenue by adding all sales values.
2.	total_revenue → Stores the sum of all sales in the dataset.
3.	print(total_revenue) → Displays the total revenue.

total_revenue = merged_df['Total_Sales'].sum()
print(total_revenue)
 
Revenue By Reigon
1.	merged_df.groupby('Region') → Groups sales data by each region.
2.	['Total_Sales'].sum() → Calculates the total sales for each region.
3.	revenue_by_region → Stores the regional revenue summary.

revenue_by_region = merged_df.groupby('Region')['Total_Sales'].sum()
revenue_by_region
 

Revenue By Contract
revenue_by_contract = merged_df.groupby('Contract')['Total_Sales'].sum()
revenue_by_contract
 

Top Customers
top_customers = (
    merged_df.groupby('Customer_ID')['Total_Sales']
    .sum()
    .sort_values(ascending=False)
    .head(10)
)
top_customers
 

Orders Per Customer
1.	orders_per_customer = merged_df.groupby('Customer_ID')['Date'].nunique()
o	Counts how many unique purchase days each customer has.
2.	orders_per_customer > 1
o	Marks customers who bought more than once as True.
3.	.mean()
o	Calculates the fraction of customers who are repeat buyers.
4.	* 100
o	Converts the fraction into a percentage.
5.	Interpretation:
o	Retention rate = percentage of customers who made multiple purchases; higher = better customer loyalty.

orders_per_customer = merged_df.groupby('Customer_ID')['Date'].nunique()
retention_rate = (orders_per_customer > 1).mean() * 100
retention_rate
 

Monthly sales
1.	groupby(['Year', 'Month']) → Groups sales by year and month.
2.	['Total_Sales'].sum() → Calculates total revenue for each month.
3.	.reset_index() → Converts grouped data back into a DataFrame.
4.	Result (monthly_sales) → Shows monthly total sales trends.
5.	Use → Ideal for line charts and seasonal trend analysis.

monthly_sales = (
    merged_df.groupby(['Year', 'Month'])['Total_Sales']
    .sum()
    .reset_index()
)
monthly_sales

 


Top Products
1.	groupby('Product') → Groups sales by each product.
2.	['Quantity'].sum() → Calculates total quantity sold per product.
3.	.sort_values(ascending=False) → Sorts products from highest to lowest sales.
4.	.head(10) → Selects the top 10 best-selling products.
5.	Result (top_products) → Shows the most popular products by quantity sold.

top_products = (
    merged_df.groupby('Product')['Quantity']
    .sum()
    .sort_values(ascending=False)
    .head(10)
)
top_products

 

Total Sales for each reigon.
1.	pd.pivot_table() → Creates a table that summarizes data across two dimensions.
2.	values='Total_Sales' → Uses total sales as the numbers to summarize.
3.	index='Region' → Rows represent different regions.
4.	columns='Product' → Columns represent different products.
5.	aggfunc='sum' → Sums sales for each region-product combination.
6.	Result (pivot_sales) → Shows total sales of each product in each region.

pivot_sales = pd.pivot_table(
    merged_df,
    values='Total_Sales',
    index='Region',
    columns='Product',
    aggfunc='sum'
)
pivot_sales
 

churn rate by contract type and region
1.	pd.pivot_table() → Creates a summarized table for analysis.
2.	values='ChurnFlag' → Uses the churn indicator (0/1) as the metric.
3.	index='Contract' → Rows represent different contract types.
4.	columns='Region' → Columns represent different regions.
5.	aggfunc='mean' → Calculates the average churn rate for each contract-region pair.
6.	Result (pivot_churn) → Shows churn rate by contract type and region.

pivot_churn = pd.pivot_table(
    merged_df,
    values='ChurnFlag',
    index='Contract',
    columns='Region',
    aggfunc='mean'
)
pivot_churn
 

DAY 4 Visualization
Monthly sales Trends
1.	import matplotlib.pyplot as plt → Imports Matplotlib for plotting and figure control.
2.	import seaborn as sns → Imports Seaborn for easy, high-level visualizations.
3.	sns.lineplot(data=monthly_sales, x='Month', y='Total_Sales', hue='Year') → Plots a line chart of monthly sales, with separate lines for each year.
4.	plt.title("Monthly Sales Trend") → Adds a title to the chart.
5.	plt.show() → Displays the chart.

import matplotlib.pyplot as plt
import seaborn as sns
sns.lineplot(data=monthly_sales, x='Month', y='Total_Sales', hue='Year')
plt.title("Monthly Sales Trend")
plt.show()
 


Total revenue in each region.
1.	revenue_by_region.plot(kind='bar', title="Revenue by Region") → Creates a vertical bar chart of total sales by region.
2.	plt.ylabel("Total Sales") → Labels the y-axis as “Total Sales.”
3.	plt.show() → Displays the chart.

revenue_by_region.plot(kind='bar', title="Revenue by Region")
plt.ylabel("Total Sales")
plt.show()
 

Customers contributed the most revenue
1.	top_customers.plot(kind='barh', title="Top 10 Customers by Revenue") → Creates a horizontal bar chart of the top 10 customers based on revenue.
2.	plt.show() → Displays the chart.

top_customers.plot(kind='barh', title="Top 10 Customers by Revenue")
plt.show()
 

10 most profitable products
1.	merged_df.groupby('Product')['Total_Sales'].sum() → Calculates total revenue for each product.
2.	.sort_values(ascending=False).head(10) → Selects the top 10 highest-revenue products.
3.	.reset_index() → Converts the grouped data back into a DataFrame.
4.	plt.figure(figsize=(10,6)) → Sets the size of the plot.
5.	sns.barplot(data=top_products_revenue, y='Product', x='Total_Sales', palette="viridis") → Creates a horizontal bar chart of top products by revenue.
6.	plt.xlabel/plt.ylabel/plt.title → Adds axis labels and a chart title.
7.	plt.show() → Displays the chart.

# Aggregate total sales per product
top_products_revenue = merged_df.groupby('Product')['Total_Sales'].sum().sort_values(ascending=False).head(10).reset_index()

# Plot
plt.figure(figsize=(10,6))
sns.barplot(data=top_products_revenue, y='Product', x='Total_Sales', palette="viridis")
plt.xlabel("Total Revenue")
plt.ylabel("Product")
plt.title("Top 10 Products by Revenue")
plt.show()
 

monthly sales trends across years
1.	plt.figure(figsize=(10,6)) → Sets the size of the figure for better readability.
2.	sns.lineplot(..., marker="o") → Plots monthly sales as a line chart with markers on each data point.
3.	plt.title("Monthly Sales Trend") → Adds a title to the chart.
4.	plt.ylabel("Total Sales") / plt.xlabel("Month") → Labels the y-axis and x-axis.
5.	plt.xticks(range(1,13)) → Ensures x-axis shows all months from 1 to 12.
6.	plt.legend(title="Year") → Adds a legend to differentiate sales lines by year.
7.	plt.show() → Displays the chart.

plt.figure(figsize=(10,6))
sns.lineplot(data=monthly_sales, x='Month', y='Total_Sales', hue='Year', marker="o")
plt.title("Monthly Sales Trend")
plt.ylabel("Total Sales")
plt.xlabel("Month")
plt.xticks(range(1,13))
plt.legend(title="Year")
plt.show()

 
