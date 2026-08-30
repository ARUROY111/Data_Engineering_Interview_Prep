# Interview Preparation Guide: Python & SQL Coding Questions

This document compiles and structures all interview questions, logic, and code solutions extracted from your screenshots.

---

## 📑 Table of Contents
- [Part 1: Python Programming Problems](#part-1-python-programming-problems)
  - [1. Reverse a List](#1-reverse-a-list)
  - [2. Find the Nth Largest Element Without Built-in Sort](#2-find-the-nth-largest-element-without-built-in-sort)
  - [3. Remove Duplicates While Maintaining Order](#3-remove-duplicates-while-maintaining-order)
  - [4. Count Frequency of Elements](#4-count-frequency-of-elements)
  - [5. Merge Two Dictionaries](#5-merge-two-dictionaries)
  - [6. Find Common Elements Between Two Lists](#6-find-common-elements-between-two-lists)
  - [7. Sum Digits Present Inside Strings](#7-sum-digits-present-inside-strings)
  - [8. Reverse Each Word in a Sentence](#8-reverse-each-word-in-a-sentence)
  - [9. Check if a String is a Palindrome](#9-check-if-a-string-is-a-palindrome)
  - [10. First Non-Repeating Character in a String](#10-first-non-repeating-character-in-a-string)
  - [11. Find All Pairs in a List That Sum to Target](#11-find-all-pairs-in-a-list-that-sum-to-target)
  - [12. Flatten a Nested List](#12-flatten-a-nested-list)
  - [13. Implement Fibonacci Sequence](#13-implement-fibonacci-sequence)
  - [14. Find Duplicate Elements in a List](#14-find-duplicate-elements-in-a-list)
  - [15. Sort a List of Dictionaries by Key](#15-sort-a-list-of-dictionaries-by-key)
  - [16. Rotate a List K Positions to the Right](#16-rotate-a-list-k-positions-to-the-right)
  - [17. Move All Zeroes to the End of the List](#17-move-all-zeroes-to-the-end-of-the-list)
  - [18. Longest Word in a Sentence](#18-longest-word-in-a-sentence)
  - [19. Reverse String Preserving Non-Alphanumeric Character Positions](#19-reverse-string-preserving-non-alphanumeric-character-positions)
  - [20. Longest Common Prefix](#20-longest-common-prefix)
  - [21. Sort a String Alphabetically Without Built-in Sort](#21-sort-a-string-alphabetically-without-built-in-sort)
- [Part 2: SQL Top 15 Interview Queries](#part-2-sql-top-15-interview-queries)
  - [Q1. Second Highest Salary](#q1-second-highest-salary)
  - [Q2. Employees Earning More Than Average Salary](#q2-employees-earning-more-than-average-salary)
  - [Q3. Employees Earning More Than Department Average](#q3-employees-earning-more-than-department-average)
  - [Q4. List Employees With the Same Manager](#q4-list-employees-with-the-same-manager)
  - [Q5. Nth Highest Salary for Each Department](#q5-nth-highest-salary-for-each-department)
  - [Q6. Find Continuous Sequence Gaps in IDs](#q6-find-continuous-sequence-gaps-in-ids)
  - [Q7. Count Employees by Salary Band](#q7-count-employees-by-salary-band)
  - [Q8. Find Duplicate Records in a Table](#q8-find-duplicate-records-in-a-table)
  - [Q9. Top 3 Products by Sales for Each Month](#q9-top-3-products-by-sales-for-each-month)
  - [Q10. Cumulative Sum of Sales for Each Region](#q10-cumulative-sum-of-sales-for-each-region)
  - [Q11. Employees Who Joined Before Their Manager](#q11-employees-who-joined-before-their-manager)
  - [Q12. Delete Duplicate Records Keeping One](#q12-delete-duplicate-records-keeping-one)
  - [Q13. Salary Higher Than All Employees in Another Department](#q13-salary-higher-than-all-employees-in-another-department)
  - [Q14. Second Most Recent Order for Each Customer](#q14-second-most-recent-order-for-each-customer)
  - [Q15. Departments Where All Employees Earn Over 5000](#q15-departments-where-all-employees-earn-over-5000)

---

# Part 1: Python Programming Problems

### 1. Reverse a List
- **Input:** `[1, 2, 3, 4]`
- **Output:** `[4, 3, 2, 1]`
- **Focus:** Slicing, loops, list operations

```python
Input = [1, 2, 3, 4]
Output = []
for i in Input[::-1]:
    Output.append(i)
print("reverse list:", Output)
```

---

### 2. Find the Nth Largest Element Without Built-in Sort
- **Input:** `[10, 20, 40, 30, 40]`, `n = 3`
- **Output:** `20` / `30`
- **Focus:** Loops, conditional logic, maintaining top values

```python
Input = [10, 20, 40, 30, 40]

def nth_largest(Input, n):
    top = set()
    for i in Input:
        top.add(i)
        if len(top) > n:
            top.remove(min(top)) # Maintain exactly n largest elements
    if len(top) < n:
        return "nth largest element doesn't exist"
    return min(top)

print("Third largest element:", nth_largest(Input, 3))
```

---

### 3. Remove Duplicates While Maintaining Order
- **Input:** `[1, 2, 3, 2, 1]`
- **Output:** `[1, 2, 3]`
- **Focus:** Sets, lists, iteration

```python
Input = [1, 2, 3, 2, 1]
Output = []
for i in Input:
    if i not in Output:
        Output.append(i)
print("unique list with order:", Output)
```

---

### 4. Count Frequency of Elements
- **Input:** `['a', 'b', 'a']`
- **Output:** `{'a': 2, 'b': 1}`
- **Focus:** Dictionaries, frequency counting

```python
Input = ["a", "b", "a"]
Output = {}
for i in Input:
    if i in Output:
        Output[i] += 1
    else:
        Output[i] = 1
print("frequency of elements:", Output)
```

---

### 5. Merge Two Dictionaries
- **Input:** `{'a': 1}` & `{'b': 2}`
- **Output:** `{'a': 1, 'b': 2}`
- **Focus:** Dictionary merging

```python
d1 = {"a": 1}
d2 = {"b": 2}
d1.update(d2)
print("Merged d1 and d2:", d1)
```

---

### 6. Find Common Elements Between Two Lists
- **Input:** `[1, 2, 3]` & `[2, 3, 4]`
- **Output:** `[2, 3]`
- **Focus:** Loops, sets, membership checking

```python
l1 = [1, 2, 3]
l2 = [2, 3, 4]
Output = []
for i in l1:
    if i in l2:
        Output.append(i)
print("common elements:", Output)
```

---

### 7. Sum Digits Present Inside Strings
- **Input:** `["a1b2", "34c", "x9"]`
- **Output:** `19` (1 + 2 + 3 + 4 + 9)
- **Focus:** String traversal, `.isdigit()`, integer conversion

```python
Input = ["a1b2", "34c", "x9"]
total = 0
for word in Input:
    for letter in word:
        if letter.isdigit():
            total += int(letter)
print("Total sum:", total)
```

---

### 8. Reverse Each Word in a Sentence
- **Input:** `"hello world"`
- **Output:** `"olleh dlrow"`
- **Focus:** String slicing, `.split()`, `.join()`

```python
Input = "hello world"
input_list = Input.split()
Output = []
for i in input_list:
    Output.append(i[::-1])
result = " ".join(Output)
print("Reversed words:", result)
```

---

### 9. Check if a String is a Palindrome
- **Input:** `'level'`
- **Output:** `True`
- **Focus:** String slicing / reversal

```python
Input = 'level'
Output = ""
for i in Input[::-1]:
    Output += i
print("Is palindrome:", Input == Output)
```

---

### 10. First Non-Repeating Character in a String
- **Input:** `"aabbcdde"`
- **Output:** `"c"`
- **Focus:** Frequency hash map, linear scan

```python
Input = "aabbcdde"
freq = {}
for i in Input:
    freq[i] = freq.get(i, 0) + 1

for ch in Input:
    if freq[ch] == 1:
        print("First non-repeating character:", ch)
        break
```

---

### 11. Find All Pairs in a List That Sum to Target
- **Input:** `[1, 2, 3, 4]`, `target = 5`
- **Output:** `[(1, 4), (2, 3)]`
- **Focus:** Two-sum approach with deduplication set

```python
Input = [1, 2, 3, 4]
target = 5
Output = []
seen = set()
for i in Input:
    com = target - i
    if com in Input and com not in seen:
        Output.append((i, com))
        seen.add(i)
        seen.add(com)
print("Pairs:", Output)
```

---

### 12. Flatten a Nested List
- **Input:** `[1, [2, 3], [4, [5]]]`
- **Output:** `[1, 2, 3, 4, 5]`
- **Focus:** Stack (iterative) or Recursion

```python
# Iterative using Stack:
Input = [1, [2, 3], [4, [5]]]
Output = []
reverse = Input[::-1]
while reverse:
    item = reverse.pop()
    if isinstance(item, list):
        reverse.extend(item[::-1])
    else:
        Output.append(item)
print("Flattened list (Iterative):", Output)

# Recursive:
Output_rec = []
def flatten_list(lst):
    for i in lst:
        if isinstance(i, list):
            flatten_list(i)
        else:
            Output_rec.append(i)

flatten_list(Input)
print("Flattened list (Recursive):", Output_rec)
```

---

### 13. Implement Fibonacci Sequence
- **Input:** `n = 5`
- **Output:** `[0, 1, 1, 2, 3]`
- **Focus:** Iterative sequence generation

```python
n = 5
Output = [0, 1]
for i in range(2, n):
    Output.append(Output[-1] + Output[-2])
print("Fibonacci series:", Output)
```

---

### 14. Find Duplicate Elements in a List
- **Input:** `[1, 2, 2, 3, 3, 3]`
- **Output:** `[2, 3]`
- **Focus:** Seen set / list tracking

```python
Input = [1, 2, 2, 3, 3, 3]
Output = []
duplicate_item = []
for i in Input:
    if i in Output:
        duplicate_item.append(i)
    else:
        Output.append(i)
print("Duplicates:", list(set(duplicate_item)))
```

---

### 15. Sort a List of Dictionaries by Key
- **Input:** `[{'name':'Alice','age':25}, {'name':'Bob','age':20}]`
- **Output:** `[{'name':'Bob','age':20}, {'name':'Alice','age':25}]`
- **Focus:** `sorted()` with `lambda`

```python
Input = [{'name': 'Alice', 'age': 25}, {'name': 'Bob', 'age': 20}]
Output = sorted(Input, key=lambda x: x['age'])
print("Sorted dictionaries:", Output)
```

---

### 16. Rotate a List K Positions to the Right
- **Input:** `[1, 2, 3, 4, 5]`, `k = 2`
- **Output:** `[4, 5, 1, 2, 3]`
- **Focus:** Modulo arithmetic, list slicing

```python
arr = [1, 2, 3, 4, 5]
k = 2
k = k % len(arr)
rotated = arr[-k:] + arr[:-k]
print("Rotated list:", rotated)
```

---

### 17. Move All Zeroes to the End of the List
- **Input:** `[0, 1, 0, 3, 12]`
- **Output:** `[1, 3, 12, 0, 0]`
- **Focus:** Filtering and appending

```python
Input = [0, 1, 0, 3, 12]
Output = []
count = 0
for i in Input:
    if i == 0:
        count += 1
    else:
        Output.append(i)
for _ in range(count):
    Output.append(0)
print("Zeroes moved to end:", Output)
```

---

### 18. Longest Word in a Sentence
- **Input:** `"Python is awesome"`
- **Output:** `"awesome"`
- **Focus:** String length finding

```python
s = "Python is awesome"
s_list = s.split()
result = [len(i) for i in s_list]
index = result.index(max(result))
print("Longest word:", s_list[index])
```

---

### 19. Reverse String Preserving Non-Alphanumeric Character Positions
- **Input:** `"a!b@c#1$2"`
- **Output:** `"2!1@c#b$a"`
- **Focus:** Character classification, pointer/stack replacement

```python
def rev_alnum(Input):
    Input = list(Input)
    Output = [i for i in Input if i.isalnum()]
    rev_output = Output[::-1]
    
    result = ""
    idx = 0
    for char in Input:
        if char.isalnum():
            result += rev_output[idx]
            idx += 1
        else:
            result += char
    return result

print("Result:", rev_alnum("a!b@c#1$2"))
```

---

### 20. Longest Common Prefix
- **Input:** `["flower", "flow", "flight"]`
- **Output:** `"fl"`
- **Focus:** Prefix reduction with `.startswith()`

```python
strs = ["flower", "flow", "flight"]

def longest_common_prefix(strs):
    if not strs:
        return ""
    prefix = strs[0]
    for char in strs[1:]:
        while not char.startswith(prefix):
            prefix = prefix[:-1]
            if not prefix:
                return ""
    return prefix

print("Longest common prefix:", longest_common_prefix(strs))
```

---

### 21. Sort a String Alphabetically Without Built-in Sort
- **Input:** `"dcba"`
- **Output:** `"abcd"`
- **Focus:** Selection sort logic on string

```python
s = "dcba"
def str_sort(s):
    result = ""
    while s:
        smallest = s[0]
        for i in s:
            if i < smallest:
                smallest = i
        result += smallest
        s = s.replace(smallest, "", 1)
    return result

print("Sorted string:", str_sort(s))
```

---
---

# Part 2: SQL Top 15 Interview Queries

### Q1. Second Highest Salary
```sql
-- Method 1: Subquery with MAX()
SELECT MAX(salary) AS second_highest_salary
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);

-- Method 2: Window Function (ROW_NUMBER / DENSE_RANK)
SELECT salary
FROM (
    SELECT salary, ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
    FROM employee
) t
WHERE rn = 2;
```

---

### Q2. Employees Earning More Than Average Salary
```sql
SELECT employee, salary, (SELECT AVG(salary) FROM employee) AS avg_salary
FROM employee
WHERE salary > (SELECT AVG(salary) FROM employee);
```

---

### Q3. Employees Earning More Than Department Average
```sql
SELECT employee, salary, avg_dept_salary
FROM (
    SELECT 
        employee,
        salary, 
        AVG(salary) OVER (PARTITION BY department) AS avg_dept_salary
    FROM employee
) t
WHERE salary > avg_dept_salary;
```

---

### Q4. List Employees With the Same Manager
```sql
SELECT 
    e1.emp_id, 
    e1.emp_name, 
    e1.manager_id
FROM employee e1
JOIN employee e2
    ON e1.manager_id = e2.manager_id
    AND e1.emp_id <> e2.emp_id;
```

---

### Q5. Nth Highest Salary for Each Department (e.g. n = 5)
```sql
SELECT employee, salary
FROM (
    SELECT 
        employee,
        salary, 
        DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employee
) t
WHERE rn = 5;
```

---

### Q6. Find Continuous Sequence Gaps in IDs
```sql
-- Identifies gap when current ID and previous ID differ by 2
SELECT (prev_id + 1) AS missing_id
FROM (
    SELECT 
        emp_id, 
        LAG(emp_id) OVER (ORDER BY emp_id) AS prev_id
    FROM employee
) t
WHERE (emp_id - prev_id) = 2;
```

---

### Q7. Count Employees by Salary Band
```sql
SELECT 
    CASE
        WHEN salary BETWEEN 0 AND 5000 THEN '0-5000'
        WHEN salary BETWEEN 5001 AND 10000 THEN '5001-10000'
        ELSE '15001+'
    END AS salary_band,
    COUNT(*) AS employee_count
FROM employee
GROUP BY 
    CASE
        WHEN salary BETWEEN 0 AND 5000 THEN '0-5000'
        WHEN salary BETWEEN 5001 AND 10000 THEN '5001-10000'
        ELSE '15001+'
    END
ORDER BY salary_band;
```

---

### Q8. Find Duplicate Records in a Table
```sql
SELECT emp_name, department, salary, COUNT(*) AS total_count
FROM employee
GROUP BY emp_name, department, salary
HAVING COUNT(*) > 1;
```

---

### Q9. Top 3 Products by Sales for Each Month
```sql
SELECT product, sales, month
FROM (
    SELECT 
        product, 
        sales, 
        month, 
        ROW_NUMBER() OVER (PARTITION BY month ORDER BY sales DESC) AS rn
    FROM employee
) t
WHERE rn <= 3;
```

---

### Q10. Cumulative Sum of Sales for Each Region
```sql
SELECT region, cumulative_sales
FROM (
    SELECT 
        region, 
        SUM(sales) OVER (PARTITION BY region ORDER BY date) AS cumulative_sales
    FROM employee
) t;
```

---

### Q11. Employees Who Joined Before Their Manager
```sql
SELECT 
    e.emp_id    AS emp_id, 
    e.emp_name  AS emp_name, 
    e.hire_date AS emp_hire_date,
    m.emp_id    AS mgr_id, 
    m.emp_name  AS mgr_name, 
    m.hire_date AS mgr_hire_date
FROM employee e
JOIN employee m
    ON e.manager_id = m.emp_id
WHERE e.hire_date < m.hire_date;
```

---

### Q12. Delete Duplicate Records Keeping One
```sql
WITH duplicates AS (
    SELECT 
        emp_id, 
        emp_name, 
        ROW_NUMBER() OVER (PARTITION BY emp_id, emp_name ORDER BY emp_id) AS rn
    FROM employee
)
DELETE FROM employee
WHERE emp_id IN (
    SELECT emp_id 
    FROM duplicates 
    WHERE rn > 1
);
```

---

### Q13. Salary Higher Than All Employees in Another Department
```sql
SELECT emp, salary
FROM employee
WHERE department = 'A'
  AND salary > (
      SELECT MAX(salary)
      FROM employee
      WHERE department IN ('B', 'C')
  );
```

---

### Q14. Second Most Recent Order for Each Customer
```sql
WITH cust_cte AS (
    SELECT 
        customer_id, 
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM customer
)
SELECT customer_id
FROM cust_cte
WHERE rn = 2;
```

---

### Q15. Departments Where All Employees Earn Over 5000
```sql
SELECT department, COUNT(*) AS total_employees
FROM employee
GROUP BY department
HAVING COUNT(*) = COUNT(CASE WHEN salary > 5000 THEN 1 END);
```
