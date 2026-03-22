# Object-Oriented Programming -- Assignment
### Four Real-World Problems to Solve Using OOP

---

> This assignment covers OOP Basics and Intermediate concepts.
> For each problem, read the description carefully, understand what the
> system needs to do, and design your classes before writing any code.
> Planning on paper first is not optional -- it is part of the exercise.

---

## How to Approach Each Problem

Before writing a single line of code for any problem:

1. Read the entire problem description
2. Identify the classes you need
3. List the attributes each class should have
4. List the methods each class should have
5. Identify relationships between classes (inheritance, composition)
6. Only then start coding

The OOP concepts tested in each problem are clearly labelled so you know
what to focus on.

---

## Problem 1 -- Library Management System

### Difficulty: Beginner to Intermediate
### Concepts tested: Classes, instance methods, class variables, `__str__`, `__repr__`, custom exceptions, properties

---

### Background

You are building a digital system for a public library in Pune.
The library has a collection of books. Members can borrow books and return them.
The system needs to track which books are available, which are borrowed,
and by whom.

### What You Need to Build

#### Class: `Book`

Each book in the library has the following information:
- Title
- Author
- ISBN (a unique identifier -- no two books share an ISBN)
- Total copies in the library
- Copies currently available (starts equal to total copies)

The Book class must:
- Raise a `ValueError` if total copies is less than 1
- Have a property `is_available` that returns True if at least one copy is available
- Have a method `borrow()` that reduces available copies by 1. Raise a custom
  exception `BookNotAvailableError` if no copies are available
- Have a method `return_book()` that increases available copies by 1. Raise a
  `ValueError` if available copies would exceed total copies
- Have a class variable `total_books_in_library` that tracks the total number
  of Book objects created
- `__str__` should return something readable for a member: e.g. `"The White Tiger by Aravind Adiga (2 copies available)"`
- `__repr__` should return something useful for a developer

#### Class: `Member`

A library member has:
- Name
- Member ID (auto-generated -- use a class variable counter starting from M001)
- A list of books currently borrowed (starts empty)
- Maximum books allowed at once (default: 3)

The Member class must:
- Have a method `borrow_book(book)` that calls `book.borrow()`, adds the book
  to the member's borrowed list, and prints a confirmation. Raise a
  `BorrowLimitError` if the member already has the maximum number of books.
- Have a method `return_book(book)` that calls `book.return_book()`, removes
  it from the borrowed list, and prints a confirmation. Raise a `ValueError`
  if the member does not have that book.
- Have a method `borrowed_titles()` that returns a list of titles currently borrowed
- `__str__` should show the member's name, ID, and how many books they have borrowed

#### Class: `Library`

The library itself:
- Has a name and city
- Maintains a collection of books (a dict mapping ISBN to Book objects)
- Maintains a collection of members (a dict mapping member ID to Member objects)

The Library class must:
- Have a method `add_book(book)` that adds a book to the collection
- Have a method `register_member(member)` that registers a new member
- Have a method `search_by_title(title)` that returns all books whose title
  contains the search string (case-insensitive)
- Have a method `search_by_author(author)` that returns all books by that author
- Have a method `available_books()` that returns all books with at least one copy available
- Have a method `generate_report()` that prints a summary of total books,
  total members, and total books currently borrowed

#### Custom Exceptions

Create these exception classes:
- `LibraryError(Exception)` -- base exception for the library system
- `BookNotAvailableError(LibraryError)` -- raised when a book has no copies left
- `BorrowLimitError(LibraryError)` -- raised when a member tries to borrow beyond their limit

### Expected Behaviour

```python
# Sample usage (your code should produce similar output)

library = Library("Pune Central Library", "Pune")

b1 = Book("The White Tiger",    "Aravind Adiga",   "ISBN001", total_copies=2)
b2 = Book("A Suitable Boy",     "Vikram Seth",     "ISBN002", total_copies=1)
b3 = Book("The God of Small Things", "Arundhati Roy", "ISBN003", total_copies=3)

library.add_book(b1)
library.add_book(b2)
library.add_book(b3)

m1 = Member("Aarav Sharma")
m2 = Member("Priya Patel")

library.register_member(m1)
library.register_member(m2)

m1.borrow_book(b1)   # "Aarav Sharma borrowed: The White Tiger"
m1.borrow_book(b2)   # "Aarav Sharma borrowed: A Suitable Boy"
m2.borrow_book(b1)   # "Priya Patel borrowed: The White Tiger"

print(b1)            # The White Tiger by Aravind Adiga (0 copies available)
print(b1.is_available)  # False

m2.borrow_book(b1)   # BookNotAvailableError

m1.return_book(b2)   # "Aarav Sharma returned: A Suitable Boy"

library.generate_report()
# Pune Central Library -- Report
# Total books    : 3 titles
# Total members  : 2
# Books borrowed : 2
```

