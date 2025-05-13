# wm-sales-analysis

Exploring a dataset of over 1000 Walmart sales, this project uses Python along with the _Pandas_ library to process and clean the data, along with _SQL_ via _MySQL_ software to form impactful insights commonly used in the business world. Using _Jupyter Notebook_, this project interactively cleaned up the data, which was then sent over to _MySQL_ via a _sqlmyalchemy_ import _create_engine_. Below overviews some of the conclusions this project aims to discover. 

## Results and Insights 

### Identifying which payment methods dominate, and how many items move through each. 
This has use cases in terms of negotiating process fees for crewdit cards, and identifying where to invest promotional incentives.

```sq
SELECT
  payment_method,
  COUNT(*) AS transactions,
  SUM(quantity) AS total_units_sold
FROM walmart
GROUP BY payment_method;
```

### Identifying each Walmart branch's highest rated product categories.
This allows decision makers to tailor inventory and prioritize promotions on a local basis to cater to unique customer needs.
```sq
SELECT branch, category, avg_rating
FROM (
  SELECT
    branch,
    category,
    AVG(rating) AS avg_rating,
    RANK() OVER (
      PARTITION BY branch
      ORDER BY AVG(rating) DESC
    ) AS rnk
  FROM walmart
  GROUP BY branch, category
) AS ranked
WHERE rnk = 1;
```

### Which day of the week do Walmart stores have the highest foot traffic?
This allows for the optimization of staff levels, restocking, and promotion incentives to correspond with peak shopping days.
```sq
SELECT branch, day_name, transaction_count
FROM (
  SELECT
    branch,
    DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
    COUNT(*) AS transaction_count,
    RANK() OVER (
      PARTITION BY branch
      ORDER BY COUNT(*) DESC
    ) AS rnk
  FROM walmart
  GROUP BY branch, day_name
) AS daily_rank
WHERE rnk = 1;
```

### Which categories generate the most profit?
Identifying high profit margin items can allow for a priortization of products in terms of upselling, bundling and target marketing.
```sq
SELECT
  category,
  SUM(unit_price * quantity * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
```
### How does customer activity change as the time of day shifts?
This allows for branches to prepare and optimize their stores based on busy shopping windows, maximizing their oppurtunities of succsess.
```sq
SELECT
  branch,
  CASE
    WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
    WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
    ELSE 'Evening'
  END AS shift,
  COUNT(*) AS invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, invoices DESC;
```

### Which branches are experiences year-over-year revenue declines, and how steep are these declines?
Singling out underperforming locations allows for a deeper competitive analysis, and possible corrective action.
```sq
WITH revenue_last_year AS (
  SELECT branch, SUM(purchase_amnt) AS revenue
  FROM walmart
  WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2022
  GROUP BY branch
), revenue_this_year AS (
  SELECT branch, SUM(purchase_amnt) AS revenue
  FROM walmart
  WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2023
  GROUP BY branch
)
SELECT
  r1.branch,
  r1.revenue AS rev_2022,
  r2.revenue AS rev_2023,
  ROUND(((r1.revenue - r2.revenue) / r1.revenue) * 100, 2)
    AS pct_decline
FROM revenue_last_year r1
JOIN revenue_this_year r2
  ON r1.branch = r2.branch
WHERE r2.revenue < r1.revenue
ORDER BY pct_decline DESC
LIMIT 5;
```


