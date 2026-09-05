# SQL Interview Prep --- Important Queries

## 1. Average Salary by Department

Find departments where the average salary is greater than 75,000.

``` sql
SELECT department, AVG(salary) AS AverageSalary
FROM Employee
GROUP BY department
HAVING AVG(salary) > 75000;
```

**Pattern:** `GROUP BY` → aggregate → `HAVING`

------------------------------------------------------------------------

## 2. Total Completed Order Amount per Customer

``` sql
SELECT
    c.CustomerID,
    c.Name,
    SUM(o.Amount) AS TotalAmount
FROM Customers c
JOIN Orders o
    ON c.CustomerID = o.CustomerID
WHERE o.Status = 'Completed'
GROUP BY c.CustomerID, c.Name;
```

**Pattern:** `JOIN` → `WHERE` → `GROUP BY` → `SUM`

------------------------------------------------------------------------

## 3. Customers Who Never Placed an Order

``` sql
SELECT c.CustomerID, c.Name
FROM Customers c
LEFT JOIN Orders o
    ON c.CustomerID = o.CustomerID
WHERE o.OrderID IS NULL;
```

**Pattern:** `LEFT JOIN` + `IS NULL`

------------------------------------------------------------------------

## 4. Departments Where Every Employee Earns More Than 5,000

``` sql
SELECT department, COUNT(*) AS total_employees
FROM employee
GROUP BY department
HAVING COUNT(*) = COUNT(
    CASE WHEN salary > 5000 THEN 1 END
);
```

**Pattern:** - Everyone satisfies condition →
`COUNT(*) = COUNT(CASE...)` - At least one → `COUNT(CASE...) > 0` - None
→ `COUNT(CASE...) = 0`

------------------------------------------------------------------------

## 5. Employees Earning More Than Overall Average Salary

### Subquery

``` sql
SELECT employee, salary
FROM employee
WHERE salary > (
    SELECT AVG(salary)
    FROM employee
);
```

### CTE version

``` sql
WITH avg_salary AS (
    SELECT AVG(salary) AS AverageSalary
    FROM Employee
)
SELECT e.Name, e.Salary, a.AverageSalary
FROM Employee e
CROSS JOIN avg_salary a
WHERE e.Salary > a.AverageSalary;
```

**Pattern:** Calculate one value first → compare every row against it.

------------------------------------------------------------------------

## 6. Find Duplicate Records

If duplicates are defined by `name`, `salary`, and `car`:

``` sql
SELECT
    name,
    salary,
    car,
    COUNT(*) AS duplicate_count
FROM employees
GROUP BY name, salary, car
HAVING COUNT(*) > 1;
```

**Pattern:**

``` text
GROUP BY duplicate-defining columns
HAVING COUNT(*) > 1
```

------------------------------------------------------------------------

## 7. Delete Duplicate Records Safely

Use a unique row identifier such as `employee_id`.

``` sql
WITH duplicates AS (
    SELECT
        employee_id,
        ROW_NUMBER() OVER (
            PARTITION BY emp_id, emp_name
            ORDER BY employee_id
        ) AS rn
    FROM employee
)
DELETE FROM employee
WHERE employee_id IN (
    SELECT employee_id
    FROM duplicates
    WHERE rn > 1
);
```

**Important:** Do not delete using only the duplicated columns. You may
delete both the original and duplicate rows.

------------------------------------------------------------------------

## 8. Second Most Recent Order per Customer

``` sql
WITH cust_cte AS (
    SELECT
        customer_id,
        order_id,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC
        ) AS rn
    FROM customer
)
SELECT customer_id, order_id, order_date
FROM cust_cte
WHERE rn = 2;
```

**Pattern:** `ROW_NUMBER()` + `PARTITION BY` + `ORDER BY DESC`

------------------------------------------------------------------------

## 9. Employee Earning More Than Their Manager

``` sql
SELECT
    e.Name AS EmployeeName,
    e.Salary AS EmployeeSalary,
    m.Name AS ManagerName,
    m.Salary AS ManagerSalary
FROM Employees e
JOIN Employees m
    ON e.ManagerID = m.EmployeeID
WHERE e.Salary > m.Salary;
```

**Pattern:** Self-join the employee table.

------------------------------------------------------------------------

## 10. Second Highest Salary per Department