### What to Highlight in Your Solution

Mark the following in your code comments:
- Where **encapsulation** is used (private data, controlled access)
- Where **properties** are used
- Where **custom exceptions** are used
- Where **class variables** are used vs **instance variables**

---

## Problem 2 -- Employee Payroll System

### Difficulty: Intermediate
### Concepts tested: Inheritance, method overriding, `super()`, polymorphism, abstract base class, `@classmethod`, `@staticmethod`

---

### Background

A Bangalore tech company has three types of employees:
- Full-time employees (fixed monthly salary)
- Contract employees (paid per hour worked)
- Intern employees (fixed stipend, capped at 6 months)

All employees share some common attributes and behaviour, but the way their
pay is calculated is different for each type. The payroll system should be
able to process any employee through the same interface regardless of type.

### What You Need to Build

#### Abstract Class: `Employee`

Use Python's `ABC` module. This class should:
- Have `__init__` accepting: `name`, `employee_id`, `department`
- Store `name`, `employee_id`, `department`, and `joining_date` (set to today automatically)
- Have an `@abstractmethod` called `calculate_pay()` that returns a float
- Have an `@abstractmethod` called `employment_type()` that returns a string
  like "Full-Time", "Contract", or "Intern"
- Have a concrete method `generate_payslip()` that prints a formatted payslip.
  It calls `calculate_pay()` and `employment_type()` internally -- it works
  for ALL employee types without any modification
- Have a `@classmethod` called `from_dict(data)` that creates an employee from
  a dictionary (you will implement this on each subclass)
- Have a `@staticmethod` called `is_valid_department(dept)` that returns True
  if the department is one of: Engineering, Product, Sales, HR, Marketing, Finance
- Have `__str__` return: `"[EmployeeType] Name (ID) -- Department"`

#### Class: `FullTimeEmployee(Employee)`

Additional attributes:
- `monthly_salary` (float)
- `performance_rating` (float, 1.0 to 5.0, default 3.0)

`calculate_pay()` must:
- Return monthly salary
- Add a 10% bonus if performance rating is 4.5 or above
- Add a 5% bonus if performance rating is between 4.0 and 4.49

`employment_type()` returns `"Full-Time"`

#### Class: `ContractEmployee(Employee)`

Additional attributes:
- `hourly_rate` (float)
- `hours_worked` (float, this month)

`calculate_pay()` returns `hourly_rate * hours_worked`

`employment_type()` returns `"Contract"`

#### Class: `Intern(Employee)`

Additional attributes:
- `monthly_stipend` (float, must not exceed 25000)
- `duration_months` (int, must not exceed 6)

`calculate_pay()` returns `monthly_stipend`

`employment_type()` returns `"Intern"`

Raise a `ValueError` in `__init__` if stipend exceeds 25000 or duration exceeds 6.

#### Function: `run_payroll(employees)`

Write a standalone function (not inside a class) that:
- Accepts a list of any mix of Employee subclass objects
- Calls `generate_payslip()` on each one
- Prints a summary at the end: total payroll amount, number of each employee type

This function demonstrates **polymorphism** -- it treats every employee
the same way regardless of type.

### Expected Behaviour

```python
employees = [
    FullTimeEmployee("Vikram Nair",   "E001", "Engineering", monthly_salary=95000,
                     performance_rating=4.7),
    FullTimeEmployee("Ananya Shah",   "E002", "Product",     monthly_salary=85000,
                     performance_rating=3.8),
    ContractEmployee("Suresh Kumar",  "C001", "Sales",       hourly_rate=800,
                     hours_worked=160),
    Intern("Divya Menon",             "I001", "Engineering", monthly_stipend=20000,
           duration_months=3),
]

run_payroll(employees)

# Expected payslip output for Vikram (salary + 10% bonus):
# ==========================================
# PAYSLIP
# Name           : Vikram Nair
# ID             : E001
# Department     : Engineering
# Type           : Full-Time
# Joining Date   : 2024-03-15
# Gross Pay      : Rs.1,04,500.00
# ==========================================

# Summary:
# Full-Time  : 2 employees  |  Total: Rs.1,97,750.00
# Contract   : 1 employee   |  Total: Rs.1,28,000.00
# Intern     : 1 employee   |  Total: Rs.20,000.00
# Grand Total: Rs.3,45,750.00
```

