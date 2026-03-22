# Object-Oriented Programming -- Assignment Solutions
### Full Solutions with Concept Annotations

---

> Each solution includes inline comments marking exactly where each
> OOP concept is being applied. Read the comments as carefully as the code.

---

## Solution 1 -- Library Management System

```python
# ─── Custom Exceptions ─────────────────────────────────────────────────────
# CONCEPT: Custom exception hierarchy (OOP basics)
# We inherit from a base LibraryError so callers can catch all library
# errors with one except clause, or be specific when needed.

class LibraryError(Exception):
    """Base exception for all library system errors."""
    pass

class BookNotAvailableError(LibraryError):
    """Raised when a book has no copies available to borrow."""
    def __init__(self, title):
        self.title = title
        super().__init__(f"'{title}' has no copies available.")

class BorrowLimitError(LibraryError):
    """Raised when a member tries to borrow beyond their allowed limit."""
    def __init__(self, member_name, limit):
        super().__init__(
            f"{member_name} has already borrowed the maximum of {limit} books."
        )


# ─── Book ──────────────────────────────────────────────────────────────────

class Book:

    # CONCEPT: Class variable
    # Shared across all Book instances -- tracks how many Book objects exist
    total_books_in_library = 0

    def __init__(self, title, author, isbn, total_copies):
        if total_copies < 1:
            raise ValueError("A book must have at least 1 copy.")

        # CONCEPT: Instance variables -- unique to each Book object
        self.title           = title
        self.author          = author
        self.isbn            = isbn
        self._total_copies   = total_copies       # private -- controlled via property
        self._available      = total_copies       # private -- starts equal to total

        # CONCEPT: Modifying class variable at instance creation
        Book.total_books_in_library += 1

    # CONCEPT: Property -- controlled read access to private attribute
    @property
    def total_copies(self):
        return self._total_copies

    # CONCEPT: Computed property -- no setter, derived from state
    @property
    def is_available(self):
        return self._available > 0

    @property
    def available_copies(self):
        return self._available

    def borrow(self):
        """Reduce available copies by 1. Raise if none left."""
        # CONCEPT: Encapsulation -- the borrow logic is owned by the Book
        # No external code can directly set _available
        if not self.is_available:
            raise BookNotAvailableError(self.title)
        self._available -= 1

    def return_book(self):
        """Increase available copies by 1. Raise if already at maximum."""
        if self._available >= self._total_copies:
            raise ValueError(
                f"All copies of '{self.title}' are already in the library."
            )
        self._available += 1

    # CONCEPT: __str__ -- human-readable, shown to end users
    def __str__(self):
        status = f"{self._available} cop{'y' if self._available == 1 else 'ies'} available"
        return f"{self.title} by {self.author} ({status})"

    # CONCEPT: __repr__ -- developer-readable, useful for debugging
    def __repr__(self):
        return (f"Book(title={self.title!r}, author={self.author!r}, "
                f"isbn={self.isbn!r}, available={self._available}/{self._total_copies})")


# ─── Member ────────────────────────────────────────────────────────────────

class Member:

    # CONCEPT: Class variable -- auto-incrementing ID counter
    _id_counter = 0

    def __init__(self, name, max_books=3):
        Member._id_counter += 1

        # CONCEPT: Instance variables
        self.name         = name
        self.member_id    = f"M{Member._id_counter:03d}"  # M001, M002, ...
        self._borrowed    = []    # private list -- accessed via methods only
        self.max_books    = max_books

    def borrow_book(self, book):
        """Borrow a book. Raises BorrowLimitError or BookNotAvailableError."""
        # CONCEPT: Encapsulation -- this method controls the rules of borrowing
        if len(self._borrowed) >= self.max_books:
            raise BorrowLimitError(self.name, self.max_books)

        # Delegate to Book's own borrow logic (Book owns its availability state)
        book.borrow()
        self._borrowed.append(book)
        print(f"{self.name} borrowed: {book.title}")

    def return_book(self, book):
        """Return a borrowed book."""
        if book not in self._borrowed:
            raise ValueError(f"{self.name} has not borrowed '{book.title}'.")
        book.return_book()
        self._borrowed.remove(book)
        print(f"{self.name} returned: {book.title}")

    def borrowed_titles(self):
        """Return list of titles currently borrowed."""
        return [book.title for book in self._borrowed]

    def __str__(self):
        return (f"Member({self.name}, ID: {self.member_id}, "
                f"Books borrowed: {len(self._borrowed)}/{self.max_books})")


# ─── Library ───────────────────────────────────────────────────────────────

class Library:

    def __init__(self, name, city):
        self.name     = name
        self.city     = city
        self._books   = {}    # isbn -> Book   (encapsulated collection)
        self._members = {}    # member_id -> Member

    def add_book(self, book):
        self._books[book.isbn] = book

    def register_member(self, member):
        self._members[member.member_id] = member

    def search_by_title(self, title):
        """Return all books whose title contains the search string."""
        query = title.lower()
        return [b for b in self._books.values() if query in b.title.lower()]

    def search_by_author(self, author):
        """Return all books by a given author (case-insensitive)."""
        query = author.lower()
        return [b for b in self._books.values() if query in b.author.lower()]

    def available_books(self):
        """Return all books with at least one available copy."""
        return [b for b in self._books.values() if b.is_available]

    def generate_report(self):
        """Print a summary of the library's current state."""
        total_borrowed = sum(
            1
            for m in self._members.values()
            for _ in m._borrowed
        )
        print(f"\n{self.name} -- Report")
        print(f"  Total books   : {len(self._books)} titles")
        print(f"  Total members : {len(self._members)}")
        print(f"  Books borrowed: {total_borrowed}")


# ─── Demo ──────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    library = Library("Pune Central Library", "Pune")

    b1 = Book("The White Tiger",         "Aravind Adiga",   "ISBN001", total_copies=2)
    b2 = Book("A Suitable Boy",          "Vikram Seth",     "ISBN002", total_copies=1)
    b3 = Book("The God of Small Things", "Arundhati Roy",   "ISBN003", total_copies=3)

    library.add_book(b1)
    library.add_book(b2)
    library.add_book(b3)

    m1 = Member("Aarav Sharma")
    m2 = Member("Priya Patel")

    library.register_member(m1)
    library.register_member(m2)

    m1.borrow_book(b1)
    m1.borrow_book(b2)
    m2.borrow_book(b1)

    print(b1)
    print(b1.is_available)

    try:
        m2.borrow_book(b1)
    except BookNotAvailableError as e:
        print(f"Error: {e}")

    m1.return_book(b2)

    library.generate_report()
    print(f"\nTotal Book objects created: {Book.total_books_in_library}")
```

