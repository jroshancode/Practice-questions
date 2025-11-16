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

-- 🧠 What This Query Does

It calculates:

1. 1️⃣ prev_three → 3 rows BEFORE the current row (inside the same Ship Mode)
2. 2️⃣ next_three → 3 rows AFTER the current row (inside the same Ship Mode)

But it does this separately for each Ship Mode.


📌 Step-by-step Explanation
PARTITION BY Ship Mode

This groups the data inside each Ship Mode.

Example:

1. “First Class” rows form one group

2. “Second Class” rows form another group

3. Each group is processed independently.

| sales |
| ----- |
| 100   |
| 150   |
| 200   |
| 300   |
| 350   |

🔙 LAG(sales, 3)

“Give me the value of sales from 3 rows above the current row.”
| Row | sales | LAG(sales,3) |
| --- | ----- | ------------ |
| 1   | 100   | NULL         |
| 2   | 150   | NULL         |
| 3   | 200   | NULL         |
| 4   | 300   | **100**      |
| 5   | 350   | **150**      |

Because:

Row 4 → 3 rows back = row 1 → 100

Row 5 → 3 rows back = row 2 → 150

➡️ LEAD(sales, 3)

“Give me the value of sales from 3 rows below the current row.”

| Row | sales | LEAD(sales,3)           |
| --- | ----- | ----------------------- |
| 1   | 100   | **300**                 |
| 2   | 150   | **350**                 |
| 3   | 200   | NULL? (no 3 rows ahead) |
| 4   | 300   | NULL                    |
| 5   | 350   | NULL                    |

Because:

Row 1 → 3 rows ahead = row 4 → 300

Row 2 → 3 rows ahead = row 5 → 350

🎯 Why We Use This?

It is used for:

Trend analysis

Comparing a value with previous/next entries

Time-series analysis

Checking patterns inside groups

Detecting jumps in sales

📊 Final Output Columns

Your result will show:

Category	Sub-Category	Ship Mode	sales	prev_three	next_three

Where:

prev_three = 3 previous sales within same Ship Mode

next_three = 3 next sales within same Ship Mode


SELECT 
    category, 
    new_order_date,
    `Order Date`, 
    sales,
    LAG(sales,1)  OVER(ORDER BY sales DESC) AS pre_date,
    LAG(sales,3)  OVER(ORDER BY sales DESC) AS pre_third,
    LEAD(sales,1) OVER(ORDER BY sales DESC) AS next_date,
    LEAD(sales,2) OVER(ORDER BY sales DESC) AS next_second
FROM train;

🧠 What This Query Is Doing

You're ordering all rows by sales descending, then using:

LAG()

LAG(sales, 1) → Sales from 1 row above

LAG(sales, 3) → Sales from 3 rows above

LEAD()

LEAD(sales, 1) → Sales from 1 row below

LEAD(sales, 2) → Sales from 2 rows below

And this is applied over the entire dataset (no partition).

📌 Step-by-step Understanding

Suppose after ordering by sales DESC, the sorted rows look like:

| Row | sales |
| --- | ----- |
| 1   | 900   |
| 2   | 850   |
| 3   | 800   |
| 4   | 700   |
| 5   | 650   |
| 6   | 500   |

🔙 LAG(sales, 1)
| Row | sales | LAG(1) |
| --- | ----- | ------ |
| 1   | 900   | NULL   |
| 2   | 850   | 900    |
| 3   | 800   | 850    |
| 4   | 700   | 800    |
| 5   | 650   | 700    |
| 6   | 500   | 650    |

🔙 LAG(sales, 3)

| Row | sales | LAG(3)  |
| --- | ----- | ------- |
| 1   | 900   | NULL    |
| 2   | 850   | NULL    |
| 3   | 800   | NULL    |
| 4   | 700   | **900** |
| 5   | 650   | **850** |
| 6   | 500   | **800** |

Because:

Row 4 (700) → 3 rows above = 900

Row 5 (650) → 3 rows above = 850

Row 6 (500) → 3 rows above = 800

➡️ LEAD(sales, 1)
| Row | sales | LEAD(1) |
| --- | ----- | ------- |
| 1   | 900   | 850     |
| 2   | 850   | 800     |
| 3   | 800   | 700     |
| 4   | 700   | 650     |
| 5   | 650   | 500     |
| 6   | 500   | NULL    |

➡️ LEAD(sales, 2)

| Row | sales | LEAD(2) |
| --- | ----- | ------- |
| 1   | 900   | 800     |
| 2   | 850   | 700     |
| 3   | 800   | 650     |
| 4   | 700   | 500     |
| 5   | 650   | NULL    |
| 6   | 500   | NULL    |


🎯 What You Achieve With This Query

For every row, you get:

| Column      | Meaning                    |
| ----------- | -------------------------- |
| pre_date    | Sales value 1 step before  |
| pre_third   | Sales value 3 steps before |
| next_date   | Sales value 1 step after   |
| next_second | Sales 2 steps after        |


📊 Example Output (simplified)
| category    | Order Date | sales | pre_date | pre_third | next_date | next_second |
| ----------- | ---------- | ----- | -------- | --------- | --------- | ----------- |
| Chairs      | 2020-01-12 | 900   | NULL     | NULL      | 850       | 800         |
| Phones      | 2020-01-05 | 850   | 900      | NULL      | 800       | 700         |
| Tables      | 2020-02-01 | 800   | 850      | NULL      | 700       | 650         |
| Bookcases   | 2020-03-07 | 700   | 800      | 900       | 650       | 500         |
| Accessories | 2020-04-08 | 650   | 700      | 850       | 500       | NULL        |
| Supplies    | 2020-06-10 | 500   | 650      | 800       | NULL      | NULL        |









