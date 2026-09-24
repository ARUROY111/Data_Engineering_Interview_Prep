# Complete Python Interview Questions
## 60+ Commonly Asked Problems with Inputs & Outputs

---

## SECTION 1: ARRAYS & LISTS (15 Questions)

### Q1. Reverse a List
**Difficulty:** Easy  
**Focus:** Loops, slicing, list operations

**INPUT:**  
```
Input = [1, 2, 3, 4]
```

**EXPECTED OUTPUT:**  
```
[4, 3, 2, 1]
```

**Explanation:**  
Reverse the order of elements. Use slicing `[::-1]`, `.reverse()`, or two-pointer approach.

**Solutions:**
```python
# Method 1: Slicing
return lst[::-1]

# Method 2: In-place
lst.reverse()
return lst

# Method 3: Two-pointer
left, right = 0, len(lst) - 1
while left < right:
    lst[left], lst[right] = lst[right], lst[left]
    left += 1
    right -= 1
return lst
```

---

### Q2. Find the nth Largest Element (without sort())
**Difficulty:** Medium  
**Focus:** Loops, conditional logic, heap operations

**INPUT:**  
```
Input = [10, 20, 40, 30, 40], n = 3
```

**EXPECTED OUTPUT:**  
```
20
```

**Explanation:**  
Find the nth largest unique element without using sort(). Use heap for O(n log k) complexity.

**Key Points:**
- Remove duplicates first: `unique_arr = list(set(arr))`
- Use heap: `heapq.nsmallest(n, unique_arr)[-1]`
- Or maintain min-heap of size n: O(n log k)

**Solutions:**
```python
# Method 1: Heap (BEST - O(n log k))
import heapq
def find_nth_largest(arr, n):
    unique = list(set(arr))
    if n > len(unique):
        return None
    heap = unique[:n]
    heapq.heapify(heap)
    for num in unique[n:]:
        if num > heap[0]:
            heapq.heapreplace(heap, num)
    return heap[0]

# Method 2: Simple
return sorted(set(arr), reverse=True)[n-1]

# Method 3: QuickSelect (average O(n))
# Complex but optimal for random data
```

---

### Q3. Remove Duplicates While Maintaining Order
**Difficulty:** Easy  
**Focus:** Sets, lists, iteration, order preservation

**INPUT:**  
```
Input = [1, 2, 3, 2, 1]
```

**EXPECTED OUTPUT:**  
```
[1, 2, 3]
```

**Explanation:**  
Remove duplicate elements while keeping original order of first occurrences.

**Solutions:**
```python
# Method 1: Set tracking (BEST)
seen = set()
result = []
for item in lst:
    if item not in seen:
        seen.add(item)
        result.append(item)
return result

# Method 2: Dict (Python 3.7+)
return list(dict.fromkeys(lst))

# Method 3: List comprehension
return [x for i, x in enumerate(lst) if x not in lst[:i]]  # O(n²)
```

---

### Q4. Count Frequency of Elements in a List
**Difficulty:** Easy  
**Focus:** Dictionary, loops, collections.Counter

**INPUT:**  
```
Input = ['a', 'b', 'a']
```

**EXPECTED OUTPUT:**  
```
{'a': 2, 'b': 1}
```

**Explanation:**  
Count occurrences of each element. Return as dictionary.

**Solutions:**
```python
# Method 1: Counter (BEST)
from collections import Counter
return dict(Counter(lst))

# Method 2: Dictionary
freq = {}
for item in lst:
    freq[item] = freq.get(item, 0) + 1
return freq

# Method 3: defaultdict
from collections import defaultdict
freq = defaultdict(int)
for item in lst:
    freq[item] += 1
return dict(freq)
```

---

### Q5. Merge Two Dictionaries
**Difficulty:** Easy  
**Focus:** Dictionary merging, unpacking

**INPUT:**  
```
d1 = {'a': 1}
d2 = {'b': 2}
```

**EXPECTED OUTPUT:**  
```
{'a': 1, 'b': 2}
```

**Explanation:**  
Combine two dictionaries. If duplicate keys, second dictionary wins.

**Solutions:**
```python
# Method 1: Unpacking (Python 3.5+, BEST)
return {**d1, **d2}

# Method 2: Pipe operator (Python 3.9+)
return d1 | d2

# Method 3: update()
d1_copy = d1.copy()
d1_copy.update(d2)
return d1_copy

# Method 4: Constructor
return dict(list(d1.items()) + list(d2.items()))
```

---

### Q6. Find Common Elements Between Two Lists
**Difficulty:** Easy  
**Focus:** Sets, list comprehension, set intersection

**INPUT:**  
```
l1 = [1, 2, 3]
l2 = [2, 3, 4]
```

**EXPECTED OUTPUT:**  
```
[2, 3]
```

**Explanation:**  
Find elements present in both lists.

**Solutions:**
```python
# Method 1: Set intersection (BEST)
return list(set(l1) & set(l2))

# Method 2: List comprehension
return [x for x in l1 if x in l2]

# Method 3: Set operations
return list(set(l1).intersection(set(l2)))

# Method 4: Filter + set lookup
s2 = set(l2)
return [x for x in l1 if x in s2]
```

---

### Q13. Find Duplicates in a List
**Difficulty:** Easy  
**Focus:** Dictionary counting, set operations

**INPUT:**  
```
Input = [1, 2, 2, 3, 3, 3]
```

**EXPECTED OUTPUT:**  
```
[2, 3]
```

**Explanation:**  
Find all elements that appear more than once.

**Solutions:**
```python
# Method 1: Set tracking (BEST)
seen = set()
duplicates = set()
for num in lst:
    if num in seen:
        duplicates.add(num)
    else:
        seen.add(num)
return list(duplicates)

# Method 2: Counter
from collections import Counter
freq = Counter(lst)
return [num for num, count in freq.items() if count > 1]

# Method 3: Dictionary
freq = {}
for num in lst:
    freq[num] = freq.get(num, 0) + 1
return [num for num, count in freq.items() if count > 1]
```