### What to Highlight in Your Solution

Mark the following in your code comments:
- Where **inheritance** is used and how `super()` is called
- Where **method overriding** happens (each subclass overriding `calculate_pay`)
- Where **polymorphism** happens (the `run_payroll` function)
- Where **abstraction** is enforced (the ABC preventing direct instantiation)

---

## Problem 3 -- Online Food Ordering System

### Difficulty: Intermediate
### Concepts tested: Encapsulation with properties, composition, dunder methods, class methods, the `__enter__`/`__exit__` context manager, operator overloading

---

### Background

You are building the core order management system for a Zomato-like food
delivery app. The system needs to handle menu items, customer carts,
and orders.

### What You Need to Build

#### Class: `MenuItem`

Represents a single item on a restaurant's menu:
- `name` (str)
- `price` (float, must be positive -- use a property with validation)
- `category` (str: "Starter", "Main", "Dessert", "Beverage")
- `is_available` (bool, default True)

Must implement:
- `__str__`: `"Masala Dosa -- Rs.80 (Main)"`
- `__repr__`: developer-friendly representation
- `__eq__`: two MenuItems are equal if they have the same name and price
- `__lt__`: items are ordered by price (enables sorting a menu by price)

#### Class: `CartItem`

A MenuItem in a cart with a quantity:
- `item` (a MenuItem object -- composition)
- `quantity` (int, must be >= 1 -- use a property with validation)

Must implement:
- `subtotal` property: returns `item.price * quantity`
- `__str__`: `"Masala Dosa x2 = Rs.160"`

#### Class: `Cart`

A customer's shopping cart:
- `customer_name` (str)
- `_items` (list of CartItem objects, private)
- `restaurant_name` (str)

Must implement:
- `add_item(menu_item, quantity=1)`: adds a CartItem. If the item is already
  in the cart, increase the quantity instead of adding a duplicate
- `remove_item(menu_item)`: removes the CartItem for that MenuItem
- `update_quantity(menu_item, quantity)`: updates quantity for an existing item
- `total` property: sum of all CartItem subtotals
- `item_count` property: total number of individual items (sum of quantities)
- `is_empty` property: True if cart has no items
- `clear()`: empties the cart
- `__len__`: returns number of distinct items in cart
- `__str__`: prints a formatted cart summary
- `__add__`: combine two carts (merge their items) and return a new Cart
  (useful for group orders)
- Works as a context manager: entering the context returns the cart,
  exiting it prints a summary and clears it if empty

#### Class: `Order`

Created from a Cart when the customer checks out:
- `order_id` (auto-generated, format: "ORD-XXXX" where XXXX is a 4-digit number)
- `customer_name`
- `restaurant_name`
- `items` (list of CartItem -- copied from the Cart)
- `order_time` (set to now automatically)
- `status` (starts as "Placed")
- `delivery_address` (str)
- `subtotal` property
- `delivery_fee` property: Rs.0 if subtotal >= 300, else Rs.40
- `gst` property: 5% of subtotal
- `total` property: subtotal + delivery_fee + gst (rounded to 2 decimal places)

Must implement:
- `update_status(new_status)`: updates status. Valid statuses are:
  "Placed", "Confirmed", "Preparing", "Out for Delivery", "Delivered", "Cancelled"
  Raise a `ValueError` for invalid statuses.
- `@classmethod from_cart(cart, delivery_address)`: creates an Order from a Cart
- `__str__`: a complete formatted order receipt

### Expected Behaviour

```python
# Building a menu
dosa      = MenuItem("Masala Dosa",    80,  "Main")
coffee    = MenuItem("Filter Coffee",  40,  "Beverage")
idli      = MenuItem("Idli Sambar",    60,  "Main")
gulab     = MenuItem("Gulab Jamun",    50,  "Dessert")
vada      = MenuItem("Medu Vada",      45,  "Starter")

# Using the Cart as a context manager
with Cart("Aarav Sharma", "Udupi Palace") as cart:
    cart.add_item(dosa, 2)
    cart.add_item(coffee, 1)
    cart.add_item(idli, 1)
    cart.add_item(dosa, 1)   # should increase dosa quantity to 3, not add duplicate

    print(len(cart))          # 3 (three distinct items)
    print(cart.total)         # 80*3 + 40 + 60 = 340

    order = Order.from_cart(cart, "42, Koregaon Park, Pune")
    print(order)

# Order receipt should show:
# =========================================
# ORDER RECEIPT
# Order ID    : ORD-0001
# Customer    : Aarav Sharma
# Restaurant  : Udupi Palace
# Time        : 15 Mar 2024, 14:30
# -----------------------------------------
# Masala Dosa       x3    Rs.240
# Filter Coffee     x1    Rs.40
# Idli Sambar       x1    Rs.60
# -----------------------------------------
# Subtotal          :    Rs.340.00
# Delivery Fee      :    Rs.0.00
# GST (5%)          :    Rs.17.00
# Total             :    Rs.357.00
# -----------------------------------------
# Status: Placed
# =========================================

# Sorting menu items by price (uses __lt__)
menu = [dosa, coffee, idli, gulab, vada]
print(sorted(menu))

# Combining two carts (uses __add__)
cart2 = Cart("Priya Patel", "Udupi Palace")
cart2.add_item(gulab, 2)
cart2.add_item(vada, 1)

combined = cart + cart2
print(combined.total)
```

