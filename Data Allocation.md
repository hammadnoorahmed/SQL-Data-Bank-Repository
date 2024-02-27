### C. Data Allocation Challenge

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- **Option 1:** data is allocated based on the amount of money at the end of the previous month
- **Option 2:** data is allocated on the average amount of money kept in the account in the previous 30 days
- **Option 3:** data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact of each transaction
- customer balance at the end of each month
- minimum, average, and maximum values of the running balance for each customer

Using all of the data available - how much data would have been required for each option on a monthly basis?


- **running customer balance column that includes the impact of each transaction**

Using the below query we are finding the impact of each transaction on the balance of customer.

```sql
Select customer_id, txn_date, txn_type, txn_amount, sum(case
when txn_type = 'deposit' then txn_amount else 0 - txn_amount end) 
over (partition by customer_id order by customer_id ASC, txn_date ASC) as CustomerBalance 
from customer_transactions;
```

**Output:**
| customer_id | txn_date   | txn_type   | txn_amount | CustomerBalance |
|-------------|------------|------------|------------|-----------------|
| 1           | 2020-01-02 | deposit    | 312        |312              |
| 1           | 2020-03-05 | purchase   | 612        |-300             |
| 1           | 2020-03-17 | deposit    | 324        |24               |
| 1           | 2020-03-19 | purchase   | 664        |-640             |
| 2           | 2020-01-03 | deposit    | 549        |549              |
| 2           | 2020-03-04 | deposit    | 61         |610              |
| 3           | 2020-01-27 | deposit    | 144        |144              |
| 3           | 2020-02-22 | purchase   | 965        |-821             |
| 3           | 2020-03-05 | withdrawal | 213        |-1034            |

Please note that this output is limited to save up on space.

- **customer balance at the end of each month**

```sql
Select customer_id, month(txn_date) as month_, case 
when month(txn_date) = 1 then 'Jan'
when month(txn_date) = 2 then 'Feb'
when month (txn_date) = 3 then 'Mar' Else 'Apr'
End as Month_name,
sum(case when txn_type = 'deposit' then txn_amount
when txn_type = 'withdrawal' then -txn_amount
when txn_type = 'purchase' then -txn_amount 
else 0 end) as current_balance
from customer_transactions
group by customer_id, Month_name, month_
Order by customer_id, month_;
```

**Output:** 
| customer_id | month_ |Month_name| current_balance |
|-------------|--------|----------|-----------------|
| 1           |1       |Jan       |312              |
| 1           |3       |Mar       |-952             |
| 2           |1       |Jan       |549              |
| 2           |3       |Mar       |61               |
| 3           |1       |Jan       |144              |
| 3           |2       |Feb       |-965             |
| 3           |3       |Mar       |-401             |
| 3           |4       |Apr       |493              |

Please note that this output is limited to save up on space.

- **minimum, average, and maximum values of the running balance for each customer**

```sql
with running_balance as
(
Select
customer_id, txn_type, txn_amount, txn_date, sum(case 
when txn_type = 'deposit' then txn_amount
when txn_type = 'withdrawal' then -txn_amount
when txn_type = 'purchase' then -txn_amount 
else 0 end) over (partition by customer_id order by txn_date) as runningbalance
from customer_transactions)
Select customer_id,
Min(runningbalance) as lowest_balance,
Max(runningbalance) as highest_balance,
round(Avg(runningbalance),0) as average_balance
from running_balance 
group by customer_id
order by customer_id;
```

**Output:**
| customer_id | lowest_balance          | highest_balance         | average_balance |
|-------------|-------------------------|-------------------------|-----------------|
| 1           | -640                    | 312                     | -151            |
| 2           | 549                     | 610                     | 579             |
| 3           | -1222                   | 144                     | -732            |
| 4           | 458                     | 848                     | 653             |
| 5           | -2413                   | 1780                    | -135            |
| 6           | -552                    | 2197                    | 624             |
| 7           | 887                     | 3539                    | 2269            |

Please note that this output is limited to save up on space.

Using all of the data available - how much data would have been required for each option on a monthly basis?

- **Option 1: Data is allocated based on the amount of money at the end of the previous month**
```sql
with monthlybalance 
as(
select customer_id, month(txn_date) as month_, sum(case
when txn_type = 'deposit' then txn_amount else -1*txn_amount end) as balance
from customer_transactions
group by customer_id, month_
),
closing_balance as 
(select month_, customer_id,
sum(balance) over (partition by customer_id order by month_) as closing_balance
from monthlybalance),
previousmonthbalance as 
(select customer_id, month_, 
last_value(closing_balance) over (partition by customer_id, month_ order by month_ ASC) as previous_balance
from closing_balance)
Select month_, sum(case when previous_balance > 0 then previous_balance else 0 end) as data_required 
from previousmonthbalance 
group by month_
order by month_ ASC;
```

**Output:**
| month_ | data_required |
|--------|---------------|
| 1      | 235595        |
| 2      | 238492        |
| 3      | 240065        |
| 4      | 157033        |


- **Option 2: Data is allocated on the average amount of money kept in the account in the previous 30 days**

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
Averagebalance as 
(select customer_id, month_,
avg(closing_balance) over (partition by customer_id) as average_balance
from closing_balance
group by customer_id, month_)
select month_, round(sum(case when average_balance>0 then average_balance else 0 end),0) as data_required
from Averagebalance
group by month_
order by month_;
```

**Output:**

| month_ | data_required |
|--------|---------------|
| 1      | 218197        |
| 2      | 196413        |
| 3      | 201383        |
| 4      | 124976        |


- **Option 3: Data is updated real-time**

```sql
with balance as
(
Select customer_id, txn_date, sum(case
when txn_type = 'deposit' then txn_amount else 0 - txn_amount end) 
over (partition by customer_id order by customer_id ASC, txn_date ASC) as CustomerBalance 
from customer_transactions)
Select month(txn_date) as month_,
sum(case when CustomerBalance > 0 then CustomerBalance else 0 end) as data_required
from balance
group by month(txn_date)
order by month(txn_date);
```

**Output:**

| month_ | data_required |
|--------|---------------|
| 1      | 717947        |
| 2      | 959673        |
| 3      | 934717        |
| 4      | 412635        |