---

### Q17. Move All Zeroes to the End
**Difficulty:** Medium  
**Focus:** Two-pointer, list partitioning

**INPUT:**  
```
Input = [0, 1, 0, 3, 12]
```

**EXPECTED OUTPUT:**  
```
[1, 3, 12, 0, 0]
```

**Explanation:**  
Move all zeros to the end while maintaining relative order of non-zero elements.

**Solutions:**
```python
# Method 1: Two-pointer (in-place, BEST)
pointer = 0
for i in range(len(lst)):
    if lst[i] != 0:
        lst[pointer], lst[i] = lst[i], lst[pointer]
        pointer += 1
return lst

# Method 2: Separate and concatenate
non_zeros = [x for x in lst if x != 0]
zeros = [0] * (len(lst) - len(non_zeros))
return non_zeros + zeros

# Method 3: Count zeros
num_zeros = lst.count(0)
non_zeros = [x for x in lst if x != 0]
return non_zeros + [0] * num_zeros
```

---

### Q18. Find All Pairs with Target Sum
**Difficulty:** Medium  
**Focus:** Hashing, two-pointer optimization

**INPUT:**  
```
Input = [1, 2, 3, 4]
target = 5
```

**EXPECTED OUTPUT:**  
```
[(1, 4), (2, 3)]
```

**Explanation:**  
Find all unique pairs that sum to target.

**Solutions:**
```python
# Method 1: Hash set (BEST - O(n))
seen = set()
pairs = set()
for num in lst:
    complement = target - num
    if complement in seen:
        pair = tuple(sorted([num, complement]))
        pairs.add(pair)
    seen.add(num)
return list(pairs)

# Method 2: Two-pointer (requires sorting)
lst = sorted(lst)
pairs = []
left, right = 0, len(lst) - 1
while left < right:
    total = lst[left] + lst[right]
    if total == target:
        pairs.append((lst[left], lst[right]))
        left += 1
        right -= 1
    elif total < target:
        left += 1
    else:
        right -= 1
return pairs

# Method 3: Nested loop (O(n²))
pairs = set()
for i in range(len(lst)):
    for j in range(i+1, len(lst)):
        if lst[i] + lst[j] == target:
            pairs.add(tuple(sorted([lst[i], lst[j]])))
return list(pairs)
```

---

### Q15. Rotate List k Positions Right
**Difficulty:** Medium  
**Focus:** Slicing, modulo, array rotation

**INPUT:**  
```
Input = [1, 2, 3, 4, 5]
k = 2
```

**EXPECTED OUTPUT:**  
```
[4, 5, 1, 2, 3]
```

**Explanation:**  
Rotate list to the right k times. Elements that fall off reappear on left.

**Solutions:**
```python
# Method 1: Slicing (BEST - Pythonic)
if not lst:
    return lst
k = k % len(lst)
return lst[-k:] + lst[:-k] if k else lst

# Method 2: In-place reversal
k = k % len(lst)
def reverse(start, end):
    while start < end:
        lst[start], lst[end] = lst[end], lst[start]
        start += 1
        end -= 1
reverse(0, len(lst) - 1)
reverse(0, k - 1)
reverse(k, len(lst) - 1)
return lst

# Method 3: Deque
from collections import deque
dq = deque(lst)
dq.rotate(k)
return list(dq)
```

---

### Q19. Find Longest Word in Sentence
**Difficulty:** Easy  
**Focus:** String split, max, iteration

**INPUT:**  
```
s = 'Python is awesome'
```

**EXPECTED OUTPUT:**  
```
'awesome'
```

**Explanation:**  
Find the word with maximum length.

**Solutions:**
```python
# Method 1: max with key (BEST)
return max(s.split(), key=len)

# Method 2: Loop tracking
words = s.split()
longest = ""
for word in words:
    if len(word) > len(longest):
        longest = word
return longest

# Method 3: Sorted
return sorted(s.split(), key=len)[-1]
```

---

### Q14. Sort List of Dictionaries by Key
**Difficulty:** Easy  
**Focus:** sorted() with key=lambda, dictionary sorting

**INPUT:**  
```
[{'name':'Alice','age':25}, {'name':'Bob','age':20}]
```

**EXPECTED OUTPUT:**  
```
[{'name':'Bob','age':20}, {'name':'Alice','age':25}]
```

**Explanation:**  
Sort list of dictionaries by specific key (age in ascending order).

**Solutions:**
```python
# Method 1: sorted with lambda (BEST)
return sorted(lst, key=lambda x: x['age'])

# Method 2: Reverse order
return sorted(lst, key=lambda x: x['age'], reverse=True)

# Method 3: Multiple keys
return sorted(lst, key=lambda x: (x['age'], x['name']))

# Method 4: Using operator
from operator import itemgetter
return sorted(lst, key=itemgetter('age'))
```

---

### Q16. Find Third Largest Number (without sorting)
**Difficulty:** Medium  
**Focus:** Maintaining top 3 values manually

**INPUT:**  
```
Input = [10, 20, 40, 30, 40]
```

**EXPECTED OUTPUT:**  
```
20
```

**Explanation:**  
Same as Q2 but asking for 3rd specifically. Use heap or tracking variables.

**Solutions:**
```python
# Method 1: Heap
import heapq
unique = list(set(arr))
if len(unique) < 3:
    return None
return heapq.nsmallest(3, unique)[-1]

# Method 2: Manual tracking
first = second = third = float('-inf')
for num in arr:
    if num > first:
        third = second
        second = first
        first = num
    elif num > second:
        third = second
        second = num
    elif num > third:
        third = num
return third if third != float('-inf') else None

# Method 3: Sort unique and get
return sorted(set(arr), reverse=True)[2] if len(set(arr)) >= 3 else None
```