---

## Solution 2 -- Employee Payroll System

```python
from abc import ABC, abstractmethod
from datetime import date


# ─── Abstract Base Class ───────────────────────────────────────────────────
# CONCEPT: Abstraction
# Employee defines the CONTRACT that all employee types must follow.
# You cannot instantiate Employee directly -- it exists only as a template.

class Employee(ABC):

    VALID_DEPARTMENTS = {
        "Engineering", "Product", "Sales", "HR", "Marketing", "Finance"
    }

    def __init__(self, name, employee_id, department):
        if not self.is_valid_department(department):
            raise ValueError(
                f"'{department}' is not a valid department. "
                f"Choose from: {sorted(self.VALID_DEPARTMENTS)}"
            )
        # CONCEPT: Instance variables set in abstract base
        # All subclasses automatically have these
        self.name         = name
        self.employee_id  = employee_id
        self.department   = department
        self.joining_date = date.today()

    # CONCEPT: abstractmethod -- forces every subclass to implement this
    # If a subclass does not implement calculate_pay(), Python raises TypeError
    @abstractmethod
    def calculate_pay(self) -> float:
        """Calculate and return the employee's pay for this period."""
        pass

    @abstractmethod
    def employment_type(self) -> str:
        """Return a string describing the employment type."""
        pass

    # CONCEPT: Concrete method in abstract class
    # This method works for ALL employee types without modification.
    # It calls calculate_pay() and employment_type() which are overridden
    # in each subclass -- this is POLYMORPHISM in action inside the class.
    def generate_payslip(self):
        """Print a formatted payslip. Works for all employee types."""
        pay = self.calculate_pay()
        print("=" * 44)
        print("PAYSLIP")
        print(f"  Name           : {self.name}")
        print(f"  ID             : {self.employee_id}")
        print(f"  Department     : {self.department}")
        print(f"  Type           : {self.employment_type()}")
        print(f"  Joining Date   : {self.joining_date.strftime('%d %b %Y')}")
        print(f"  Gross Pay      : Rs.{pay:,.2f}")
        print("=" * 44)
        return pay

    # CONCEPT: classmethod -- alternative constructor
    # Each subclass overrides this to build from a dictionary
    @classmethod
    def from_dict(cls, data):
        raise NotImplementedError("Subclasses must implement from_dict()")

    # CONCEPT: staticmethod -- utility that belongs to the class
    # but needs no object or class state
    @staticmethod
    def is_valid_department(dept):
        return dept in Employee.VALID_DEPARTMENTS

    def __str__(self):
        return f"[{self.employment_type()}] {self.name} ({self.employee_id}) -- {self.department}"

    def __repr__(self):
        return f"{self.__class__.__name__}(name={self.name!r}, id={self.employee_id!r})"


# ─── FullTimeEmployee ──────────────────────────────────────────────────────
# CONCEPT: Inheritance
# FullTimeEmployee inherits all attributes and methods from Employee,
# then adds its own and overrides calculate_pay().

class FullTimeEmployee(Employee):

    def __init__(self, name, employee_id, department, monthly_salary,
                 performance_rating=3.0):
        # CONCEPT: super() -- initialise the parent class first
        super().__init__(name, employee_id, department)

        # Additional instance variables specific to full-time employees
        self.monthly_salary     = monthly_salary
        self.performance_rating = performance_rating

    # CONCEPT: Method overriding -- replacing the parent's abstract method
    # with a concrete implementation specific to this type
    def calculate_pay(self):
        """Salary plus performance bonus if applicable."""
        pay = self.monthly_salary
        if self.performance_rating >= 4.5:
            pay *= 1.10     # 10% bonus
        elif self.performance_rating >= 4.0:
            pay *= 1.05     # 5% bonus
        return round(pay, 2)

    def employment_type(self):
        return "Full-Time"

    @classmethod
    def from_dict(cls, data):
        return cls(
            name=data["name"],
            employee_id=data["employee_id"],
            department=data["department"],
            monthly_salary=data["monthly_salary"],
            performance_rating=data.get("performance_rating", 3.0),
        )


# ─── ContractEmployee ──────────────────────────────────────────────────────
# CONCEPT: Inheritance -- another subclass of Employee

class ContractEmployee(Employee):

    def __init__(self, name, employee_id, department, hourly_rate, hours_worked):
        super().__init__(name, employee_id, department)
        self.hourly_rate   = hourly_rate
        self.hours_worked  = hours_worked

    # CONCEPT: Method overriding -- different pay calculation for contracts
    def calculate_pay(self):
        return round(self.hourly_rate * self.hours_worked, 2)

    def employment_type(self):
        return "Contract"

    @classmethod
    def from_dict(cls, data):
        return cls(
            name=data["name"],
            employee_id=data["employee_id"],
            department=data["department"],
            hourly_rate=data["hourly_rate"],
            hours_worked=data["hours_worked"],
        )


# ─── Intern ────────────────────────────────────────────────────────────────

class Intern(Employee):

    MAX_STIPEND   = 25000
    MAX_DURATION  = 6   # months

    def __init__(self, name, employee_id, department,
                 monthly_stipend, duration_months):
        super().__init__(name, employee_id, department)

        if monthly_stipend > self.MAX_STIPEND:
            raise ValueError(
                f"Stipend cannot exceed Rs.{self.MAX_STIPEND:,}. Got: Rs.{monthly_stipend:,}"
            )
        if duration_months > self.MAX_DURATION:
            raise ValueError(
                f"Internship cannot exceed {self.MAX_DURATION} months. Got: {duration_months}"
            )

        self.monthly_stipend  = monthly_stipend
        self.duration_months  = duration_months

    # CONCEPT: Method overriding
    def calculate_pay(self):
        return self.monthly_stipend

    def employment_type(self):
        return "Intern"

    @classmethod
    def from_dict(cls, data):
        return cls(
            name=data["name"],
            employee_id=data["employee_id"],
            department=data["department"],
            monthly_stipend=data["monthly_stipend"],
            duration_months=data["duration_months"],
        )


# ─── run_payroll ──────────────────────────────────────────────────────────
# CONCEPT: Polymorphism
# This function does not know or care whether it receives a FullTimeEmployee,
# ContractEmployee, or Intern. It treats them ALL identically through the
# shared Employee interface. When generate_payslip() is called, Python
# automatically calls the right calculate_pay() for each type.

def run_payroll(employees):
    """Process payroll for a mixed list of any Employee subclass objects."""
    print("\n" + "=" * 44)
    print("PAYROLL RUN")
    print("=" * 44)

    totals = {}
    grand_total = 0.0

    for employee in employees:
        pay  = employee.generate_payslip()
        emp_type = employee.employment_type()

        if emp_type not in totals:
            totals[emp_type] = {"count": 0, "amount": 0.0}
        totals[emp_type]["count"]  += 1
        totals[emp_type]["amount"] += pay
        grand_total += pay

    print("\nPAYROLL SUMMARY")
    print("-" * 44)
    for emp_type, data in totals.items():
        count = data["count"]
        label = "employee" if count == 1 else "employees"
        print(f"  {emp_type:<12}: {count} {label:<10} | Rs.{data['amount']:>10,.2f}")
    print("-" * 44)
    print(f"  {'Grand Total':<24} | Rs.{grand_total:>10,.2f}")


# ─── Demo ──────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    employees = [
        FullTimeEmployee("Vikram Nair",  "E001", "Engineering",
                         monthly_salary=95000, performance_rating=4.7),
        FullTimeEmployee("Ananya Shah",  "E002", "Product",
                         monthly_salary=85000, performance_rating=3.8),
        ContractEmployee("Suresh Kumar", "C001", "Sales",
                         hourly_rate=800, hours_worked=160),
        Intern("Divya Menon",            "I001", "Engineering",
               monthly_stipend=20000, duration_months=3),
    ]

    run_payroll(employees)

    # Demonstrate that Employee cannot be instantiated directly
    try:
        e = Employee("Test", "T001", "Engineering")
    except TypeError as err:
        print(f"\nAbstraction enforced: {err}")

    # Demonstrate @staticmethod
    print(f"\n'Engineering' valid: {Employee.is_valid_department('Engineering')}")
    print(f"'Accounts' valid: {Employee.is_valid_department('Accounts')}")

    # Demonstrate @classmethod alternative constructor
    data = {"name": "Meera Joshi", "employee_id": "E003",
            "department": "HR", "monthly_salary": 70000}
    emp = FullTimeEmployee.from_dict(data)
    print(f"\nCreated from dict: {emp}")
```

