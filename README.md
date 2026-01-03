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

#### Q5. Identify shifts with insufficient staff  
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


  