### What to Highlight in Your Solution

Mark the following in your code comments:
- Where **encapsulation** is used (private `_items`, properties with validation)
- Where **composition** is used (CartItem has a MenuItem, Cart has CartItems)
- Where **dunder methods** make your objects feel native to Python
- Where the **context manager** protocol is implemented

---

## Problem 4 -- Student Grade Management System (Full System)

### Difficulty: Intermediate to Advanced
### Concepts tested: All four pillars together, mixins, dataclasses, multiple inheritance, `__slots__`, comprehensive dunder methods

---

### Background

Build a complete grade management system for a college. The system handles
students, courses, grade records, and generates reports. This problem
intentionally requires you to think about the full design before writing code.

### What You Need to Build

#### Mixin: `SerializableMixin`

Adds the ability to convert any object to a dictionary and to a JSON string.
- `to_dict()`: returns a dict of all non-private attributes
- `to_json()`: returns a JSON string (use the `json` module)

#### Mixin: `TimestampMixin`

Adds creation and update timestamps to any class.
- `created_at`: set to now in `__init__`
- `updated_at`: set to now in `__init__`
- `touch()`: updates `updated_at` to now

#### Dataclass: `GradeRecord`

Use `@dataclass` to represent a single grade entry:
- `subject` (str)
- `marks_obtained` (float)
- `max_marks` (float, default 100)
- `exam_type` (str: "Internal", "Midterm", "Final", default "Final")
- `percentage` property (computed: marks_obtained / max_marks * 100)
- `grade_letter` property (A+: >=90, A: >=80, B: >=70, C: >=60, D: >=50, F: below 50)
- The dataclass should be orderable (sort by percentage)

#### Class: `Student(SerializableMixin, TimestampMixin)`

Uses both mixins. Has `__slots__` for memory efficiency since the college
has thousands of students.

Slots: `name`, `student_id`, `department`, `year`, `_grade_records`

Must:
- Auto-generate `student_id` in format "STU-YYYY-NNNN" where YYYY is current
  year and NNNN is a zero-padded counter (STU-2024-0001, STU-2024-0002, etc.)
- `_grade_records` is a private list starting empty
- `add_grade(grade_record)`: adds a GradeRecord. Raise `TypeError` if not a GradeRecord
- `grades` property: returns a copy of the grade records list
- `cgpa` property: average percentage across all grade records, on a 10-point scale
  (percentage / 10). Returns 0.0 if no grades yet
- `top_subject` property: returns the GradeRecord with the highest percentage
- `failed_subjects` property: returns list of GradeRecords where grade_letter is "F"
- `generate_transcript()`: prints a formatted transcript
- `__str__`: name, ID, department, CGPA
- `__repr__`: developer-friendly
- `__eq__`: two students are equal if they have the same student_id
- `__lt__`: students are ordered by CGPA (for ranking)

#### Class: `Course`

- `course_code` (str, e.g. "CS101")
- `course_name` (str)
- `credits` (int)
- `instructor` (str)
- `enrolled_students` (list of Student objects, starts empty)
- `enroll(student)`: adds student if not already enrolled
- `unenroll(student)`: removes student
- `average_cgpa` property: mean CGPA of all enrolled students
- `top_student` property: Student with highest CGPA
- `__len__`: number of enrolled students
- `__contains__`: supports `student in course`

#### Class: `Department`

- `name` (str)
- `head` (str, name of department head)
- `courses` (dict of course_code: Course)
- `students` (dict of student_id: Student)
- `add_course(course)`
- `add_student(student)`
- `generate_report()`: prints department-wide report including:
  - Total students
  - Total courses
  - Department average CGPA
  - Topper (highest CGPA student)
  - Number of students with CGPA below 5.0 (at risk)
  - Distribution of grade letters across all grade records

