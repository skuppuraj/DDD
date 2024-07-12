### Aggregates and Repositories in Domain Driven Design (DDD)

#### Aggregates
An **aggregate** is a cluster of related objects (entities and value objects) that are treated as a single unit. Aggregates help maintain consistency and enforce invariants within the boundaries they define. Each aggregate has a **root entity** called the **aggregate root**, which is the only entity accessible from outside the aggregate. The aggregate root ensures the integrity and consistency of the aggregate.

### Example: Online Bookstore

Let's expand our online bookstore example to illustrate aggregates.

#### Aggregate: Order
An `Order` aggregate might include an `Order` entity, several `OrderLine` entities, and possibly some value objects like `Address` and `PaymentDetails`.

```python
class Order:
    def __init__(self, order_id, customer, shipping_address):
        self.order_id = order_id  # Aggregate Root ID
        self.customer = customer
        self.shipping_address = shipping_address
        self.order_lines = []
        self.status = 'NEW'

    def add_order_line(self, book, quantity):
        self.order_lines.append(OrderLine(book, quantity))

    def calculate_total_price(self):
        return sum(line.calculate_price() for line in self.order_lines)

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

In this example, `Order` is the aggregate root. `OrderLine` entities and the `Address` value object are part of the `Order` aggregate, but they cannot be accessed directly from outside the `Order` aggregate. 

### Repositories

A **repository** is a mechanism for encapsulating storage, retrieval, and search behavior, which emulates a collection of aggregates. Repositories provide an abstraction over data storage, making the persistence of objects transparent to the client code.

### Example: Order Repository

Let's create a repository for the `Order` aggregate:

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

### How Aggregates and Repositories Work Together

1. **Creating an Order**:
   - A new `Order` is created and populated with order lines.
   - The `OrderRepository` is used to save the `Order` aggregate.

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

2. **Retrieving an Order**:
   - Retrieve an order by its ID using the `OrderRepository`.

```python
retrieved_order = order_repository.get(order_id=1001)
print(f"Order ID: {retrieved_order.order_id}, Total Price: {retrieved_order.calculate_total_price()}")
```

3. **Finding Orders by Customer**:
   - Use the repository to find all orders placed by a specific customer.

```python
customer_orders = order_repository.find_by_customer(customer)
for order in customer_orders:
    print(f"Order ID: {order.order_id}, Total Price: {order.calculate_total_price()}")
```

### Benefits of Using Aggregates and Repositories

- **Consistency**: Aggregates ensure that changes within their boundaries maintain integrity and consistency.
- **Encapsulation**: Aggregates encapsulate related entities and value objects, reducing complexity.
- **Abstraction**: Repositories provide a clear and simple way to interact with data storage, hiding the complexities of the underlying persistence mechanism.

By organizing your domain model using aggregates and repositories, you create a robust, maintainable, and scalable system that aligns closely with the business domain.
