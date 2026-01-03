# ☕ Coffee Shop SQL Analytics Project

## 📌 Project Overview
This project showcases **end-to-end SQL data analysis** on a fictional **coffee shop business database**, designed to reflect real-world analytical scenarios faced by **Data Analysts and Business Intelligence professionals**.  

The repository contains **business-driven SQL questions and solutions** covering workforce management, sales performance, customer behavior, inventory forecasting, and employee productivity.  
It is ideal for:
- Aspiring **Data Analysts**
- **SQL interview preparation**
- Portfolio demonstration for **Analytics & BI roles**

---

## 🧠 Skills & Concepts Covered
- SQL Joins (INNER, LEFT)
- Aggregations & Grouping
- Window Functions (`RANK`, `DENSE_RANK`, `ROW_NUMBER`)
- Common Table Expressions (CTEs)
- Subqueries
- Date & Time Functions
- Conditional Logic (`CASE WHEN`)
- String Aggregations
- Views
- Business KPI calculations
- Ranking & Segmentation
- Pivot-style analysis

---

## 🗂 Dataset Overview
The project assumes a **business-style relational database** for a coffee shop, including:
- `staff` – employee details  
- `coffeeshop` – shift assignments  
- `shift` – shift timings  
- `rota` – staff scheduling  
- `orders` – customer orders  
- `menu_items` – product catalog  
- `ingredients` & `inventory` – stock tracking  

The schema supports **operational, sales, and workforce analytics**.

---

## 🧩 Analysis Sections & SQL Solutions

### 🔹 1. Employee Workload & Shift Management  
**Difficulty:** Intermediate  

#### Q1. Weekly Employee Working Hours  
**Business Problem:**  
Management needs to track how many hours each employee works per week to ensure fair scheduling.

**SQL Approach:**  
- Join staff, shift, and assignment tables  
- Aggregate shift durations weekly using `DATE_TRUNC`

```sql
SELECT sf.staff_id, sf.first_name, sf.last_name,
       DATE_TRUNC('week', cs.date) AS week_start,
       SUM(s.end_time - s.start_time) AS total_worked_hours
FROM staff sf
JOIN coffeeshop cs ON sf.staff_id = cs.staff_id
JOIN shift s ON cs.shift_id = s.shift_id
GROUP BY 1,2,3,4
ORDER BY sf.staff_id;

```

#### Q2. Identify Overtime Employees  
**Business Problem:** 
Detect employees exceeding 25 working hours per week.

**SQL Approach:**  
- Reuse weekly aggregation  
- Filter using a subquery and interval comparison

```sql
SELECT *
FROM (
  SELECT sf.staff_id, sf.first_name, sf.last_name,
         DATE_TRUNC('week', cs.date) AS week_start,
         SUM(s.end_time - s.start_time) AS total_worked_hours
  FROM staff sf
  JOIN coffeeshop cs ON sf.staff_id = cs.staff_id
  JOIN shift s ON cs.shift_id = s.shift_id
  GROUP BY 1,2,3,4
) t
WHERE total_worked_hours > INTERVAL '25 HOURS';

```

#### Q3. Rank Employees by Workload 
**Business Problem:** 
Identify top-working employees for performance and burnout analysis.

**SQL Approach:**  
- Use CTE for weekly hours 
- Apply `DENSE_RANK()` for ranking

```sql
WITH emp_worked_hours AS (
  SELECT sf.staff_id, sf.first_name, sf.last_name,
         DATE_TRUNC('week', cs.date) AS week_start,
         SUM(s.end_time - s.start_time) AS total_worked_hours
  FROM staff sf
  JOIN coffeeshop cs ON sf.staff_id = cs.staff_id
  JOIN shift s ON cs.shift_id = s.shift_id
  GROUP BY 1,2,3,4
)
SELECT *,
       DENSE_RANK() OVER (ORDER BY total_worked_hours DESC) AS rank_top_working_employees
FROM emp_worked_hours;

```

#### Q4. Optimize Shift Allocation 
**Business Problem:** 
Suggest reallocation between overworked and underworked staff.

**SQL Approach:**  
- Calculate total hours per employee 
- Segment into overworked vs underworked using CTEs

```sql
WITH EmployeeHours AS (
  SELECT sf.staff_id, sf.first_name, sf.last_name,
         SUM(EXTRACT(EPOCH FROM (s.end_time - s.start_time)) / 3600) AS total_worked_hours
  FROM staff sf
  JOIN coffeeshop cs ON sf.staff_id = cs.staff_id
  JOIN shift s ON cs.shift_id = s.shift_id
  GROUP BY 1,2,3
),
OverWorked AS (
  SELECT * FROM EmployeeHours WHERE total_worked_hours > 25
),
UnderWorked AS (
  SELECT * FROM EmployeeHours WHERE total_worked_hours < 25
)
SELECT o.staff_id AS overworked_id, u.staff_id AS underworked_id,
       'Consider Shift reallocation' AS suggestion
FROM OverWorked o
CROSS JOIN UnderWorked u;

```
### 🔹 2. Preventing Shift Overlaps & Scheduling Optimization  