---

### Q7. Sum Digits Inside Strings
**Difficulty:** Easy  
**Focus:** String iteration, isdigit()

**INPUT:**  
```
Input = ['a1b2', '34c', 'x9']
```

**EXPECTED OUTPUT:**  
```
19 (1+2+3+4+9)
```

**Explanation:**  
Extract all digits from strings and sum them.

**Solutions:**
```python
# Method 1: Nested loops with isdigit (BEST)
total = 0
for string in lst:
    for char in string:
        if char.isdigit():
            total += int(char)
return total

# Method 2: List comprehension
return sum(int(char) for string in lst for char in string if char.isdigit())

# Method 3: Regex
import re
total = 0
for string in lst:
    digits = re.findall(r'\d', string)
    total += sum(int(d) for d in digits)
return total
```

---

## SECTION 2: STRINGS (12 Questions)

### Q8. Reverse Each Word in Sentence
**Difficulty:** Easy  
**Focus:** String split, slicing

**INPUT:**  
```
Input = 'hello world'
```

**EXPECTED OUTPUT:**  
```
'olleh dlrow'
```

**Explanation:**  
Reverse each word individually, maintain word order.

**Solutions:**
```python
# Method 1: List comprehension (BEST)
return ' '.join(word[::-1] for word in s.split())

# Method 2: Map
return ' '.join(map(lambda word: word[::-1], s.split()))

# Method 3: Loop
words = s.split()
reversed_words = [word[::-1] for word in words]
return ' '.join(reversed_words)
```

---

### Q9. Check if String is Palindrome
**Difficulty:** Easy  
**Focus:** String slicing, comparison

**INPUT:**  
```
Input = 'level'
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if string reads same forwards and backwards.

**Solutions:**
```python
# Method 1: Slicing (BEST)
return s == s[::-1]

# Method 2: Two-pointer
left, right = 0, len(s) - 1
while left < right:
    if s[left] != s[right]:
        return False
    left += 1
    right -= 1
return True

# Method 3: Case-insensitive + ignore spaces
cleaned = ''.join(c.lower() for c in s if c.isalnum())
return cleaned == cleaned[::-1]
```

---

### Q10. Find First Non-Repeating Character
**Difficulty:** Medium  
**Focus:** Dictionary, frequency counting

**INPUT:**  
```
Input = 'aabbcdde'
```

**EXPECTED OUTPUT:**  
```
'c'
```

**Explanation:**  
Find first character that appears exactly once.

**Solutions:**
```python
# Method 1: Counter + iteration (BEST)
from collections import Counter
char_count = Counter(s)
for char in s:
    if char_count[char] == 1:
        return char
return None

# Method 2: Dictionary
char_count = {}
for char in s:
    char_count[char] = char_count.get(char, 0) + 1
for char in s:
    if char_count[char] == 1:
        return char
return None

# Method 3: Two-pass
freq = {}
for c in s:
    freq[c] = freq.get(c, 0) + 1
for c in s:
    if freq[c] == 1:
        return c
return None
```

---

### Q20. Reverse Only Alphanumeric Characters
**Difficulty:** Hard  
**Focus:** Two-pointer, isalnum()

**INPUT:**  
```
Input = 'a1b@c#1$2'
```

**EXPECTED OUTPUT:**  
```
'2$1#c@b1a'
```

**Explanation:**  
Reverse only alphanumeric chars, keep special chars in place.

**Solutions:**
```python
# Method 1: Two-pointer (BEST - O(n))
s_list = list(s)
left, right = 0, len(s) - 1
while left < right:
    while left < right and not s_list[left].isalnum():
        left += 1
    while left < right and not s_list[right].isalnum():
        right -= 1
    s_list[left], s_list[right] = s_list[right], s_list[left]
    left += 1
    right -= 1
return ''.join(s_list)

# Method 2: Extract, reverse, rebuild
alphanumeric = [c for c in s if c.isalnum()]
alphanumeric.reverse()
result = []
alnum_idx = 0
for char in s:
    if char.isalnum():
        result.append(alphanumeric[alnum_idx])
        alnum_idx += 1
    else:
        result.append(char)
return ''.join(result)
```

---

### Q21. Find Longest Common Prefix
**Difficulty:** Medium  
**Focus:** String comparison, startswith()

**INPUT:**  
```
strs = ['flower', 'flow', 'flight']
```

**EXPECTED OUTPUT:**  
```
'fl'
```

**Explanation:**  
Find longest string that is prefix of all strings.

**Solutions:**
```python
# Method 1: Vertical scanning (BEST)
if not strs:
    return ""
for i in range(len(strs[0])):
    char = strs[0][i]
    for j in range(1, len(strs)):
        if i >= len(strs[j]) or strs[j][i] != char:
            return strs[0][:i]
return strs[0]

# Method 2: zip
prefix = ""
for chars in zip(*strs):
    if len(set(chars)) == 1:
        prefix += chars[0]
    else:
        break
return prefix

# Method 3: Horizontal scanning
prefix = strs[0]
for s in strs[1:]:
    while not s.startswith(prefix):
        prefix = prefix[:-1]
        if not prefix:
            return ""
return prefix
```

---

### Q22. Sort String Without Using sort()
**Difficulty:** Medium  
**Focus:** Selection sort, bubble sort, insertion sort

**INPUT:**  
```
s = 'dcba'
```

**EXPECTED OUTPUT:**  
```
'abcd'
```

**Explanation:**  
Sort string alphabetically without built-in sort().

**Solutions:**
```python
# Method 1: Selection sort (BEST - O(n²))
s_list = list(s)
for i in range(len(s_list)):
    min_idx = i
    for j in range(i + 1, len(s_list)):
        if s_list[j] < s_list[min_idx]:
            min_idx = j
    s_list[i], s_list[min_idx] = s_list[min_idx], s_list[i]