### Expected Behaviour

```python
# Create department
cs_dept = Department("Computer Science", "Dr. Ramesh Kumar")

# Create courses
python_course = Course("CS101", "Python Programming", credits=4,
                        instructor="Prof. Priya Nair")
ml_course     = Course("CS301", "Machine Learning",   credits=4,
                        instructor="Prof. Vikram Sharma")

cs_dept.add_course(python_course)
cs_dept.add_course(ml_course)

# Create students
s1 = Student("Aarav Sharma",  "Computer Science", year=2)
s2 = Student("Priya Patel",   "Computer Science", year=2)
s3 = Student("Rohan Verma",   "Computer Science", year=3)

# Add grades using GradeRecord dataclass
s1.add_grade(GradeRecord("Python Programming", 88,  exam_type="Final"))
s1.add_grade(GradeRecord("Machine Learning",   91,  exam_type="Final"))
s1.add_grade(GradeRecord("Data Structures",    79,  exam_type="Midterm"))

s2.add_grade(GradeRecord("Python Programming", 95))
s2.add_grade(GradeRecord("Machine Learning",   88))
s2.add_grade(GradeRecord("Data Structures",    72))

s3.add_grade(GradeRecord("Python Programming", 45))   # failed
s3.add_grade(GradeRecord("Data Structures",    58))

# Enroll in courses
python_course.enroll(s1)
python_course.enroll(s2)
python_course.enroll(s3)
ml_course.enroll(s1)
ml_course.enroll(s2)

cs_dept.add_student(s1)
cs_dept.add_student(s2)
cs_dept.add_student(s3)

# Generate transcript
s1.generate_transcript()

# Output:
# ============================================
# ACADEMIC TRANSCRIPT
# Name       : Aarav Sharma
# ID         : STU-2024-0001
# Department : Computer Science
# Year       : 2
# CGPA       : 8.60 / 10
# --------------------------------------------
# Subject               Marks   Grade
# --------------------------------------------
# Python Programming    88/100   A     (Final)
# Machine Learning      91/100   A+    (Final)
# Data Structures       79/100   C     (Midterm)
# --------------------------------------------
# Top Subject : Machine Learning (91%)
# ============================================

# Serialisation (from SerializableMixin)
import json
print(s1.to_json())

# Ranking students (uses __lt__)
all_students = [s1, s2, s3]
ranked = sorted(all_students, reverse=True)

# Department report
cs_dept.generate_report()

# Check membership (uses __contains__)
print(s1 in python_course)   # True
print(s3 in ml_course)       # False
```

### What to Highlight in Your Solution

Mark the following in your code comments:
- All four pillars: **Encapsulation**, **Inheritance** (through mixins),
  **Polymorphism** (to_dict/to_json works on any class using the mixin),
  **Abstraction** (clean interfaces hiding implementation details)
- Where **mixins** are used and why they are better than a deep inheritance chain
- Where **dataclasses** reduce boilerplate
- Where **`__slots__`** is used and why
- Where **composition** is used (Student has GradeRecords, Department has Courses and Students)

---

## Submission Checklist

Before submitting your solutions, verify:

```
Problem 1 -- Library System
  [ ] Book class with all methods and properties
  [ ] Member class with borrow/return logic
  [ ] Library class with search and report
  [ ] Three custom exceptions in a hierarchy
  [ ] __str__ and __repr__ on all classes
  [ ] Class variable tracking total books

Problem 2 -- Payroll System
  [ ] Abstract Employee class using ABC
  [ ] Three concrete subclasses with calculate_pay() overridden
  [ ] generate_payslip() works without modification for all types
  [ ] run_payroll() demonstrates polymorphism
  [ ] @classmethod and @staticmethod present

Problem 3 -- Food Ordering System
  [ ] MenuItem with price validation and comparison operators
  [ ] CartItem with quantity validation
  [ ] Cart with __add__, __len__, context manager
  [ ] Order auto-generates ID and calculates all fees
  [ ] No duplicate items in cart

Problem 4 -- Grade Management System
  [ ] SerializableMixin and TimestampMixin
  [ ] GradeRecord as a proper dataclass
  [ ] Student uses __slots__
  [ ] Auto-generated student IDs
  [ ] Department report with full statistics
  [ ] All dunder methods implemented and tested
```

---

*Codeverra OOP Assignment | learn.codeverra.com*
