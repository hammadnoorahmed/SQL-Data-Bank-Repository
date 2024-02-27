### D. Extra Challenge

Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

#### Special notes:
- Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!

```sql
with dailybalance 
as
(
Select customer_id, txn_date, txn_amount, 
case	when txn_type = 'deposit' then txn_amount 
	else -1* txn_amount 
end as CustomerBalance 
from customer_transactions
order by customer_id),
daily_interest as
(Select 
customer_id, txn_date, txn_amount, CustomerBalance*(POWER(1+ (0.06/365),1)) as Interest_amount
from dailybalance)
Select month(txn_date),
round(sum(case when Interest_amount > 0 then Interest_amount else 0 end),1) as data_required
from daily_interest
group by month(txn_date)
order by month(txn_date);
```

**Output:**

| month_    | data_required |
|-----------|---------------|
| 1         | 437966        |
| 2         | 357098.7      |
| 3         | 390167.1      |
| 4         | 174159.6      |