return ''.join(s_list)

# Method 2: Insertion sort
s_list = list(s)
for i in range(1, len(s_list)):
    key = s_list[i]
    j = i - 1
    while j >= 0 and s_list[j] > key:
        s_list[j + 1] = s_list[j]
        j -= 1
    s_list[j + 1] = key
return ''.join(s_list)

# Method 3: Bubble sort
s_list = list(s)
n = len(s_list)
for i in range(n):
    for j in range(0, n - i - 1):
        if s_list[j] > s_list[j + 1]:
            s_list[j], s_list[j + 1] = s_list[j + 1], s_list[j]
return ''.join(s_list)
```

---

### Q23. Reverse String with Special Characters
**Difficulty:** Medium  
**Focus:** String manipulation

**INPUT:**  
```
s = "a-bC-dEf-ghIj"
```

**EXPECTED OUTPUT:**  
```
"j-Ih-gfE-dCba"
```

**Explanation:**  
Reverse only letters, keep special chars and case.

**Solutions:**
```python
# Method 1: Two-pointer with condition
s_list = list(s)
left, right = 0, len(s) - 1
while left < right:
    while left < right and not s_list[left].isalpha():
        left += 1
    while left < right and not s_list[right].isalpha():
        right -= 1
    s_list[left], s_list[right] = s_list[right], s_list[left]
    left += 1
    right -= 1
return ''.join(s_list)
```

---

### Q24. Valid Parentheses String
**Difficulty:** Easy  
**Focus:** Stack, dictionary

**INPUT:**  
```
s = "()[]{}"
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if parentheses are balanced and properly nested.

**Solutions:**
```python
# Method 1: Stack (BEST)
stack = []
pairs = {'(': ')', '[': ']', '{': '}'}
for char in s:
    if char in pairs:
        stack.append(char)
    else:
        if not stack or pairs[stack.pop()] != char:
            return False
return len(stack) == 0

# Method 2: Stack with counter
stack = []
for char in s:
    if char in '([{':
        stack.append(char)
    else:
        if not stack:
            return False
        if (char == ')' and stack[-1] != '(') or \
           (char == ']' and stack[-1] != '[') or \
           (char == '}' and stack[-1] != '{'):
            return False
        stack.pop()
return len(stack) == 0
```

---

### Q25. Longest Substring Without Repeating
**Difficulty:** Medium  
**Focus:** Sliding window, dictionary

**INPUT:**  
```
s = "abcabcbb"
```

**EXPECTED OUTPUT:**  
```
3 (substring "abc")
```

**Explanation:**  
Find length of longest substring without duplicate characters.

**Solutions:**
```python
# Method 1: Sliding window (BEST)
char_index = {}
max_length = 0
start = 0
for i, char in enumerate(s):
    if char in char_index and char_index[char] >= start:
        start = char_index[char] + 1
    char_index[char] = i
    max_length = max(max_length, i - start + 1)
return max_length

# Method 2: Two-pointer with set
left = 0
char_set = set()
max_length = 0
for right in range(len(s)):
    while s[right] in char_set:
        char_set.remove(s[left])
        left += 1
    char_set.add(s[right])
    max_length = max(max_length, right - left + 1)
return max_length
```

---

### Q26. String Compression
**Difficulty:** Medium  
**Focus:** String building, character counting

**INPUT:**  
```
s = "aabbcc"
```

**EXPECTED OUTPUT:**  
```
"a2b2c2"
```

**Explanation:**  
Compress string by counting consecutive characters.

**Solutions:**
```python
# Method 1: Iteration (BEST)
if not s:
    return ""
result = []
count = 1
for i in range(len(s)):
    if i + 1 < len(s) and s[i] == s[i + 1]:
        count += 1
    else:
        result.append(s[i] + str(count))
        count = 1
return ''.join(result)

# Method 2: itertools.groupby
from itertools import groupby
return ''.join(char + str(len(list(group))) 
               for char, group in groupby(s))
```

---

## SECTION 3: DICTIONARIES & SETS (8 Questions)

### Q27. Group Anagrams
**Difficulty:** Medium  
**Focus:** Dictionary, sorting, hashing

**INPUT:**  
```
strs = ["eat", "tea", "ate", "tan", "nat"]
```

**EXPECTED OUTPUT:**  
```
[["eat", "tea", "ate"], ["tan", "nat"]]
```

**Explanation:**  
Group words that are anagrams of each other.

**Solutions:**
```python
# Method 1: Sort each word as key (BEST)
from collections import defaultdict
anagrams = defaultdict(list)
for word in strs:
    sorted_word = ''.join(sorted(word))
    anagrams[sorted_word].append(word)
return list(anagrams.values())

# Method 2: Character count as key
from collections import Counter
anagrams = defaultdict(list)
for word in strs:
    key = tuple(sorted(Counter(word).items()))
    anagrams[key].append(word)
return list(anagrams.values())
```

---

### Q28. Two Sum (Return Indices)
**Difficulty:** Easy  
**Focus:** Hash map, target finding

**INPUT:**  
```
nums = [2, 7, 11, 15]
target = 9
```

**EXPECTED OUTPUT:**  
```
[0, 1]
```

**Explanation:**  
Find indices of two numbers that add to target.

**Solutions:**
```python
# Method 1: Hash map (BEST - O(n))
seen = {}
for i, num in enumerate(nums):
    complement = target - num
    if complement in seen:
        return [seen[complement], i]
    seen[num] = i
return []

# Method 2: Two-pointer (requires sorting)
nums_sorted = sorted(enumerate(nums), key=lambda x: x[1])
left, right = 0, len(nums_sorted) - 1
while left < right:
    total = nums_sorted[left][1] + nums_sorted[right][1]
    if total == target:
        return sorted([nums_sorted[left][0], nums_sorted[right][0]])
    elif total < target:
        left += 1
    else:
        right -= 1
return []
```

