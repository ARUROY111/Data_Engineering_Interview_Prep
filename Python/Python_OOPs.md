# 🐍 Python Object-Oriented Programming: The Master Reference Handbook
### *100 Practical Coding Problems, Solutions, Architectural Patterns & Concept Breakdowns*
> **Based on "Python Object-Oriented Programming" by @kartikmungole**  
> **Enhanced with comprehensive deep-dive explanations, architectural pointers, Python internals (CPython), and design pattern best practices.**

---

## 📑 Table of Contents

1. [Executive Overview: The OOP Paradigm in Python](#executive-overview-the-oop-paradigm-in-python)
2. [Chapter 01: Classes & Objects Basics (Problems 001–010)](#chapter-01-classes--objects-basics)
3. [Chapter 02: Encapsulation (Problems 011–020)](#chapter-02-encapsulation)
4. [Chapter 03: Inheritance (Problems 021–035)](#chapter-03-inheritance)
5. [Chapter 04: Polymorphism (Problems 036–047)](#chapter-04-polymorphism)
6. [Chapter 05: Abstraction (Problems 048–057)](#chapter-05-abstraction)
7. [Chapter 06: Magic Methods & Operator Overloading (Problems 058–069)](#chapter-06-magic-methods--operator-overloading)
8. [Chapter 07: Static & Class Members (Problems 070–077)](#chapter-07-static--class-members)
9. [Chapter 08: Design Patterns (Problems 078–087)](#chapter-08-design-patterns)
10. [Chapter 09: Composition & Mixins (Problems 088–095)](#chapter-09-composition--mixins)
11. [Chapter 10: Exception Handling in OOP (Problems 096–100)](#chapter-10-exception-handling-in-oop)
12. [Appendix: Dunder Methods & Pattern Cheat Sheet](#appendix-dunder-methods--pattern-cheat-sheet)

---

## Executive Overview: The OOP Paradigm in Python

In Python, **everything is an object**—including functions, modules, and integers. Python's object-oriented architecture is centered around the **Python Data Model** (often referred to as the *dunder* or double-underscore protocol). 

```
                               ┌───────────────────────────┐
                               │   Python Object System    │
                               └─────────────┬─────────────┘
                                             │
      ┌────────────────┬─────────────────────┼─────────────────────┬────────────────┐
      ▼                ▼                     ▼                     ▼                ▼
┌──────────────┐ ┌──────────────┐    ┌──────────────┐      ┌──────────────┐ ┌──────────────┐
│Encapsulation │ │ Inheritance  │    │ Polymorphism │      │ Abstraction  │ │ Composition  │
│• @property   │ │• Single/MRO  │    │• Duck Typing │      │• abc.ABC     │ │• Has-A       │
│• Name Mangle │ │• super()     │    │• Dunder Ops  │      │• Protocol    │ │• Mixins      │
│• __slots__   │ │• Cooperativ. │    │• Dispatch    │      │• Interfaces  │ │• Delegation  │
└──────────────┘ └──────────────┘    └──────────────┘      └──────────────┘ └──────────────┘
```

### Key Python Philosophy Pointers:
- **Explicit is better than implicit**: Methods always take `self` or `cls` as their first argument.
- **EAFP (Easier to Ask for Forgiveness than Permission)**: Python favours attempting operations and catching exceptions rather than excessive type checking.
- **Duck Typing**: *"If it walks like a duck and quacks like a duck, it's a duck."* Interface compatibility is evaluated by behavior (methods available) rather than explicit nominal inheritance.

---

# CHAPTER 01: Classes & Objects Basics

### 001. Create a basic class with attributes
**Problem**: Create a class `Book` with attributes `title`, `author`, and `price`. Create an instance and print its details.

```python
class Book:
    def __init__(self, title, author, price):
        self.title = title
        self.author = author
        self.price = price

    def display(self):
        print(f"{self.title} by {self.author} - ${self.price:.2f}")

b = Book("Clean Code", "Robert C. Martin", 34.99)
b.display()
```
**Output**:
```text
Clean Code by Robert C. Martin - $34.99
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `__init__` is an **initializer**, not a constructor. The actual object instantiation is performed earlier by `__new__`.
> - `self` represents the specific instance being created/modified. When calling `b.display()`, Python transforms it behind the scenes into `Book.display(b)`.

---

### 002. Instance counter
**Problem**: Create a class `Employee` that keeps track of how many `Employee` objects have been created using a class attribute.

```python
class Employee:
    count = 0

    def __init__(self, name):
        self.name = name
        Employee.count += 1

e1 = Employee("Alice")
e2 = Employee("Bob")
e3 = Employee("Carol")
print(f"Total employees: {Employee.count}")
```
**Output**:
```text
Total employees: 3
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - `count` is stored in the class namespace (`Employee.__dict__`), not in individual instance dictionaries.
> - Modifying via `Employee.count += 1` increments the class variable directly. If you mistakenly wrote `self.count += 1`, Python would shadow the class attribute by creating a new instance attribute on `self`.

---

### 003. Bank account class with deposit/withdraw
**Problem**: Create a `BankAccount` class with a balance, and methods `deposit()` and `withdraw()` that validate the amount and prevent overdrafts.

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount

acc = BankAccount("Priya", 100)
acc.deposit(50)
acc.withdraw(30)
print(acc.balance)
```
**Output**:
```text
120
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Encapsulates state mutations inside business logic methods.
> - Using explicit guard clauses with `raise ValueError` enforces invariant state constraints immediately rather than leaving the object in a corrupt state.

---

### 004. Rectangle class with area and perimeter
**Problem**: Create a `Rectangle` class with `width` and `height`, and methods to compute area and perimeter.

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

r = Rectangle(4, 5)
print(r.area())
print(r.perimeter())
```
**Output**:
```text
20
18
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Computes values dynamically on call rather than storing them as redundant attributes. This guarantees consistency even if `width` or `height` change later.

---

### 005. Compare two objects for equality by attribute
**Problem**: Create a `Point` class with `x` and `y` attributes, and implement equality comparison so two points with the same coordinates are considered equal.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(3, 4)
print(p1 == p2)
print(p1 == p3)
```
**Output**:
```text
True
False
```
> [!IMPORTANT]
> **💡 Info Pointer & Deep Dive:**
> - By default, Python tests equality with `is` (identity / memory address).
> - Returning `NotImplemented` instead of `False` allows Python to try the reverse comparison `other.__eq__(self)` before falling back to identity comparison.

---

### 006. Student grade average calculator
**Problem**: Create a `Student` class that stores a list of grades and has a method to compute the average grade.

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.grades = []

    def add_grade(self, grade):
        self.grades.append(grade)

    def average(self):
        if not self.grades:
            return 0
        return sum(self.grades) / len(self.grades)

s = Student("Kenji")
s.add_grade(85)
s.add_grade(92)
s.add_grade(78)
print(f"{s.average():.2f}")
```
**Output**:
```text
85.00
```
> [!WARNING]
> **💡 Info Pointer & Deep Dive:**
> - Initializing `self.grades = []` inside `__init__` gives each instance its own fresh list.
> - **Never** declare mutable default arguments in `def __init__(self, grades=[])` because that list is shared across every instance created without arguments!

---

### 007. Object with a dynamic `__dict__` attribute
**Problem**: Write a function that takes an object and prints all its instance attribute names and values using `__dict__`.

```python
class Car:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year

def print_attributes(obj):
    for key, value in obj.__dict__.items():
        print(f"{key}: {value}")

c = Car("Toyota", "Corolla", 2022)
print_attributes(c)
```
**Output**:
```text
make: Toyota
model: Corolla
year: 2022
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - In Python, user-defined instances store their writable attributes inside an internal hash map dictionary accessible via `__dict__`.
> - Attribute lookup `c.make` internally translates to `c.__dict__['make']` (after checking descriptors and class namespace).

---

### 008. Shallow copy vs deep copy demonstration
**Problem**: Demonstrate the difference between a shallow copy and a deep copy using a class that contains a mutable list attribute.

```python
import copy

class Team:
    def __init__(self, members):
        self.members = members

original = Team(["Alice", "Bob"])
shallow = copy.copy(original)
deep = copy.deepcopy(original)

shallow.members.append("Carol")
print(original.members)  # Shared list
print(deep.members)      # Independent list
```
**Output**:
```text
['Alice', 'Bob', 'Carol']
['Alice', 'Bob']
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `copy.copy()` duplicates the outer container/object, but leaves internal references pointing to the exact same nested objects in memory.
> - `copy.deepcopy()` recursively traverses and copies every nested child object to guarantee full isolation.

---

### 009. Using `__slots__` to restrict attributes
**Problem**: Create a `Coordinate` class that uses `__slots__` to only allow `x` and `y` attributes, and show that adding another attribute raises an error.

```python
class Coordinate:
    __slots__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

c = Coordinate(1, 2)
try:
    c.z = 3
except AttributeError as e:
    print(f"Error: {e}")
```
**Output**:
```text
Error: 'Coordinate' object has no attribute 'z'
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - `__slots__` prevents the creation of the default `__dict__` dictionary for each instance.
> - Instead, CPython allocates a fixed-size C array for attribute pointers. This dramatically reduces memory footprint (up to 40–60% RAM savings for millions of objects) and speeds up attribute access.

---

### 010. Class representing a 2D vector with basic arithmetic
**Problem**: Create a `Vector2D` class with `x` and `y`, and a method to compute its magnitude.

```python
import math

class Vector2D:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def magnitude(self):
        return math.sqrt(self.x ** 2 + self.y ** 2)

v = Vector2D(3, 4)
print(v.magnitude())
```
**Output**:
```text
5.0
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Uses Euclidean distance formula $\sqrt{x^2 + y^2}$.
> - `math.hypot(self.x, self.y)` can also be used as a high-precision alternative that avoids intermediate overflow.

---

# CHAPTER 02: Encapsulation

### 011. Private attribute with getter and setter
**Problem**: Create a `Person` class with a private `__age` attribute, exposing it through a getter and a setter that validates the age is non-negative.

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.__age = age

    def get_age(self):
        return self.__age

    def set_age(self, age):
        if age < 0:
            raise ValueError("Age cannot be negative")
        self.__age = age

p = Person("Nina", 30)
print(p.get_age())
p.set_age(31)
print(p.get_age())
```
**Output**:
```text
30
31
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Demonstrates traditional getter/setter style often seen in Java/C++. In Python, `@property` (Problem 012) is the idiomatic way to handle this without changing the public calling syntax.

---

### 012. Using `@property` for validated attribute access
**Problem**: Rewrite a `Temperature` class so that its `celsius` attribute can be read and written using dot notation, but raises an error for temperatures below absolute zero.

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius  # Invokes the setter

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < -273.15:
            raise ValueError("Temperature below absolute zero is not possible")
        self._celsius = value

t = Temperature(25)
print(t.celsius)
try:
    t.celsius = -300
except ValueError as e:
    print(f"Error: {e}")
```
**Output**:
```text
25
Error: Temperature below absolute zero is not possible
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - `@property` implements the **Python Descriptor Protocol** (`__get__`, `__set__`).
> - Calling `self.celsius = celsius` inside `__init__` routes directly through the setter, ensuring validation runs immediately at initialization time!

---

### 013. Read-only computed property
**Problem**: Create a `Circle` class with a radius, exposing a read-only `area` property computed from the radius.

```python
import math

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return math.pi * self.radius ** 2

c = Circle(3)
print(round(c.area, 2))
try:
    c.area = 100
except AttributeError as e:
    print(f"Error: {e}")
```
**Output**:
```text
28.27
Error: can't set attribute 'area'
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - When a property defines only a getter (`@property`) without a corresponding `@area.setter`, attempting to assign a value to `c.area` automatically raises an `AttributeError`.

---

### 014. Name-mangled private attribute demo
**Problem**: Demonstrate Python's name mangling by creating a class with a `__balance` attribute and accessing it from outside using the mangled name.

```python
class Wallet:
    def __init__(self, balance):
        self.__balance = balance

w = Wallet(100)
# print(w.__balance) # would raise AttributeError
print(w._Wallet__balance)
```
**Output**:
```text
100
```
> [!WARNING]
> **💡 Info Pointer & Deep Dive:**
> - Any identifier with at least two leading underscores and at most one trailing underscore (e.g. `__balance`) is textually replaced with `_ClassName__attribute` by the compiler.
> - This is designed to **prevent accidental name collisions in subclasses**, not to serve as absolute security. "We are all consenting adults here."

---

### 015. Encapsulated inventory item with quantity validation
**Problem**: Create an `InventoryItem` class where the quantity cannot be set directly to a negative number, using a property setter.

```python
class InventoryItem:
    def __init__(self, name, quantity):
        self.name = name
        self.quantity = quantity

    @property
    def quantity(self):
        return self._quantity

    @quantity.setter
    def quantity(self, value):
        if value < 0:
            raise ValueError("Quantity cannot be negative")
        self._quantity = value

item = InventoryItem("Widget", 10)
item.quantity -= 5
print(item.quantity)
try:
    item.quantity = -1
except ValueError as e:
    print(f"Error: {e}")
```
**Output**:
```text
5
Error: Quantity cannot be negative
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Notice that in-place subtraction `item.quantity -= 5` seamlessly invokes both the getter (to retrieve 10) and the setter (to store 5 with validation).

---

### 016. Encapsulated password field with masking
**Problem**: Create a `UserAccount` class that stores a password privately and exposes only a masked version through a property.

```python
class UserAccount:
    def __init__(self, username, password):
        self.username = username
        self.__password = password

    @property
    def masked_password(self):
        return "*" * len(self.__password)

    def check_password(self, attempt):
        return attempt == self.__password

u = UserAccount("dev_user", "s3cr3t!")
print(u.masked_password)
print(u.check_password("s3cr3t!"))
```
**Output**:
```text
*******
True
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - State hiding: The actual plaintext is protected from unintentional exposure in logging or string formatting, while verification is delegated to a domain method `check_password()`.

---

### 017. Immutable point class using properties
**Problem**: Create a `Point` class whose `x` and `y` coordinates cannot be changed after creation (read-only properties).

```python
class Point:
    def __init__(self, x, y):
        self._x = x
        self._y = y

    @property
    def x(self):
        return self._x

    @property
    def y(self):
        return self._y

    def __repr__(self):
        return f"Point({self._x}, {self._y})"

p = Point(2, 3)
print(p)
try:
    p.x = 10
except AttributeError as e:
    print(f"Error: {e}")
```
**Output**:
```text
Point(2, 3)
Error: property 'x' of 'Point' object has no setter
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - For modern Python, `@dataclass(frozen=True)` or `typing.NamedTuple` can also create immutable records. Manual property encapsulation is the foundation upon which those decorators work.

---

### 018. Encapsulated Celsius/Fahrenheit conversion
**Problem**: Create a `Weather` class storing temperature in Celsius internally, but exposing a `fahrenheit` property to get and set the temperature in Fahrenheit.

```python
class Weather:
    def __init__(self, celsius=0):
        self.celsius = celsius

    @property
    def fahrenheit(self):
        return self.celsius * 9 / 5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self.celsius = (value - 32) * 5 / 9

w = Weather(celsius=20)
print(w.fahrenheit)
w.fahrenheit = 98.6
print(round(w.celsius, 1))
```
**Output**:
```text
68.0
37.0
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Single Source of Truth: Data is stored in only one canonical unit (`self.celsius`), avoiding data desynchronization bugs while offering multiple user-facing representations.

---

### 019. Validated email field using a property
**Problem**: Create a `Contact` class whose `email` property validates that the value contains an `'@'` symbol before allowing assignment.

```python
class Contact:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    @property
    def email(self):
        return self._email

    @email.setter
    def email(self, value):
        if "@" not in value:
            raise ValueError("Invalid email address")
        self._email = value

c = Contact("Sam", "sam@example.com")
print(c.email)
try:
    c.email = "not-an-email"
except ValueError as e:
    print(f"Error: {e}")
```
**Output**:
```text
sam@example.com
Error: Invalid email address
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Encapsulation ensures business invariants are checked continuously over the lifetime of the object, preventing invalid email states at either instantiation or assignment.

---

### 020. Encapsulated stack with controlled access
**Problem**: Implement a `Stack` class that hides its underlying list and exposes only `push`, `pop`, `peek`, and `is_empty` methods.

```python
class Stack:
    def __init__(self):
        self.__items = []

    def push(self, item):
        self.__items.append(item)

    def pop(self):
        if self.is_empty():
            raise IndexError("pop from empty stack")
        return self.__items.pop()

    def peek(self):
        if self.is_empty():
            raise IndexError("peek from empty stack")
        return self.__items[-1]

    def is_empty(self):
        return len(self.__items) == 0

s = Stack()
s.push(1)
s.push(2)
print(s.peek())
print(s.pop())
print(s.is_empty())
```
**Output**:
```text
2
2
False
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Restricts arbitrary operations (like `insert(0, ...)` or sorting) that would violate the LIFO (Last-In First-Out) stack invariants.

---

# CHAPTER 03: Inheritance

### 021. Basic single inheritance
**Problem**: Create an `Animal` base class with a `speak()` method, and a `Dog` subclass that overrides `speak()`.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return f"{self.name} makes a sound."

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

a = Animal("Generic Animal")
d = Dog("Rex")
print(a.speak())
print(d.speak())
```
**Output**:
```text
Generic Animal makes a sound.
Rex says Woof!
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Method Overriding allows a child class to provide a specific implementation of a method that is already defined in its superclass.

---

### 022. Using `super()` to extend parent constructor
**Problem**: Create a `Vehicle` base class and a `Car` subclass that adds a `doors` attribute while still initializing the parent's attributes via `super()`.

```python
class Vehicle:
    def __init__(self, make, model):
        self.make = make
        self.model = model

class Car(Vehicle):
    def __init__(self, make, model, doors):
        super().__init__(make, model)
        self.doors = doors

    def __repr__(self):
        return f"{self.make} {self.model} ({self.doors} doors)"

c = Car("Honda", "Civic", 4)
print(c)
```
**Output**:
```text
Honda Civic (4 doors)
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - `super().__init__(make, model)` invokes the superclass constructor dynamically according to the MRO (Method Resolution Order), adhering to the DRY (Don't Repeat Yourself) principle.

---

### 023. Multilevel inheritance chain
**Problem**: Create a three-level hierarchy: `LivingBeing` -> `Animal` -> `Dog`, each adding new behavior.

```python
class LivingBeing:
    def breathe(self):
        return "Breathing..."

class Animal(LivingBeing):
    def move(self):
        return "Moving..."

class Dog(Animal):
    def bark(self):
        return "Barking!"

d = Dog()
print(d.breathe())
print(d.move())
print(d.bark())
```
**Output**:
```text
Breathing...
Moving...
Barking!
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Methods cascade down the inheritance tree. When calling `d.breathe()`, Python checks `Dog`, then `Animal`, then `LivingBeing`, and finally `object`.

---

### 024. Hierarchical inheritance with a shared parent
**Problem**: Create a `Shape` base class with an abstract-like `area()` method, and `Circle` and `Square` subclasses that each implement it differently.

```python
class Shape:
    def area(self):
        raise NotImplementedError("Subclasses must implement area()")

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

shapes = [Circle(3), Square(4)]
for s in shapes:
    print(round(s.area(), 2))
```
**Output**:
```text
28.27
16
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Raising `NotImplementedError` is the standard manual way in Python to define an interface contract before using formal `abc.ABC`.

---

### 025. Multiple inheritance and MRO inspection
**Problem**: Create classes `A`, `B`, and `C` where `C` inherits from both `A` and `B`, and print `C`'s Method Resolution Order.

```python
class A:
    def greet(self):
        return "Hello from A"

class B:
    def greet(self):
        return "Hello from B"

class C(A, B):
    pass

c = C()
print(c.greet())
print([cls.__name__ for cls in C.__mro__])
```
**Output**:
```text
Hello from A
['C', 'A', 'B', 'object']
```
> [!IMPORTANT]
> **💡 Info Pointer & Deep Dive:**
> - Python resolves multiple inheritance using the **C3 Linearization Algorithm** (accessible via `Class.__mro__` or `Class.mro()`).
> - Since `A` is listed before `B` in `class C(A, B)`, `A.greet` shadows `B.greet`.

---

### 026. Diamond inheritance resolved with `super()`
**Problem**: Demonstrate the diamond inheritance pattern where `D` inherits from `B` and `C`, which both inherit from `A`, and show that `A`'s method runs only once when using `super()` cooperatively.

```python
class A:
    def __init__(self):
        print("A init")
        self.value = 1

class B(A):
    def __init__(self):
        super().__init__()
        print("B init")

class C(A):
    def __init__(self):
        super().__init__()
        print("C init")

class D(B, C):
    def __init__(self):
        super().__init__()
        print("D init")

D()
```
**Output**:
```text
A init
C init
B init
D init
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Without `super()`, calling `A.__init__(self)` explicitly in both `B` and `C` would execute `A`'s initializer **twice**.
> - With cooperative `super()`, Python traverses `D -> B -> C -> A -> object` sequentially, executing `A.__init__` exactly **once**.

---

### 027. Overriding a method and still calling the parent version
**Problem**: Create a `Logger` base class with a `log()` method, and a `TimestampLogger` subclass that overrides `log()` to add a timestamp while still calling the original `log()` method.

```python
import datetime

class Logger:
    def log(self, message):
        print(f"LOG: {message}")

class TimestampLogger(Logger):
    def log(self, message):
        timestamp = datetime.datetime(2026, 1, 1, 12, 0, 0).isoformat()
        super().log(f"[{timestamp}] {message}")

logger = TimestampLogger()
logger.log("Application started")
```
**Output**:
```text
LOG: [2026-01-01T12:00:00] Application started
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Extension pattern: Modifies or wraps inputs/outputs before or after delegating the core workload to `super()`.

---

### 028. Employee/Manager hierarchy with salary calculation
**Problem**: Create an `Employee` class with a base salary calculation, and a `Manager` subclass that adds a bonus on top of the inherited calculation.

```python
class Employee:
    def __init__(self, name, base_salary):
        self.name = name
        self.base_salary = base_salary

    def calculate_salary(self):
        return self.base_salary

class Manager(Employee):
    def __init__(self, name, base_salary, bonus):
        super().__init__(name, base_salary)
        self.bonus = bonus

    def calculate_salary(self):
        return super().calculate_salary() + self.bonus

e = Employee("Tom", 50000)
m = Manager("Ana", 70000, 15000)
print(e.calculate_salary())
print(m.calculate_salary())
```
**Output**:
```text
50000
85000
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Polymorphic method extension: If the formula for `Employee.calculate_salary()` changes (e.g. taxes or deductions), `Manager` automatically absorbs that change without modifying its own code.

---

### 029. Abstract base with concrete shared behavior and required overrides
**Problem**: Create a `PaymentMethod` base class with a shared `validate()` method and an abstract-like `process_payment()` method that subclasses must implement.

```python
class PaymentMethod:
    def validate(self, amount):
        if amount <= 0:
            raise ValueError("Amount must be positive")
        return True

    def process_payment(self, amount):
        raise NotImplementedError("Subclasses must implement process_payment()")

class CreditCardPayment(PaymentMethod):
    def process_payment(self, amount):
        self.validate(amount)
        return f"Charged ${amount:.2f} to credit card"

class PayPalPayment(PaymentMethod):
    def process_payment(self, amount):
        self.validate(amount)
        return f"Charged ${amount:.2f} via PayPal"

for method in (CreditCardPayment(), PayPalPayment()):
    print(method.process_payment(49.99))
```
**Output**:
```text
Charged $49.99 to credit card
Charged $49.99 via PayPal
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Establishes the **Template Method Pattern**: common utility logic (`validate`) is inherited, while variant business operations (`process_payment`) are enforced upon subclasses.

---

### 030. Mixin combined with inheritance
**Problem**: Create a `JSONSerializableMixin` that adds a `to_json()` method, and combine it with a `Product` class using multiple inheritance.

```python
import json

class JSONSerializableMixin:
    def to_json(self):
        return json.dumps(self.__dict__)

class Product(JSONSerializableMixin):
    def __init__(self, name, price):
        self.name = name
        self.price = price

p = Product("Keyboard", 79.99)
print(p.to_json())
```
**Output**:
```text
{"name": "Keyboard", "price": 79.99}
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - **Mixins** are lightweight classes designed to provide stand-alone, orthogonal capabilities to other classes without participating in the primary domain hierarchy.
> - Mixins should never define their own `__init__` state.

---

### 031. Preventing further subclassing
**Problem**: Create a `FinalConfig` class that raises an error if anyone tries to subclass it, using `__init_subclass__`.

```python
class FinalConfig:
    def __init_subclass__(cls, **kwargs):
        raise TypeError(f"Cannot subclass FinalConfig (attempted by {cls.__name__})")

try:
    class ExtendedConfig(FinalConfig):
        pass
except TypeError as e:
    print(f"Error: {e}")
```
**Output**:
```text
Error: Cannot subclass FinalConfig (attempted by ExtendedConfig)
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Introduced in Python 3.6 (PEP 487), `__init_subclass__` hook runs automatically at class definition time whenever a class is derived, providing a clean alternative to full metaclasses.

---

### 032. Shape hierarchy with a common constructor pattern
**Problem**: Create an abstract-style `Shape` base storing a color attribute shared by all subclasses like `Triangle` and `Hexagon`.

```python
class Shape:
    def __init__(self, color):
        self.color = color

    def describe(self):
        return f"A {self.color} shape"

class Triangle(Shape):
    def __init__(self, color, base, height):
        super().__init__(color)
        self.base = base
        self.height = height

    def area(self):
        return 0.5 * self.base * self.height

t = Triangle("red", 6, 4)
print(t.describe())
print(t.area())
```
**Output**:
```text
A red shape
12.0
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - The parent encapsulates general identity attributes (`color`), while the child specializes domain geometry (`base`, `height`).

---

### 033. Vehicle rental hierarchy with polymorphic pricing
**Problem**: Create a `RentalVehicle` base class and `Car` and `Motorcycle` subclasses, each with its own daily rate calculation formula.

```python
class RentalVehicle:
    def __init__(self, daily_rate):
        self.daily_rate = daily_rate

    def total_cost(self, days):
        return self.daily_rate * days

class Car(RentalVehicle):
    def total_cost(self, days):
        base = super().total_cost(days)
        return base + 10 * days  # Insurance surcharge

class Motorcycle(RentalVehicle):
    def total_cost(self, days):
        return super().total_cost(days) * 0.9  # 10% discount

c = Car(50)
m = Motorcycle(30)
print(c.total_cost(3))
print(m.total_cost(3))
```
**Output**:
```text
180
81.0
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Open/Closed Principle (SOLID): You can introduce new vehicle types (e.g. `ElectricScooter`) without modifying existing rental billing workflows.

---

### 034. Extending a built-in type
**Problem**: Create a `UniqueList` class that inherits from Python's built-in `list` but overrides `append()` to avoid adding duplicate values.

```python
class UniqueList(list):
    def append(self, item):
        if item not in self:
            super().append(item)

ul = UniqueList()
ul.append(1)
ul.append(2)
ul.append(1)
print(ul)
```
**Output**:
```text
[1, 2]
```
> [!WARNING]
> **💡 Info Pointer & Deep Dive:**
> - When extending built-in C types like `list` or `dict`, C methods may bypass overridden Python methods (e.g., `extend()` will not call `append()`).
> - For production libraries, inherit from `collections.UserList` or `collections.abc.MutableSequence` instead of `list`.

---

### 035. Interface-like abstract base enforced across a hierarchy
**Problem**: Create a `Comparable` style base class that requires subclasses to implement `compare_to()`, and use it to sort custom objects.

```python
import functools

class Comparable:
    def compare_to(self, other):
        raise NotImplementedError

class Money(Comparable):
    def __init__(self, amount):
        self.amount = amount

    def compare_to(self, other):
        return (self.amount > other.amount) - (self.amount < other.amount)

    def __repr__(self):
        return f"${self.amount}"

amounts = [Money(50), Money(10), Money(30)]
amounts.sort(key=functools.cmp_to_key(lambda a, b: a.compare_to(b)))
print(amounts)
```
**Output**:
```text
[$10, $30, $50]
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `(self.amount > other.amount) - (self.amount < other.amount)` calculates the 3-way comparison spaceship operator `(-1, 0, 1)`.
> - `functools.cmp_to_key` converts traditional comparator functions into Python 3 key functions.

---

# CHAPTER 04: Polymorphism

### 036. Polymorphic `speak()` across multiple subclasses
**Problem**: Create `Animal` subclasses `Cat`, `Dog`, and `Bird` that each override `speak()`, then call `speak()` polymorphically in a loop.

```python
class Animal:
    def speak(self):
        raise NotImplementedError

class Cat(Animal):
    def speak(self):
        return "Meow"

class Dog(Animal):
    def speak(self):
        return "Woof"

class Bird(Animal):
    def speak(self):
        return "Tweet"

for animal in (Cat(), Dog(), Bird()):
    print(animal.speak())
```
**Output**:
```text
Meow
Woof
Tweet
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Dynamic Dispatch: Python resolves the appropriate method implementation at runtime based on the actual concrete type of the object instance.

---

### 037. Duck typing with unrelated classes
**Problem**: Create two unrelated classes, `Duck` and `Robot`, that both implement a `quack()` method, and call `quack()` on each without checking their type.

```python
class Duck:
    def quack(self):
        return "Quack!"

class Robot:
    def quack(self):
        return "BEEP - simulated quack"

def make_it_quack(entity):
    print(entity.quack())

make_it_quack(Duck())
make_it_quack(Robot())
```
**Output**:
```text
Quack!
BEEP - simulated quack
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Pure Duck Typing: Notice neither `Duck` nor `Robot` inherit from a common parent class. Python only cares that `entity` possesses an invokable `quack()` attribute.

---

### 038. Polymorphic `total_area` function over shapes
**Problem**: Write a function `total_area(shapes)` that sums the `area()` of any list of shape objects, regardless of their concrete class.

```python
class Circle:
    def __init__(self, r):
        self.r = r

    def area(self):
        return 3.14159 * self.r ** 2

class Rectangle:
    def __init__(self, w, h):
        self.w, self.h = w, h

    def area(self):
        return self.w * self.h

def total_area(shapes):
    return sum(shape.area() for shape in shapes)

shapes = [Circle(2), Rectangle(3, 4)]
print(round(total_area(shapes), 2))
```
**Output**:
```text
24.57
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Eliminates fragile `if isinstance(s, Circle): ... elif isinstance(s, Rectangle): ...` type inspection blocks.

---

### 039. Operator overloading enabling polymorphic `+` across custom types
**Problem**: Create a `Money` class that overloads `+` so two `Money` objects, or a `Money` and a plain number, can be added together.

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        if isinstance(other, Money):
            return Money(self.amount + other.amount)
        elif isinstance(other, (int, float)):
            return Money(self.amount + other)
        return NotImplemented

    def __repr__(self):
        return f"${self.amount:.2f}"

m1 = Money(10)
m2 = Money(5.5)
print(m1 + m2)
print(m1 + 4)
```
**Output**:
```text
$15.50
$14.00
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Polymorphic operator overloading allows your custom domain classes to interact seamlessly with Python's arithmetic operators.

---

### 040. Using `functools.singledispatch` for type-based dispatch
**Problem**: Write a generic `describe` function that behaves differently depending on the runtime type of its argument, using `functools.singledispatch`.

```python
from functools import singledispatch

@singledispatch
def describe(value):
    return f"A value: {value}"

@describe.register
def _(value: int):
    return f"An integer: {value}"

@describe.register
def _(value: str):
    return f"A string of length {len(value)}: '{value}'"

print(describe(42))
print(describe("hello"))
print(describe(3.14))
```
**Output**:
```text
An integer: 42
A string of length 5: 'hello'
A value: 3.14
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - In Python, you cannot overload functions with identical names in the same scope (the last `def` overwrites previous ones). `singledispatch` provides clean, functional function overloading based on the first argument's type.

---

### 041. Polymorphism with an abstract Notification base
**Problem**: Create a `Notification` base class and `EmailNotification` / `SMSNotification` subclasses, each implementing `send()` differently, then send all notifications from a single list.

```python
class Notification:
    def send(self, message):
        raise NotImplementedError

class EmailNotification(Notification):
    def send(self, message):
        return f"Emailing: {message}"

class SMSNotification(Notification):
    def send(self, message):
        return f"Texting: {message}"

notifications = [EmailNotification(), SMSNotification()]
for n in notifications:
    print(n.send("Your order has shipped"))
```
**Output**:
```text
Emailing: Your order has shipped
Texting: Your order has shipped
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Uniform Interface: Callers do not need to know the channel details (SMTP server, SMS gateway API); they only invoke `.send()`.

---

### 042. Overloading `__len__` for polymorphic `len()` support
**Problem**: Create a `Playlist` class that stores songs and supports Python's built-in `len()` function by implementing `__len__`.

```python
class Playlist:
    def __init__(self, songs=None):
        self.songs = songs or []

    def __len__(self):
        return len(self.songs)

p = Playlist(["Song A", "Song B", "Song C"])
print(len(p))
```
**Output**:
```text
3
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Python's built-in `len(p)` directly invokes `p.__len__()` in CPython (specifically accessing `tp_as_sequence->sq_length` for fast O(1) performance). Must return a non-negative integer.

---

### 043. Comparable objects sorted polymorphically
**Problem**: Create a `Product` class with `__lt__` implemented so a list of Products can be sorted directly using the built-in `sorted()` function.

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __lt__(self, other):
        return self.price < other.price

    def __repr__(self):
        return f"{self.name}(${self.price})"

products = [Product("Mug", 12), Product("Pen", 2), Product("Bag", 25)]
print(sorted(products))
```
**Output**:
```text
[Pen($2), Mug($12), Bag($25)]
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Python's **Timsort** algorithm only requires the less-than operator (`__lt__`) to perform full ascending sorts.

---

### 044. Polymorphic file-like objects sharing a write interface
**Problem**: Create two classes, `ConsoleWriter` and `ListWriter`, that both implement a `write(text)` method, and use them interchangeably in a shared logging function.

```python
class ConsoleWriter:
    def write(self, text):
        print(f"CONSOLE: {text}")

class ListWriter:
    def __init__(self):
        self.lines = []

    def write(self, text):
        self.lines.append(text)

def log_message(writer, message):
    writer.write(message)

cw = ConsoleWriter()
lw = ListWriter()
log_message(cw, "Startup complete")
log_message(lw, "Startup complete")
print(lw.lines)
```
**Output**:
```text
CONSOLE: Startup complete
['Startup complete']
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Mimics standard I/O protocols like `sys.stdout` and `io.StringIO`. Highly useful for mocking output during automated unit testing.

---

### 045. Overriding `__call__` for polymorphic callable objects
**Problem**: Create two classes, `Doubler` and `Squarer`, that are both callable (implement `__call__`), and apply them polymorphically inside a `map()`-like loop.

```python
class Doubler:
    def __call__(self, x):
        return x * 2

class Squarer:
    def __call__(self, x):
        return x ** 2

def apply_to_all(func, values):
    return [func(v) for v in values]

print(apply_to_all(Doubler(), [1, 2, 3]))
print(apply_to_all(Squarer(), [1, 2, 3]))
```
**Output**:
```text
[2, 4, 6]
[1, 4, 9]
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Implementing `__call__` turns instances into **Functors** (stateful callable objects). They can be passed anywhere a standard function callback is expected.

---

### 046. Polymorphic discount strategies applied to an order
**Problem**: Create a `DiscountStrategy` base class with a couple of subclasses implementing `apply(price)` differently, then apply whichever strategy is passed to an `Order`.

```python
class DiscountStrategy:
    def apply(self, price):
        raise NotImplementedError

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percent):
        self.percent = percent

    def apply(self, price):
        return price * (1 - self.percent / 100)

class FlatDiscount(DiscountStrategy):
    def __init__(self, amount):
        self.amount = amount

    def apply(self, price):
        return max(0, price - self.amount)

class Order:
    def __init__(self, price, discount: DiscountStrategy):
        self.price = price
        self.discount = discount

    def final_price(self):
        return self.discount.apply(self.price)

o1 = Order(100, PercentageDiscount(20))
o2 = Order(100, FlatDiscount(15))
print(o1.final_price())
print(o2.final_price())
```
**Output**:
```text
80.0
85
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Strategy Pattern: Swaps algorithms at runtime via dependency injection.

---

### 047. Polymorphic iterator objects
**Problem**: Create two custom iterable classes, `EvenNumbers` and `OddNumbers`, that both implement `__iter__` and `__next__`, and consume them interchangeably in a `for` loop.

```python
class EvenNumbers:
    def __init__(self, limit):
        self.limit = limit
        self.n = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.n >= self.limit:
            raise StopIteration
        value = self.n
        self.n += 2
        return value

class OddNumbers:
    def __init__(self, limit):
        self.limit = limit
        self.n = 1

    def __iter__(self):
        return self

    def __next__(self):
        if self.n >= self.limit:
            raise StopIteration
        value = self.n
        self.n += 2
        return value

for seq in (EvenNumbers(6), OddNumbers(6)):
    print(list(seq))
```
**Output**:
```text
[0, 2, 4]
[1, 3, 5]
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - The **Iterator Protocol**: `__iter__` returns the iterator object itself, and `__next__` returns the next element or raises `StopIteration` when exhausted.

---

# CHAPTER 05: Abstraction

### 048. Abstract base class with `abstractmethod`
**Problem**: Create an abstract `Shape` class using `ABC` and `abstractmethod` that forces subclasses to implement `area()`, and show that instantiating `Shape` directly fails.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

try:
    s = Shape()
except TypeError as e:
    print(f"Error: {e}")

sq = Square(4)
print(sq.area())
```
**Output**:
```text
Error: Can't instantiate abstract class Shape with abstract method area
16
```
> [!IMPORTANT]
> **💡 Info Pointer & Deep Dive:**
> - `abc.ABC` uses the metaclass `ABCMeta`. Python intercepts instantiation at `__call__` and raises `TypeError` if any `@abstractmethod` remains un-overridden.

---

### 049. Abstract class with both abstract and concrete methods
**Problem**: Create an abstract `Employee` class with an abstract `calculate_pay()` method and a concrete `display_info()` method shared by all subclasses.

```python
from abc import ABC, abstractmethod

class Employee(ABC):
    def __init__(self, name):
        self.name = name

    @abstractmethod
    def calculate_pay(self):
        pass

    def display_info(self):
        print(f"{self.name}: ${self.calculate_pay():.2f}")

class HourlyEmployee(Employee):
    def __init__(self, name, hours, rate):
        super().__init__(name)
        self.hours = hours
        self.rate = rate

    def calculate_pay(self):
        return self.hours * self.rate

he = HourlyEmployee("Lena", 40, 25)
he.display_info()
```
**Output**:
```text
Lena: $1000.00
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Template Method: The concrete `display_info()` calls the abstract `calculate_pay()`, providing shared reporting formatting while delegating calculations.

---

### 050. Abstract database connector interface
**Problem**: Create an abstract `DatabaseConnector` with abstract `connect()` and `disconnect()` methods, implemented differently by `MySQLConnector` and `PostgresConnector`.

```python
from abc import ABC, abstractmethod

class DatabaseConnector(ABC):
    @abstractmethod
    def connect(self):
        pass

    @abstractmethod
    def disconnect(self):
        pass

class MySQLConnector(DatabaseConnector):
    def connect(self):
        return "Connected to MySQL"

    def disconnect(self):
        return "Disconnected from MySQL"

class PostgresConnector(DatabaseConnector):
    def connect(self):
        return "Connected to PostgreSQL"

    def disconnect(self):
        return "Disconnected from PostgreSQL"

for connector in (MySQLConnector(), PostgresConnector()):
    print(connector.connect())
```
**Output**:
```text
Connected to MySQL
Connected to PostgreSQL
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Enforces API uniformity across multiple database drivers. Application services only depend on the `DatabaseConnector` abstraction.

---

### 051. Enforcing an abstract property
**Problem**: Create an abstract `Account` class that requires subclasses to implement an `interest_rate` property.

```python
from abc import ABC, abstractmethod

class Account(ABC):
    @property
    @abstractmethod
    def interest_rate(self):
        pass

class SavingsAccount(Account):
    @property
    def interest_rate(self):
        return 0.03

class CheckingAccount(Account):
    @property
    def interest_rate(self):
        return 0.0

print(SavingsAccount().interest_rate)
print(CheckingAccount().interest_rate)
```
**Output**:
```text
0.03
0.0
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Note decorator order: `@property` must wrap `@abstractmethod` (outermost to innermost).

---

### 052. Incomplete subclass raises `TypeError`
**Problem**: Show what happens when a subclass of an abstract class fails to implement all required abstract methods.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

class Incomplete(Shape):
    def area(self):
        return 0
    # perimeter() is missing

try:
    Incomplete()
except TypeError as e:
    print(f"Error: {e}")
```
**Output**:
```text
Error: Can't instantiate abstract class Incomplete with abstract method perimeter
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Python prevents partial implementations from ever being instantiated at runtime, guarding against runtime `AttributeError`s downstream.

---

### 053. Abstract report generator template
**Problem**: Create an abstract `ReportGenerator` whose `generate()` method defines the overall steps, calling abstract `fetch_data()` and `format_data()` methods that subclasses must implement (Template Method pattern).

```python
from abc import ABC, abstractmethod

class ReportGenerator(ABC):
    def generate(self):
        data = self.fetch_data()
        formatted = self.format_data(data)
        return f"REPORT:\n{formatted}"

    @abstractmethod
    def fetch_data(self):
        pass

    @abstractmethod
    def format_data(self, data):
        pass

class SalesReport(ReportGenerator):
    def fetch_data(self):
        return [100, 200, 300]

    def format_data(self, data):
        return f"Total sales: {sum(data)}"

print(SalesReport().generate())
```
**Output**:
```text
REPORT:
Total sales: 600
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Skeleton algorithm: High-level workflow orchestration (`generate`) is locked in the base class, while individual step implementations are customized.

---

### 054. Using `typing.Protocol` for structural abstraction
**Problem**: Define a `Flyable` Protocol requiring a `fly()` method, and write a function that accepts any object matching that structure without formal inheritance.

```python
from typing import Protocol

class Flyable(Protocol):
    def fly(self) -> str:
        ...

class Airplane:
    def fly(self):
        return "Airplane is flying"

class Bird:
    def fly(self):
        return "Bird is flying"

def make_it_fly(entity: Flyable):
    print(entity.fly())

make_it_fly(Airplane())
make_it_fly(Bird())
```
**Output**:
```text
Airplane is flying
Bird is flying
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Introduced in Python 3.8 (PEP 544), `Protocol` enables **Structural Subtyping** (static duck typing). Static type checkers like `mypy` verify compatibility without requiring explicit class inheritance!

---

### 055. Abstract shape hierarchy with a shared `scale()` method
**Problem**: Create an abstract `Shape2D` class with an abstract `area()` method and a concrete `scale()` method that multiplies a stored scale factor and reports it.

```python
from abc import ABC, abstractmethod

class Shape2D(ABC):
    def __init__(self):
        self.scale_factor = 1.0

    @abstractmethod
    def area(self):
        pass

    def scale(self, factor):
        self.scale_factor *= factor
        return f"Scaled by {factor}, cumulative factor: {self.scale_factor}"

class Square(Shape2D):
    def __init__(self, side):
        super().__init__()
        self.side = side

    def area(self):
        return (self.side * self.scale_factor) ** 2

sq = Square(5)
print(sq.scale(2))
print(sq.area())
```
**Output**:
```text
Scaled by 2, cumulative factor: 2.0
100.0
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Demonstrates stateful base abstractions modifying common scaling behavior that immediately reflects across derived mathematical formulas.

---

### 056. Abstract state machine interface
**Problem**: Create an abstract `State` class with an abstract `handle()` method, used to implement a simple traffic-light state machine.

```python
from abc import ABC, abstractmethod

class State(ABC):
    @abstractmethod
    def handle(self):
        pass

class RedLight(State):
    def handle(self):
        return "Stop"

class GreenLight(State):
    def handle(self):
        return "Go"

class TrafficLight:
    def __init__(self, state: State):
        self.state = state

    def change_state(self, state: State):
        self.state = state

    def report(self):
        return self.state.handle()

light = TrafficLight(RedLight())
print(light.report())
light.change_state(GreenLight())
print(light.report())
```
**Output**:
```text
Stop
Go
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - State Pattern: Encapsulates state-specific behavior in discrete classes instead of massive conditional `switch/match/if-elif` statements.

---

### 057. Abstract Shape hierarchy with `__str__` shared implementation
**Problem**: Create an abstract `Shape` class that provides a shared `__str__` implementation using an abstract `name()` method and abstract `area()` method.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def name(self):
        pass

    @abstractmethod
    def area(self):
        pass

    def __str__(self):
        return f"{self.name()} with area {self.area():.2f}"

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def name(self):
        return "Circle"

    def area(self):
        return 3.14159 * self.radius ** 2

print(str(Circle(2)))
```
**Output**:
```text
Circle with area 12.57
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - High-level formatting contracts are maintained at the root, while subclasses only provide basic metadata primitives (`name`, `area`).

---

# CHAPTER 06: Magic Methods & Operator Overloading

### 058. Overload `+` and `-` for a Vector class
**Problem**: Create a `Vector` class supporting addition and subtraction using `__add__` and `__sub__`.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(2, 3)
v2 = Vector(1, 1)
print(v1 + v2)
print(v1 - v2)
```
**Output**:
```text
Vector(3, 4)
Vector(1, 2)
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Magic methods hook into CPython's arithmetic evaluation bytecode (`BINARY_OP` / `BINARY_ADD`).

---

### 059. Implement `__str__` and `__repr__` differently
**Problem**: Create a `Product` class where `__str__` gives a friendly description and `__repr__` gives a developer-oriented, evaluable representation.

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __str__(self):
        return f"{self.name} costs ${self.price:.2f}"

    def __repr__(self):
        return f"Product(name={self.name!r}, price={self.price!r})"

p = Product("Mouse", 19.99)
print(str(p))
print(repr(p))
```
**Output**:
```text
Mouse costs $19.99
Product(name='Mouse', price=19.99)
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Rule of Thumb: `__str__` is for end users (readable); `__repr__` is for developers and debuggers (unambiguous, and ideally valid Python code to recreate the object).
> - Using `!r` in f-strings wraps strings in quotes automatically.

---

### 060. Implement full ordering with `total_ordering`
**Problem**: Create a `Card` class ranked by value, implementing `__eq__` and `__lt__`, and use `functools.total_ordering` to get the rest of the comparisons for free.

```python
from functools import total_ordering

@total_ordering
class Card:
    def __init__(self, rank):
        self.rank = rank

    def __eq__(self, other):
        return self.rank == other.rank

    def __lt__(self, other):
        return self.rank < other.rank

    def __repr__(self):
        return f"Card({self.rank})"

cards = [Card(10), Card(2), Card(7)]
print(sorted(cards))
print(Card(5) >= Card(3))
```
**Output**:
```text
[Card(2), Card(7), Card(10)]
True
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Implementing `__eq__`, `__lt__`, `__le__`, `__gt__`, `__ge__`, `__ne__` manually is boilerplate-heavy and error-prone. `@total_ordering` auto-generates `<=`, `>`, `>=` from `__eq__` and `__lt__`.

---

### 061. Make an object indexable with `__getitem__` and `__setitem__`
**Problem**: Create a `Matrix` class that supports indexing like `matrix[i][j]` using `__getitem__` and `__setitem__` on rows.

```python
class Matrix:
    def __init__(self, rows):
        self.rows = rows

    def __getitem__(self, index):
        return self.rows[index]

    def __setitem__(self, index, value):
        self.rows[index] = value

    def __repr__(self):
        return "\n".join(str(row) for row in self.rows)

m = Matrix([[1, 2], [3, 4]])
print(m[0])
m[1] = [9, 9]
print(m)
```
**Output**:
```text
[1, 2]
[1, 2]
[9, 9]
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `__getitem__` is triggered whenever square bracket indexing `obj[key]` is used. It also supports slice objects (e.g. `obj[1:3]`).

---

### 062. Custom iterable class with `__iter__` and `__next__`
**Problem**: Create a `Fibonacci` class that generates Fibonacci numbers up to a limit using the iterator protocol.

```python
class Fibonacci:
    def __init__(self, limit):
        self.limit = limit

    def __iter__(self):
        self.a, self.b = 0, 1
        return self

    def __next__(self):
        if self.a > self.limit:
            raise StopIteration
        value = self.a
        self.a, self.b = self.b, self.a + self.b
        return value

for num in Fibonacci(20):
    print(num, end=" ")
print()
```
**Output**:
```text
0 1 1 2 3 5 8 13 
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Memory efficiency: Numbers are generated on-demand one by one in $O(1)$ memory without creating an entire pre-computed list.

---

### 063. Make an object callable with `__call__`
**Problem**: Create a `Multiplier` class whose instances behave like functions that multiply their input by a stored factor.

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, value):
        return value * self.factor

double = Multiplier(2)
triple = Multiplier(3)
print(double(5))
print(triple(5))
```
**Output**:
```text
10
15
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `callable(double)` returns `True`. Enables closure-like state retention within a class structure.

---

### 064. Context manager class using `__enter__` and `__exit__`
**Problem**: Create a `Timer` context manager class that records and prints elapsed time when a `with` block exits.

```python
import time

class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.4f} seconds")
        return False  # Don't suppress exceptions

with Timer():
    total = sum(range(1_000_000))
    print(total)
```
**Output**:
```text
499999500000
Elapsed: 0.0312 seconds
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - The context manager protocol guarantees `__exit__` execution even if unhandled exceptions occur within the block.
> - Returning `False` propagates exceptions upward; returning `True` suppresses them.

---

### 065. Support `'in'` operator with `__contains__`
**Problem**: Create a `Playlist` class that supports `'song in playlist'` membership checks using `__contains__`.

```python
class Playlist:
    def __init__(self, songs):
        self.songs = songs

    def __contains__(self, song):
        return song in self.songs

p = Playlist(["Song A", "Song B"])
print("Song A" in p)
print("Song Z" in p)
```
**Output**:
```text
True
False
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Implementing `__contains__` allows using the clean `in` keyword. If not implemented, Python falls back to iterating over `__iter__` or `__getitem__`.

---

### 066. Overload `__eq__` and `__hash__` for use in sets
**Problem**: Create a `Coordinate` class that can be safely stored in a set by implementing both `__eq__` and `__hash__` consistently.

```python
class Coordinate:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __eq__(self, other):
        return isinstance(other, Coordinate) and self.x == other.x and self.y == other.y

    def __hash__(self):
        return hash((self.x, self.y))

    def __repr__(self):
        return f"({self.x}, {self.y})"

points = {Coordinate(1, 2), Coordinate(1, 2), Coordinate(3, 4)}
print(points)
```
**Output**:
```text
{(1, 2), (3, 4)}
```
> [!IMPORTANT]
> **💡 Info Pointer & Deep Dive:**
> - **The Hashing Invariant**: If two objects compare equal (`a == b`), they **must** have identical hash values (`hash(a) == hash(b)`).
> - When you override `__eq__`, Python automatically sets `__hash__ = None` to prevent hashing mutable objects unless you explicitly define `__hash__`.

---

### 067. Overload `__bool__` for truthiness checks
**Problem**: Create a `ShoppingCart` class that is considered 'falsy' when empty, by implementing `__bool__`.

```python
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)

    def __bool__(self):
        return len(self.items) > 0

cart = ShoppingCart()
print("Cart has items" if cart else "Cart is empty")
cart.add("Book")
print("Cart has items" if cart else "Cart is empty")
```
**Output**:
```text
Cart is empty
Cart has items
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `if cart:` checks `cart.__bool__()`. If `__bool__` is not defined, Python falls back to `len(cart) != 0` via `__len__`.

---

### 068. Overload `__len__` and `__getitem__` together for a custom sequence
**Problem**: Create a `Queue` class that behaves like a read-only sequence by supporting `len()` and indexing.

```python
class Queue:
    def __init__(self, items):
        self._items = list(items)

    def __len__(self):
        return len(self._items)

    def __getitem__(self, index):
        return self._items[index]

q = Queue([10, 20, 30])
print(len(q))
print(q[1])
print(list(q))
```
**Output**:
```text
3
20
[10, 20, 30]
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Implementing both `__len__` and `__getitem__` automatically gives you free iteration and slicing support via Python's sequence protocol fallback.

---

### 069. Overload `__radd__` for reflected addition
**Problem**: Create a `Money` class that supports `sum([Money(5), Money(10)])` by handling the case where `int` (0) is added on the left via `__radd__`.

```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __add__(self, other):
        other_amount = other.amount if isinstance(other, Money) else other
        return Money(self.amount + other_amount)

    def __radd__(self, other):
        return self.__add__(other)

    def __repr__(self):
        return f"${self.amount}"

total = sum([Money(5), Money(10), Money(3)])
print(total)
```
**Output**:
```text
$18
```
> [!IMPORTANT]
> **💡 Info Pointer & Deep Dive:**
> - Python's built-in `sum()` starts with `0` as the accumulator.
> - When evaluating `0 + Money(5)`, Python first tries `int.__add__(0, Money(5))` which returns `NotImplemented`, then falls back to `Money(5).__radd__(0)`.

---

# CHAPTER 07: Static & Class Members

### 070. Static utility method
**Problem**: Create a `MathUtils` class with a static method `is_prime()` that does not depend on any instance or class state.

```python
class MathUtils:
    @staticmethod
    def is_prime(n):
        if n < 2:
            return False
        for i in range(2, int(n ** 0.5) + 1):
            if n % i == 0:
                return False
        return True

print(MathUtils.is_prime(17))
print(MathUtils.is_prime(18))
```
**Output**:
```text
True
False
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `@staticmethod` functions as a plain function placed inside the class namespace for logical grouping. It receives neither `self` nor `cls`.

---

### 071. Class method as an alternative constructor
**Problem**: Create a `Date` class with a `classmethod` `from_string()` that parses a `'YYYY-MM-DD'` string and returns a new `Date` instance.

```python
class Date:
    def __init__(self, year, month, day):
        self.year, self.month, self.day = year, month, day

    @classmethod
    def from_string(cls, date_str):
        year, month, day = map(int, date_str.split("-"))
        return cls(year, month, day)

    def __repr__(self):
        return f"{self.year:04d}-{self.month:02d}-{self.day:02d}"

d = Date.from_string("2026-08-04")
print(d)
```
**Output**:
```text
2026-08-04
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Using `cls(...)` inside the classmethod instead of `Date(...)` ensures that if `Date` is subclassed, the alternative constructor creates an instance of the subclass, not the parent!

---

### 072. Class method returning the correct subclass
**Problem**: Create a `Shape` base class with a `classmethod` `default()` that returns an instance of whatever subclass it's called on.

```python
class Shape:
    @classmethod
    def default(cls):
        return cls()

    def describe(self):
        return f"I am a {type(self).__name__}"

class Circle(Shape):
    pass

class Square(Shape):
    pass

print(Circle.default().describe())
print(Square.default().describe())
```
**Output**:
```text
I am a Circle
I am a Square
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Polymorphic Factory: Demonstrates how `cls` dynamically binds to the calling subclass (`Circle` or `Square`).

---

### 073. Class attribute shared across instances
**Problem**: Create a `Config` class with a shared class attribute `settings` dictionary, and show that modifying it through one reference affects all instances.

```python
class Config:
    settings = {"debug": False}

c1 = Config()
c2 = Config()
Config.settings["debug"] = True
print(c1.settings["debug"])
print(c2.settings["debug"])
```
**Output**:
```text
True
True
```
> [!WARNING]
> **💡 Info Pointer & Deep Dive:**
> - Mutable class attributes are shared globally across every instance of the class in the Python runtime.

---

### 074. Instance counter using a class method
**Problem**: Extend a `Widget` class to track total instances created, with a `classmethod` `get_count()` to retrieve the current count.

```python
class Widget:
    _count = 0

    def __init__(self):
        Widget._count += 1

    @classmethod
    def get_count(cls):
        return cls._count

Widget()
Widget()
Widget()
print(Widget.get_count())
```
**Output**:
```text
3
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Provides a clean, encapsulated read-only interface (`get_count`) for class-level metrics.

---

### 075. Static method used for input validation before construction
**Problem**: Create a `User` class with a static method `validate_username()` used inside `__init__` to check the username before assignment.

```python
class User:
    def __init__(self, username):
        if not User.validate_username(username):
            raise ValueError("Invalid username")
        self.username = username

    @staticmethod
    def validate_username(name):
        return name.isalnum() and len(name) >= 3

u = User("dev123")
print(u.username)
try:
    User("a!")
except ValueError as e:
    print(f"Error: {e}")
```
**Output**:
```text
dev123
Error: Invalid username
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - The validator can be tested independently without instantiating dummy `User` objects: `User.validate_username("test")`.

---

### 076. Class-level registry populated via classmethod
**Problem**: Create a `Plugin` base class with a class-level list that every registered plugin subclass gets added to automatically via a classmethod.

```python
class Plugin:
    registry = []

    @classmethod
    def register(cls, plugin_cls):
        cls.registry.append(plugin_cls)
        return plugin_cls

@Plugin.register
class LoggerPlugin:
    pass

@Plugin.register
class CachePlugin:
    pass

print([p.__name__ for p in Plugin.registry])
```
**Output**:
```text
['LoggerPlugin', 'CachePlugin']
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Decorator Registry Pattern: Widely used in frameworks like Flask/Django for registering routes, tasks, and extension plugins.

---

### 077. Singleton implemented with a class attribute
**Problem**: Implement a simple Singleton class that reuses the same instance on every construction attempt by overriding `__new__`.

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

a = Singleton()
b = Singleton()
print(a is b)
```
**Output**:
```text
True
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - `__new__` controls object creation and returns the instance reference. `a is b` confirms both variables reference the exact same memory address.

---

# CHAPTER 08: Design Patterns

### 078. Singleton pattern with thread safety
**Problem**: Implement a thread-safe Singleton using a lock to guard instance creation.

```python
import threading

class Singleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

a = Singleton()
b = Singleton()
print(a is b)
```
**Output**:
```text
True
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - **Double-Checked Locking**: The outer check prevents acquiring the lock once the instance already exists (preserving performance), while the inner check guarantees atomic creation in multithreaded environments.

---

### 079. Factory Method pattern
**Problem**: Implement a factory function that returns the correct `Shape` subclass instance based on a string type name.

```python
class Circle:
    def area(self):
        return "circle area"

class Square:
    def area(self):
        return "square area"

def shape_factory(shape_type):
    shapes = {"circle": Circle, "square": Square}
    if shape_type not in shapes:
        raise ValueError(f"Unknown shape type: {shape_type}")
    return shapes[shape_type]()

s = shape_factory("circle")
print(s.area())
```
**Output**:
```text
circle area
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Decouples object creation from object utilization. Client code only passes a string identifier and receives a standardized product.

---

### 080. Builder pattern for constructing a complex object
**Problem**: Implement a Builder pattern for constructing a `Pizza` object step by step with optional toppings.

```python
class Pizza:
    def __init__(self):
        self.toppings = []
        self.size = "medium"

    def __repr__(self):
        return f"{self.size} pizza with {', '.join(self.toppings) or 'no toppings'}"

class PizzaBuilder:
    def __init__(self):
        self.pizza = Pizza()

    def set_size(self, size):
        self.pizza.size = size
        return self

    def add_topping(self, topping):
        self.pizza.toppings.append(topping)
        return self

    def build(self):
        return self.pizza

pizza = (PizzaBuilder()
         .set_size("large")
         .add_topping("cheese")
         .add_topping("mushroom")
         .build())
print(pizza)
```
**Output**:
```text
large pizza with cheese, mushroom
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - **Method Chaining (Fluent Interface)**: Returning `self` from builder methods allows chaining configuration calls cleanly.

---

### 081. Observer pattern
**Problem**: Implement the Observer pattern where a `Publisher` notifies multiple `Subscriber` objects when its state changes.

```python
class Publisher:
    def __init__(self):
        self._subscribers = []

    def subscribe(self, subscriber):
        self._subscribers.append(subscriber)

    def notify(self, message):
        for sub in self._subscribers:
            sub.update(message)

class Subscriber:
    def __init__(self, name):
        self.name = name

    def update(self, message):
        print(f"{self.name} received: {message}")

pub = Publisher()
pub.subscribe(Subscriber("Alice"))
pub.subscribe(Subscriber("Bob"))
pub.notify("New article published!")
```
**Output**:
```text
Alice received: New article published!
Bob received: New article published!
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Publish-Subscribe model: The publisher maintains no tight coupling to specific subscriber implementations.

---

### 082. Strategy pattern for interchangeable algorithms
**Problem**: Implement the Strategy pattern to compute a route using either `FastestRoute` or `ShortestRoute` strategies, chosen at runtime.

```python
class RouteStrategy:
    def calculate(self, start, end):
        raise NotImplementedError

class FastestRoute(RouteStrategy):
    def calculate(self, start, end):
        return f"Fastest route from {start} to {end} via highway"

class ShortestRoute(RouteStrategy):
    def calculate(self, start, end):
        return f"Shortest route from {start} to {end} via back roads"

class Navigator:
    def __init__(self, strategy: RouteStrategy):
        self.strategy = strategy

    def get_route(self, start, end):
        return self.strategy.calculate(start, end)

nav = Navigator(FastestRoute())
print(nav.get_route("A", "B"))
nav.strategy = ShortestRoute()
print(nav.get_route("A", "B"))
```
**Output**:
```text
Fastest route from A to B via highway
Shortest route from A to B via back roads
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Enables hot-swapping algorithms on a live `Navigator` instance at runtime without recreating the coordinator object.

---

### 083. Decorator pattern for extending object behavior
**Problem**: Implement the Decorator pattern to add extra costs (milk, sugar) to a base `Coffee` object without subclassing for every combination.

```python
class Coffee:
    def cost(self):
        return 3.0

    def description(self):
        return "Coffee"

class MilkDecorator:
    def __init__(self, coffee):
        self._coffee = coffee

    def cost(self):
        return self._coffee.cost() + 0.5

    def description(self):
        return self._coffee.description() + " + Milk"

class SugarDecorator:
    def __init__(self, coffee):
        self._coffee = coffee

    def cost(self):
        return self._coffee.cost() + 0.2

    def description(self):
        return self._coffee.description() + " + Sugar"

order = SugarDecorator(MilkDecorator(Coffee()))
print(order.description())
print(order.cost())
```
**Output**:
```text
Coffee + Milk + Sugar
3.7
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Solves class explosion: Instead of creating $2^N$ subclasses (`CoffeeWithMilk`, `CoffeeWithSugar`, `CoffeeWithMilkAndSugar`), decorators wrap instances dynamically.

---

### 084. Adapter pattern to bridge incompatible interfaces
**Problem**: Implement an Adapter that lets an old `LegacyPrinter` class be used through a modern `Printer` interface expected by client code.

```python
class LegacyPrinter:
    def old_print(self, text):
        return f"[LEGACY] {text}"

class Printer:
    def print(self, text):
        raise NotImplementedError

class LegacyPrinterAdapter(Printer):
    def __init__(self, legacy_printer):
        self.legacy_printer = legacy_printer

    def print(self, text):
        return self.legacy_printer.old_print(text)

def client_code(printer: Printer, text):
    print(printer.print(text))

client_code(LegacyPrinterAdapter(LegacyPrinter()), "Hello World")
```
**Output**:
```text
[LEGACY] Hello World
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Acts as a translator layer between legacy third-party subsystems and new application architectures.

---

### 085. Facade pattern simplifying a complex subsystem
**Problem**: Implement a `HomeTheaterFacade` class that provides a single `watch_movie()` method hiding the complexity of coordinating a projector, sound system, and lights.

```python
class Projector:
    def on(self):
        return "Projector ON"

class SoundSystem:
    def on(self):
        return "Sound system ON"

class Lights:
    def dim(self):
        return "Lights dimmed"

class HomeTheaterFacade:
    def __init__(self):
        self.projector = Projector()
        self.sound = SoundSystem()
        self.lights = Lights()

    def watch_movie(self):
        steps = [self.lights.dim(), self.projector.on(), self.sound.on()]
        return " -> ".join(steps)

theater = HomeTheaterFacade()
print(theater.watch_movie())
```
**Output**:
```text
Lights dimmed -> Projector ON -> Sound system ON
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Provides a simple high-level unified interface to a complex collection of subsystem micro-components.

---

### 086. Command pattern with undo support
**Problem**: Implement the Command pattern for a simple text editor supporting an insert action and an `undo()` operation.

```python
class InsertTextCommand:
    def __init__(self, document, text):
        self.document = document
        self.text = text

    def execute(self):
        self.document.content += self.text

    def undo(self):
        self.document.content = self.document.content[:-len(self.text)]

class Document:
    def __init__(self):
        self.content = ""

doc = Document()
cmd = InsertTextCommand(doc, "Hello")
cmd.execute()
print(doc.content)
cmd.undo()
print(repr(doc.content))
```
**Output**:
```text
Hello
''
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Encapsulates actions as standalone objects, enabling action queues, transactional execution, and multi-level undo/redo stacks.

---

### 087. Composite pattern for a file-system-like tree
**Problem**: Implement the Composite pattern with `File` and `Folder` classes so folders can contain files or other folders, and both support a common `size()` method.

```python
class File:
    def __init__(self, name, size):
        self.name = name
        self._size = size

    def size(self):
        return self._size

class Folder:
    def __init__(self, name):
        self.name = name
        self.children = []

    def add(self, item):
        self.children.append(item)

    def size(self):
        return sum(child.size() for child in self.children)

root = Folder("root")
root.add(File("a.txt", 100))
sub = Folder("subfolder")
sub.add(File("b.txt", 250))
root.add(sub)
print(root.size())
```
**Output**:
```text
350
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Treats individual leaf objects (`File`) and composition trees (`Folder`) uniformly through recursion.

---

# CHAPTER 09: Composition & Mixins

### 088. Basic composition: Car has an Engine
**Problem**: Model a `Car` class that is composed of an `Engine` object rather than inheriting from it.

```python
class Engine:
    def __init__(self, horsepower):
        self.horsepower = horsepower

    def start(self):
        return f"Engine starting with {self.horsepower} HP"

class Car:
    def __init__(self, make, horsepower):
        self.make = make
        self.engine = Engine(horsepower)  # Composition (HAS-A)

    def start(self):
        return f"{self.make}: {self.engine.start()}"

c = Car("Ford", 300)
print(c.start())
```
**Output**:
```text
Ford: Engine starting with 300 HP
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - **Favor Composition over Inheritance**: A Car is NOT an Engine (IS-A); a Car HAS an Engine (HAS-A).

---

### 089. Delegation through composition
**Problem**: Create a `Playlist` class that delegates storage operations to an internal list, exposing only `add()` and `play_all()`.

```python
class Playlist:
    def __init__(self):
        self._songs = []

    def add(self, song):
        self._songs.append(song)

    def play_all(self):
        for song in self._songs:
            print(f"Playing: {song}")

p = Playlist()
p.add("Song A")
p.add("Song B")
p.play_all()
```
**Output**:
```text
Playing: Song A
Playing: Song B
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Avoids leaking irrelevant list methods (e.g. `.sort()`, `.reverse()`, `.pop()`) that would occur with direct inheritance from `list`.

---

### 090. Multiple mixins combined on one class
**Problem**: Create `ComparableMixin` and `PrintableMixin` classes that each add reusable behavior, then combine both into a single `Product` class.

```python
class ComparableMixin:
    def __eq__(self, other):
        return self.value == other.value

    def __lt__(self, other):
        return self.value < other.value

class PrintableMixin:
    def print_info(self):
        print(f"{type(self).__name__}: {self.value}")

class Product(ComparableMixin, PrintableMixin):
    def __init__(self, value):
        self.value = value

p1 = Product(10)
p2 = Product(20)
p1.print_info()
print(p1 < p2)
```
**Output**:
```text
Product: 10
True
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Mixin composability: Independent capabilities can be mixed and matched across different domain classes without rigid hierarchical inheritance trees.

---

### 091. Composition-based validation pipeline
**Problem**: Build a `Form` class composed of independent `Validator` objects (e.g., `NotEmptyValidator`, `MaxLengthValidator`) that are each applied to the input.

```python
class NotEmptyValidator:
    def validate(self, value):
        return len(value) > 0

class MaxLengthValidator:
    def __init__(self, max_len):
        self.max_len = max_len

    def validate(self, value):
        return len(value) <= self.max_len

class Form:
    def __init__(self, validators):
        self.validators = validators

    def is_valid(self, value):
        return all(v.validate(value) for v in self.validators)

form = Form([NotEmptyValidator(), MaxLengthValidator(10)])
print(form.is_valid("hello"))
print(form.is_valid(""))
print(form.is_valid("way too long"))
```
**Output**:
```text
True
False
False
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Pipeline Architecture: Validations are broken down into small, single-responsibility units that can be configured dynamically per form field.

---

### 092. Strategy injected via composition instead of inheritance
**Problem**: Instead of subclassing a `Sorter` for each algorithm, inject a sorting function into the `Sorter`'s constructor.

```python
class Sorter:
    def __init__(self, strategy):
        self.strategy = strategy

    def sort(self, data):
        return self.strategy(data)

ascending = Sorter(sorted)
descending = Sorter(lambda data: sorted(data, reverse=True))

print(ascending.sort([3, 1, 2]))
print(descending.sort([3, 1, 2]))
```
**Output**:
```text
[1, 2, 3]
[3, 2, 1]
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - First-class functions: In Python, functions are objects. Passing callables directly avoids the overhead of creating dedicated strategy classes for trivial algorithms.

---

### 093. Mixin adding serialization to multiple unrelated classes
**Problem**: Create a `DictSerializableMixin` that any class can inherit from to gain a `to_dict()` method based on its `__dict__`.

```python
class DictSerializableMixin:
    def to_dict(self):
        return dict(self.__dict__)

class User(DictSerializableMixin):
    def __init__(self, name, email):
        self.name = name
        self.email = email

class Order(DictSerializableMixin):
    def __init__(self, order_id, total):
        self.order_id = order_id
        self.total = total

print(User("Ana", "ana@example.com").to_dict())
print(Order(101, 59.99).to_dict())
```
**Output**:
```text
{'name': 'Ana', 'email': 'ana@example.com'}
{'order_id': 101, 'total': 59.99}
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Reusable cross-cutting concern: Provides uniform serialization across unrelated entities (`User`, `Order`).

---

### 094. Composing a Logger into unrelated services
**Problem**: Instead of every service class inheriting from a `LoggerMixin`, inject a shared `Logger` object into each service via composition.

```python
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")

class OrderService:
    def __init__(self, logger):
        self.logger = logger

    def place_order(self, order_id):
        self.logger.log(f"Order {order_id} placed")

class PaymentService:
    def __init__(self, logger):
        self.logger = logger

    def process_payment(self, amount):
        self.logger.log(f"Payment of ${amount} processed")

logger = Logger()
OrderService(logger).place_order(101)
PaymentService(logger).process_payment(49.99)
```
**Output**:
```text
[LOG] Order 101 placed
[LOG] Payment of $49.99 processed
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - **Dependency Injection**: Services receive dependencies from outside, simplifying testing through mock injections.

---

### 095. Has-a relationship modeled with a list of composed objects
**Problem**: Model a `House` class composed of multiple `Room` objects, with a method to compute the total area of the house.

```python
class Room:
    def __init__(self, name, area):
        self.name = name
        self.area = area

class House:
    def __init__(self):
        self.rooms = []

    def add_room(self, room):
        self.rooms.append(room)

    def total_area(self):
        return sum(room.area for room in self.rooms)

house = House()
house.add_room(Room("Bedroom", 20))
house.add_room(Room("Kitchen", 15))
print(house.total_area())
```
**Output**:
```text
35
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Aggregation / Composition: The lifecycle of composite parts is managed within the parent container.

---

# CHAPTER 10: Exception Handling in OOP

### 096. Custom exception class
**Problem**: Create a custom `InsufficientFundsError` exception and raise it from a `BankAccount`'s `withdraw()` method.

```python
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f"Cannot withdraw {amount}: balance is only {balance}")

class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(self.balance, amount)
        self.balance -= amount

acc = BankAccount(50)
try:
    acc.withdraw(100)
except InsufficientFundsError as e:
    print(f"Error: {e}")
```
**Output**:
```text
Error: Cannot withdraw 100: balance is only 50
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - Custom exceptions inherit from `Exception` (never `BaseException`, which is reserved for system-exiting exceptions like `KeyboardInterrupt`).
> - Attaching attributes (`self.balance`, `self.amount`) allows exception catchers to inspect data programmatically.

---

### 097. Exception hierarchy for a custom library
**Problem**: Design a small exception hierarchy: a base `ValidationError` with two subclasses `RequiredFieldError` and `InvalidFormatError`.

```python
class ValidationError(Exception):
    pass

class RequiredFieldError(ValidationError):
    def __init__(self, field):
        super().__init__(f"'{field}' is required")

class InvalidFormatError(ValidationError):
    def __init__(self, field, value):
        super().__init__(f"'{field}' has invalid value: {value}")

def validate(data):
    if "email" not in data:
        raise RequiredFieldError("email")
    if "@" not in data["email"]:
        raise InvalidFormatError("email", data["email"])

try:
    validate({"email": "bademail"})
except ValidationError as e:
    print(f"Validation failed: {e}")
```
**Output**:
```text
Validation failed: 'email' has invalid value: bademail
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Hierarchical Catching: Callers can catch specific errors (`except RequiredFieldError:`) or catch all domain validation errors generically (`except ValidationError:`).

---

### 098. Exception chaining with `'raise ... from'`
**Problem**: Catch a low-level exception and re-raise a more descriptive custom exception while preserving the original cause.

```python
class DataProcessingError(Exception):
    pass

def parse_number(value):
    try:
        return int(value)
    except ValueError as original_error:
        raise DataProcessingError(f"Could not parse '{value}' as a number") from original_error

try:
    parse_number("abc")
except DataProcessingError as e:
    print(f"Error: {e}")
    print(f"Caused by: {e.__cause__}")
```
**Output**:
```text
Error: Could not parse 'abc' as a number
Caused by: invalid literal for int() with base 10: 'abc'
```
> [!TIP]
> **💡 Info Pointer & Deep Dive:**
> - PEP 3134 Exception Chaining: `raise NewException from original_error` sets the `__cause__` attribute, preserving the full traceback for debugging while exposing clean high-level exceptions.

---

### 099. Context manager guaranteeing cleanup even on exception
**Problem**: Create a `ManagedResource` context manager class that always releases a resource in `__exit__`, even if the code inside the `with` block raises an exception.

```python
class ManagedResource:
    def __enter__(self):
        print("Resource acquired")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Resource released")
        return False  # Propagate any exception

try:
    with ManagedResource():
        raise RuntimeError("Something went wrong")
except RuntimeError as e:
    print(f"Caught: {e}")
```
**Output**:
```text
Resource acquired
Resource released
Caught: Something went wrong
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Resource Safety: Guarantees teardown execution (closing sockets, releasing database locks, flushing file buffers) regardless of runtime failures.

---

### 100. Catching a base exception type to handle multiple subclasses generically
**Problem**: Define two specific exceptions that both inherit from a shared `AppError` base, and catch them generically in one `except` block.

```python
class AppError(Exception):
    pass

class NetworkError(AppError):
    pass

class TimeoutError_(AppError):
    pass

def risky_operation(kind):
    if kind == "network":
        raise NetworkError("Connection refused")
    elif kind == "timeout":
        raise TimeoutError_("Request timed out")

for kind in ("network", "timeout"):
    try:
        risky_operation(kind)
    except AppError as e:
        print(f"Handled {type(e).__name__}: {e}")
```
**Output**:
```text
Handled NetworkError: Connection refused
Handled TimeoutError_: Request timed out
```
> [!NOTE]
> **💡 Info Pointer & Deep Dive:**
> - Subtype Polymorphism in Exception Handling: An `except BaseClass` block catches instances of `BaseClass` as well as any of its derived subclasses.

---

## Appendix: Dunder Methods & Pattern Cheat Sheet

### 1. Magic Methods (Dunder Protocol) Quick Reference

| Method Category | Dunder Methods | Invocation Trigger |
| :--- | :--- | :--- |
| **Object Lifecycle** | `__new__(cls, ...)`, `__init__(self, ...)`, `__del__(self)` | Creation, initialization, garbage collection |
| **String / Representation** | `__str__(self)`, `__repr__(self)`, `__format__(self, spec)` | `str(x)`, `print(x)`, `repr(x)`, f-strings |
| **Arithmetic Operators** | `__add__`, `__sub__`, `__mul__`, `__truediv__`, `__floordiv__`, `__mod__`, `__pow__` | `a + b`, `a - b`, `a * b`, `a / b`, `a // b`, `a % b`, `a ** b` |
| **Reflected Operators** | `__radd__`, `__rsub__`, `__rmul__`, `__rtruediv__` | `b + a` (when `type(b)` doesn't support `a`) |
| **In-place Operators** | `__iadd__`, `__isub__`, `__imul__`, `__itruediv__` | `a += b`, `a -= b`, `a *= b` |
| **Comparison Operators** | `__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__` | `==`, `!=`, `<`, `<=`, `>`, `>=` |
| **Containers & Slicing** | `__len__`, `__getitem__`, `__setitem__`, `__delitem__`, `__contains__` | `len(x)`, `x[i]`, `x[i] = v`, `del x[i]`, `item in x` |
| **Iteration** | `__iter__`, `__next__` | `iter(x)`, `next(x)`, `for item in x:` |
| **Callable Objects** | `__call__(self, *args, **kwargs)` | `instance(*args, **kwargs)` |
| **Context Management** | `__enter__(self)`, `__exit__(self, exc_type, exc_val, exc_tb)` | `with instance:` |
| **Attribute Access** | `__getattr__`, `__getattribute__`, `__setattr__`, `__delattr__` | `x.attr`, `x.attr = val` |

---

### 2. Design Patterns Classification in Python

```
                       ┌──────────────────────────────────────────────┐
                       │        Gang of Four (GoF) Patterns           │
                       └──────────────────────┬───────────────────────┘
                                              │
        ┌─────────────────────────────┼─────────────────────────────┐
        ▼                             ▼                             ▼
 ┌──────────────┐              ┌──────────────┐              ┌──────────────┐
 │  Creational  │              │  Structural  │              │  Behavioral  │
 ├──────────────┤              ├──────────────┤              ├──────────────┤
 │• Singleton   │              │• Adapter     │              │• Strategy    │
 │• Factory     │              │• Decorator   │              │• Observer    │
 │• Builder     │              │• Facade      │              │• Command     │
 │              │              │• Composite   │              │• Template    │
 └──────────────┘              └──────────────┘              └──────────────┘
```

---
*End of Master Document.*