---

## Solution 3 -- Online Food Ordering System

```python
import json
from datetime import datetime


# ─── Custom Exception ──────────────────────────────────────────────────────

class OrderError(Exception):
    pass


# ─── MenuItem ──────────────────────────────────────────────────────────────

class MenuItem:

    VALID_CATEGORIES = {"Starter", "Main", "Dessert", "Beverage"}

    def __init__(self, name, price, category, is_available=True):
        # CONCEPT: Encapsulation -- price and category go through setters
        self.name         = name
        self.price        = price        # calls the setter below
        self.category     = category     # calls the setter below
        self.is_available = is_available

    # CONCEPT: Property with validation -- encapsulates the price field
    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if not isinstance(value, (int, float)) or value <= 0:
            raise ValueError(f"Price must be a positive number. Got: {value}")
        self._price = float(value)

    @property
    def category(self):
        return self._category

    @category.setter
    def category(self, value):
        if value not in self.VALID_CATEGORIES:
            raise ValueError(
                f"'{value}' is not a valid category. "
                f"Choose from: {self.VALID_CATEGORIES}"
            )
        self._category = value

    # CONCEPT: Dunder methods -- makes MenuItem feel like a native Python type
    def __str__(self):
        return f"{self.name} -- Rs.{self._price:.0f} ({self._category})"

    def __repr__(self):
        return f"MenuItem(name={self.name!r}, price={self._price}, category={self._category!r})"

    def __eq__(self, other):
        """Two items are equal if same name and price."""
        if not isinstance(other, MenuItem):
            return NotImplemented
        return self.name == other.name and self._price == other._price

    def __lt__(self, other):
        """Order items by price -- enables sorted(menu_list)."""
        if not isinstance(other, MenuItem):
            return NotImplemented
        return self._price < other._price

    def __hash__(self):
        # Required when __eq__ is defined and we want to use items in sets/dicts
        return hash((self.name, self._price))


# ─── CartItem ──────────────────────────────────────────────────────────────
# CONCEPT: Composition -- CartItem HAS-A MenuItem

class CartItem:

    def __init__(self, item, quantity=1):
        if not isinstance(item, MenuItem):
            raise TypeError("item must be a MenuItem instance.")
        self.item     = item    # composition: holds a reference to a MenuItem
        self.quantity = quantity  # calls the setter

    @property
    def quantity(self):
        return self._quantity

    @quantity.setter
    def quantity(self, value):
        if not isinstance(value, int) or value < 1:
            raise ValueError("Quantity must be a positive integer.")
        self._quantity = value

    @property
    def subtotal(self):
        """Total price for this item x quantity."""
        return self.item.price * self._quantity

    def __str__(self):
        return (f"{self.item.name} x{self._quantity} "
                f"= Rs.{self.subtotal:.0f}")


# ─── Cart ──────────────────────────────────────────────────────────────────
# CONCEPT: Composition -- Cart HAS-A list of CartItems

class Cart:

    def __init__(self, customer_name, restaurant_name):
        self.customer_name   = customer_name
        self.restaurant_name = restaurant_name
        self._items          = []    # CONCEPT: Encapsulation -- private list

    def _find_item(self, menu_item):
        """Return the CartItem for a given MenuItem, or None."""
        for cart_item in self._items:
            if cart_item.item == menu_item:
                return cart_item
        return None

    def add_item(self, menu_item, quantity=1):
        """Add item to cart. If already present, increase quantity."""
        if not menu_item.is_available:
            raise OrderError(f"'{menu_item.name}' is not currently available.")
        existing = self._find_item(menu_item)
        if existing:
            existing.quantity += quantity
        else:
            self._items.append(CartItem(menu_item, quantity))

    def remove_item(self, menu_item):
        """Remove a MenuItem from the cart entirely."""
        cart_item = self._find_item(menu_item)
        if not cart_item:
            raise OrderError(f"'{menu_item.name}' is not in the cart.")
        self._items.remove(cart_item)

    def update_quantity(self, menu_item, quantity):
        """Update the quantity for an existing item."""
        cart_item = self._find_item(menu_item)
        if not cart_item:
            raise OrderError(f"'{menu_item.name}' is not in the cart.")
        cart_item.quantity = quantity

    def clear(self):
        self._items.clear()

    @property
    def total(self):
        return sum(ci.subtotal for ci in self._items)

    @property
    def item_count(self):
        return sum(ci.quantity for ci in self._items)

    @property
    def is_empty(self):
        return len(self._items) == 0

    # CONCEPT: Dunder methods

    def __len__(self):
        """Number of distinct items in the cart."""
        return len(self._items)

    def __str__(self):
        if self.is_empty:
            return f"Cart for {self.customer_name} (empty)"
        lines = [f"Cart -- {self.customer_name} @ {self.restaurant_name}"]
        lines.append("-" * 40)
        for ci in self._items:
            lines.append(f"  {ci}")
        lines.append("-" * 40)
        lines.append(f"  Total: Rs.{self.total:.2f}")
        return "\n".join(lines)

    def __add__(self, other):
        """Merge two carts. Combines quantities for shared items."""
        if not isinstance(other, Cart):
            return NotImplemented
        merged = Cart(
            f"{self.customer_name} + {other.customer_name}",
            self.restaurant_name
        )
        for ci in self._items:
            merged.add_item(ci.item, ci.quantity)
        for ci in other._items:
            merged.add_item(ci.item, ci.quantity)
        return merged

    # CONCEPT: Context manager protocol
    # __enter__ and __exit__ make Cart usable with the 'with' statement

    def __enter__(self):
        """Enter context -- return the cart itself."""
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        """Exit context -- print summary, clear if empty."""
        if not self.is_empty:
            print(f"\nCart session ended for {self.customer_name}.")
            print(f"  Items: {self.item_count}  |  Total: Rs.{self.total:.2f}")
        else:
            print(f"Cart for {self.customer_name} is empty -- nothing to summarise.")
        return False   # do not suppress exceptions


# ─── Order ─────────────────────────────────────────────────────────────────

class Order:

    VALID_STATUSES = [
        "Placed", "Confirmed", "Preparing",
        "Out for Delivery", "Delivered", "Cancelled"
    ]
    _order_counter = 0

    def __init__(self, customer_name, restaurant_name, items, delivery_address):
        Order._order_counter += 1
        self.order_id         = f"ORD-{Order._order_counter:04d}"
        self.customer_name    = customer_name
        self.restaurant_name  = restaurant_name
        self.items            = list(items)   # copy, not reference
        self.delivery_address = delivery_address
        self.order_time       = datetime.now()
        self.status           = "Placed"

    @property
    def subtotal(self):
        return sum(ci.subtotal for ci in self.items)

    @property
    def delivery_fee(self):
        return 0.0 if self.subtotal >= 300 else 40.0

    @property
    def gst(self):
        return round(self.subtotal * 0.05, 2)

    @property
    def total(self):
        return round(self.subtotal + self.delivery_fee + self.gst, 2)

    def update_status(self, new_status):
        if new_status not in self.VALID_STATUSES:
            raise ValueError(
                f"'{new_status}' is not a valid status. "
                f"Choose from: {self.VALID_STATUSES}"
            )
        self.status = new_status
        print(f"Order {self.order_id} status updated to: {self.status}")

    # CONCEPT: classmethod as alternative constructor
    @classmethod
    def from_cart(cls, cart, delivery_address):
        """Create an Order from a Cart object."""
        if cart.is_empty:
            raise OrderError("Cannot create an order from an empty cart.")
        return cls(
            customer_name=cart.customer_name,
            restaurant_name=cart.restaurant_name,
            items=cart._items,
            delivery_address=delivery_address,
        )

    def __str__(self):
        time_str = self.order_time.strftime("%d %b %Y, %H:%M")
        lines = [
            "=" * 45,
            "ORDER RECEIPT",
            f"  Order ID    : {self.order_id}",
            f"  Customer    : {self.customer_name}",
            f"  Restaurant  : {self.restaurant_name}",
            f"  Time        : {time_str}",
            f"  Deliver to  : {self.delivery_address}",
            "-" * 45,
        ]
        for ci in self.items:
            lines.append(f"  {ci.item.name:<20} x{ci.quantity:<2}  Rs.{ci.subtotal:.0f}")
        lines.extend([
            "-" * 45,
            f"  {'Subtotal':<28}: Rs.{self.subtotal:.2f}",
            f"  {'Delivery Fee':<28}: Rs.{self.delivery_fee:.2f}",
            f"  {'GST (5%)':<28}: Rs.{self.gst:.2f}",
            f"  {'Total':<28}: Rs.{self.total:.2f}",
            "-" * 45,
            f"  Status: {self.status}",
            "=" * 45,
        ])
        return "\n".join(lines)


# ─── Demo ──────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    dosa   = MenuItem("Masala Dosa",    80,  "Main")
    coffee = MenuItem("Filter Coffee",  40,  "Beverage")
    idli   = MenuItem("Idli Sambar",    60,  "Main")
    gulab  = MenuItem("Gulab Jamun",    50,  "Dessert")
    vada   = MenuItem("Medu Vada",      45,  "Starter")

    # CONCEPT: Context manager -- 'with' ensures clean entry/exit
    with Cart("Aarav Sharma", "Udupi Palace") as cart:
        cart.add_item(dosa, 2)
        cart.add_item(coffee, 1)
        cart.add_item(idli, 1)
        cart.add_item(dosa, 1)   # increases dosa to 3, no duplicate

        print(f"Distinct items: {len(cart)}")       # 3
        print(f"Total items   : {cart.item_count}") # 5
        print(f"Cart total    : Rs.{cart.total}")   # 340

        order = Order.from_cart(cart, "42, Koregaon Park, Pune")
        print(order)

        order.update_status("Confirmed")
        order.update_status("Preparing")

    # Sort menu by price (uses __lt__)
    menu = [dosa, coffee, idli, gulab, vada]
    print("\nMenu sorted by price:")
    for item in sorted(menu):
        print(f"  {item}")

    # Combine two carts (uses __add__)
    cart2 = Cart("Priya Patel", "Udupi Palace")
    cart2.add_item(gulab, 2)
    cart2.add_item(vada, 1)

    combined = cart + cart2
    print(f"\nCombined cart total: Rs.{combined.total}")
```

