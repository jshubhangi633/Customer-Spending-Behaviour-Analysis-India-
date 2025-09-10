# Customer-Spending-Behaviour-Analysis-India-
This SQL project analyzes credit card transactions across various cities and card types in India. The analysis focuses on user spending patterns, trends across expense types, and interesting business questions.

## Key Business Questions Solved
# 
### 1	Top 5 cities with highest spends and % contribution
### 2	Highest spend month for each card type
### 3	Transaction when each card reached ₹1M cumulative
### 4	Female spend contribution by expense type
### 5	Weekend city with highest spend/transaction ratio

## 1-  Top 5 cities with highest spends and % contribution
```sql 
WITH cte AS (
    SELECT city,
           SUM(amount) AS total_spend
    FROM credit_card_transcations
    GROUP BY city
),
amt_sum AS (
    SELECT SUM(total_spend) AS total_amount FROM cte
)
SELECT c.city,
       c.total_spend,
       ROUND(c.total_spend / a.total_amount * 100, 2) AS per_contribution
FROM cte c
JOIN amt_sum a ON 1=1
ORDER BY c.total_spend DESC
LIMIT 5;
```
### Result - city	            total_spend ,       	per_contribution
### Greater Mumbai, India	        576718921.00,	    14.15
### Bengaluru, India	            572326739.00,	      14.05
### Ahmedabad, India	567794310.00,	13.93
### Delhi, India	556929212.00,	13.67
### Kolkata, India	115466943.00,	2.83


## 2- Highest spend month for each card type

```sql
WITH yr_mnt AS (
    SELECT card_type,
           YEAR(transaction_date) AS yr,
           MONTH(transaction_date) AS mnt,
           SUM(amount) AS sales
    FROM credit_card_transcations
    GROUP BY card_type, YEAR(transaction_date), MONTH(transaction_date)
)
SELECT card_type, yr, mnt, sales
FROM (
    SELECT *,
           RANK() OVER (PARTITION BY card_type ORDER BY sales DESC) AS rn
    FROM yr_mnt
) a
WHERE rn = 1;
```
### Result- mnt	, sales
### Gold	,	984539536.00
### Platinum,		1007606464.00
### Signature	,	1013041105.00
### Silver, 	1069613713.00
## 3- Transaction details when cumulative spend ≥ 1,000,000
```sql
WITH cte AS (
    SELECT c.*,
           SUM(amount) OVER (PARTITION BY card_type ORDER BY transaction_date, transaction_id) AS total_spend
    FROM credit_card_transactions c
)
SELECT *
FROM (
    SELECT *,
           RANK() OVER (PARTITION BY card_type ORDER BY total_spend) AS rn
    FROM cte
    WHERE total_spend >= 1000000
) a
WHERE rn = 1;
```

## Result
### 13431	Delhi, India	1-Apr-14	Gold	Grocery	M	23697.00	1000956.00	1
### 18394	Surandai, India	1-Apr-14	Platinum	Entertainment	F	290618.00	1060394.00	1


## 4- Female spend contribution by expense type
```sql
SELECT exp_type,
       SUM(CASE WHEN gender = 'F' THEN amount ELSE 0 END) / SUM(amount) AS per_female_contribution
FROM credit_card_transactions
GROUP BY exp_type
ORDER BY per_female_contribution DESC;
```
### Result- exp_type	per_female_contribution
### Bills	0.639446
### Food	0.549053
### Travel	0.511329
### Grocery	0.509110
### Fuel	0.497104
### Entertainment	0.493729

## 5-Weekend city with highest spend/transaction ratio
```sql
SELECT city,
       SUM(amount) / COUNT(*) AS ratio
FROM credit_card_transcations
WHERE DAYOFWEEK(transaction_date) IN (1, 7) -- Sunday=1, Saturday=7 in MySQL
GROUP BY city
ORDER BY ratio DESC
LIMIT 1;
```
## Result 
### Delhi has the highest weekend spend efficiency.


