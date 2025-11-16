# 2025 - 11 - 15 

<img width="303" height="166" alt="image" src="https://github.com/user-attachments/assets/c5abc224-b538-4b28-a234-d537bd5e676c" />

# 2025 - 11 - 16 
-- For each category, find the top 7 days where total sales amount was the highest.
SELECT 
    category,
    order_date,
    daily_sales
FROM (
    SELECT 
        category,
        order_date,
        SUM(amount) AS daily_sales,
        ROW_NUMBER() OVER (
            PARTITION BY category 
            ORDER BY SUM(amount) DESC
        ) AS rn
    FROM orders
    GROUP BY category, order_date
) t
WHERE rn <= 7
ORDER BY category, daily_sales DESC;