---

### Q29. Contains Duplicate
**Difficulty:** Easy  
**Focus:** Set operations, hash lookup

**INPUT:**  
```
nums = [1, 2, 3, 1]
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if array contains duplicates.

**Solutions:**
```python
# Method 1: Set (BEST)
return len(nums) != len(set(nums))

# Method 2: Hash map
seen = set()
for num in nums:
    if num in seen:
        return True
    seen.add(num)
return False

# Method 3: Sorting
return len(nums) != len(set(nums))
```

---

### Q30. Valid Anagram
**Difficulty:** Easy  
**Focus:** Sorting, character counting

**INPUT:**  
```
s = "anagram"
t = "nagaram"
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if two strings are anagrams.

**Solutions:**
```python
# Method 1: Sorting (BEST)
return sorted(s) == sorted(t)

# Method 2: Counter
from collections import Counter
return Counter(s) == Counter(t)

# Method 3: Character frequency
freq = {}
for char in s:
    freq[char] = freq.get(char, 0) + 1
for char in t:
    if char not in freq:
        return False
    freq[char] -= 1
    if freq[char] < 0:
        return False
return all(count == 0 for count in freq.values())
```

---

## SECTION 4: ADVANCED STRUCTURES (12 Questions)

### Q11. Flatten Nested List
**Difficulty:** Medium  
**Focus:** Recursion, iteration, type checking

**INPUT:**  
```
Input = [1, [2, 3], [4, [5]]]
```

**EXPECTED OUTPUT:**  
```
[1, 2, 3, 4, 5]
```

**Explanation:**  
Convert nested list to flat list.

**Solutions:**
```python
# Method 1: Recursion (BEST)
def flatten_recursive(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten_recursive(item))
        else:
            result.append(item)
    return result

# Method 2: Generator (Memory efficient)
def flatten_generator(lst):
    for item in lst:
        if isinstance(item, list):
            yield from flatten_generator(item)
        else:
            yield item
return list(flatten_generator(lst))

# Method 3: Iterative with stack
def flatten_iterative(lst):
    stack = list(reversed(lst))
    result = []
    while stack:
        item = stack.pop()
        if isinstance(item, list):
            stack.extend(reversed(item))
        else:
            result.append(item)
    return result
```

---

### Q12. Fibonacci Sequence
**Difficulty:** Easy  
**Focus:** Loops, recursion, memoization

**INPUT:**  
```
n = 5
```

**EXPECTED OUTPUT:**  
```
[0, 1, 1, 2, 3]
```

**Explanation:**  
Generate first n Fibonacci numbers.

**Solutions:**
```python
# Method 1: Iterative (BEST - O(n))
def fibonacci_iterative(n):
    if n <= 0:
        return []
    elif n == 1:
        return [0]
    result = [0, 1]
    for i in range(2, n):
        result.append(result[-1] + result[-2])
    return result

# Method 2: Memoized recursion (O(n))
def fibonacci_memo(n):
    memo = {}
    def fib(x):
        if x in memo:
            return memo[x]
        if x <= 1:
            return x
        memo[x] = fib(x-1) + fib(x-2)
        return memo[x]
    return [fib(i) for i in range(n)]

# Method 3: Generator
def fibonacci_generator(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b
return list(fibonacci_generator(n))
```

---

### Q31. Binary Search
**Difficulty:** Medium  
**Focus:** Sorted array, divide and conquer

**INPUT:**  
```
nums = [1, 3, 5, 7, 9, 11]
target = 7
```

**EXPECTED OUTPUT:**  
```
3
```

**Explanation:**  
Find index of target in sorted array. Return -1 if not found.

**Solutions:**
```python
# Method 1: Iterative (BEST)
def binary_search_iterative(nums, target):
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# Method 2: Recursive
def binary_search_recursive(nums, target, left=0, right=None):
    if right is None:
        right = len(nums) - 1
    if left > right:
        return -1
    mid = (left + right) // 2
    if nums[mid] == target:
        return mid
    elif nums[mid] < target:
        return binary_search_recursive(nums, target, mid + 1, right)
    else:
        return binary_search_recursive(nums, target, left, mid - 1)
```

---

### Q32. Merge Two Sorted Arrays
**Difficulty:** Easy  
**Focus:** Two-pointer, array merging

**INPUT:**  
```
nums1 = [1, 3, 5]
nums2 = [2, 4, 6]
```

**EXPECTED OUTPUT:**  
```
[1, 2, 3, 4, 5, 6]
```

**Explanation:**  
Merge two sorted arrays into one sorted array.

**Solutions:**
```python
# Method 1: Two-pointer (BEST)
def merge_arrays(nums1, nums2):
    result = []
    i = j = 0
    while i < len(nums1) and j < len(nums2):
        if nums1[i] <= nums2[j]:
            result.append(nums1[i])
            i += 1
        else:
            result.append(nums2[j])
            j += 1
    result.extend(nums1[i:])
    result.extend(nums2[j:])
    return result

# Method 2: Simple (less efficient)
return sorted(nums1 + nums2)
```

---

### Q33. Intersection of Two Arrays
**Difficulty:** Easy  
**Focus:** Sets, filtering

**INPUT:**  
```
nums1 = [1, 2, 2, 1]
nums2 = [2, 2]
```

**EXPECTED OUTPUT:**  
```
[2]
```

**Explanation:**  
Find common elements between two arrays (count duplicates).

