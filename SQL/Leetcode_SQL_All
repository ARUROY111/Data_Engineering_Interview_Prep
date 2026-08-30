<a id="top"></a>
# LeetCode SQL Problem Solving Questions With Solutions

---

## 🧭 Quick Navigation
Click below to jump directly to any difficulty section:

- 🟢 [**Easy Problems (Questions 1 – 35)**](#easy-problems)
- 🟡 [**Medium Problems (Questions 36 – 62)**](#medium-problems)
- 🔴 [**Hard Problems (Questions 63 – 73)**](#hard-problems)

---

<a id="easy-problems"></a>
## 🟢 Section 1: EASY PROBLEMS (35 Questions)
[⬆ Back to Top](#top)

---

### 1. Combine Two Tables (LeetCode #175) | Easy

**Table Schema:**

`Person` table:
| Column Name | Type |
| :--- | :--- |
| PersonId (PK) | int |
| FirstName | varchar |
| LastName | varchar |

`Address` table:
| Column Name | Type |
| :--- | :--- |
| AddressId (PK) | int |
| PersonId | int |
| City | varchar |
| State | varchar |

**Problem Statement:**
Write a SQL query for a report that provides the following information for each person in the `Person` table, regardless of whether an address exists for each person:
`FirstName, LastName, City, State`

#### Query Solution:
```sql
SELECT 
    p.FirstName, 
    p.LastName, 
    a.City, 
    a.State
FROM Person p
LEFT JOIN Address a 
    ON p.PersonId = a.PersonId;
```

---

### 2. Second Highest Salary (LeetCode #176) | Easy

**Sample Input:**

`Employee` table:
| Id | Salary |
| :--- | :--- |
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |

**Sample Output:**
| SecondHighestSalary |
| :--- |
| 200 |

**Problem Statement:**
Write a SQL query to get the second highest salary from the `Employee` table. If there is no second highest salary, return `null`.

#### Query Solution:
```sql
SELECT (
    SELECT DISTINCT Salary
    FROM Employee
    ORDER BY Salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

---

### 3. Employees Earning More Than Their Managers (LeetCode #181) | Easy

**Sample Input:**

`Employee` table:
| Id | Name | Salary | ManagerId |
| :--- | :--- | :--- | :--- |
| 1 | Joe | 70000 | 3 |
| 2 | Henry | 80000 | 4 |
| 3 | Sam | 60000 | NULL |
| 4 | Max | 90000 | NULL |

**Sample Output:**
| Employee |
| :--- |
| Joe |

**Problem Statement:**
Given the `Employee` table, write a SQL query that finds out employees who earn more than their managers.

#### Query Solution:
```sql
SELECT e.Name AS Employee
FROM Employee e
JOIN Employee m 
    ON e.ManagerId = m.Id
WHERE e.Salary > m.Salary;
```

---

### 4. Duplicate Emails (LeetCode #182) | Easy

**Sample Input:**

`Person` table:
| Id | Email |
| :--- | :--- |
| 1 | a@b.com |
| 2 | c@d.com |
| 3 | a@b.com |

**Sample Output:**
| Email |
| :--- |
| a@b.com |

**Problem Statement:**
Write a SQL query to find all duplicate emails in a table named `Person`.

#### Query Solution:
```sql
SELECT Email
FROM Person
GROUP BY Email
HAVING COUNT(Email) > 1;
```

---

### 5. Customers Who Never Order (LeetCode #183) | Easy

**Sample Input:**

`Customers` table:
| Id | Name |
| :--- | :--- |
| 1 | Joe |
| 2 | Henry |
| 3 | Sam |
| 4 | Max |

`Orders` table:
| Id | CustomerId |
| :--- | :--- |
| 1 | 3 |
| 2 | 1 |

**Sample Output:**
| Customers |
| :--- |
| Henry |
| Max |

**Problem Statement:**
Suppose that a website contains two tables, the `Customers` table and the `Orders` table. Write a SQL query to find all customers who never order anything.

#### Query Solution:
```sql
SELECT c.Name AS Customers
FROM Customers c
LEFT JOIN Orders o 
    ON c.Id = o.CustomerId
WHERE o.CustomerId IS NULL;
```

---

### 6. Delete Duplicate Emails (LeetCode #196) | Easy

**Sample Input:**

`Person` table:
| Id | Email |
| :--- | :--- |
| 1 | john@example.com |
| 2 | bob@example.com |
| 3 | john@example.com |

**Sample Output (after execution):**
| Id | Email |
| :--- | :--- |
| 1 | john@example.com |
| 2 | bob@example.com |

**Problem Statement:**
Write a SQL query to delete all duplicate email entries in a table named `Person`, keeping only unique emails based on its smallest `Id`.

#### Query Solution:
```sql
DELETE p1
FROM Person p1
JOIN Person p2 
    ON p1.Email = p2.Email AND p1.Id > p2.Id;
```

---

### 7. Rising Temperature (LeetCode #197) | Easy

**Sample Input:**

`Weather` table:
| id | recordDate | temperature |
| :--- | :--- | :--- |
| 1 | 2015-01-01 | 10 |
| 2 | 2015-01-02 | 25 |
| 3 | 2015-01-03 | 20 |
| 4 | 2015-01-04 | 30 |

**Sample Output:**
| id |
| :--- |
| 2 |
| 4 |

**Problem Statement:**
Write a SQL query to find all dates' `id` with higher temperatures compared to their previous dates (yesterday).

#### Query Solution:
```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2 
    ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

---

### 8. Game Play Analysis I (LeetCode #511) | Easy

**Sample Input:**

`Activity` table:
| player_id | device_id | event_date | games_played |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 2016-03-01 | 5 |
| 1 | 2 | 2016-05-02 | 6 |
| 2 | 3 | 2017-06-25 | 1 |
| 3 | 1 | 2016-03-02 | 0 |
| 3 | 4 | 2018-07-03 | 5 |

**Sample Output:**
| player_id | first_login |
| :--- | :--- |
| 1 | 2016-03-01 |
| 2 | 2017-06-25 |
| 3 | 2016-03-02 |

**Problem Statement:**
Write a SQL query that reports the first login date for each player.

#### Query Solution:
```sql
SELECT 
    player_id, 
    MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

---

### 9. Game Play Analysis II (LeetCode #512) | Easy

**Sample Input:**

`Activity` table:
| player_id | device_id | event_date | games_played |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 2016-03-01 | 5 |
| 1 | 2 | 2016-05-02 | 6 |
| 2 | 3 | 2017-06-25 | 1 |
| 3 | 1 | 2016-03-02 | 0 |
| 3 | 4 | 2018-07-03 | 5 |

**Sample Output:**
| player_id | device_id |
| :--- | :--- |
| 1 | 2 |
| 2 | 3 |
| 3 | 1 |

**Problem Statement:**
Write a SQL query that reports the device that was first logged in for each player.

#### Query Solution:
```sql
SELECT player_id, device_id
FROM Activity
WHERE (player_id, event_date) IN (
    SELECT player_id, MIN(event_date)
    FROM Activity
    GROUP BY player_id
);
```

---

### 10. Employee Bonus (LeetCode #577) | Easy

**Sample Input:**

`Employee` table:
| empId | name | supervisor | salary |
| :--- | :--- | :--- | :--- |
| 1 | John | 3 | 1000 |
| 2 | Dan | 3 | 2000 |
| 3 | Brad | null | 4000 |
| 4 | Thomas | 3 | 4000 |

`Bonus` table:
| empId | bonus |
| :--- | :--- |
| 2 | 500 |
| 4 | 2000 |

**Sample Output:**
| name | bonus |
| :--- | :--- |
| John | null |
| Dan | 500 |
| Brad | null |

**Problem Statement:**
Select all employees' names and bonuses whose bonus is $< 1000$ or `null`.

#### Query Solution:
```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b 
    ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

---

### 11. Find Customer Referee (LeetCode #584) | Easy

**Sample Input:**

`customer` table:
| id | name | referee_id |
| :--- | :--- | :--- |
| 1 | Will | NULL |
| 2 | Jane | NULL |
| 3 | Alex | 2 |
| 4 | Bill | NULL |
| 5 | Zack | 1 |
| 6 | Mark | 2 |

**Sample Output:**
| name |
| :--- |
| Will |
| Jane |
| Bill |
| Zack |

**Problem Statement:**
Write a query to return the list of customers NOT referred by the person with `id = 2`.

#### Query Solution:
```sql
SELECT name
FROM customer
WHERE referee_id != 2 OR referee_id IS NULL;
```

---

### 12. Customer Placing the Largest Number of Orders (LeetCode #586) | Easy

**Sample Input:**

`orders` table:
| order_number | customer_number | order_date | required_date | shipped_date | status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 2017-04-09 | 2017-04-13 | 2017-04-12 | Closed |
| 2 | 2 | 2017-04-15 | 2017-04-20 | 2017-04-18 | Closed |
| 3 | 3 | 2017-04-16 | 2017-04-25 | 2017-04-20 | Closed |
| 4 | 3 | 2017-04-18 | 2017-04-28 | 2017-04-25 | Closed |

**Sample Output:**
| customer_number |
| :--- |
| 3 |

**Problem Statement:**
Query the `customer_number` from the `orders` table for the customer who has placed the largest number of orders.

#### Query Solution:
```sql
SELECT customer_number
FROM orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;
```

---

### 13. Big Countries (LeetCode #595) | Easy

**Sample Input:**

`World` table:
| name | continent | area | population | gdp |
| :--- | :--- | :--- | :--- | :--- |
| Afghanistan | Asia | 652230 | 25500100 | 20343000 |
| Albania | Europe | 28748 | 2831741 | 12960000 |
| Algeria | Africa | 2381741 | 37100000 | 188681000 |
| Andorra | Europe | 468 | 78115 | 3712000 |
| Angola | Africa | 1246700 | 20609294 | 100990000 |

**Sample Output:**
| name | population | area |
| :--- | :--- | :--- |
| Afghanistan | 25500100 | 652230 |
| Algeria | 37100000 | 2381741 |

**Problem Statement:**
A country is big if it has an area greater than 3 million $\text{km}^2$ or a population of more than 25 million. Write a SQL solution to output big countries' name, population, and area.

#### Query Solution:
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

---

### 14. Classes More Than 5 Students (LeetCode #596) | Easy

**Sample Input:**

`courses` table:
| student | class |
| :--- | :--- |
| A | Math |
| B | English |
| C | Math |
| D | Biology |
| E | Math |
| F | Computer |
| G | Math |
| H | Math |
| I | Math |

**Sample Output:**
| class |
| :--- |
| Math |

**Problem Statement:**
Please list out all classes which have more than or equal to 5 students.

#### Query Solution:
```sql
SELECT class
FROM courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5;
```

---

### 15. Friend Requests I: Overall Acceptance Rate (LeetCode #597) | Easy

**Sample Input:**

`friend_request` table:
| sender_id | send_to_id | request_date |
| :--- | :--- | :--- |
| 1 | 2 | 2016-06-01 |
| 1 | 3 | 2016-06-01 |
| 1 | 4 | 2016-06-01 |
| 2 | 3 | 2016-06-02 |
| 3 | 4 | 2016-06-09 |

`request_accepted` table:
| requester_id | accepter_id | accept_date |
| :--- | :--- | :--- |
| 1 | 2 | 2016-06-03 |
| 1 | 3 | 2016-06-08 |
| 2 | 3 | 2016-06-08 |
| 3 | 4 | 2016-06-09 |
| 3 | 4 | 2016-06-10 |

**Sample Output:**
| accept_rate |
| :--- |
| 0.80 |

**Problem Statement:**
Write a query to find the overall acceptance rate of requests rounded to 2 decimals, which is the number of acceptance divided by the number of requests.

#### Query Solution:
```sql
SELECT ROUND(
    IFNULL(
        (SELECT COUNT(DISTINCT requester_id, accepter_id) FROM request_accepted) /
        NULLIF((SELECT COUNT(DISTINCT sender_id, send_to_id) FROM friend_request), 0),
        0.00
    ), 
    2
) AS accept_rate;
```

---

### 16. Consecutive Available Seats (LeetCode #603) | Easy

**Sample Input:**

`cinema` table:
| seat_id | free |
| :--- | :--- |
| 1 | 1 |
| 2 | 0 |
| 3 | 1 |
| 4 | 1 |
| 5 | 1 |

**Sample Output:**
| seat_id |
| :--- |
| 3 |
| 4 |
| 5 |

**Problem Statement:**
Query all consecutive available seats (`free = 1`, 2 or more seats consecutive) ordered by `seat_id`.

#### Query Solution:
```sql
SELECT DISTINCT c1.seat_id
FROM cinema c1
JOIN cinema c2 
    ON ABS(c1.seat_id - c2.seat_id) = 1
WHERE c1.free = 1 AND c2.free = 1
ORDER BY c1.seat_id;
```

---

### 17. Sales Person (LeetCode #607) | Easy

**Sample Input:**

`salesperson` table:
| sales_id | name | salary | commission_rate | hire_date |
| :--- | :--- | :--- | :--- | :--- |
| 1 | John | 100000 | 6 | 4/1/2006 |
| 2 | Amy | 120000 | 5 | 5/1/2010 |
| 3 | Mark | 65000 | 12 | 12/25/2008 |
| 4 | Pam | 25000 | 25 | 1/1/2005 |
| 5 | Alex | 50000 | 10 | 2/3/2007 |

`company` table:
| com_id | name | city |
| :--- | :--- | :--- |
| 1 | RED | Boston |
| 2 | ORANGE | New York |
| 3 | YELLOW | Boston |
| 4 | GREEN | Austin |

`orders` table:
| order_id | date | com_id | sales_id | amount |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1/1/2014 | 3 | 4 | 100000 |
| 2 | 2/1/2014 | 4 | 5 | 5000 |
| 3 | 3/1/2014 | 1 | 1 | 50000 |
| 4 | 4/1/2014 | 1 | 4 | 25000 |

**Sample Output:**
| name |
| :--- |
| Amy |
| Mark |
| Alex |

**Problem Statement:**
Output all the names in the table `salesperson` who didn't have sales related to company `'RED'`.

#### Query Solution:
```sql
SELECT name
FROM salesperson
WHERE sales_id NOT IN (
    SELECT o.sales_id
    FROM orders o
    JOIN company c 
        ON o.com_id = c.com_id
    WHERE c.name = 'RED'
);
```

---

### 18. Triangle Judgement (LeetCode #610) | Easy

**Sample Input:**

`triangle` table:
| x | y | z |
| :--- | :--- | :--- |
| 13 | 15 | 30 |
| 10 | 20 | 15 |

**Sample Output:**
| x | y | z | triangle |
| :--- | :--- | :--- | :--- |
| 13 | 15 | 30 | No |
| 10 | 20 | 15 | Yes |

**Problem Statement:**
Write a query to judge whether three side segments can form a triangle (`x + y > z` AND `y + z > x` AND `x + z > y`).

#### Query Solution:
```sql
SELECT 
    x, y, z,
    CASE 
        WHEN x + y > z AND y + z > x AND x + z > y THEN 'Yes'
        ELSE 'No'
    END AS triangle
FROM triangle;
```

---

### 19. Shortest Distance in a Line (LeetCode #613) | Easy

**Sample Input:**

`point` table:
| x |
| :--- |
| -1 |
| 0 |
| 2 |

**Sample Output:**
| shortest |
| :--- |
| 1 |

**Problem Statement:**
Write a query to find the shortest distance between two points in table `point`.

#### Query Solution:
```sql
SELECT MIN(ABS(p1.x - p2.x)) AS shortest
FROM point p1
JOIN point p2 
    ON p1.x != p2.x;
```

---

### 20. Biggest Single Number (LeetCode #619) | Easy

**Sample Input:**

`number` table:
| num |
| :--- |
| 8 |
| 8 |
| 3 |
| 3 |
| 1 |
| 4 |
| 5 |
| 6 |

**Sample Output:**
| num |
| :--- |
| 6 |

**Problem Statement:**
Find the biggest number that only appears once. If there is no such number, output `null`.

#### Query Solution:
```sql
SELECT MAX(num) AS num
FROM (
    SELECT num
    FROM number
    GROUP BY num
    HAVING COUNT(*) = 1
) t;
```

---

### 21. Not Boring Movies (LeetCode #620) | Easy

**Sample Input:**

`cinema` table:
| id | movie | description | rating |
| :--- | :--- | :--- | :--- |
| 1 | War | great 3D | 8.9 |
| 2 | Science | fiction | 8.5 |
| 3 | irish | boring | 6.2 |
| 4 | Ice song | Fantacy | 8.6 |
| 5 | House card | Interesting | 9.1 |

**Sample Output:**
| id | movie | description | rating |
| :--- | :--- | :--- | :--- |
| 5 | House card | Interesting | 9.1 |
| 1 | War | great 3D | 8.9 |

**Problem Statement:**
Write a SQL query to output movies with an odd numbered ID and a description that is not `'boring'`. Order the result by rating.

#### Query Solution:
```sql
SELECT id, movie, description, rating
FROM cinema
WHERE id % 2 = 1 
  AND description <> 'boring'
ORDER BY rating DESC;
```

---

### 22. Swap Salary (LeetCode #627) | Easy

**Sample Input:**

`Salary` table:
| id | name | sex | salary |
| :--- | :--- | :--- | :--- |
| 1 | A | m | 2500 |
| 2 | B | f | 1500 |
| 3 | C | m | 5500 |
| 4 | D | f | 500 |

**Sample Output:**
| id | name | sex | salary |
| :--- | :--- | :--- | :--- |
| 1 | A | f | 2500 |
| 2 | B | m | 1500 |
| 3 | C | f | 5500 |
| 4 | D | m | 500 |

**Problem Statement:**
Swap all `'f'` and `'m'` values with a single update statement and no intermediate temp table(s).

#### Query Solution:
```sql
UPDATE Salary
SET sex = CASE sex 
    WHEN 'm' THEN 'f' 
    ELSE 'm' 
END;
```

---

### 23. Actors and Directors Who Cooperated At Least Three Times (LeetCode #1050) | Easy

**Sample Input:**

`ActorDirector` table:
| actor_id | director_id | timestamp |
| :--- | :--- | :--- |
| 1 | 1 | 0 |
| 1 | 1 | 1 |
| 1 | 1 | 2 |
| 1 | 2 | 3 |
| 1 | 2 | 4 |
| 2 | 1 | 5 |
| 2 | 1 | 6 |

**Sample Output:**
| actor_id | director_id |
| :--- | :--- |
| 1 | 1 |

**Problem Statement:**
Write a SQL query for a report that provides the pairs `(actor_id, director_id)` where the actor has cooperated with the director at least 3 times.

#### Query Solution:
```sql
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3;
```

---

### 24. Product Sales Analysis I (LeetCode #1068) | Easy

**Sample Input:**

`Sales` table:
| sale_id | product_id | year | quantity | price |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 100 | 2008 | 10 | 5000 |
| 2 | 100 | 2009 | 12 | 5000 |
| 7 | 200 | 2011 | 15 | 9000 |

`Product` table:
| product_id | product_name |
| :--- | :--- |
| 100 | Nokia |
| 200 | Apple |
| 300 | Samsung |

**Sample Output:**
| product_name | year | price |
| :--- | :--- | :--- |
| Nokia | 2008 | 5000 |
| Nokia | 2009 | 5000 |
| Apple | 2011 | 9000 |

**Problem Statement:**
Write an SQL query that reports all product names of the products in the `Sales` table along with their selling year and price.

#### Query Solution:
```sql
SELECT p.product_name, s.year, s.price
FROM Sales s
JOIN Product p 
    ON s.product_id = p.product_id;
```

---

### 25. Product Sales Analysis II (LeetCode #1069) | Easy

**Sample Input:**
*(Same tables as #1068)*

**Sample Output:**
| product_id | total_quantity |
| :--- | :--- |
| 100 | 22 |
| 200 | 15 |

**Problem Statement:**
Write an SQL query that reports the total quantity sold for every product id.

#### Query Solution:
```sql
SELECT product_id, SUM(quantity) AS total_quantity
FROM Sales
GROUP BY product_id;
```

---

### 26. Project Employees I (LeetCode #1075) | Easy

**Sample Input:**

`Project` table:
| project_id | employee_id |
| :--- | :--- |
| 1 | 1 |
| 1 | 2 |
| 1 | 3 |
| 2 | 1 |
| 2 | 4 |

`Employee` table:
| employee_id | name | experience_years |
| :--- | :--- | :--- |
| 1 | Khaled | 3 |
| 2 | Ali | 2 |
| 3 | John | 1 |
| 4 | Doe | 2 |

**Sample Output:**
| project_id | average_years |
| :--- | :--- |
| 1 | 2.00 |
| 2 | 2.50 |

**Problem Statement:**
Write an SQL query that reports the average experience years of all the employees for each project, rounded to 2 digits.

#### Query Solution:
```sql
SELECT 
    p.project_id, 
    ROUND(AVG(e.experience_years), 2) AS average_years
FROM Project p
JOIN Employee e 
    ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

---

### 27. Project Employees II (LeetCode #1076) | Easy

**Sample Input:**
*(Same tables as #1075)*

**Sample Output:**
| project_id |
| :--- |
| 1 |

**Problem Statement:**
Write an SQL query that reports all the projects that have the most employees.

#### Query Solution:
```sql
SELECT project_id
FROM Project
GROUP BY project_id
HAVING COUNT(employee_id) = (
    SELECT COUNT(employee_id)
    FROM Project
    GROUP BY project_id
    ORDER BY COUNT(employee_id) DESC
    LIMIT 1
);
```

---

### 28. Sales Analysis I (LeetCode #1082) | Easy

**Sample Input:**

`Product` table:
| product_id | product_name | unit_price |
| :--- | :--- | :--- |
| 1 | S8 | 1000 |
| 2 | G4 | 800 |
| 3 | iPhone | 1400 |

`Sales` table:
| seller_id | product_id | buyer_id | sale_date | quantity | price |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 1 | 2019-01-21 | 2 | 2000 |
| 1 | 2 | 2 | 2019-02-17 | 1 | 800 |
| 2 | 2 | 3 | 2019-06-02 | 1 | 800 |
| 3 | 3 | 4 | 2019-05-13 | 2 | 2800 |

**Sample Output:**
| seller_id |
| :--- |
| 1 |
| 3 |

**Problem Statement:**
Write an SQL query that reports the best seller by total sales price. If there is a tie, report them all.

#### Query Solution:
```sql
SELECT seller_id
FROM Sales
GROUP BY seller_id
HAVING SUM(price) = (
    SELECT SUM(price)
    FROM Sales
    GROUP BY seller_id
    ORDER BY SUM(price) DESC
    LIMIT 1
);
```

---

### 29. Sales Analysis II (LeetCode #1083) | Easy

**Sample Input:**

`Sales` table:
| seller_id | product_id | buyer_id | sale_date | quantity | price |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 1 | 2019-01-21 | 2 | 2000 |
| 1 | 2 | 2 | 2019-02-17 | 1 | 800 |
| 2 | 1 | 3 | 2019-06-02 | 1 | 800 |
| 3 | 3 | 3 | 2019-05-13 | 2 | 2800 |

**Sample Output:**
| buyer_id |
| :--- |
| 1 |

**Problem Statement:**
Write an SQL query that reports the buyers who have bought `'S8'` but not `'iPhone'`.

#### Query Solution:
```sql
SELECT DISTINCT s.buyer_id
FROM Sales s
JOIN Product p 
    ON s.product_id = p.product_id
WHERE p.product_name = 'S8'
  AND s.buyer_id NOT IN (
      SELECT s2.buyer_id
      FROM Sales s2
      JOIN Product p2 
          ON s2.product_id = p2.product_id
      WHERE p2.product_name = 'iPhone'
  );
```

---

### 30. Sales Analysis III (LeetCode #1084) | Easy

**Sample Input:**
*(Same tables as #1082 / #1083)*

**Sample Output:**
| product_id | product_name |
| :--- | :--- |
| 1 | S8 |

**Problem Statement:**
Reports the products that were only sold in spring 2019. That is, between `2019-01-01` and `2019-03-31` inclusive.

#### Query Solution:
```sql
SELECT 
    p.product_id, 
    p.product_name
FROM Product p
JOIN Sales s 
    ON p.product_id = s.product_id
GROUP BY p.product_id, p.product_name
HAVING MIN(s.sale_date) >= '2019-01-01' 
   AND MAX(s.sale_date) <= '2019-03-31';
```

---

### 31. Reported Posts (LeetCode #1113) | Easy

**Sample Input:**

`Actions` table:
| user_id | post_id | action_date | action | extra |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 2019-07-01 | view | null |
| 1 | 1 | 2019-07-01 | like | null |
| 1 | 1 | 2019-07-01 | share | null |
| 2 | 4 | 2019-07-04 | view | null |
| 2 | 4 | 2019-07-04 | report | spam |
| 3 | 4 | 2019-07-04 | view | null |
| 3 | 4 | 2019-07-04 | report | spam |
| 4 | 3 | 2019-07-02 | view | null |
| 4 | 3 | 2019-07-02 | report | spam |
| 5 | 2 | 2019-07-04 | view | null |
| 5 | 2 | 2019-07-04 | report | racism |
| 5 | 5 | 2019-07-04 | view | null |
| 5 | 5 | 2019-07-04 | report | racism |

**Sample Output:**
| report_reason | report_count |
| :--- | :--- |
| spam | 1 |
| racism | 2 |

**Problem Statement:**
Write an SQL query that reports the number of posts reported yesterday for each report reason. Assume today is `2019-07-05`.

#### Query Solution:
```sql
SELECT 
    extra AS report_reason, 
    COUNT(DISTINCT post_id) AS report_count
FROM Actions
WHERE action_date = '2019-07-04' 
  AND action = 'report'
GROUP BY extra;
```

---

### 32. User Activity for the Past 30 Days I (LeetCode #1141) | Easy

**Sample Input:**

`Activity` table:
| user_id | session_id | activity_date | activity_type |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 2019-07-20 | open_session |
| 1 | 1 | 2019-07-20 | scroll_down |
| 1 | 1 | 2019-07-20 | end_session |
| 2 | 4 | 2019-07-20 | open_session |
| 2 | 4 | 2019-07-21 | send_message |
| 2 | 4 | 2019-07-21 | end_session |
| 3 | 2 | 2019-07-21 | open_session |
| 3 | 2 | 2019-07-21 | send_message |
| 3 | 2 | 2019-07-21 | end_session |
| 4 | 3 | 2019-06-25 | open_session |
| 4 | 3 | 2019-06-25 | end_session |

**Sample Output:**
| day | active_users |
| :--- | :--- |
| 2019-07-20 | 2 |
| 2019-07-21 | 2 |

**Problem Statement:**
Write an SQL query to find the daily active user count for a period of 30 days ending `2019-07-27` inclusively.

#### Query Solution:
```sql
SELECT 
    activity_date AS day, 
    COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27'
GROUP BY activity_date;
```

---

### 33. User Activity for the Past 30 Days II (LeetCode #1142) | Easy

**Sample Input:**
*(Activity table from LeetCode #1141 with multiple user sessions)*

**Sample Output:**
| average_sessions_per_user |
| :--- |
| 1.33 |

**Problem Statement:**
Write an SQL query to find the average number of sessions per user for a period of 30 days ending `2019-07-27` inclusively, rounded to 2 decimal places.

#### Query Solution:
```sql
SELECT IFNULL(ROUND(COUNT(DISTINCT session_id) / COUNT(DISTINCT user_id), 2), 0.00) AS average_sessions_per_user
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27';
```

---

### 34. Article Views I (LeetCode #1148) | Easy

**Sample Input:**

`Views` table:
| article_id | author_id | viewer_id | view_date |
| :--- | :--- | :--- | :--- |
| 1 | 3 | 5 | 2019-08-01 |
| 1 | 3 | 6 | 2019-08-02 |
| 2 | 7 | 7 | 2019-08-01 |
| 2 | 7 | 6 | 2019-08-02 |
| 4 | 7 | 1 | 2019-07-22 |
| 3 | 4 | 4 | 2019-07-21 |
| 3 | 4 | 4 | 2019-07-21 |

**Sample Output:**
| id |
| :--- |
| 4 |
| 7 |

**Problem Statement:**
Write an SQL query to find all the authors that viewed at least one of their own articles, sorted in ascending order by their id.

#### Query Solution:
```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY id ASC;
```

---

### 35. Immediate Food Delivery I (LeetCode #1173) | Easy

**Sample Input:**

`Delivery` table:
| delivery_id | customer_id | order_date | customer_pref_delivery_date |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 2019-08-01 | 2019-08-02 |
| 2 | 5 | 2019-08-02 | 2019-08-02 |
| 3 | 1 | 2019-08-11 | 2019-08-11 |
| 4 | 3 | 2019-08-24 | 2019-08-26 |
| 5 | 4 | 2019-08-21 | 2019-08-22 |
| 6 | 2 | 2019-08-11 | 2019-08-13 |

**Sample Output:**
| immediate_percentage |
| :--- |
| 33.33 |

**Problem Statement:**
If the preferred delivery date of the customer is the same as the order date, then the order is called immediate; otherwise, it is called scheduled. Write an SQL query to find the percentage of immediate orders in the table, rounded to 2 decimal places.

#### Query Solution:
```sql
SELECT ROUND(
    AVG(order_date = customer_pref_delivery_date) * 100, 
    2
) AS immediate_percentage
FROM Delivery;
```

---

<a id="medium-problems"></a>
## 🟡 Section 2: MEDIUM PROBLEMS (27 Questions)
[⬆ Back to Top](#top)

---

### 36. Nth Highest Salary (LeetCode #177) | Medium

**Sample Input:**

`Employee` table:
| Id | Salary |
| :--- | :--- |
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |

**Sample Output ($N = 2$):**
| getNthHighestSalary(2) |
| :--- |
| 200 |

**Problem Statement:**
Write a SQL query to get the $n^{\text{th}}$ highest salary from the `Employee` table.

#### Query Solution:
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  SET N = N - 1;
  RETURN (
      SELECT DISTINCT Salary
      FROM Employee
      ORDER BY Salary DESC
      LIMIT 1 OFFSET N
  );
END;
```

---

### 37. Rank Scores (LeetCode #178) | Medium

**Sample Input:**

`Scores` table:
| Id | Score |
| :--- | :--- |
| 1 | 3.50 |
| 2 | 3.65 |
| 3 | 4.00 |
| 4 | 3.85 |
| 5 | 4.00 |
| 6 | 3.65 |

**Sample Output:**
| score | Rank |
| :--- | :--- |
| 4.00 | 1 |
| 4.00 | 1 |
| 3.85 | 2 |
| 3.65 | 3 |
| 3.65 | 3 |
| 3.50 | 4 |

**Problem Statement:**
Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value (no gaps).

#### Query Solution:
```sql
SELECT 
    Score, 
    DENSE_RANK() OVER (ORDER BY Score DESC) AS `Rank`
FROM Scores;
```

---

### 38. Consecutive Numbers (LeetCode #180) | Medium

**Sample Input:**

`Logs` table:
| Id | Num |
| :--- | :--- |
| 1 | 1 |
| 2 | 1 |
| 3 | 1 |
| 4 | 2 |
| 5 | 1 |
| 6 | 2 |
| 7 | 2 |

**Sample Output:**
| ConsecutiveNums |
| :--- |
| 1 |

**Problem Statement:**
Write an SQL query to find all numbers that appear at least three times consecutively.

#### Query Solution:
```sql
SELECT DISTINCT l1.Num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l1.Id = l2.Id - 1 AND l1.Num = l2.Num
JOIN Logs l3 ON l1.Id = l3.Id - 2 AND l1.Num = l3.Num;
```

---

### 39. Department Highest Salary (LeetCode #184) | Medium

**Sample Input:**

`Employee` table:
| Id | Name | Salary | DepartmentId |
| :--- | :--- | :--- | :--- |
| 1 | Joe | 70000 | 1 |
| 2 | Jim | 90000 | 1 |
| 3 | Henry | 80000 | 2 |
| 4 | Sam | 60000 | 2 |
| 5 | Max | 90000 | 1 |

`Department` table:
| Id | Name |
| :--- | :--- |
| 1 | IT |
| 2 | Sales |

**Sample Output:**
| Department | Employee | Salary |
| :--- | :--- | :--- |
| IT | Max | 90000 |
| IT | Jim | 90000 |
| Sales | Henry | 80000 |

**Problem Statement:**
Write a SQL query to find employees who have the highest salary in each of the departments.

#### Query Solution:
```sql
SELECT 
    d.Name AS Department, 
    e.Name AS Employee, 
    e.Salary
FROM Employee e
JOIN Department d 
    ON e.DepartmentId = d.Id
WHERE (e.DepartmentId, e.Salary) IN (
    SELECT DepartmentId, MAX(Salary)
    FROM Employee
    GROUP BY DepartmentId
);
```

---

### 40. Game Play Analysis III (LeetCode #534) | Medium

**Sample Input:**

`Activity` table:
| player_id | device_id | event_date | games_played |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 2016-03-01 | 5 |
| 1 | 2 | 2016-05-02 | 6 |
| 1 | 3 | 2017-06-25 | 1 |
| 3 | 1 | 2016-03-02 | 0 |
| 3 | 4 | 2018-07-03 | 5 |

**Sample Output:**
| player_id | event_date | games_played_so_far |
| :--- | :--- | :--- |
| 1 | 2016-03-01 | 5 |
| 1 | 2016-05-02 | 11 |
| 1 | 2017-06-25 | 12 |
| 3 | 2016-03-02 | 0 |
| 3 | 2018-07-03 | 5 |

**Problem Statement:**
Write an SQL query that reports for each player and date, how many games played so far by the player.

#### Query Solution:
```sql
SELECT 
    player_id, 
    event_date,
    SUM(games_played) OVER (
        PARTITION BY player_id 
        ORDER BY event_date
    ) AS games_played_so_far
FROM Activity;
```

---

### 41. Game Play Analysis IV (LeetCode #550) | Medium

**Sample Input:**
*(Activity table from #534)*

**Sample Output:**
| fraction |
| :--- |
| 0.33 |

**Problem Statement:**
Write an SQL query that reports the fraction of players that logged in again on the day after the day they first logged in, rounded to 2 decimal places.

#### Query Solution:
```sql
SELECT ROUND(
    COUNT(DISTINCT a.player_id) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 
    2
) AS fraction
FROM Activity a
JOIN (
    SELECT player_id, MIN(event_date) AS first_login
    FROM Activity
    GROUP BY player_id
) f 
    ON a.player_id = f.player_id 
   AND a.event_date = DATE_ADD(f.first_login, INTERVAL 1 DAY);
```

---

### 42. Managers with at Least 5 Direct Reports (LeetCode #570) | Medium

**Sample Input:**

`Employee` table:
| Id | Name | Department | ManagerId |
| :--- | :--- | :--- | :--- |
| 101 | John | A | null |
| 102 | Dan | A | 101 |
| 103 | James | A | 101 |
| 104 | Amy | A | 101 |
| 105 | Anne | A | 101 |
| 106 | Ron | B | 101 |

**Sample Output:**
| Name |
| :--- |
| John |

**Problem Statement:**
Given the `Employee` table, write a SQL query that finds out managers with at least 5 direct reports.

#### Query Solution:
```sql
SELECT Name
FROM Employee
WHERE Id IN (
    SELECT ManagerId
    FROM Employee
    GROUP BY ManagerId
    HAVING COUNT(DISTINCT Id) >= 5
);
```

---

### 43. Winning Candidate (LeetCode #574) | Medium

**Sample Input:**

`Candidate` table:
| id | Name |
| :--- | :--- |
| 1 | A |
| 2 | B |
| 3 | C |
| 4 | D |
| 5 | E |

`Vote` table:
| id | CandidateId |
| :--- | :--- |
| 1 | 2 |
| 2 | 4 |
| 3 | 3 |
| 4 | 2 |
| 5 | 5 |

**Sample Output:**
| Name |
| :--- |
| B |

**Problem Statement:**
Write a SQL query to find the name of the winning candidate (candidate with the most votes).

#### Query Solution:
```sql
SELECT c.Name
FROM Candidate c
JOIN (
    SELECT CandidateId
    FROM Vote
    GROUP BY CandidateId
    ORDER BY COUNT(*) DESC
    LIMIT 1
) v ON c.id = v.CandidateId;
```

---

### 44. Get Highest Answer Rate Question (LeetCode #578) | Medium

**Sample Input:**

`survey_log` table:
| uid | action | question_id | answer_id | q_num | timestamp |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 5 | show | 285 | null | 1 | 123 |
| 5 | answer | 285 | 124124 | 1 | 124 |
| 5 | show | 369 | null | 2 | 125 |
| 5 | skip | 369 | null | 2 | 126 |

**Sample Output:**
| survey_log |
| :--- |
| 285 |

**Problem Statement:**
Write a SQL query to identify the question which has the highest answer rate (number of answers divided by number of shows).

#### Query Solution:
```sql
SELECT question_id AS survey_log
FROM survey_log
GROUP BY question_id
ORDER BY (SUM(IF(action = 'answer', 1, 0)) / SUM(IF(action = 'show', 1, 0))) DESC, question_id ASC
LIMIT 1;
```

---

### 45. Count Student Number in Departments (LeetCode #580) | Medium

**Sample Input:**

`student` table:
| student_id | student_name | gender | dept_id |
| :--- | :--- | :--- | :--- |
| 1 | Jack | M | 1 |
| 2 | Jane | F | 1 |
| 3 | Mark | M | 2 |

`department` table:
| dept_id | dept_name |
| :--- | :--- |
| 1 | Engineering |
| 2 | Science |
| 3 | Law |

**Sample Output:**
| dept_name | student_number |
| :--- | :--- |
| Engineering | 2 |
| Science | 1 |
| Law | 0 |

**Problem Statement:**
Write a query to print the respective department name and number of students majoring in each department for all departments in the `department` table (even ones with no current students).

#### Query Solution:
```sql
SELECT 
    d.dept_name, 
    COUNT(s.student_id) AS student_number
FROM department d
LEFT JOIN student s 
    ON d.dept_id = s.dept_id
GROUP BY d.dept_id, d.dept_name
ORDER BY student_number DESC, d.dept_name ASC;
```

---

### 46. Investments in 2016 (LeetCode #585) | Medium

**Sample Input:**

`insurance` table:
| PID | TIV_2015 | TIV_2016 | LAT | LON |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 10 | 5 | 10 | 10 |
| 2 | 20 | 20 | 20 | 20 |
| 3 | 10 | 30 | 20 | 20 |
| 4 | 10 | 40 | 40 | 40 |

**Sample Output:**
| TIV_2016 |
| :--- |
| 45.00 |

**Problem Statement:**
Sum of all total investment values in 2016 (`TIV_2016`), to a scale of 2 decimal places, for all policyholders who:
1. Have the same `TIV_2015` value as one or more other policyholders.
2. Are not located in the same city as any other policyholder (unique `LAT, LON`).

#### Query Solution:
```sql
SELECT ROUND(SUM(TIV_2016), 2) AS TIV_2016
FROM insurance
WHERE TIV_2015 IN (
    SELECT TIV_2015
    FROM insurance
    GROUP BY TIV_2015
    HAVING COUNT(*) > 1
)
AND (LAT, LON) IN (
    SELECT LAT, LON
    FROM insurance
    GROUP BY LAT, LON
    HAVING COUNT(*) = 1
);
```

---

### 47. Friend Requests II: Who Has the Most Friends (LeetCode #602) | Medium

**Sample Input:**

`request_accepted` table:
| requester_id | accepter_id | accept_date |
| :--- | :--- | :--- |
| 1 | 2 | 2016_06-03 |
| 1 | 3 | 2016_06-08 |
| 2 | 3 | 2016_06-08 |
| 3 | 4 | 2016_06-09 |

**Sample Output:**
| id | num |
| :--- | :--- |
| 3 | 3 |

**Problem Statement:**
Write a query to find the person who has the most friends and the most friends number.

#### Query Solution:
```sql
WITH AllFriends AS (
    SELECT requester_id AS id FROM request_accepted
    UNION ALL
    SELECT accepter_id AS id FROM request_accepted
)
SELECT id, COUNT(*) AS num
FROM AllFriends
GROUP BY id
ORDER BY num DESC
LIMIT 1;
```

---

### 48. Tree Node (LeetCode #608) | Medium

**Sample Input:**

`tree` table:
| id | p_id |
| :--- | :--- |
| 1 | null |
| 2 | 1 |
| 3 | 1 |
| 4 | 2 |
| 5 | 2 |

**Sample Output:**
| id | Type |
| :--- | :--- |
| 1 | Root |
| 2 | Inner |
| 3 | Leaf |
| 4 | Leaf |
| 5 | Leaf |

**Problem Statement:**
Each node in the tree can be one of three types:
- `Root`: if the node is the root of the tree.
- `Leaf`: if the node is a leaf node.
- `Inner`: if the node is neither a leaf node nor a root node.
Write a query to print the node id and the type of the node. Sort by `id`.

#### Query Solution:
```sql
SELECT 
    id,
    CASE
        WHEN p_id IS NULL THEN 'Root'
        WHEN id IN (SELECT DISTINCT p_id FROM tree WHERE p_id IS NOT NULL) THEN 'Inner'
        ELSE 'Leaf'
    END AS Type
FROM tree
ORDER BY id;
```

---

### 49. Shortest Distance in a Plane (LeetCode #612) | Medium

**Sample Input:**

`point_2d` table:
| x | y |
| :--- | :--- |
| -1 | -1 |
| 0 | 0 |
| -1 | -2 |

**Sample Output:**
| shortest |
| :--- |
| 1.00 |

**Problem Statement:**
Write a query to find the shortest distance between points in a 2D plane rounded to 2 decimals.

#### Query Solution:
```sql
SELECT ROUND(MIN(SQRT(POW(p1.x - p2.x, 2) + POW(p1.y - p2.y, 2))), 2) AS shortest
FROM point_2d p1
JOIN point_2d p2 
    ON (p1.x != p2.x OR p1.y != p2.y);
```

---

### 50. Second Degree Follower (LeetCode #614) | Medium

**Sample Input:**

`follow` table:
| followee | follower |
| :--- | :--- |
| A | B |
| B | C |
| B | D |
| D | E |

**Sample Output:**
| follower | num |
| :--- | :--- |
| B | 2 |
| D | 1 |

**Problem Statement:**
Please write a SQL query to get the amount of each follower's follower if he/she has one. Display in follower's alphabetical order.

#### Query Solution:
```sql
SELECT 
    f1.follower, 
    COUNT(DISTINCT f2.follower) AS num
FROM follow f1
JOIN follow f2 
    ON f1.follower = f2.followee
GROUP BY f1.follower
ORDER BY f1.follower ASC;
```

---

### 51. Exchange Seats (LeetCode #626) | Medium

**Sample Input:**

`seat` table:
| id | student |
| :--- | :--- |
| 1 | Abbot |
| 2 | Doris |
| 3 | Emerson |
| 4 | Green |
| 5 | Jeames |

**Sample Output:**
| id | student |
| :--- | :--- |
| 1 | Doris |
| 2 | Abbot |
| 3 | Green |
| 4 | Emerson |
| 5 | Jeames |

**Problem Statement:**
Mary wants to change seats for adjacent students. Write a SQL query to output the result. If the number of students is odd, no need to change the last one's seat.

#### Query Solution:
```sql
SELECT
    CASE
        WHEN id % 2 = 1 AND id = (SELECT MAX(id) FROM seat) THEN id
        WHEN id % 2 = 1 THEN id + 1
        ELSE id - 1
    END AS id,
    student
FROM seat
ORDER BY id;
```

---

### 52. Customers Who Bought All Products (LeetCode #1045) | Medium

**Sample Input:**

`Customer` table:
| customer_id | product_key |
| :--- | :--- |
| 1 | 5 |
| 2 | 6 |
| 3 | 5 |
| 3 | 6 |
| 1 | 6 |

`Product` table:
| product_key |
| :--- |
| 5 |
| 6 |

**Sample Output:**
| customer_id |
| :--- |
| 1 |
| 3 |

**Problem Statement:**
Write an SQL query for a report that provides the customer ids from the `Customer` table that bought all the products in the `Product` table.

#### Query Solution:
```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
```

---

### 53. Product Sales Analysis III (LeetCode #1070) | Medium

**Sample Input:**

`Sales` table:
| sale_id | product_id | year | quantity | price |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 100 | 2008 | 10 | 5000 |
| 2 | 100 | 2009 | 12 | 5000 |
| 7 | 200 | 2011 | 15 | 9000 |

`Product` table:
| product_id | product_name |
| :--- | :--- |
| 100 | Nokia |
| 200 | Apple |
| 300 | Samsung |

**Sample Output:**
| product_id | first_year | quantity | price |
| :--- | :--- | :--- | :--- |
| 100 | 2008 | 10 | 5000 |
| 200 | 2011 | 15 | 9000 |

**Problem Statement:**
Write an SQL query that selects the `product_id, year, quantity, price` for the first year of every product sold.

#### Query Solution:
```sql
SELECT product_id, year AS first_year, quantity, price
FROM Sales
WHERE (product_id, year) IN (
    SELECT product_id, MIN(year)
    FROM Sales
    GROUP BY product_id
);
```

---

### 54. Project Employees III (LeetCode #1077) | Medium

**Sample Input:**

`Project` table:
| project_id | employee_id |
| :--- | :--- |
| 1 | 1 |
| 1 | 2 |
| 1 | 3 |
| 2 | 1 |
| 2 | 4 |

`Employee` table:
| employee_id | name | experience_years |
| :--- | :--- | :--- |
| 1 | Khaled | 3 |
| 2 | Ali | 2 |
| 3 | John | 3 |
| 4 | Doe | 2 |

**Sample Output:**
| project_id | employee_id |
| :--- | :--- |
| 1 | 1 |
| 1 | 3 |
| 2 | 1 |

**Problem Statement:**
Write an SQL query that reports the most experienced employees in each project. In case of a tie, report all employees with the maximum number of experience years.

#### Query Solution:
```sql
WITH Ranked AS (
    SELECT 
        p.project_id, 
        p.employee_id,
        DENSE_RANK() OVER (
            PARTITION BY p.project_id 
            ORDER BY e.experience_years DESC
        ) AS rnk
    FROM Project p
    JOIN Employee e 
        ON p.employee_id = e.employee_id
)
SELECT project_id, employee_id
FROM Ranked
WHERE rnk = 1;
```

---

### 55. Unpopular Books (LeetCode #1098) | Medium

**Sample Input:**

`Books` table:
| book_id | name | available_from |
| :--- | :--- | :--- |
| 1 | "Kalila And Demna" | 2010-01-01 |
| 2 | "28 Letters" | 2012-05-12 |
| 3 | "The Hobbit" | 2019-06-10 |
| 4 | "13 Reasons Why" | 2019-06-01 |
| 5 | "The Hunger Games" | 2008-09-21 |

`Orders` table:
| order_id | book_id | quantity | dispatch_date |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 2 | 2018-07-26 |
| 2 | 1 | 1 | 2018-11-05 |
| 3 | 3 | 8 | 2019-06-11 |
| 4 | 4 | 6 | 2019-06-05 |
| 5 | 4 | 5 | 2019-06-20 |
| 6 | 5 | 9 | 2009-02-02 |
| 7 | 5 | 8 | 2010-04-13 |

**Sample Output:**
| book_id | name |
| :--- | :--- |
| 1 | "Kalila And Demna" |
| 2 | "28 Letters" |
| 5 | "The Hunger Games" |

**Problem Statement:**
Write an SQL query that reports the books that have sold less than 10 copies in the last year, excluding books that have been available for less than 1 month from today. Assume today is `2019-06-23`.

#### Query Solution:
```sql
SELECT b.book_id, b.name
FROM Books b
LEFT JOIN Orders o 
    ON b.book_id = o.book_id 
   AND o.dispatch_date BETWEEN '2018-06-23' AND '2019-06-23'
WHERE b.available_from < '2019-05-23'
GROUP BY b.book_id, b.name
HAVING IFNULL(SUM(o.quantity), 0) < 10;
```

---

### 56. New Users Daily Count (LeetCode #1107) | Medium

**Sample Input:**

`Traffic` table:
| user_id | activity | activity_date |
| :--- | :--- | :--- |
| 1 | login | 2019-05-01 |
| 1 | homepage | 2019-05-01 |
| 1 | logout | 2019-05-01 |
| 2 | login | 2019-06-21 |
| 2 | logout | 2019-06-21 |
| 3 | login | 2019-01-01 |
| 3 | jobs | 2019-01-01 |
| 3 | logout | 2019-01-01 |
| 4 | login | 2019-06-21 |
| 4 | groups | 2019-06-21 |
| 4 | logout | 2019-06-21 |
| 5 | login | 2019-03-01 |
| 5 | logout | 2019-03-01 |
| 5 | login | 2019-06-21 |
| 5 | logout | 2019-06-21 |

**Sample Output:**
| login_date | user_count |
| :--- | :--- |
| 2019-05-01 | 1 |
| 2019-06-21 | 2 |

**Problem Statement:**
Write an SQL query that reports for every date within at most 90 days from today, the number of users that logged in for the first time on that date. Assume today is `2019-06-30`.

#### Query Solution:
```sql
WITH FirstLogins AS (
    SELECT user_id, MIN(activity_date) AS login_date
    FROM Traffic
    WHERE activity = 'login'
    GROUP BY user_id
)
SELECT login_date, COUNT(user_id) AS user_count
FROM FirstLogins
WHERE login_date BETWEEN DATE_SUB('2019-06-30', INTERVAL 90 DAY) AND '2019-06-30'
GROUP BY login_date;
```

---

### 57. Highest Grade For Each Student (LeetCode #1112) | Medium

**Sample Input:**

`Enrollments` table:
| student_id | course_id | grade |
| :--- | :--- | :--- |
| 2 | 2 | 95 |
| 2 | 3 | 95 |
| 1 | 1 | 90 |
| 1 | 2 | 99 |
| 3 | 1 | 80 |
| 3 | 2 | 75 |
| 3 | 3 | 82 |

**Sample Output:**
| student_id | course_id | grade |
| :--- | :--- | :--- |
| 1 | 2 | 99 |
| 2 | 2 | 95 |
| 3 | 3 | 82 |

**Problem Statement:**
Write a SQL query to find the highest grade with its corresponding course for each student. In case of a tie, you should find the course with the smallest `course_id`. Sort by `student_id` ascending.

#### Query Solution:
```sql
WITH RankedEnrollments AS (
    SELECT 
        student_id, 
        course_id, 
        grade,
        ROW_NUMBER() OVER (
            PARTITION BY student_id 
            ORDER BY grade DESC, course_id ASC
        ) AS rnk
    FROM Enrollments
)
SELECT student_id, course_id, grade
FROM RankedEnrollments
WHERE rnk = 1
ORDER BY student_id ASC;
```

---

### 58. Active Businesses (LeetCode #1126) | Medium

**Sample Input:**

`Events` table:
| business_id | event_type | occurences |
| :--- | :--- | :--- |
| 1 | reviews | 7 |
| 3 | reviews | 3 |
| 1 | ads | 11 |
| 2 | ads | 7 |
| 3 | ads | 6 |
| 1 | page views | 3 |
| 2 | page views | 12 |

**Sample Output:**
| business_id |
| :--- |
| 1 |

**Problem Statement:**
Write an SQL query to find all active businesses. An active business is a business that has more than one event type with occurrences greater than the average occurrences of that event type among all businesses.

#### Query Solution:
```sql
WITH EventAverages AS (
    SELECT 
        business_id, 
        event_type, 
        occurences,
        AVG(occurences) OVER (PARTITION BY event_type) AS avg_occurences
    FROM Events
)
SELECT business_id
FROM EventAverages
WHERE occurences > avg_occurences
GROUP BY business_id
HAVING COUNT(event_type) > 1;
```

---

### 59. Reported Posts II (LeetCode #1132) | Medium

**Sample Input:**

`Actions` table:
| user_id | post_id | action_date | action | extra |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 2019-07-01 | view | null |
| 1 | 1 | 2019-07-01 | like | null |
| 1 | 1 | 2019-07-01 | share | null |
| 2 | 2 | 2019-07-04 | view | null |
| 2 | 2 | 2019-07-04 | report | spam |
| 3 | 4 | 2019-07-04 | view | null |
| 3 | 4 | 2019-07-04 | report | spam |
| 4 | 3 | 2019-07-02 | view | null |
| 4 | 3 | 2019-07-02 | report | spam |
| 5 | 2 | 2019-07-03 | view | null |
| 5 | 2 | 2019-07-03 | report | racism |
| 5 | 5 | 2019-07-03 | view | null |
| 5 | 5 | 2019-07-03 | report | racism |

`Removals` table:
| post_id | remove_date |
| :--- | :--- |
| 2 | 2019-07-20 |
| 3 | 2019-07-18 |

**Sample Output:**
| average_daily_percent |
| :--- |
| 75.00 |

**Problem Statement:**
Write an SQL query to find the average for daily percentage of posts that got removed after being reported as spam, rounded to 2 decimal places.

#### Query Solution:
```sql
WITH DailySpamStats AS (
    SELECT 
        a.action_date,
        COUNT(DISTINCT r.post_id) / COUNT(DISTINCT a.post_id) * 100.0 AS daily_pct
    FROM Actions a
    LEFT JOIN Removals r 
        ON a.post_id = r.post_id
    WHERE a.action = 'report' AND a.extra = 'spam'
    GROUP BY a.action_date
)
SELECT ROUND(AVG(daily_pct), 2) AS average_daily_percent
FROM DailySpamStats;
```

---

### 60. Article Views II (LeetCode #1149) | Medium

**Sample Input:**

`Views` table:
| article_id | author_id | viewer_id | view_date |
| :--- | :--- | :--- | :--- |
| 1 | 3 | 5 | 2019-08-01 |
| 3 | 4 | 5 | 2019-08-01 |
| 1 | 3 | 6 | 2019-08-02 |
| 2 | 7 | 7 | 2019-08-01 |
| 2 | 7 | 6 | 2019-08-02 |
| 4 | 7 | 1 | 2019-07-22 |
| 3 | 4 | 4 | 2019-07-21 |
| 3 | 4 | 4 | 2019-07-21 |

**Sample Output:**
| id |
| :--- |
| 5 |
| 6 |

**Problem Statement:**
Write an SQL query to find all the people who viewed more than one article on the same date, sorted in ascending order by their id.

#### Query Solution:
```sql
SELECT DISTINCT viewer_id AS id
FROM Views
GROUP BY viewer_id, view_date
HAVING COUNT(DISTINCT article_id) > 1
ORDER BY id ASC;
```

---

### 61. Market Analysis I (LeetCode #1158) | Medium

**Sample Input:**

`Users` table:
| user_id | join_date | favorite_brand |
| :--- | :--- | :--- |
| 1 | 2018-01-01 | Lenovo |
| 2 | 2018-02-09 | Samsung |
| 3 | 2018-01-19 | LG |
| 4 | 2018-05-21 | HP |

`Orders` table:
| order_id | order_date | item_id | buyer_id | seller_id |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 2019-08-01 | 4 | 1 | 2 |
| 2 | 2018-08-02 | 2 | 1 | 3 |
| 3 | 2019-08-03 | 3 | 2 | 3 |
| 4 | 2018-08-04 | 1 | 4 | 2 |
| 5 | 2018-08-04 | 1 | 3 | 4 |
| 6 | 2019-08-05 | 2 | 2 | 4 |

`Items` table:
| item_id | item_brand |
| :--- | :--- |
| 1 | Samsung |
| 2 | Lenovo |
| 3 | LG |
| 4 | HP |

**Sample Output:**
| buyer_id | join_date | orders_in_2019 |
| :--- | :--- | :--- |
| 1 | 2018-01-01 | 1 |
| 2 | 2018-02-09 | 2 |
| 3 | 2018-01-19 | 0 |
| 4 | 2018-05-21 | 0 |

**Problem Statement:**
Write an SQL query to find for each user, the join date and the number of orders they made as a buyer in 2019.

#### Query Solution:
```sql
SELECT 
    u.user_id AS buyer_id, 
    u.join_date,
    COUNT(o.order_id) AS orders_in_2019
FROM Users u
LEFT JOIN Orders o 
    ON u.user_id = o.buyer_id 
   AND YEAR(o.order_date) = 2019
GROUP BY u.user_id, u.join_date;
```

---

### 62. Product Price at a Given Date (LeetCode #1164) | Medium

**Sample Input:**

`Products` table:
| product_id | new_price | change_date |
| :--- | :--- | :--- |
| 1 | 20 | 2019-08-14 |
| 2 | 50 | 2019-08-14 |
| 1 | 30 | 2019-08-15 |
| 1 | 35 | 2019-08-16 |
| 2 | 65 | 2019-08-17 |
| 3 | 20 | 2019-08-18 |

**Sample Output:**
| product_id | price |
| :--- | :--- |
| 2 | 50 |
| 1 | 35 |
| 3 | 10 |

**Problem Statement:**
Write an SQL query to find the prices of all products on `2019-08-16`. Assume the price of all products before any change is 10.

#### Query Solution:
```sql
WITH LatestPrices AS (
    SELECT 
        product_id, 
        new_price,
        ROW_NUMBER() OVER (
            PARTITION BY product_id 
            ORDER BY change_date DESC
        ) AS rnk
    FROM Products
    WHERE change_date <= '2019-08-16'
),
AllProducts AS (
    SELECT DISTINCT product_id 
    FROM Products
)
SELECT 
    ap.product_id, 
    IFNULL(lp.new_price, 10) AS price
FROM AllProducts ap
LEFT JOIN LatestPrices lp 
    ON ap.product_id = lp.product_id 
   AND lp.rnk = 1;
```

---

<a id="hard-problems"></a>
## 🔴 Section 3: HARD PROBLEMS (11 Questions)
[⬆ Back to Top](#top)

---

### 63. Department Top Three Salaries (LeetCode #185) | Hard

**Sample Input:**

`Employee` table:
| Id | Name | Salary | DepartmentId |
| :--- | :--- | :--- | :--- |
| 1 | Joe | 85000 | 1 |
| 2 | Henry | 80000 | 2 |
| 3 | Sam | 60000 | 2 |
| 4 | Max | 90000 | 1 |
| 5 | Janet | 69000 | 1 |
| 6 | Randy | 85000 | 1 |
| 7 | Will | 70000 | 1 |

`Department` table:
| Id | Name |
| :--- | :--- |
| 1 | IT |
| 2 | Sales |

**Sample Output:**
| Department | Employee | Salary |
| :--- | :--- | :--- |
| IT | Max | 90000 |
| IT | Randy | 85000 |
| IT | Joe | 85000 |
| IT | Will | 70000 |
| Sales | Henry | 80000 |
| Sales | Sam | 60000 |

**Problem Statement:**
Write a SQL query to find employees who earn the top three salaries in each of the department.

#### Query Solution:
```sql
WITH RankedEmployee AS (
    SELECT 
        DepartmentId,
        Name AS Employee,
        Salary,
        DENSE_RANK() OVER (
            PARTITION BY DepartmentId 
            ORDER BY Salary DESC
        ) AS rnk
    FROM Employee
)
SELECT 
    d.Name AS Department,
    r.Employee,
    r.Salary
FROM RankedEmployee r
JOIN Department d 
    ON r.DepartmentId = d.Id
WHERE r.rnk <= 3;
```

---

### 64. Trips and Users (LeetCode #262) | Hard

**Sample Input:**

`Trips` table:
| Id | Client_Id | Driver_Id | City_Id | Status | Request_at |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 10 | 1 | completed | 2013-10-01 |
| 2 | 2 | 11 | 1 | cancelled_by_driver | 2013-10-01 |
| 3 | 3 | 12 | 6 | completed | 2013-10-01 |
| 4 | 4 | 13 | 6 | cancelled_by_client | 2013-10-01 |
| 5 | 1 | 10 | 1 | completed | 2013-10-02 |
| 6 | 2 | 11 | 6 | completed | 2013-10-02 |
| 7 | 3 | 12 | 6 | completed | 2013-10-02 |
| 8 | 2 | 12 | 12 | completed | 2013-10-03 |
| 9 | 3 | 10 | 12 | completed | 2013-10-03 |
| 10 | 4 | 13 | 12 | cancelled_by_driver | 2013-10-03 |

`Users` table:
| Users_Id | Banned | Role |
| :--- | :--- | :--- |
| 1 | No | client |
| 2 | Yes | client |
| 3 | No | client |
| 4 | No | client |
| 10 | No | driver |
| 11 | No | driver |
| 12 | No | driver |
| 13 | No | driver |

**Sample Output:**
| Day | Cancellation Rate |
| :--- | :--- |
| 2013-10-01 | 0.33 |
| 2013-10-02 | 0.00 |
| 2013-10-03 | 0.50 |

**Problem Statement:**
Write a SQL query to find the cancellation rate of requests with unbanned users (both client and driver must not be banned) each day between `"2013-10-01"` and `"2013-10-03"`. Round to two decimal points.

#### Query Solution:
```sql
SELECT 
    Request_at AS Day,
    ROUND(SUM(IF(Status <> 'completed', 1, 0)) / COUNT(*), 2) AS `Cancellation Rate`
FROM Trips
WHERE Request_at BETWEEN '2013-10-01' AND '2013-10-03'
  AND Client_Id NOT IN (SELECT Users_Id FROM Users WHERE Banned = 'Yes')
  AND Driver_Id NOT IN (SELECT Users_Id FROM Users WHERE Banned = 'Yes')
GROUP BY Request_at;
```

---

### 65. Median Employee Salary (LeetCode #569) | Hard

**Sample Input:**

`Employee` table:
| Id | Company | Salary |
| :--- | :--- | :--- |
| 1 | A | 2341 |
| 2 | A | 341 |
| 3 | A | 15 |
| 4 | A | 15314 |
| 5 | A | 451 |
| 6 | A | 513 |
| 7 | B | 15 |
| 8 | B | 13 |
| 9 | B | 1154 |
| 10 | B | 1345 |
| 11 | B | 1221 |
| 12 | B | 234 |
| 13 | C | 2345 |
| 14 | C | 2645 |
| 15 | C | 2645 |
| 16 | C | 2652 |
| 17 | C | 65 |

**Sample Output:**
| Id | Company | Salary |
| :--- | :--- | :--- |
| 5 | A | 451 |
| 6 | A | 513 |
| 12 | B | 234 |
| 9 | B | 1154 |
| 14 | C | 2645 |

**Problem Statement:**
Write a SQL query to find the median salary of each company. Bonus points if you can solve it without using any built-in SQL functions.

#### Query Solution:
```sql
WITH Ranked AS (
    SELECT 
        Id, 
        Company, 
        Salary,
        ROW_NUMBER() OVER (PARTITION BY Company ORDER BY Salary, Id) AS rnk,
        COUNT(*) OVER (PARTITION BY Company) AS total_count
    FROM Employee
)
SELECT Id, Company, Salary
FROM Ranked
WHERE rnk BETWEEN total_count / 2.0 AND total_count / 2.0 + 1;
```

---

### 66. Find Median Given Frequency of Numbers (LeetCode #571) | Hard

**Sample Input:**

`Numbers` table:
| Number | Frequency |
| :--- | :--- |
| 0 | 7 |
| 1 | 1 |
| 2 | 3 |
| 3 | 1 |

**Sample Output:**
| median |
| :--- |
| 0.0000 |

**Problem Statement:**
Write a query to find the median of all numbers and name the result as `median`.

#### Query Solution:
```sql
WITH Cumulative AS (
    SELECT 
        Number, 
        Frequency,
        SUM(Frequency) OVER (ORDER BY Number) AS cum_freq,
        SUM(Frequency) OVER () AS total_freq
    FROM Numbers
)
SELECT ROUND(AVG(Number * 1.0), 4) AS median
FROM Cumulative
WHERE total_freq / 2.0 <= cum_freq
  AND total_freq / 2.0 >= cum_freq - Frequency;
```

---

### 67. Find Cumulative Salary of an Employee (LeetCode #579) | Hard

**Sample Input:**

`Employee` table:
| Id | Month | Salary |
| :--- | :--- | :--- |
| 1 | 1 | 20 |
| 2 | 1 | 20 |
| 1 | 2 | 30 |
| 2 | 2 | 30 |
| 3 | 2 | 40 |
| 1 | 3 | 40 |
| 3 | 3 | 60 |
| 1 | 4 | 60 |
| 3 | 4 | 70 |

**Sample Output:**
| Id | Month | Salary |
| :--- | :--- | :--- |
| 1 | 3 | 90 |
| 1 | 2 | 50 |
| 1 | 1 | 20 |
| 2 | 1 | 20 |
| 3 | 3 | 100 |
| 3 | 2 | 40 |

**Problem Statement:**
Write a SQL to get the cumulative sum of an employee's salary over a period of 3 months but exclude the most recent month. Display by `Id` ascending and `Month` descending.

#### Query Solution:
```sql
WITH RankedSalary AS (
    SELECT 
        Id, 
        Month, 
        Salary,
        SUM(Salary) OVER (
            PARTITION BY Id 
            ORDER BY Month 
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ) AS cum_salary,
        DENSE_RANK() OVER (PARTITION BY Id ORDER BY Month DESC) AS rnk
    FROM Employee
)
SELECT Id, Month, cum_salary AS Salary
FROM RankedSalary
WHERE rnk > 1
ORDER BY Id ASC, Month DESC;
```

---

### 68. Human Traffic of Stadium (LeetCode #601) | Hard

**Sample Input:**

`Stadium` table:
| id | visit_date | people |
| :--- | :--- | :--- |
| 1 | 2017-01-01 | 10 |
| 2 | 2017-01-02 | 109 |
| 3 | 2017-01-03 | 150 |
| 4 | 2017-01-04 | 99 |
| 5 | 2017-01-05 | 145 |
| 6 | 2017-01-06 | 1455 |
| 7 | 2017-01-07 | 199 |
| 8 | 2017-01-09 | 188 |

**Sample Output:**
| id | visit_date | people |
| :--- | :--- | :--- |
| 5 | 2017-01-05 | 145 |
| 6 | 2017-01-06 | 1455 |
| 7 | 2017-01-07 | 199 |
| 8 | 2017-01-09 | 188 |

**Problem Statement:**
Write an SQL query to display the records with three or more rows with consecutive id's, and the number of people is greater than or equal to 100 for each. Return the result table ordered by `visit_date` in ascending order.

#### Query Solution:
```sql
WITH GroupedTraffic AS (
    SELECT 
        id, 
        visit_date, 
        people,
        id - ROW_NUMBER() OVER (ORDER BY id) AS grp
    FROM Stadium
    WHERE people >= 100
)
SELECT id, visit_date, people
FROM GroupedTraffic
WHERE grp IN (
    SELECT grp
    FROM GroupedTraffic
    GROUP BY grp
    HAVING COUNT(*) >= 3
)
ORDER BY visit_date ASC;
```

---

### 69. Average Salary: Departments VS Company (LeetCode #615) | Hard

**Sample Input:**

`salary` table:
| id | employee_id | amount | pay_date |
| :--- | :--- | :--- | :--- |
| 1 | 1 | 9000 | 2017-03-31 |
| 2 | 2 | 6000 | 2017-03-31 |
| 3 | 3 | 10000 | 2017-03-31 |
| 4 | 1 | 7000 | 2017-02-28 |
| 5 | 2 | 6000 | 2017-02-28 |
| 6 | 3 | 8000 | 2017-02-28 |

`employee` table:
| employee_id | department_id |
| :--- | :--- |
| 1 | 1 |
| 2 | 2 |
| 3 | 2 |

**Sample Output:**
| pay_month | department_id | comparison |
| :--- | :--- | :--- |
| 2017-03 | 1 | higher |
| 2017-03 | 2 | lower |
| 2017-02 | 1 | same |
| 2017-02 | 2 | same |

**Problem Statement:**
Given two tables as above, write a query to display the comparison result (higher/lower/same) of the average salary of employees in a department to the company's average salary.

#### Query Solution:
```sql
WITH MonthlyDeptAvg AS (
    SELECT 
        DATE_FORMAT(s.pay_date, '%Y-%m') AS pay_month,
        e.department_id,
        AVG(s.amount) AS dept_avg,
        AVG(AVG(s.amount)) OVER (PARTITION BY DATE_FORMAT(s.pay_date, '%Y-%m')) AS comp_avg
    FROM salary s
    JOIN employee e 
        ON s.employee_id = e.employee_id
    GROUP BY DATE_FORMAT(s.pay_date, '%Y-%m'), e.department_id
)
SELECT 
    pay_month, 
    department_id,
    CASE
        WHEN dept_avg > comp_avg THEN 'higher'
        WHEN dept_avg < comp_avg THEN 'lower'
        ELSE 'same'
    END AS comparison
FROM MonthlyDeptAvg
ORDER BY pay_month DESC, department_id;
```

---

### 70. Students Report By Geography (LeetCode #618) | Hard

**Sample Input:**

`student` table:
| name | continent |
| :--- | :--- |
| Jack | America |
| Pascal | Europe |
| Xi | Asia |
| Jane | America |

**Sample Output:**
| America | Asia | Europe |
| :--- | :--- | :--- |
| Jack | Xi | Pascal |
| Jane | null | null |

**Problem Statement:**
Pivot the `continent` column in this table so that each name is sorted alphabetically and displayed underneath its corresponding continent.

#### Query Solution:
```sql
WITH Ranked AS (
    SELECT 
        name, 
        continent,
        ROW_NUMBER() OVER (PARTITION BY continent ORDER BY name) AS rnk
    FROM student
)
SELECT
    MAX(CASE WHEN continent = 'America' THEN name END) AS America,
    MAX(CASE WHEN continent = 'Asia' THEN name END) AS Asia,
    MAX(CASE WHEN continent = 'Europe' THEN name END) AS Europe
FROM Ranked
GROUP BY rnk;
```

---

### 71. Game Play Analysis V (LeetCode #1097) | Hard

**Sample Input:**

`Activity` table:
| player_id | device_id | event_date | games_played |
| :--- | :--- | :--- | :--- |
| 1 | 2 | 2016-03-01 | 5 |
| 1 | 2 | 2016-03-02 | 6 |
| 2 | 3 | 2017-06-25 | 1 |
| 3 | 1 | 2016-03-01 | 0 |
| 3 | 4 | 2016-07-03 | 5 |

**Sample Output:**
| install_dt | installs | Day1_retention |
| :--- | :--- | :--- |
| 2016-03-01 | 2 | 0.50 |
| 2017-06-25 | 1 | 0.00 |

**Problem Statement:**
We define the install date of a player to be the first login day of that player. We also define day 1 retention to be the number of players whose install date is $X$ and they logged back in on the day right after $X$, divided by the number of players whose install date is $X$, rounded to 2 decimal places.

#### Query Solution:
```sql
WITH FirstLogins AS (
    SELECT player_id, MIN(event_date) AS install_dt
    FROM Activity
    GROUP BY player_id
)
SELECT 
    f.install_dt,
    COUNT(f.player_id) AS installs,
    ROUND(COUNT(a.player_id) / COUNT(f.player_id), 2) AS Day1_retention
FROM FirstLogins f
LEFT JOIN Activity a 
    ON f.player_id = a.player_id 
   AND a.event_date = DATE_ADD(f.install_dt, INTERVAL 1 DAY)
GROUP BY f.install_dt;
```

---

### 72. User Purchase Platform (LeetCode #1127) | Hard

**Sample Input:**

`Spending` table:
| user_id | spend_date | platform | amount |
| :--- | :--- | :--- | :--- |
| 1 | 2019-07-01 | mobile | 100 |
| 1 | 2019-07-01 | desktop | 100 |
| 2 | 2019-07-01 | mobile | 100 |
| 2 | 2019-07-02 | mobile | 100 |
| 3 | 2019-07-01 | desktop | 100 |
| 3 | 2019-07-02 | desktop | 100 |

**Sample Output:**
| spend_date | platform | total_amount | total_users |
| :--- | :--- | :--- | :--- |
| 2019-07-01 | desktop | 100 | 1 |
| 2019-07-01 | mobile | 100 | 1 |
| 2019-07-01 | both | 200 | 1 |
| 2019-07-02 | desktop | 100 | 1 |
| 2019-07-02 | mobile | 100 | 1 |
| 2019-07-02 | both | 0 | 0 |

**Problem Statement:**
Write an SQL query to find the total number of users and the total amount spent using mobile only, desktop only, and both mobile and desktop together for each date.

#### Query Solution:
```sql
WITH DatesPlatforms AS (
    SELECT DISTINCT spend_date, 'desktop' AS platform FROM Spending
    UNION
    SELECT DISTINCT spend_date, 'mobile' AS platform FROM Spending
    UNION
    SELECT DISTINCT spend_date, 'both' AS platform FROM Spending
),
UserDailySpending AS (
    SELECT 
        spend_date, 
        user_id,
        CASE 
            WHEN COUNT(DISTINCT platform) = 2 THEN 'both'
            ELSE MAX(platform)
        END AS platform,
        SUM(amount) AS total_amount
    FROM Spending
    GROUP BY spend_date, user_id
)
SELECT 
    dp.spend_date, 
    dp.platform,
    IFNULL(SUM(uds.total_amount), 0) AS total_amount,
    COUNT(uds.user_id) AS total_users
FROM DatesPlatforms dp
LEFT JOIN UserDailySpending uds 
    ON dp.spend_date = uds.spend_date 
   AND dp.platform = uds.platform
GROUP BY dp.spend_date, dp.platform;
```

---

### 73. Market Analysis II (LeetCode #1159) | Hard

**Sample Input:**

`Users` table:
| user_id | join_date | favorite_brand |
| :--- | :--- | :--- |
| 1 | 2019-01-01 | Lenovo |
| 2 | 2019-02-09 | Samsung |
| 3 | 2019-01-19 | LG |
| 4 | 2019-05-21 | HP |

`Orders` table:
| order_id | order_date | item_id | buyer_id | seller_id |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 2019-08-01 | 4 | 1 | 2 |
| 2 | 2019-08-02 | 2 | 1 | 3 |
| 3 | 2019-08-03 | 3 | 2 | 3 |
| 4 | 2019-08-04 | 1 | 4 | 2 |
| 5 | 2019-08-04 | 1 | 3 | 4 |
| 6 | 2019-08-05 | 2 | 2 | 4 |

`Items` table:
| item_id | item_brand |
| :--- | :--- |
| 1 | Samsung |
| 2 | Lenovo |
| 3 | LG |
| 4 | HP |

**Sample Output:**
| seller_id | 2nd_item_fav_brand |
| :--- | :--- |
| 1 | no |
| 2 | yes |
| 3 | yes |
| 4 | no |

**Problem Statement:**
Write an SQL query to find for each user, whether the brand of the second item (by date) they sold is their favorite brand. If a user sold less than two items, report the answer for that user as no.

#### Query Solution:
```sql
WITH RankedSales AS (
    SELECT 
        o.seller_id, 
        i.item_brand,
        ROW_NUMBER() OVER (
            PARTITION BY o.seller_id 
            ORDER BY o.order_date
        ) AS rnk
    FROM Orders o
    JOIN Items i 
        ON o.item_id = i.item_id
)
SELECT 
    u.user_id AS seller_id,
    CASE 
        WHEN u.favorite_brand = rs.item_brand THEN 'yes'
        ELSE 'no'
    END AS 2nd_item_fav_brand
FROM Users u
LEFT JOIN RankedSales rs 
    ON u.user_id = rs.seller_id 
   AND rs.rnk = 2;
```