``` sql
SELECT Department, Salary AS SecondHighestSalary
FROM (
    SELECT
        Department,
        Salary,
        DENSE_RANK() OVER (
            PARTITION BY Department
            ORDER BY Salary DESC
        ) AS rnk
    FROM Employee
) e
WHERE rnk = 2;
```

**Pattern:**

``` sql
DENSE_RANK() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

Then filter on the required rank.

------------------------------------------------------------------------

## 11. Nth Highest Salary

For the 3rd highest distinct salary:

``` sql
SELECT name, salary
FROM (
    SELECT
        name,
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS sal_rank
    FROM employees
) t
WHERE sal_rank = 3;
```

**Why `DENSE_RANK()`?**

It handles ties as the same salary rank.

Example:

``` text
Salary    Rank
100000      1
90000       2
80000       3
80000       3
75000       4
```

So rank `3` returns everyone earning 80,000.

------------------------------------------------------------------------

## 12. 3rd, 4th and 5th Highest Rows Using LIMIT/OFFSET

``` sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3 OFFSET 2;
```

Meaning:

-   `ORDER BY salary DESC` → highest salary first
-   `OFFSET 2` → skip first 2 rows
-   `LIMIT 3` → return next 3 rows

**Important:** This works by **rows**, not distinct salary values.

------------------------------------------------------------------------

## 13. Running Total per Customer

``` sql
SELECT
    CustomerID,
    OrderDate,
    Amount,
    SUM(Amount) OVER (
        PARTITION BY CustomerID
        ORDER BY OrderDate
    ) AS RunningTotal
FROM Orders;
```

**Pattern:**

``` sql
SUM(value) OVER (
    PARTITION BY group
    ORDER BY date
)
```

------------------------------------------------------------------------

## 14. Previous Order Amount Using LAG

``` sql
SELECT
    CustomerID,
    OrderDate,
    Amount,
    LAG(Amount) OVER (
        PARTITION BY CustomerID
        ORDER BY OrderDate
    ) AS PreviousAmount
FROM Orders;
```

`LAG()` looks at the previous row within each customer.

------------------------------------------------------------------------

## 15. Difference From Previous Order

### Direct version

``` sql
SELECT
    CustomerID,
    OrderDate,
    Amount,
    LAG(Amount) OVER (
        PARTITION BY CustomerID
        ORDER BY OrderDate
    ) AS PreviousAmount,
    Amount - LAG(Amount) OVER (
        PARTITION BY CustomerID
        ORDER BY OrderDate
    ) AS Difference
FROM Orders;
```

### Cleaner CTE version

``` sql
WITH x AS (
    SELECT
        CustomerID,
        OrderDate,
        Amount,
        LAG(Amount) OVER (
            PARTITION BY CustomerID
            ORDER BY OrderDate
        ) AS PreviousAmount
    FROM Orders
)
SELECT
    CustomerID,
    OrderDate,
    Amount,
    PreviousAmount,
    Amount - PreviousAmount AS Difference
FROM x;
```

**Important:** A SELECT-list alias such as `PreviousAmount` generally
cannot be reused in another expression in the same SELECT list.

------------------------------------------------------------------------

## 16. Highest-Value Order per Customer

``` sql
WITH ranked AS (
    SELECT
        CustomerID,
        OrderID,
        Amount,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerID
            ORDER BY Amount DESC
        ) AS rn
    FROM Orders
)
SELECT CustomerID, OrderID, Amount
FROM ranked
WHERE rn = 1;
```

**Pattern:** Top N per group → ranking window function + CTE/subquery.

------------------------------------------------------------------------

## 17. Top 3 Employees per Department

``` sql
WITH ranked AS (
    SELECT
        EmployeeID,
        Name,
        Department,
        Salary,
        DENSE_RANK() OVER (
            PARTITION BY Department
            ORDER BY Salary DESC
        ) AS rnk
    FROM Employee
)
SELECT EmployeeID, Name, Department, Salary
FROM ranked
WHERE rnk <= 3;
```

Use `ROW_NUMBER()` instead when you specifically need exactly 3 rows per
department.

------------------------------------------------------------------------

## 18. ROW_NUMBER vs RANK vs DENSE_RANK

### ROW_NUMBER

``` sql
ROW_NUMBER() OVER (ORDER BY salary DESC)
```

Every row gets a unique number.

### RANK

``` sql
RANK() OVER (ORDER BY salary DESC)
```

Ties share rank, and gaps appear.

Example:

``` text
100000 → 1
90000  → 2
90000  → 2
80000  → 4
```

### DENSE_RANK

``` sql
DENSE_RANK() OVER (ORDER BY salary DESC)
```

Ties share rank, but there are no gaps.

``` text
100000 → 1
90000  → 2
90000  → 2
80000  → 3
```

**Interview shortcut:**

-   Need unique rows → `ROW_NUMBER`
-   Need ranking with gaps → `RANK`
-   Need distinct salary ranking → `DENSE_RANK`

------------------------------------------------------------------------

## 19. Customers With More Than 3 Orders

``` sql
SELECT CustomerID, COUNT(*) AS OrderCount
FROM Orders
GROUP BY CustomerID
HAVING COUNT(*) > 3;
```

------------------------------------------------------------------------

## 20. Customers Spending More Than 50,000

``` sql
SELECT
    CustomerID,
    SUM(Amount) AS TotalSpent
