## B. Customer Transactions
**1. What is the unique count and total amount for each transaction type?**
```sql
Select txn_type, count(txn_amount) as Transaction_Total, sum(txn_amount) as total_transaction_amount
from customer_transactions
group by txn_type;
```

**Output:**

| txn_type   | Transaction_Total   | total_transaction_amount |
|------------|---------------------|--------------------------|
| deposit    | 2671                | 1359168                  |
| withdrawal | 1580                | 793003                   |
| purchase   | 1617                | 806537                   |


**2. What is the average total historical deposit counts and amounts for all customers?**

```sql
With total_deposits as
(
Select customer_id, txn_type, count(txn_type) as Total_deposit, sum(txn_amount) as total_transaction_amount
from customer_transactions
where txn_type = 'deposit'
group by customer_id, txn_type)
Select txn_type, round(avg(Total_deposit),0) as avg_total_deposit,
round(avg(total_transaction_amount),1) as avg_deposit_amount
from total_deposits
group by txn_type;
```

**Output:**

| txn_type | avg_total_deposit     | average_deposit_amount |
|----------|-----------------------|------------------------|
| deposit  | 5                     | 2718.3                 |


**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

```sql
with customer_account
as
(
SELECT 
  customer_id, Month(txn_date) AS month_,
  count(CASE WHEN txn_type = 'Deposit' THEN 1 END) AS deposits,
  count(CASE WHEN txn_type = 'Purchase' THEN 1 END) AS purchases,
  count(CASE WHEN txn_type = 'Withdrawal' THEN 1 END) AS withdrawals
FROM customer_transactions
GROUP BY customer_id, month_
)
Select case 
when month_ = 1 then 'Jan'
when month_ = 2 then 'Feb' 
when month_ = 3 then 'Mar' else 'Apr' end as Month_name, 
count(distinct customer_id) as active_customer_accounts
from customer_account
where deposits > 1 AND (purchases > 0 OR withdrawals > 0)
group by month_
order by active_customer_accounts DESC;
```

**Output:**

| Month_name| active_customer_accounts   |
|-----------|----------------------------|
| Mar       | 192                        |
| Feb       | 181                        |
| Jan       | 168                        |
| Apr       | 70                         |


**4. What is the closing balance for each customer at the end of the month?**

```sql
with Balance as
(
select customer_id, Month(txn_date) as month_, case 
when month(txn_date) =1 then 'Jan'
when month(txn_date) =2 then 'Feb'
When month(txn_date) =3 then 'Mar'
Else 'Apr' End as Month_name, sum(case
when txn_type = 'deposit' then txn_amount else -1*txn_amount end) as amount
from customer_transactions
group by customer_id, month_, Month_name
Order by customer_id ASC)
Select customer_id, month_, month_name, sum(amount)
over (partition by customer_id order by month_) as closing_balance
from Balance; 

```

**Output:**

| customer_id | month_ | month_name|closing_balance|
|-------------|--------|-----------|---------------|
| 1           | 1      | Jan       |312            |
| 1           | 3      | Mar       |-640           |
| 2           | 1      | Jan       |549            |
| 2           | 3      | Mar       |610            |
| 3           | 1      | Jan       |144            |
| 3           | 2      | Feb       |-821           |


Please note that this output is limited to save the space.
**5. What is the percentage of customers who increase their closing balance by more than 5%?**

```sql
with monthlybalance -- cte for monthly balance as we did above
as(
select month(txn_date) as month_, customer_id, sum(case
when txn_type = 'deposit' then txn_amount else -1*txn_amount end) as balance
from customer_transactions
group by customer_id, month_
),
closing_balance as 
(select month_, customer_id,
sum(balance) over (partition by customer_id order by month_) as closing_balance
from monthlybalance),
percentgrowth -- cte for finding percent growth
as
(Select month_, customer_id, closing_balance, 
lag(closing_balance) over(partition by customer_id order by month_) as previous_balance,
((closing_balance - lag(closing_balance) over(partition by customer_id order by month_)) / lag(closing_balance) over(partition by customer_id order by month_))*100 as Percent_growth
from closing_balance)  -- cte for table containing values only for growth
Select  -- finding only percentage of customers with having more than five percent increase in their monthly closing balances
round((count(distinct customer_id) / (select count(distinct customer_id) from monthlybalance)) * 100,1) as percentage_customers_more5pct_increase
from 
percentgrowth
where percent_growth > 5;

```

**Output:**

| percentage_of_customers |
|-------------------------|
| 75.8                    |