**Solutions:**
```python
# Method 1: Counter (handles duplicates)
from collections import Counter
count1 = Counter(nums1)
result = []
for num in nums2:
    if count1[num] > 0:
        result.append(num)
        count1[num] -= 1
return result

# Method 2: Set intersection
return list(set(nums1) & set(nums2))

# Method 3: Two-pointer (requires sorted)
nums1.sort()
nums2.sort()
result = []
i = j = 0
while i < len(nums1) and j < len(nums2):
    if nums1[i] == nums2[j]:
        if not result or result[-1] != nums1[i]:
            result.append(nums1[i])
        i += 1
        j += 1
    elif nums1[i] < nums2[j]:
        i += 1
    else:
        j += 1
return result
```

---

### Q34. Majority Element
**Difficulty:** Medium  
**Focus:** Hash map, voting algorithm

**INPUT:**  
```
nums = [3, 2, 3]
```

**EXPECTED OUTPUT:**  
```
3
```

**Explanation:**  
Find element appearing more than n/2 times.

**Solutions:**
```python
# Method 1: Counter (BEST)
from collections import Counter
return Counter(nums).most_common(1)[0][0]

# Method 2: Hash map
freq = {}
for num in nums:
    freq[num] = freq.get(num, 0) + 1
return max(freq, key=freq.get)

# Method 3: Boyer-Moore Voting (O(1) space)
def majority_element(nums):
    candidate = None
    count = 0
    for num in nums:
        if count == 0:
            candidate = num
        count += (1 if num == candidate else -1)
    return candidate
```

---

### Q35. Missing Number
**Difficulty:** Easy  
**Focus:** Math, set operations

**INPUT:**  
```
nums = [3, 0, 1]
```

**EXPECTED OUTPUT:**  
```
2
```

**Explanation:**  
Find missing number in array of range [0, n].

**Solutions:**
```python
# Method 1: Sum math (BEST)
n = len(nums)
expected_sum = n * (n + 1) // 2
return expected_sum - sum(nums)

# Method 2: Set difference
return list(set(range(len(nums) + 1)) - set(nums))[0]

# Method 3: XOR trick
result = 0
for i, num in enumerate(nums):
    result ^= i ^ num
return result ^ len(nums)
```

---

### Q36. Maximum Subarray (Kadane's Algorithm)
**Difficulty:** Medium  
**Focus:** Dynamic programming, greedy

**INPUT:**  
```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
```

**EXPECTED OUTPUT:**  
```
6 (subarray [4, -1, 2, 1])
```

**Explanation:**  
Find maximum sum of contiguous subarray.

**Solutions:**
```python
# Method 1: Kadane's Algorithm (BEST)
max_sum = current_sum = nums[0]
for num in nums[1:]:
    current_sum = max(num, current_sum + num)
    max_sum = max(max_sum, current_sum)
return max_sum

# Method 2: DP
n = len(nums)
dp = [0] * n
dp[0] = nums[0]
max_sum = dp[0]
for i in range(1, n):
    dp[i] = max(nums[i], dp[i-1] + nums[i])
    max_sum = max(max_sum, dp[i])
return max_sum
```

---

### Q37. 3Sum
**Difficulty:** Medium  
**Focus:** Two-pointer, sorting, three loops

**INPUT:**  
```
nums = [-1, 0, 1, 2, -1, -4]
target = 0
```

**EXPECTED OUTPUT:**  
```
[[-1, -1, 2], [-1, 0, 1]]
```

**Explanation:**  
Find all unique triplets that sum to target.

**Solutions:**
```python
# Method 1: Two-pointer with sorting
nums.sort()
result = []
for i in range(len(nums) - 2):
    if i > 0 and nums[i] == nums[i-1]:
        continue
    left, right = i + 1, len(nums) - 1
    while left < right:
        total = nums[i] + nums[left] + nums[right]
        if total == target:
            result.append([nums[i], nums[left], nums[right]])
            while left < right and nums[left] == nums[left+1]:
                left += 1
            while left < right and nums[right] == nums[right-1]:
                right -= 1
            left += 1
            right -= 1
        elif total < target:
            left += 1
        else:
            right -= 1
return result
```

---

### Q38. Longest Increasing Subsequence
**Difficulty:** Medium  
**Focus:** Dynamic programming, binary search

**INPUT:**  
```
nums = [10, 9, 2, 5, 3, 7, 101, 18]
```

**EXPECTED OUTPUT:**  
```
4 (length of [2, 3, 7, 101])
```

**Explanation:**  
Find length of longest increasing subsequence.

**Solutions:**
```python
# Method 1: DP (O(n²))
n = len(nums)
dp = [1] * n
for i in range(1, n):
    for j in range(i):
        if nums[j] < nums[i]:
            dp[i] = max(dp[i], dp[j] + 1)
return max(dp)

# Method 2: Binary search (O(n log n))
import bisect
tails = []
for num in nums:
    pos = bisect.bisect_left(tails, num)
    if pos == len(tails):
        tails.append(num)
    else:
        tails[pos] = num
return len(tails)
```

---

## SECTION 5: SPECIALIZED PROBLEMS (8 Questions)

### Q39. Single Number (Appears Once, Others Twice)
**Difficulty:** Easy  
**Focus:** XOR, bit operations

**INPUT:**  
```
nums = [2, 2, 1]
```

**EXPECTED OUTPUT:**  
```
1
```

**Explanation:**  
Find number appearing once when others appear twice.

**Solutions:**
```python
# Method 1: XOR (BEST - O(1) space)
result = 0
for num in nums:
    result ^= num
return result

# Method 2: Set
return 2 * sum(set(nums)) - sum(nums)

# Method 3: Counter
from collections import Counter
return [k for k, v in Counter(nums).items() if v == 1][0]
```

---

### Q40. Happy Number
**Difficulty:** Easy  
**Focus:** Cycle detection, hash set

**INPUT:**  
```
n = 19
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if number reaches 1 by repeatedly summing squares of digits.

**Solutions:**
```python
# Method 1: Set tracking cycles
def is_happy(n):
    def get_next(number):
        return sum(int(digit) ** 2 for digit in str(number))
    
    seen = set()
    while n != 1 and n not in seen:
        seen.add(n)
        n = get_next(n)
    return n == 1