#### Q5. Detect Overlapping Shifts  
**Business Problem:**  
Identify scheduling conflicts where multiple employees are assigned to the same shift.

**SQL Approach:**  
- Group by shift and date  
- Use `HAVING COUNT(*) > 1`

```sql
WITH Overlappingshifts AS (
  SELECT cs.shift_id, cs.date, s.start_time, s.end_time,
         STRING_AGG(st.first_name || ' ' || st.last_name, ' | ') AS employees,
         COUNT(*) AS employees_count
  FROM coffeeshop cs
  JOIN shift s ON cs.shift_id = s.shift_id
  JOIN staff st ON cs.staff_id = st.staff_id
  GROUP BY cs.shift_id, cs.date, s.start_time, s.end_time
  HAVING COUNT(*) > 1
)
SELECT * FROM Overlappingshifts
ORDER BY date, shift_id;

```

#### Q6. Identify shifts with insufficient staff  
**Business Problem:**  
Identify scheduling conflicts where multiple employees are assigned to the same shift.

**SQL Approach:**  
- Group employees per shift
- Filter shifts having one or zero employees

```sql
SELECT cs.shift_id, cs.date, s.start_time, s.end_time,
       STRING_AGG(st.first_name || ' ' || st.last_name, ' | ') AS employees,
       COUNT(*) AS employees_count
FROM coffeeshop cs
JOIN shift s ON cs.shift_id = s.shift_id
JOIN staff st ON cs.staff_id = st.staff_id
GROUP BY cs.shift_id, cs.date, s.start_time, s.end_time
HAVING COUNT(*) <= 1;

```

### 🔹 3. Sales & Revenue Analysis  

#### Q7.Busiest Sales Hours 
**Business Problem:**  
Understand peak business hours to optimize staffing.

**SQL Approach:**  
- Extract hour from order timestamps  
- Aggregate total revenue by hour

```sql
SELECT cs.shift_id, cs.date, s.start_time, s.end_time,
       STRING_AGG(st.first_name || ' ' || st.last_name, ' | ') AS employees,
       COUNT(*) AS employees_count
FROM coffeeshop cs
JOIN shift s ON cs.shift_id = s.shift_id
JOIN staff st ON cs.staff_id = st.staff_id
GROUP BY cs.shift_id, cs.date, s.start_time, s.end_time
HAVING COUNT(*) <= 1;

```
#### Q8.Monthly revenue KPI view 
**Business Problem:**  
Track key metrics: revenue, orders, AOV by month for financial reporting.

**SQL Approach:**  
- Aggregate revenue, quantity, and order counts by month
- Store results in a reusable SQL view

```sql
CREATE VIEW monthly_kpis AS
SELECT EXTRACT(MONTH FROM o.created_at::TIMESTAMP) AS month,
       SUM(o.quantity * mi.item_price) AS revenue_per_month,
       SUM(o.quantity) AS orders_per_month,
       ROUND(SUM(o.quantity * mi.item_price) / COUNT(DISTINCT o.order_id), 2) AS avg_order_value
FROM orders o
JOIN menu_items mi ON o.item_id = mi.item_id
GROUP BY month
ORDER BY month;

```

#### Q9.Most profitable product category
**Business Problem:**  
Rank categories by total revenue for inventory and promotion planning.

**SQL Approach:**  
- Aggregate revenue by product category
- Sort and select the top category

```sql
SELECT mi.item_cat,
       SUM(o.quantity * mi.item_price) AS revenue
FROM menu_items mi
JOIN orders o ON mi.item_id = o.item_id
GROUP BY mi.item_cat
ORDER BY revenue DESC
LIMIT 1;

```

### 🔹 4. Customer Order Patterns & Retention 

#### Q10.Customers ordering at least 14 times per week 
**Business Problem:**  
Understand peak business hours to optimize staffing.

**SQL Approach:**  
- Group orders by customer and week  
- ilter high-frequency customers using `HAVING`

```sql
SELECT cust_name,
       EXTRACT(WEEK FROM created_at::TIMESTAMP) AS week,
       COUNT(*) AS total_orders
FROM orders
GROUP BY cust_name, week
HAVING COUNT(*) >= 14;

```

#### Q11.Customers inactive in the last 30 daysk 
**Business Problem:**  
Understand peak business hours to optimize staffing.

