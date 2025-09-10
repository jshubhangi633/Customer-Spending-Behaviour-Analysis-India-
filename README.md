# Customer-Spending-Behaviour-Analysis-India-
This SQL project analyzes credit card transactions across various cities and card types in India. The analysis focuses on user spending patterns, trends across expense types, and interesting business questions.

# Key Business Questions Solved
# 
## 1	Top 5 cities with highest spends and % contribution
## 2	Highest spend month for each card type
## 3	Transaction when each card reached ₹1M cumulative
## 4	Female spend contribution by expense type
## 5	Weekend city with highest spend/transaction ratio

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

## 4- Female spend contribution by expense type
```sql
SELECT exp_type,
       SUM(CASE WHEN gender = 'F' THEN amount ELSE 0 END) / SUM(amount) AS per_female_contribution
FROM credit_card_transactions
GROUP BY exp_type
ORDER BY per_female_contribution DESC;
```

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