```

---

### Q41. Is Power of Two
**Difficulty:** Easy  
**Focus:** Bit operations

**INPUT:**  
```
n = 16
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if number is power of 2.

**Solutions:**
```python
# Method 1: Bit manipulation (BEST)
return n > 0 and (n & (n - 1)) == 0

# Method 2: Logarithm
import math
return n > 0 and math.log2(n) == int(math.log2(n))

# Method 3: Loop
count = 0
while n > 0:
    count += n & 1
    n >>= 1
return count == 1
```

---

### Q42. Excel Column Number
**Difficulty:** Easy  
**Focus:** Base conversion, string manipulation

**INPUT:**  
```
columnTitle = "AB"
```

**EXPECTED OUTPUT:**  
```
28
```

**Explanation:**  
Convert Excel column letter (A, B, ... Z, AA, AB, ...) to number.

**Solutions:**
```python
# Method 1: Character to number conversion (BEST)
result = 0
for char in columnTitle:
    result = result * 26 + (ord(char) - ord('A') + 1)
return result

# Example walkthrough for "AB":
# A: result = 0 * 26 + (65 - 65 + 1) = 1
# B: result = 1 * 26 + (66 - 65 + 1) = 28
```

---

### Q43. Trapping Rain Water
**Difficulty:** Hard  
**Focus:** Dynamic programming, two-pointer

**INPUT:**  
```
height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
```

**EXPECTED OUTPUT:**  
```
6
```

**Explanation:**  
Calculate how much rain water can be trapped after raining.

**Solutions:**
```python
# Method 1: Two-pointer (BEST - O(n) space)
def trap(height):
    if not height:
        return 0
    left, right = 0, len(height) - 1
    left_max, right_max = 0, 0
    water = 0
    while left < right:
        if height[left] < height[right]:
            if height[left] >= left_max:
                left_max = height[left]
            else:
                water += left_max - height[left]
            left += 1
        else:
            if height[right] >= right_max:
                right_max = height[right]
            else:
                water += right_max - height[right]
            right -= 1
    return water
```

---

### Q44. Word Ladder
**Difficulty:** Hard  
**Focus:** BFS, graph traversal

**INPUT:**  
```
beginWord = "hit"
endWord = "cog"
wordList = ["hot", "dot", "dog", "lot", "log", "cog"]
```

**EXPECTED OUTPUT:**  
```
5
```

**Explanation:**  
Shortest path transforming one word to another (change one letter at a time).

**Solutions:**
```python
# Method 1: BFS
from collections import deque, defaultdict

def ladderLength(beginWord, endWord, wordList):
    word_set = set(wordList)
    if endWord not in word_set:
        return 0
    
    queue = deque([(beginWord, 1)])
    while queue:
        word, level = queue.popleft()
        if word == endWord:
            return level
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                new_word = word[:i] + c + word[i+1:]
                if new_word in word_set:
                    queue.append((new_word, level + 1))
                    word_set.remove(new_word)
    return 0
```

---

### Q45. LRU Cache
**Difficulty:** Hard  
**Focus:** Design, OrderedDict, doubly-linked list

**INPUT:**  
```
LRUCache(2)
put(1, 1)
put(2, 2)
get(1) → 1
put(3, 3)
get(2) → -1
```

**EXPECTED OUTPUT:**  
```
Correct cache eviction
```

**Explanation:**  
Design LRU (Least Recently Used) cache with O(1) operations.

**Solutions:**
```python
# Method 1: OrderedDict (BEST)
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key):
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)
```

---

## SECTION 6: COMPLEX ALGORITHMS (7 Questions)

### Q46. Longest Palindromic Substring
**Difficulty:** Medium  
**Focus:** Dynamic programming, expand around center

**INPUT:**  
```
s = "babad"
```

**EXPECTED OUTPUT:**  
```
"bab" or "aba"
```

**Explanation:**  
Find longest substring that is a palindrome.

**Solutions:**
```python
# Method 1: Expand around center (BEST - O(n²))
def longestPalindrome(s):
    if not s:
        return ""
    
    def expand_around_center(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return s[left + 1:right]
    
    longest = ""
    for i in range(len(s)):
        # Odd length (single character center)
        p1 = expand_around_center(i, i)
        # Even length (two character center)
        p2 = expand_around_center(i, i + 1)
        
        for p in [p1, p2]:
            if len(p) > len(longest):
                longest = p
    
    return longest

# Method 2: DP (O(n²) time, O(n²) space)
def longestPalindrome_dp(s):
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start = 0
    max_len = 0
    
    for i in range(n):
        dp[i][i] = True
    
    for i in range(n - 1):
        if s[i] == s[i + 1]:
            dp[i][i + 1] = True
            start = i
            max_len = 2
    
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i + 1][j - 1]:
                dp[i][j] = True
                start = i
                max_len = length
    
    return s[start:start + max_len]
```

---

### Q47. Word Break
**Difficulty:** Medium  
**Focus:** Dynamic programming, BFS

**INPUT:**  
```
s = "leetcode"
wordDict = ["leet", "code"]
```

**EXPECTED OUTPUT:**  
```
True
```

**Explanation:**  
Check if string can be segmented into dictionary words.

**Solutions:**
```python
# Method 1: DP (BEST - O(n²))
def wordBreak(s, wordDict):
    word_set = set(wordDict)
    dp = [False] * (len(s) + 1)
    dp[0] = True
    
    for i in range(1, len(s) + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[len(s)]

# Method 2: BFS
from collections import deque
def wordBreak_bfs(s, wordDict):
    word_set = set(wordDict)
    queue = deque([0])
    visited = set([0])
    
    while queue:
        start = queue.popleft()
        for end in range(start + 1, len(s) + 1):
            if end in visited:
                continue
            if s[start:end] in word_set:
                if end == len(s):
                    return True
                visited.add(end)
                queue.append(end)
    
    return False
```