FROM Orders
GROUP BY CustomerID
HAVING SUM(Amount) > 50000;
```

------------------------------------------------------------------------

## 21. Customers With Completed Orders

Using `EXISTS`:

``` sql
SELECT c.CustomerID, c.Name
FROM Customers c
WHERE EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
      AND o.Status = 'Completed'
);
```

------------------------------------------------------------------------

## 22. Customers With No Completed Orders

``` sql
SELECT c.CustomerID, c.Name
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
      AND o.Status = 'Completed'
);
```

------------------------------------------------------------------------

## 23. CASE --- Salary Category

``` sql
SELECT
    Name,
    Salary,
    CASE
        WHEN Salary >= 80000 THEN 'High'
        WHEN Salary >= 50000 THEN 'Medium'
        ELSE 'Low'
    END AS SalaryCategory
FROM Employees;
```

------------------------------------------------------------------------

## 24. Conditional Aggregation

Count employees earning more than 50,000 per department:

``` sql
SELECT
    Department,
    COUNT(*) AS TotalEmployees,
    COUNT(
        CASE WHEN Salary > 50000 THEN 1 END
    ) AS HighSalaryEmployees
FROM Employees
GROUP BY Department;
```

------------------------------------------------------------------------

## 25. Departments Where Everyone Earns More Than 50,000

``` sql
SELECT Department
FROM Employees
GROUP BY Department
HAVING COUNT(*) =
       COUNT(CASE WHEN Salary > 50000 THEN 1 END);
```

------------------------------------------------------------------------

## 26. First Order per Customer

``` sql
WITH ranked AS (
    SELECT
        CustomerID,
        OrderID,
        OrderDate,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerID
            ORDER BY OrderDate
        ) AS rn
    FROM Orders
)
SELECT CustomerID, OrderID, OrderDate
FROM ranked
WHERE rn = 1;
```

------------------------------------------------------------------------

## 27. Latest Order per Customer

``` sql
WITH ranked AS (
    SELECT
        CustomerID,
        OrderID,
        OrderDate,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerID
            ORDER BY OrderDate DESC
        ) AS rn
    FROM Orders
)
SELECT CustomerID, OrderID, OrderDate
FROM ranked
WHERE rn = 1;
```

------------------------------------------------------------------------

## 28. LEAD --- Next Order Amount

``` sql
SELECT
    CustomerID,
    OrderDate,
    Amount,
    LEAD(Amount) OVER (
        PARTITION BY CustomerID
        ORDER BY OrderDate
    ) AS NextAmount
