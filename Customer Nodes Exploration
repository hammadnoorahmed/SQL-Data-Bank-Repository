## A. Customer Nodes Exploration

**1. How many unique nodes are there on the Data Bank system?**
```sql
Select count(distinct node_id) as Number_of_nodes
from customer_nodes;
```
**Output:**
| Number_of_Nodes | 
|-----------------|
| 5               |

**2. What is the number of nodes per region?**
```sql
Select r.region_id, r.region_name, count(c.node_id) as number_of_nodes
from customer_nodes c join regions r on c.region_id = r.region_id
group by region_name, region_id
order by region_id;
```

**Output:**
| region_id | region_name | number_of_nodes |
|-----------|-------------|-----------------|
| 1         | Australia   | 770             |
| 2         | America     | 735             |
| 3         | Africa      | 714             |
| 4         | Asia        | 665             |
| 5         | Europe      | 616             |


**3. How many customers are allocated to each region?**
```sql
Select r.region_id, r.region_name, count(distinct c.customer_id) as number_of_customers
from customer_nodes c join regions r on c.region_id = r.region_id
group by region_name, region_id
order by region_id;
```

**Output:**
| region_id | region_name | number_of_customers |
|-----------|-------------|---------------------|
| 1         | Australia   | 110                 |
| 2         | America     | 105                 |
| 3         | Africa      | 102                 |
| 4         | Asia        | 95                  |
| 5         | Europe      | 88                  |

**4. How many days on average are customers reallocated to a different node?**
```sql
select round(Avg(DATEDIFF(end_date,start_date)),1) as Average_days_nodes
from customer_nodes
where end_date not like '%9999%';
```
**Output:**

| Average_days_nodes        | 
|---------------------------|
| 14.6                      |

**5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?**
```sql
with rows_ as ( select c.customer_id,
	r.region_name, DATEDIFF(c.end_date, c.start_date) AS days_difference,
	row_number() over (partition by r.region_name order by DATEDIFF(c.end_date, c.start_date)) AS rows_number,
	COUNT(*) over (partition by r.region_name) as total_rows  
	from
	customer_nodes c JOIN regions r ON c.region_id = r.region_id
	where c.end_date not like '%9999%')
	SELECT region_name,
	ROUND(AVG(CASE WHEN rows_number between (total_rows/2) and ((total_rows/2)+1) THEN days_difference END), 0) AS Median,
	MAX(CASE WHEN rows_number = round((0.80 * total_rows),0) THEN days_difference END) AS Percentile_80th,
	MAX(CASE WHEN rows_number = round((0.95 * total_rows),0) THEN days_difference END) AS Percentile_95th
	from rows_
	group by region_name;
```
**Output:**

| region_name | Median | Percentile_80th | Percentile_95th |
|-------------|--------|---------------- |---------------- |
| Australia   | 15     | 23              | 28              |
| America     | 15     | 23              | 28              |
| Africa      | 15     | 24              | 28              |
| Asia        | 15     | 23              | 28              |
| Europe      | 15     | 24              | 28              |