---

### Q48. Regular Expression Matching
**Difficulty:** Hard  
**Focus:** Dynamic programming, pattern matching

**INPUT:**  
```
s = "aa"
p = "a"
```

**EXPECTED OUTPUT:**  
```
False
```

**Explanation:**  
Match string with pattern ('.' matches any char, '*' matches 0 or more).

**Solutions:**
```python
# Method 1: DP (O(m*n))
def isMatch(s, p):
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    
    for j in range(2, n + 1):
        if p[j - 1] == '*':
            dp[0][j] = dp[0][j - 2]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j - 1] == '*':
                dp[i][j] = dp[i][j - 2]
                if p[j - 2] == '.' or p[j - 2] == s[i - 1]:
                    dp[i][j] = dp[i][j] or dp[i - 1][j]
            else:
                if p[j - 1] == '.' or p[j - 1] == s[i - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
    
    return dp[m][n]
```

---

## QUICK REFERENCE SUMMARY

| Q # | Topic | Difficulty | Best Approach | Time | Space |
|-----|-------|-----------|---------------|------|-------|
| 1 | Reverse List | Easy | Slicing | O(n) | O(n) |
| 2 | nth Largest | Medium | Heap | O(n log k) | O(n) |
| 3 | Remove Duplicates | Easy | Set | O(n) | O(n) |
| 4 | Frequency | Easy | Counter | O(n) | O(n) |
| 5 | Merge Dict | Easy | Unpacking | O(n) | O(n) |
| 6 | Common Elements | Easy | Set intersection | O(n) | O(n) |
| 7 | Sum Digits | Easy | Iteration | O(n) | O(1) |
| 8 | Reverse Words | Easy | Join + split | O(n) | O(n) |
| 9 | Palindrome | Easy | Slicing | O(n) | O(1) |
| 10 | First Non-repeating | Medium | Counter | O(n) | O(n) |
| 11 | Flatten | Medium | Recursion | O(n) | O(h) |
| 12 | Fibonacci | Easy | Iterative | O(n) | O(n) |
| 13 | Find Duplicates | Easy | Set | O(n) | O(n) |
| 14 | Sort Dict List | Easy | sorted()+lambda | O(n log n) | O(n) |
| 15 | Rotate List | Medium | Slicing | O(n) | O(n) |
| 16 | 3rd Largest | Medium | Heap | O(n log k) | O(n) |
| 17 | Move Zeroes | Medium | Two-pointer | O(n) | O(1) |
| 18 | Pairs Sum | Medium | Hash set | O(n) | O(n) |
| 19 | Longest Word | Easy | max()+key | O(n) | O(1) |
| 20 | Reverse Alphanumeric | Hard | Two-pointer | O(n) | O(n) |
| 21 | Common Prefix | Medium | Vertical scan | O(n*m) | O(1) |
| 22 | Sort String | Medium | Selection sort | O(n²) | O(n) |
| 23 | Reverse w/ Special | Medium | Two-pointer | O(n) | O(n) |
| 24 | Valid Parentheses | Easy | Stack | O(n) | O(n) |
| 25 | Longest Substring | Medium | Sliding window | O(n) | O(n) |
| 26 | String Compression | Medium | Iteration | O(n) | O(n) |
| 27 | Group Anagrams | Medium | Hash map | O(n*k log k) | O(n) |
| 28 | Two Sum | Easy | Hash map | O(n) | O(n) |
| 29 | Contains Duplicate | Easy | Set | O(n) | O(n) |
| 30 | Valid Anagram | Easy | Sorting | O(n log n) | O(1) |
| 31 | Binary Search | Medium | Two-pointer | O(log n) | O(1) |
| 32 | Merge Sorted | Easy | Two-pointer | O(n) | O(n) |
| 33 | Intersection | Easy | Counter | O(n) | O(n) |
| 34 | Majority Element | Medium | Counter | O(n) | O(n) |
| 35 | Missing Number | Easy | Math | O(n) | O(1) |
| 36 | Max Subarray | Medium | Kadane | O(n) | O(1) |
| 37 | 3Sum | Medium | Two-pointer | O(n²) | O(1) |
| 38 | LIS | Medium | DP + Binary | O(n log n) | O(n) |
| 39 | Single Number | Easy | XOR | O(n) | O(1) |
| 40 | Happy Number | Easy | Cycle detect | O(1) | O(1) |
| 41 | Power of Two | Easy | Bit ops | O(1) | O(1) |
| 42 | Excel Column | Easy | Base conv | O(n) | O(1) |
| 43 | Trapping Rain | Hard | Two-pointer | O(n) | O(1) |
| 44 | Word Ladder | Hard | BFS | O(n*l²) | O(n) |
| 45 | LRU Cache | Hard | OrderedDict | O(1) | O(n) |
| 46 | Longest Palindrome | Medium | Expand center | O(n²) | O(1) |
| 47 | Word Break | Medium | DP | O(n²) | O(n) |
| 48+ | Regular Expression | Hard | DP | O(m*n) | O(m*n) |

---

## STUDY STRATEGY

**Week 1: Foundations (Q1-Q13)**
- Arrays and Lists basics
- String manipulation
- Dictionary operations
- Focus: Understand concepts, write clean code

**Week 2: Intermediate (Q14-Q25)**
- Sorting, searching
- Sliding window
- Two-pointer
- Focus: Optimize complexity

**Week 3: Advanced (Q26-Q38)**
- Bit operations
- Dynamic programming
- Graph algorithms
- Focus: Different approaches

**Week 4: Specialized (Q39-Q48)**
- Complex algorithms
- System design
- Pattern matching
- Focus: Interview readiness

---

**Master these 48 questions = 20 LPA interview guarantee! 🚀**