FROM Orders;
```

**Remember:**

``` text
LAG  → previous row
LEAD → next row
```

------------------------------------------------------------------------

# CTE Recognition Cheat Sheet

When you see:

  Question                                   Think
  ------------------------------------------ ----------------
  Simple filtering                           `WHERE`
  Group totals/averages                      `GROUP BY`
  Filtering aggregates                       `HAVING`
  Previous row                               `LAG()`
  Next row                                   `LEAD()`
  Running total                              `SUM() OVER()`
  Top N per group                            Ranking + CTE
  Window result needs filtering              CTE / subquery
  Calculate something first, then query it   CTE
  Multiple transformation steps              CTE
  Hierarchy/tree                             Recursive CTE

**Important:** A CTE is not mandatory. It is mainly a clean way to break
a complicated query into steps.

------------------------------------------------------------------------

# Most Important Interview Patterns

## Pattern 1 --- Duplicate Records

``` sql
SELECT column1, column2, COUNT(*)
FROM table
GROUP BY column1, column2
HAVING COUNT(*) > 1;
```

## Pattern 2 --- Nth Highest

``` sql
SELECT *
FROM (
    SELECT
        *,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk = N;
```

## Pattern 3 --- Nth Highest Per Department

``` sql
SELECT *
FROM (
    SELECT
        *,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
) t
WHERE rnk = N;
```

## Pattern 4 --- Top N Per Group

``` sql
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
) t
WHERE rn <= N;
```

## Pattern 5 --- Previous Row

``` sql
LAG(column) OVER (
    PARTITION BY group_column
    ORDER BY date_column
)
```

## Pattern 6 --- Running Total

``` sql
SUM(amount) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
)
```

## Pattern 7 --- Employee vs Manager

``` sql
FROM Employees e
JOIN Employees m
    ON e.ManagerID = m.EmployeeID
WHERE e.Salary > m.Salary;
```

## Pattern 8 --- No Matching Records

``` sql
LEFT JOIN other_table
    ON ...
WHERE other_table.id IS NULL;
```

or:

``` sql
WHERE NOT EXISTS (...)
```

## Pattern 9 --- Every Row Meets Condition

``` sql
HAVING COUNT(*) =
       COUNT(CASE WHEN condition THEN 1 END);
```

## Pattern 10 --- Above Average

``` sql
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

------------------------------------------------------------------------

# High-Frequency SQL Interview Checklist

1.  Find duplicates
2.  Delete duplicates
3.  Find highest salary
4.  Find employee with highest salary
5.  Find second-highest salary
6.  Find Nth-highest salary
7.  Find highest salary in each department
8.  Employees above overall average
9.  Employees above department average
10. Employee earning more than manager
11. Employees without manager
12. Departments with no employees
13. Employees without a valid department
14. Employee-department joins
15. Total orders per customer
16. Customers who never ordered
17. Customers with more than N orders
18. Total spending per customer
19. Customers spending above a threshold
20. `ROW_NUMBER()`
21. `RANK()`
22. `DENSE_RANK()`
23. Top N per department
24. Running total
25. `LAG()`
26. `LEAD()`
27. Difference from previous row
28. CTE + aggregate
29. CTE + ranking
30. CASE expressions
31. Conditional aggregation
32. Departments where everyone meets a condition
33. `EXISTS`
34. `NOT EXISTS`
35. Orders in a date range
36. Orders by year/month
37. First order
38. Latest order
39. Second latest order
40. Nth-highest salary with ties
41. Top N salaries per department
42. Duplicate detection
43. Safe duplicate deletion
44. Employee vs manager
45. Latest record per customer
46. Second latest record per customer
47. Consecutive dates / gaps and islands
48. Missing IDs
49. Window functions with `PARTITION BY`
50. CTE-based multi-step queries

------------------------------------------------------------------------

# Quick Interview Rules

### `WHERE` vs `HAVING`

``` text
WHERE  → filters rows BEFORE grouping
HAVING → filters groups AFTER grouping
```

### `PARTITION BY`

``` text
PARTITION BY customer_id
```

means:

> Perform the window calculation separately for each customer.

### `ORDER BY` inside a window

``` sql
ORDER BY order_date
```

defines the row sequence for functions such as:

-   `ROW_NUMBER`
-   `RANK`
-   `DENSE_RANK`
-   `LAG`
-   `LEAD`
-   Running `SUM`

### `DESC`

``` sql
ORDER BY salary DESC
```

means highest salary first.

### `LIMIT/OFFSET`

``` sql
LIMIT 3 OFFSET 2
```

means:

> Skip 2 rows and return the next 3 rows.

------------------------------------------------------------------------

# Final 15 to Memorize Before an Interview

If time is limited, prioritize these:

1.  Duplicate records
2.  Delete duplicates safely
3.  Second-highest salary
4.  Nth-highest salary
5.  Highest salary per department
6.  Top N per department
7.  Employee earning more than manager
8.  Employees above department average
9.  Customers with no orders
10. `ROW_NUMBER`
11. `RANK` vs `DENSE_RANK`
12. Running total
13. `LAG` / `LEAD`
14. CASE + conditional aggregation
15. CTE + window function
