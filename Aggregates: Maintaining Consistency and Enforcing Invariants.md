### Aggregates: Maintaining Consistency and Enforcing Invariants

Aggregates are crucial in Domain Driven Design (DDD) because they help maintain consistency and enforce invariants within their boundaries. Here's a closer look at how they achieve this.

#### What are Invariants?
Invariants are rules or conditions that must always be true for the system to be in a valid state. For example, in an online bookstore:
- An order's total price must be the sum of its order lines' prices.
- An order cannot have a negative quantity of books.

### How Aggregates Maintain Consistency and Enforce Invariants

1. **Single Entry Point**:
   - The aggregate root is the only entry point for accessing and modifying the aggregate's internal state. This ensures that all business rules and invariants are enforced in one place.

2. **Transactional Consistency**:
   - Operations on an aggregate are treated as a single unit of work (transaction). This ensures that all changes within the aggregate are consistent and complete before being committed.

3. **Encapsulation**:
   - By encapsulating related entities and value objects, aggregates prevent external objects from directly manipulating their internal state, thus preserving their integrity.

### Example: Online Bookstore

#### Aggregate: Order

Let's revisit the `Order` aggregate with a focus on maintaining consistency and enforcing invariants.

```python
class Order:
    def __init__(self, order_id, customer, shipping_address):
        self.order_id = order_id  # Aggregate Root ID
        self.customer = customer
        self.shipping_address = shipping_address
        self.order_lines = []
        self.status = 'NEW'
        self.total_price = 0

    def add_order_line(self, book, quantity):
        if quantity <= 0:
            raise ValueError("Quantity must be greater than zero")
        
        order_line = OrderLine(book, quantity)
        self.order_lines.append(order_line)
        self._update_total_price()

    def remove_order_line(self, book_id):
        self.order_lines = [line for line in self.order_lines if line.book.book_id != book_id]
        self._update_total_price()

    def _update_total_price(self):
        self.total_price = sum(line.calculate_price() for line in self.order_lines)
        if self.total_price < 0:
            raise ValueError("Total price cannot be negative")

    def calculate_total_price(self):
        return self.total_price

class OrderLine:
    def __init__(self, book, quantity):
        self.book = book
        self.quantity = quantity

    def calculate_price(self):
        return self.book.price * self.quantity

class Address:
    def __init__(self, street, city, state, zip_code):
        self.street = street
        self.city = city
        self.state = state
        self.zip_code = zip_code
```

### Enforcing Invariants

1. **Adding an Order Line**:
   - When adding an order line, we check that the quantity is positive. This enforces the invariant that an order cannot have a negative quantity of books.
   - We update the total price after adding the order line and check that the total price is not negative.

```python
# Create customer and address
customer = Customer(customer_id=1, name="Alice", address="123 Main St")
address = Address(street="123 Main St", city="Springfield", state="IL", zip_code="62701")

# Create an order
order = Order(order_id=1001, customer=customer, shipping_address=address)
order.add_order_line(book=Book(book_id=1, title="DDD", author="Eric Evans", price=30), quantity=2)

# Save the order using the repository
order_repository = OrderRepository()
order_repository.save(order)
```

2. **Removing an Order Line**:
   - When removing an order line, we update the total price and ensure it is not negative.

```python
# Remove an order line
order.remove_order_line(book_id=1)
```

### Repositories: Accessing Aggregates

Repositories provide an abstraction for accessing aggregates, ensuring that all operations on aggregates go through the aggregate root, maintaining consistency and enforcing invariants.

```python
class OrderRepository:
    def __init__(self):
        self.orders = {}

    def save(self, order):
        self.orders[order.order_id] = order

    def get(self, order_id):
        return self.orders.get(order_id)

    def find_by_customer(self, customer):
        return [order for order in self.orders.values() if order.customer == customer]
```

### Benefits

- **Consistency**: Aggregates ensure that all changes within their boundaries are consistent and valid according to business rules.
- **Integrity**: By encapsulating related entities and value objects, aggregates protect their internal state from invalid manipulation.
- **Simplicity**: Aggregates provide a clear structure for the domain model, making it easier to understand and maintain.

By using aggregates and repositories, you create a robust and reliable system that adheres to business rules and maintains data integrity.
