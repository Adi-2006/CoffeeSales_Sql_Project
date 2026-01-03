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