**SQL Approach:**  
- Identify active customers in the last 30 days  
- Exclude them using a subquery

```sql
SELECT cust_name,
       EXTRACT(WEEK FROM created_at::TIMESTAMP) AS week,
       COUNT(*) AS total_orders
FROM orders
GROUP BY cust_name, week
HAVING COUNT(*) >= 14;

-- Lets say we are writing this query on 2024/03/15
```

#### Q12.Preferred ordering time of day
**Business Problem:**  
Determine preferred order times like morning, afternoon and evening.

**SQL Approach:**  
- Bucket order times using CASE  
- Count orders per time segment

```sql
SELECT
    COUNT(CASE WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 5 AND 10 THEN 1 END) AS morning,
    COUNT(CASE WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 12 AND 15 THEN 1 END) AS afternoon,
    COUNT(CASE WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 17 AND 19 THEN 1 END) AS evening,
    COUNT(*) AS total
FROM orders;

```

### 🔹 5. Customer Order Patterns & Retention 

#### Q13.Top 5 best-selling items 
**Business Problem:**  
Understand peak business hours to optimize staffing.

**SQL Approach:**  
- Aggregate item quantities
- Rank items by sales volume

```sql
SELECT mi.item_name,
       SUM(o.quantity) AS quantity_sold,
       SUM(o.quantity * mi.item_price) AS revenue
FROM menu_items mi
JOIN orders o ON mi.item_id = o.item_id
GROUP BY mi.item_name
ORDER BY quantity_sold DESC
LIMIT 5;

```
#### Q14.Least-selling items with recommendations
**Business Problem:**  
Find least-selling items and suggest potential removal or discounts.

**SQL Approach:**  
- Aggregate item sales
- Use `CASE` logic for business recommendations

```sql
SELECT mi.item_name,
       SUM(o.quantity) AS quantity_sold,
       CASE
           WHEN SUM(o.quantity) < 5 THEN 'Removal'
           WHEN SUM(o.quantity) <= 15 THEN 'Discount'
           ELSE 'Keep'
       END AS recommendation
FROM menu_items mi
JOIN orders o ON mi.item_id = o.item_id
GROUP BY mi.item_name
ORDER BY quantity_sold;

```

#### Q15.Marketing focus recommendations
**Business Problem:**  
 Identify best-selling items so far and recommend focus areas for marketing campaigns.

**SQL Approach:**  
- Segment products by sales volume
- Assign marketing priority using conditional logic

```sql
SELECT mi.item_name,
       SUM(o.quantity) AS total_quantity,
       CASE
           WHEN SUM(o.quantity) >= 30 THEN 'High Priority'
           WHEN SUM(o.quantity) BETWEEN 20 AND 29 THEN 'Medium Priority'
           ELSE 'Low Priority'
       END AS marketing_focus
FROM menu_items mi
JOIN orders o ON mi.item_id = o.item_id
GROUP BY mi.item_name;

```
### 🔹 6. Inventory Forecasting 

#### Q16.Ingredients running low
**Business Problem:**  
List all ingredients that are running low in inventory (quantity less than 5)

**SQL Approach:**  
- Aggregate inventory quantities per ingredient
- Filter items below threshold level

```sql
WITH ing_quantity AS (
    SELECT ing.ing_name,
           SUM(inv.quantity) AS quantity_left
    FROM ingredients ing
    JOIN inventary inv ON ing.ing_id = inv.ing_id
    GROUP BY ing.ing_name
)
SELECT *
FROM ing_quantity
WHERE quantity_left < 5;

```

#### Q17.Total shifts worked per employee
**Business Problem:**  
Estimate the number of shifts a staff member has worked since the beginning of the year.

**SQL Approach:**  
- Count shifts per staff member
- Filter from beginning of the year

```sql
SELECT staff_id, COUNT(*) AS total_shifts
FROM rota
WHERE date >= '2024-01-01'
GROUP BY staff_id;

```

#### Q18.Frequently ordered item combinations
**Business Problem:**  
Identify Frequently Ordered Menu Item Chains like Coffee -> Muffin -> Cookies.

**SQL Approach:**  
- Self-join orders table by order_id
- Create item chains and count frequency

```sql
SELECT mi1.item_name || ' -> ' || mi2.item_name AS item_chain,
       COUNT(*) AS frequency
FROM orders o1
JOIN orders o2 ON o1.order_id = o2.order_id AND o1.item_id < o2.item_id
JOIN menu_items mi1 ON o1.item_id = mi1.item_id
JOIN menu_items mi2 ON o2.item_id = mi2.item_id
GROUP BY item_chain
ORDER BY frequency DESC;

```

