# SQL Practice Setup

This section defines a small transactional schema that will be reused across all SQL practice (joins, CTEs, recursive CTEs, window functions, analytics).

The database is assumed to be PostgreSQL (e.g., running in Docker).

---

## Schema Overview

Tables:

- `regions` – region hierarchy (for recursive CTEs)
- `customers` – customer master data
- `products` – product catalog
- `orders` – order header
- `order_items` – order line items
- `payments` – payment facts

This schema is intentionally small but rich enough for:

- Multi-table joins
- Simple and recursive CTEs
- Window functions
- Analytics / business-style queries

---

## DDL: Create Tables

```sql
-- Drop if re-running
DROP TABLE IF EXISTS payments;
DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS regions;

-- Regions (for hierarchy / recursive CTE later)
CREATE TABLE regions (
    region_id SERIAL PRIMARY KEY,
    region_name VARCHAR(50),
    parent_region_id INT REFERENCES regions(region_id)
);

-- Customers
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    region_id INT REFERENCES regions(region_id),
    signup_date DATE
);

-- Products
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

-- Orders (header)
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date DATE,
    status VARCHAR(20) -- e.g. 'CREATED', 'PAID', 'CANCELLED'
);

-- Order items (detail)
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT,
    unit_price DECIMAL(10,2)
);

-- Payments
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    payment_date DATE,
    amount DECIMAL(10,2),
    payment_method VARCHAR(20) -- e.g. 'CARD', 'UPI', 'CASH'
);
```

---

## DML: Sample Data

```sql
-- Regions (hierarchy)
INSERT INTO regions (region_id, region_name, parent_region_id) VALUES
(1, 'India', NULL),
(2, 'South', 1),
(3, 'North', 1),
(4, 'East', 1),
(5, 'West', 1),
(6, 'Tamil Nadu', 2),
(7, 'Karnataka', 2);

-- Customers
INSERT INTO customers (customer_id, customer_name, email, region_id, signup_date) VALUES
(1, 'Ravi Kumar', 'ravi@example.com', 6, '2025-12-01'),
(2, 'Anita Sharma', 'anita@example.com', 3, '2025-12-10'),
(3, 'Mohamed Iqbal', 'iqbal@example.com', 7, '2026-01-05'),
(4, 'Priya Verma', 'priya@example.com', 6, '2025-11-20'),
(5, 'Rahul Singh', 'rahul@example.com', 5, '2026-01-02');

-- Products
INSERT INTO products (product_id, product_name, category, price) VALUES
(1, 'Laptop Pro 15"', 'Electronics', 1200.00),
(2, 'Mechanical Keyboard', 'Electronics', 120.00),
(3, 'Data Engineering Book', 'Books', 45.00),
(4, 'Winter Jacket', 'Clothing', 90.00),
(5, 'Running Shoes', 'Footwear', 75.00);

-- Orders
INSERT INTO orders (order_id, customer_id, order_date, status) VALUES
(1, 1, '2026-01-05', 'PAID'),
(2, 1, '2026-01-15', 'PAID'),
(3, 2, '2026-01-07', 'PAID'),
(4, 3, '2026-01-09', 'CREATED'),
(5, 3, '2026-01-20', 'PAID'),
(6, 4, '2026-01-11', 'CANCELLED'),
(7, 5, '2026-01-12', 'PAID');

-- Order items
INSERT INTO order_items (order_item_id, order_id, product_id, quantity, unit_price) VALUES
(1, 1, 1, 1, 1200.00),
(2, 1, 3, 2, 45.00),
(3, 2, 2, 1, 120.00),
(4, 3, 3, 1, 45.00),
(5, 3, 4, 1, 90.00),
(6, 4, 5, 2, 75.00),
(7, 5, 1, 1, 1200.00),
(8, 5, 5, 1, 75.00),
(9, 6, 4, 1, 90.00),
(10, 7, 2, 1, 120.00);

-- Payments
INSERT INTO payments (payment_id, order_id, payment_date, amount, payment_method) VALUES
(1, 1, '2026-01-05', 1290.00, 'CARD'),      -- 1200 + 2*45
(2, 2, '2026-01-15', 120.00, 'UPI'),
(3, 3, '2026-01-07', 135.00, 'CARD'),       -- 45 + 90
(4, 5, '2026-01-20', 1275.00, 'CARD'),      -- 1200 + 75
(5, 7, '2026-01-12', 120.00, 'CASH');
```

---