---

## Solution 4 -- Student Grade Management System

```python
import json
from datetime import datetime
from dataclasses import dataclass, field
from abc import ABC


# ─── Mixins ────────────────────────────────────────────────────────────────
# CONCEPT: Mixins -- small, focused, reusable classes that add a specific
# piece of behaviour. They are not meant to be used standalone.
# This is a form of INHERITANCE used for composition of behaviours.

class SerializableMixin:
    """Adds JSON serialisation to any class."""

    def to_dict(self):
        """Return a dict of all non-private attributes."""
        return {
            k: v for k, v in self.__dict__.items()
            if not k.startswith("_")
        }

    def to_json(self):
        """Return a JSON string representation."""
        def default_handler(obj):
            if isinstance(obj, datetime):
                return obj.isoformat()
            if hasattr(obj, "to_dict"):
                return obj.to_dict()
            return str(obj)

        return json.dumps(self.to_dict(), indent=2, default=default_handler)


class TimestampMixin:
    """Adds creation and update timestamps to any class."""

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.created_at = datetime.now()
        self.updated_at = datetime.now()

    def touch(self):
        """Update the updated_at timestamp."""
        self.updated_at = datetime.now()


# ─── GradeRecord ───────────────────────────────────────────────────────────
# CONCEPT: Dataclass -- reduces boilerplate for data-holding classes
# @dataclass auto-generates __init__, __repr__, __eq__
# With order=True it also generates __lt__, __le__, __gt__, __ge__

@dataclass(order=True)
class GradeRecord:
    """A single grade entry for one subject."""

    # sort_key is used for ordering -- we want to sort by percentage
    # field with init=False means it is not a constructor parameter
    sort_index:      float = field(init=False, repr=False)

    subject:         str
    marks_obtained:  float
    max_marks:       float  = 100.0
    exam_type:       str    = "Final"

    def __post_init__(self):
        """Called after __init__ -- compute sort_index."""
        self.sort_index = self.percentage

    @property
    def percentage(self):
        return round(self.marks_obtained / self.max_marks * 100, 2)

    @property
    def grade_letter(self):
        p = self.percentage
        if p >= 90: return "A+"
        if p >= 80: return "A"
        if p >= 70: return "B"
        if p >= 60: return "C"
        if p >= 50: return "D"
        return "F"


# ─── Student ───────────────────────────────────────────────────────────────
# CONCEPT: Multiple inheritance -- Student inherits from both mixins
# CONCEPT: __slots__ -- replaces __dict__ with fixed memory slots
# This reduces memory per object significantly for large numbers of students

class Student(SerializableMixin, TimestampMixin):

    # CONCEPT: __slots__ -- explicit list of allowed attributes
    # This reduces memory usage and slightly speeds up attribute access
    __slots__ = (
        "name", "student_id", "department", "year",
        "_grade_records", "created_at", "updated_at"
    )

    # Class-level counter for auto-generating IDs
    _counter = 0

    def __init__(self, name, department, year):
        # CONCEPT: Cooperative multiple inheritance using super()
        # TimestampMixin.__init__ is called through the MRO chain
        super().__init__()

        Student._counter += 1
        year_str = datetime.now().year

        self.name           = name
        self.student_id     = f"STU-{year_str}-{Student._counter:04d}"
        self.department     = department
        self.year           = year
        self._grade_records = []    # CONCEPT: Encapsulation -- private list

    def add_grade(self, grade_record):
        """Add a GradeRecord. Raises TypeError if wrong type."""
        if not isinstance(grade_record, GradeRecord):
            raise TypeError(
                f"Expected GradeRecord, got {type(grade_record).__name__}"
            )
        self._grade_records.append(grade_record)
        self.touch()   # update timestamp via TimestampMixin

    @property
    def grades(self):
        """Return a copy -- callers cannot modify the internal list."""
        return list(self._grade_records)

    @property
    def cgpa(self):
        """Average percentage on a 10-point scale."""
        if not self._grade_records:
            return 0.0
        avg_pct = sum(g.percentage for g in self._grade_records) / len(self._grade_records)
        return round(avg_pct / 10, 2)

    @property
    def top_subject(self):
        """GradeRecord with the highest percentage."""
        return max(self._grade_records, key=lambda g: g.percentage) \
               if self._grade_records else None

    @property
    def failed_subjects(self):
        """List of GradeRecords where grade_letter is F."""
        return [g for g in self._grade_records if g.grade_letter == "F"]

    def generate_transcript(self):
        """Print a formatted academic transcript."""
        print("\n" + "=" * 46)
        print("ACADEMIC TRANSCRIPT")
        print(f"  Name       : {self.name}")
        print(f"  ID         : {self.student_id}")
        print(f"  Department : {self.department}")
        print(f"  Year       : {self.year}")
        print(f"  CGPA       : {self.cgpa:.2f} / 10.00")
        print("-" * 46)
        print(f"  {'Subject':<24} {'Marks':>7}  {'Grade'}")
        print("-" * 46)
        for g in self._grade_records:
            marks_str = f"{g.marks_obtained:.0f}/{g.max_marks:.0f}"
            print(f"  {g.subject:<24} {marks_str:>7}  {g.grade_letter:<3}  ({g.exam_type})")
        print("-" * 46)
        if self.top_subject:
            print(f"  Top Subject: {self.top_subject.subject} "
                  f"({self.top_subject.percentage:.1f}%)")
        if self.failed_subjects:
            failed = ", ".join(g.subject for g in self.failed_subjects)
            print(f"  Failed     : {failed}")
        print("=" * 46)

    # CONCEPT: Dunder methods for natural Python behaviour

    def __str__(self):
        return (f"Student({self.name}, ID: {self.student_id}, "
                f"{self.department}, CGPA: {self.cgpa})")

    def __repr__(self):
        return f"Student(name={self.name!r}, id={self.student_id!r})"

    def __eq__(self, other):
        """Two students are equal if they have the same student_id."""
        if not isinstance(other, Student):
            return NotImplemented
        return self.student_id == other.student_id

    def __lt__(self, other):
        """Students are ordered by CGPA -- enables sorted(students)."""
        if not isinstance(other, Student):
            return NotImplemented
        return self.cgpa < other.cgpa

    def __hash__(self):
        return hash(self.student_id)

    def to_dict(self):
        """Override SerializableMixin.to_dict() to handle __slots__."""
        return {
            "name":       self.name,
            "student_id": self.student_id,
            "department": self.department,
            "year":       self.year,
            "cgpa":       self.cgpa,
            "grades":     [
                {
                    "subject":   g.subject,
                    "marks":     g.marks_obtained,
                    "max":       g.max_marks,
                    "grade":     g.grade_letter,
                    "exam_type": g.exam_type,
                }
                for g in self._grade_records
            ],
        }


# ─── Course ────────────────────────────────────────────────────────────────
# CONCEPT: Composition -- Course HAS-A list of Students

class Course:

    def __init__(self, course_code, course_name, credits, instructor):
        self.course_code        = course_code
        self.course_name        = course_name
        self.credits            = credits
        self.instructor         = instructor
        self._enrolled_students = []

    def enroll(self, student):
        if student not in self._enrolled_students:
            self._enrolled_students.append(student)

    def unenroll(self, student):
        if student in self._enrolled_students:
            self._enrolled_students.remove(student)

    @property
    def average_cgpa(self):
        if not self._enrolled_students:
            return 0.0
        return round(
            sum(s.cgpa for s in self._enrolled_students) / len(self._enrolled_students),
            2
        )

    @property
    def top_student(self):
        return max(self._enrolled_students, key=lambda s: s.cgpa) \
               if self._enrolled_students else None

    def __len__(self):
        return len(self._enrolled_students)

    def __contains__(self, student):
        """Supports: student in course"""
        return student in self._enrolled_students

    def __str__(self):
        return f"{self.course_code}: {self.course_name} ({len(self)} students)"


# ─── Department ────────────────────────────────────────────────────────────
# CONCEPT: Composition -- Department HAS-A dict of Courses and Students

class Department:

    def __init__(self, name, head):
        self.name     = name
        self.head     = head
        self._courses  = {}    # course_code: Course
        self._students = {}    # student_id: Student

    def add_course(self, course):
        self._courses[course.course_code] = course

    def add_student(self, student):
        self._students[student.student_id] = student

    def generate_report(self):
        """Print a complete department-wide report."""
        students = list(self._students.values())

        if not students:
            print(f"\n{self.name}: No students registered.")
            return

        avg_cgpa     = round(sum(s.cgpa for s in students) / len(students), 2)
        topper       = max(students, key=lambda s: s.cgpa)
        at_risk      = [s for s in students if s.cgpa < 5.0]

        # Grade distribution across all grade records
        all_grades   = [g.grade_letter for s in students for g in s.grades]
        distribution = {}
        for letter in ["A+", "A", "B", "C", "D", "F"]:
            distribution[letter] = all_grades.count(letter)

        print(f"\n{'=' * 50}")
        print(f"  DEPARTMENT REPORT: {self.name.upper()}")
        print(f"  Head: {self.head}")
        print(f"{'=' * 50}")
        print(f"  Total Students : {len(students)}")
        print(f"  Total Courses  : {len(self._courses)}")
        print(f"  Avg CGPA       : {avg_cgpa} / 10")
        print(f"  Topper         : {topper.name} ({topper.cgpa})")
        print(f"  At Risk (<5.0) : {len(at_risk)}")
        print(f"\n  Grade Distribution:")
        for letter, count in distribution.items():
            bar = "#" * count
            print(f"    {letter:>2} : {bar:<20} ({count})")
        print(f"{'=' * 50}")


# ─── Demo ──────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    cs_dept = Department("Computer Science", "Dr. Ramesh Kumar")

    python_course = Course("CS101", "Python Programming", credits=4,
                           instructor="Prof. Priya Nair")
    ml_course     = Course("CS301", "Machine Learning",   credits=4,
                           instructor="Prof. Vikram Sharma")

    cs_dept.add_course(python_course)
    cs_dept.add_course(ml_course)

    s1 = Student("Aarav Sharma", "Computer Science", year=2)
    s2 = Student("Priya Patel",  "Computer Science", year=2)
    s3 = Student("Rohan Verma",  "Computer Science", year=3)

    s1.add_grade(GradeRecord("Python Programming", 88))
    s1.add_grade(GradeRecord("Machine Learning",   91))
    s1.add_grade(GradeRecord("Data Structures",    79, exam_type="Midterm"))

    s2.add_grade(GradeRecord("Python Programming", 95))
    s2.add_grade(GradeRecord("Machine Learning",   88))
    s2.add_grade(GradeRecord("Data Structures",    72))

    s3.add_grade(GradeRecord("Python Programming", 45))   # failed
    s3.add_grade(GradeRecord("Data Structures",    58))

    python_course.enroll(s1)
    python_course.enroll(s2)
    python_course.enroll(s3)
    ml_course.enroll(s1)
    ml_course.enroll(s2)

    cs_dept.add_student(s1)
    cs_dept.add_student(s2)
    cs_dept.add_student(s3)

    # Transcript
    s1.generate_transcript()

    # Serialisation (CONCEPT: Polymorphism through mixin)
    # to_json() works on any class that uses SerializableMixin
    print("\nStudent JSON:")
    print(s1.to_json())

    # Sorting students by CGPA (uses __lt__)
    all_students = [s1, s2, s3]
    print("\nRanking (highest CGPA first):")
    for rank, student in enumerate(sorted(all_students, reverse=True), 1):
        print(f"  {rank}. {student.name} -- CGPA: {student.cgpa}")

    # __contains__ on Course
    print(f"\nAarav in Python course : {s1 in python_course}")
    print(f"Rohan in ML course     : {s3 in ml_course}")

    # Department report
    cs_dept.generate_report()
```

---

## Concept Coverage Summary

| Problem | Encapsulation | Inheritance | Polymorphism | Abstraction | Other |
|---|---|---|---|---|---|
| 1 -- Library | Properties, private attrs, custom exceptions | No | No | No | Class variables, `__str__`, `__repr__` |
| 2 -- Payroll | Property validation | 3 subclasses, `super()` | `run_payroll()` handles all types | ABC, `@abstractmethod` | `@classmethod`, `@staticmethod` |
| 3 -- Food | Properties with validation, private `_items` | Context manager protocol | Sorted via `__lt__` | No | Composition, dunder suite, `__add__` |
| 4 -- Grades | `__slots__`, private lists, properties | Mixins, multiple inheritance | `to_json()` via mixin works on any class | No | Dataclass, `@dataclass(order=True)`, MRO |

All four pillars appear across the four problems. No single problem uses
all four -- real systems rarely do. The skill is knowing which pillar
solves which design problem.

---

*Codeverra OOP Assignment | learn.codeverra.com*