### 🔹 7. Customer Segmentation & Loyalty

#### Q19.Loyalty-eligible customers
**Business Problem:**  
 Find customers with 10+ orders spread over 5 or more days for loyalty rewards.

**SQL Approach:**  
- Count total orders and distinct order days
- Apply thresholds using `HAVING`

```sql
SELECT cust_name,
       COUNT(*) AS total_orders,
       COUNT(DISTINCT created_at::DATE) AS order_days
FROM orders
GROUP BY cust_name
HAVING COUNT(*) >= 10
   AND COUNT(DISTINCT created_at::DATE) >= 5;

```

#### Q20.Most popular items by time of day
**Business Problem:**  
 Which menu items are most popular by time of day (morning, afternoon, evening)?

**SQL Approach:**  
- Categorize orders by time of day
- Rank items within each time segment
  
```sql
WITH ranked_items AS (
    SELECT item_id,
           CASE
               WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 5 AND 11 THEN 'Morning'
               WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 12 AND 16 THEN 'Afternoon'
               WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 17 AND 20 THEN 'Evening'
           END AS time_of_day,
           COUNT(*) AS order_count,
           ROW_NUMBER() OVER (
               PARTITION BY
               CASE
                   WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 5 AND 11 THEN 'Morning'
                   WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 12 AND 16 THEN 'Afternoon'
                   WHEN EXTRACT(HOUR FROM created_at::TIMESTAMP) BETWEEN 17 AND 20 THEN 'Evening'
               END
               ORDER BY COUNT(*) DESC
           ) AS rank
    FROM orders
    GROUP BY item_id, time_of_day
)
SELECT *
FROM ranked_items
WHERE rank = 1;

```

### 🔹 8. Employee Performance & Sales Contribution

#### Q21.Employees working during highest revenue shifts
**Business Problem:**  
 Find employees working during the highest-revenue shifts.
 
**SQL Approach:**  
- Calculate revenue per shift
- Identify shifts with maximum revenue
  
```sql
WITH shift_revenue AS (
    SELECT r.shift_id, r.date,
           SUM(o.quantity * mi.item_price) AS revenue
    FROM orders o
    JOIN menu_items mi ON o.item_id = mi.item_id
    JOIN rota r ON o.created_at::date = r.date
    JOIN shift s ON r.shift_id = s.shift_id
    WHERE o.created_at::time BETWEEN s.start_time AND s.end_time
    GROUP BY r.shift_id, r.date
)
SELECT *
FROM shift_revenue
ORDER BY revenue DESC
LIMIT 1;

```

#### Q22.Employees working during highest revenue shifts
**Business Problem:**  
 Find employees working during the highest-revenue shifts.
 
**SQL Approach:**  
- Attribute orders to employees using shift timing
- Aggregate revenue per employee
  
```sql
WITH employee_revenue AS (
    SELECT r.staff_id,
           SUM(o.quantity * mi.item_price) AS total_revenue
    FROM orders o
    JOIN menu_items mi ON o.item_id = mi.item_id
    JOIN rota r ON o.created_at::date = r.date
    JOIN shift s ON r.shift_id = s.shift_id
    WHERE o.created_at::time BETWEEN s.start_time AND s.end_time
    GROUP BY r.staff_id
)
SELECT staff_id,
       total_revenue,
       RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank
FROM employee_revenue;

```

---

## ▶️ How to Use This Project

1. Load the database schema and tables into **PostgreSQL**.
2. Ensure all **date and time columns** are correctly cast.
3. Execute SQL queries **section by section** as documented.
4. *(Optional)* Export query results to **Power BI, Tableau, or Excel** for visualization.

---

## 🧩 Assumptions Made

- Orders are mapped to employee shifts based on **order time**.
- Inventory quantities reflect **current available stock**.
- Business rules (e.g., **25-hour overtime threshold**) are assumed.
- One row in the `orders` table represents **one item per order**.

---

## 📈 Key Insights & Business Value

- Improved workforce scheduling and reduced overtime costs.
- Identified peak sales hours and high-performing products.
- Enabled targeted customer retention strategies.
- Reduced stockout risk through proactive inventory monitoring.
- Quantified individual employee contribution to revenue.

---

## 🛠 Tools & Technologies

- **Database:** PostgreSQL  
- **Language:** SQL  
- **Use Case:** Retail / Coffee Shop Analytics  

---

## 👤 Author

**Aditya Kumar Dwivedi**  
Aspiring Data Analyst | SQL & Business Analytics  

📧 Email: dwivediaditya2322006@gmail.com  
💼 LinkedIn: [Connect Me!](https://www.linkedin.com/in/dwivediaditya4093/)












  


