# E-commerce Database Design & SQL Optimization

This README documents the complete solution for the **Practical Database Design** assignment related to designing an e-commerce database, inserting large-scale data, writing analytical queries, and optimizing them.

---

## 1. Database Schema Design

The database is designed following **normalization rules**, **relational integrity**, and **backend best practices**.

### 1.1 Customers Table

```sql
CREATE TABLE customers (
  customer_id BIGSERIAL PRIMARY KEY,
  full_name VARCHAR(150) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.2 Categories Table

```sql
CREATE TABLE categories (
  category_id BIGSERIAL PRIMARY KEY,
  category_name VARCHAR(100) NOT NULL
);
```

### 1.3 Products Table

```sql
CREATE TABLE products (
  product_id BIGSERIAL PRIMARY KEY,
  product_name VARCHAR(200) NOT NULL,
  category_id BIGINT NOT NULL REFERENCES categories(category_id),
  price DECIMAL(10,2) NOT NULL CHECK (price > 0),
  stock_quantity INT NOT NULL CHECK (stock_quantity >= 0)
);
```

### 1.4 Orders Table

```sql
CREATE TABLE orders (
  order_id BIGSERIAL PRIMARY KEY,
  customer_id BIGINT NOT NULL REFERENCES customers(customer_id),
  order_date TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.5 Order Items Table

```sql
CREATE TABLE order_items (
  order_item_id BIGSERIAL PRIMARY KEY,
  order_id BIGINT NOT NULL REFERENCES orders(order_id),
  product_id BIGINT NOT NULL REFERENCES products(product_id),
  quantity INT NOT NULL CHECK (quantity > 0),
  price_at_time DECIMAL(10,2) NOT NULL
);
```

---

## 2. Data Insertion Functions

### 2.1 Insert Products (~10,000 rows)

```sql
INSERT INTO products (product_name, category_id, price, stock_quantity)
SELECT
  'Product ' || generate_series,
  (random() * 9 + 1)::INT,
  round((random() * 100)::numeric, 2),
  (random() * 100)::INT
FROM generate_series(1, 10000);
```

### 2.2 Insert Customers (~100,000 rows)

```sql
INSERT INTO customers (full_name, email)
SELECT
  'Customer ' || generate_series,
  'customer' || generate_series || '@mail.com'
FROM generate_series(1, 100000);
```

### 2.3 Insert Categories (~100 rows)

```sql
INSERT INTO categories (category_name)
SELECT 'Category ' || generate_series
FROM generate_series(1, 100);
```

### 2.4 Insert Orders & Order Items (~100,000 rows)

```sql
INSERT INTO orders (customer_id)
SELECT (random() * 99999 + 1)::INT
FROM generate_series(1, 100000);
```

```sql
INSERT INTO order_items (order_id, product_id, quantity, price_at_time)
SELECT
  (random() * 99999 + 1)::INT,
  (random() * 9999 + 1)::INT,
  (random() * 5 + 1)::INT,
  round((random() * 100)::numeric, 2)
FROM generate_series(1, 100000);
```

---

## 3. Analytical Queries (5 → 9)

### Query 5: Total Products per Category

```sql
SELECT c.category_name, COUNT(p.product_id)
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id
GROUP BY c.category_name;
```

### Query 6: Top Customers by Total Spending

```sql
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time) AS total_spent
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name
ORDER BY total_spent DESC
LIMIT 10;
```

### Query 7: Most Recent 1000 Orders with Customer Info

```sql
SELECT o.order_id, o.order_date, cu.full_name, cu.email
FROM orders o
JOIN customers cu ON cu.customer_id = o.customer_id
ORDER BY o.order_date DESC
LIMIT 1000;
```

### Query 8: Products with Low Stock (< 10)

```sql
SELECT product_name, stock_quantity
FROM products
WHERE stock_quantity < 10;
```

### Query 9: Revenue per Product Category

```sql
SELECT c.category_name,
       SUM(oi.quantity * oi.price_at_time) AS revenue
FROM categories c
JOIN products p ON p.category_id = c.category_id
JOIN order_items oi ON oi.product_id = p.product_id
GROUP BY c.category_name;
```

---

## 4. Query Optimization using EXPLAIN ANALYZE

### Example Optimization (Query 6)

#### Before Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Issue:** Sequential scans and expensive joins.

#### Optimization Technique

* Added indexes on foreign keys

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

#### After Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Result:** Reduced execution time and index-based joins.

---

## 5. Performance Comparison Table

| Simple Query  | Time Before | Optimization | Rewrite Query | Time After |
| ------------- | ----------- | ------------ | ------------- | ---------- |
| Top Customers | High        | Indexing     | Same          | Lower      |

---

## 6. Key Takeaways

* Design first, SQL second
* Never delete historical data (Orders)
* Use indexes based on query patterns
* Always validate with EXPLAIN ANALYZE

---

**Author:** Backend Engineer Study Notes
**Book Reference:** Practical Database Design for the Web
# E-commerce Database Design & SQL Optimization

This README documents the complete solution for the **Practical Database Design** assignment related to designing an e-commerce database, inserting large-scale data, writing analytical queries, and optimizing them using `EXPLAIN ANALYZE`.

---

## 1. Database Schema Design

The database is designed following **normalization rules**, **relational integrity**, and **backend best practices**.

### 1.1 Customers Table

```sql
CREATE TABLE customers (
  customer_id BIGSERIAL PRIMARY KEY,
  full_name VARCHAR(150) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,# E-commerce Database Design & SQL Optimization

This README documents the complete solution for the **Practical Database Design** assignment related to designing an e-commerce database, inserting large-scale data, writing analytical queries, and optimizing them using `EXPLAIN ANALYZE`.

---

## 1. Database Schema Design

The database is designed following **normalization rules**, **relational integrity**, and **backend best practices**.

### 1.1 Customers Table

```sql
CREATE TABLE customers (
  customer_id BIGSERIAL PRIMARY KEY,
  full_name VARCHAR(150) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.2 Categories Table

```sql
CREATE TABLE categories (
  category_id BIGSERIAL PRIMARY KEY,
  category_name VARCHAR(100) NOT NULL
);
```

### 1.3 Products Table

```sql
CREATE TABLE products (
  product_id BIGSERIAL PRIMARY KEY,
  product_name VARCHAR(200) NOT NULL,
  category_id BIGINT NOT NULL REFERENCES categories(category_id),
  price DECIMAL(10,2) NOT NULL CHECK (price > 0),
  stock_quantity INT NOT NULL CHECK (stock_quantity >= 0)
);
```

### 1.4 Orders Table

```sql
CREATE TABLE orders (
  order_id BIGSERIAL PRIMARY KEY,
  customer_id BIGINT NOT NULL REFERENCES customers(customer_id),
  order_date TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.5 Order Items Table

```sql
CREATE TABLE order_items (
  order_item_id BIGSERIAL PRIMARY KEY,
  order_id BIGINT NOT NULL REFERENCES orders(order_id),
  product_id BIGINT NOT NULL REFERENCES products(product_id),
  quantity INT NOT NULL CHECK (quantity > 0),
  price_at_time DECIMAL(10,2) NOT NULL
);
```

---

## 2. Data Insertion Functions

### 2.1 Insert Products (~10,000 rows)

```sql
INSERT INTO products (product_name, category_id, price, stock_quantity)
SELECT
  'Product ' || generate_series,
  (random() * 9 + 1)::INT,
  round((random() * 100)::numeric, 2),
  (random() * 100)::INT
FROM generate_series(1, 10000);
```

### 2.2 Insert Customers (~100,000 rows)

```sql
INSERT INTO customers (full_name, email)
SELECT
  'Customer ' || generate_series,
  'customer' || generate_series || '@mail.com'
FROM generate_series(1, 100000);
```

### 2.3 Insert Categories (~100 rows)

```sql
INSERT INTO categories (category_name)
SELECT 'Category ' || generate_series
FROM generate_series(1, 100);
```

### 2.4 Insert Orders & Order Items (~100,000 rows)

```sql
INSERT INTO orders (customer_id)
SELECT (random() * 99999 + 1)::INT
FROM generate_series(1, 100000);
```

```sql
INSERT INTO order_items (order_id, product_id, quantity, price_at_time)
SELECT
  (random() * 99999 + 1)::INT,
  (random() * 9999 + 1)::INT,
  (random() * 5 + 1)::INT,
  round((random() * 100)::numeric, 2)
FROM generate_series(1, 100000);
```

---

## 3. Analytical Queries (5 → 9)

### Query 5: Total Products per Category

```sql
SELECT c.category_name, COUNT(p.product_id)
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id
GROUP BY c.category_name;
```

### Query 6: Top Customers by Total Spending

```sql
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time) AS total_spent
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name
ORDER BY total_spent DESC
LIMIT 10;
```

### Query 7: Most Recent 1000 Orders with Customer Info

```sql
SELECT o.order_id, o.order_date, cu.full_name, cu.email
FROM orders o
JOIN customers cu ON cu.customer_id = o.customer_id
ORDER BY o.order_date DESC
LIMIT 1000;
```

### Query 8: Products with Low Stock (< 10)

```sql
SELECT product_name, stock_quantity
FROM products
WHERE stock_quantity < 10;
```

### Query 9: Revenue per Product Category

```sql
SELECT c.category_name,
       SUM(oi.quantity * oi.price_at_time) AS revenue
FROM categories c
JOIN products p ON p.category_id = c.category_id
JOIN order_items oi ON oi.product_id = p.product_id
GROUP BY c.category_name;
```

---

## 4. Query Optimization using EXPLAIN ANALYZE

### Example Optimization (Query 6)

#### Before Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Issue:** Sequential scans and expensive joins.

#### Optimization Technique

* Added indexes on foreign keys

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

#### After Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Result:** Reduced execution time and index-based joins.

---

## 5. Performance Comparison Table

| Simple Query  | Time Before | Optimization | Rewrite Query | Time After |
| ------------- | ----------- | ------------ | ------------- | ---------- |
| Top Customers | High        | Indexing     | Same          | Lower      |

---

## 6. Key Takeaways

* Design first, SQL second
* Never delete historical data (Orders)
* Use indexes based on query patterns
* Always validate with EXPLAIN ANALYZE

---

**Author:** Backend Engineer Study Notes
**Book Reference:** Practical Database Design for the Web

  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.2 Categories Table

```sql
CREATE TABLE categories (
  category_id BIGSERIAL PRIMARY KEY,
  category_name VARCHAR(100) NOT NULL
);
```

### 1.3 Products Table

```sql
CREATE TABLE products (
  product_id BIGSERIAL PRIMARY KEY,
  product_name VARCHAR(200) NOT NULL,
  category_id BIGINT NOT NULL REFERENCES categories(category_id),
  price DECIMAL(10,2) NOT NULL CHECK (price > 0),
  stock_quantity INT NOT NULL CHECK (stock_quantity >= 0)
);
```

### 1.4 Orders Table

```sql
CREATE TABLE orders (
  order_id BIGSERIAL PRIMARY KEY,
  customer_id BIGINT NOT NULL REFERENCES customers(customer_id),
  order_date TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.5 Order Items Table

```sql
CREATE TABLE order_items (
  order_item_id BIGSERIAL PRIMARY KEY,
  order_id BIGINT NOT NULL REFERENCES orders(order_id),
  product_id BIGINT NOT NULL REFERENCES products(product_id),
  quantity INT NOT NULL CHECK (quantity > 0),
  price_at_time DECIMAL(10,2) NOT NULL
);
```

---

## 2. Data Insertion Functions

### 2.1 Insert Products (~10,000 rows)

```sql
INSERT INTO products (product_name, category_id, price, stock_quantity)
SELECT
  'Product ' || generate_series,
  (random() * 9 + 1)::INT,
  round((random() * 100)::numeric, 2),
  (random() * 100)::INT
FROM generate_series(1, 10000);
```

### 2.2 Insert Customers (~100,000 rows)

```sql
INSERT INTO customers (full_name, email)
SELECT
  'Customer ' || generate_series,
  'customer' || generate_series || '@mail.com'
FROM generate_series(1, 100000);
```

### 2.3 Insert Categories (~100 rows)

```sql
INSERT INTO categories (category_name)
SELECT 'Category ' || generate_series
FROM generate_series(1, 100);
```

### 2.4 Insert Orders & Order Items (~100,000 rows)

```sql
INSERT INTO orders (customer_id)
SELECT (random() * 99999 + 1)::INT
FROM generate_series(1, 100000);
```

```sql
INSERT INTO order_items (order_id, product_id, quantity, price_at_time)
SELECT
  (random() * 99999 + 1)::INT,
  (random() * 9999 + 1)::INT,
  (random() * 5 + 1)::INT,
  round((random() * 100)::numeric, 2)
FROM generate_series(1, 100000);
```

---

## 3. Analytical Queries (5 → 9)

### Query 5: Total Products per Category

```sql
SELECT c.category_name, COUNT(p.product_id)
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id
GROUP BY c.category_name;
```

### Query 6: Top Customers by Total Spending

```sql
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time) AS total_spent
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name
ORDER BY total_spent DESC
LIMIT 10;
```

### Query 7: Most Recent 1000 Orders with Customer Info

```sql
SELECT o.order_id, o.order_date, cu.full_name, cu.email
FROM orders o
JOIN customers cu ON cu.customer_id = o.customer_id
ORDER BY o.order_date DESC
LIMIT 1000;
```

### Query 8: Products with Low Stock (< 10)

```sql
SELECT product_name, stock_quantity
FROM products
WHERE stock_quantity < 10;
```

### Query 9: Revenue per Product Category

```sql
SELECT c.category_name,
       SUM(oi.quantity * oi.price_at_time) AS revenue
FROM categories c
JOIN products p ON p.category_id = c.category_id
JOIN order_items oi ON oi.product_id = p.product_id
GROUP BY c.category_name;
```

---

## 4. Query Optimization using EXPLAIN ANALYZE

### Example Optimization (Query 6)

#### Before Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Issue:** Sequential scans and expensive joins.

#### Optimization Technique

* Added indexes on foreign keys

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

#### After Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Result:** Reduced execution time and index-based joins.

---

## 5. Performance Comparison Table

| Simple Query  | Time Before | Optimization | Rewrite Query | Time After |
| ------------- | ----------- | ------------ | ------------- | ---------- |
| Top Customers | High        | Indexing     | Same          | Lower      |

---


# E-commerce Database Design & SQL Optimization

This README documents the complete solution for the **Practical Database Design** assignment related to designing an e-commerce database, inserting large-scale data, writing analytical queries, and optimizing them using `EXPLAIN ANALYZE`.

---

## 1. Database Schema Design

The database is designed following **normalization rules**, **relational integrity**, and **backend best practices**.

### 1.1 Customers Table

```sql
CREATE TABLE customers (
  customer_id BIGSERIAL PRIMARY KEY,
  full_name VARCHAR(150) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.2 Categories Table

```sql
CREATE TABLE categories (
  category_id BIGSERIAL PRIMARY KEY,
  category_name VARCHAR(100) NOT NULL
);
```

### 1.3 Products Table

```sql
CREATE TABLE products (
  product_id BIGSERIAL PRIMARY KEY,
  product_name VARCHAR(200) NOT NULL,
  category_id BIGINT NOT NULL REFERENCES categories(category_id),
  price DECIMAL(10,2) NOT NULL CHECK (price > 0),
  stock_quantity INT NOT NULL CHECK (stock_quantity >= 0)
);
```

### 1.4 Orders Table

```sql
CREATE TABLE orders (
  order_id BIGSERIAL PRIMARY KEY,
  customer_id BIGINT NOT NULL REFERENCES customers(customer_id),
  order_date TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### 1.5 Order Items Table

```sql
CREATE TABLE order_items (
  order_item_id BIGSERIAL PRIMARY KEY,
  order_id BIGINT NOT NULL REFERENCES orders(order_id),
  product_id BIGINT NOT NULL REFERENCES products(product_id),
  quantity INT NOT NULL CHECK (quantity > 0),
  price_at_time DECIMAL(10,2) NOT NULL
);
```

---

## 2. Data Insertion Functions

### 2.1 Insert Products (~10,000 rows)

```sql
INSERT INTO products (product_name, category_id, price, stock_quantity)
SELECT
  'Product ' || generate_series,
  (random() * 9 + 1)::INT,
  round((random() * 100)::numeric, 2),
  (random() * 100)::INT
FROM generate_series(1, 10000);
```

### 2.2 Insert Customers (~100,000 rows)

```sql
INSERT INTO customers (full_name, email)
SELECT
  'Customer ' || generate_series,
  'customer' || generate_series || '@mail.com'
FROM generate_series(1, 100000);
```

### 2.3 Insert Categories (~100 rows)

```sql
INSERT INTO categories (category_name)
SELECT 'Category ' || generate_series
FROM generate_series(1, 100);
```

### 2.4 Insert Orders & Order Items (~100,000 rows)

```sql
INSERT INTO orders (customer_id)
SELECT (random() * 99999 + 1)::INT
FROM generate_series(1, 100000);
```

```sql
INSERT INTO order_items (order_id, product_id, quantity, price_at_time)
SELECT
  (random() * 99999 + 1)::INT,
  (random() * 9999 + 1)::INT,
  (random() * 5 + 1)::INT,
  round((random() * 100)::numeric, 2)
FROM generate_series(1, 100000);
```

---

## 3. Analytical Queries (5 → 9)

### Query 5: Total Products per Category

```sql
SELECT c.category_name, COUNT(p.product_id)
FROM categories c
LEFT JOIN products p ON p.category_id = c.category_id
GROUP BY c.category_name;
```

### Query 6: Top Customers by Total Spending

```sql
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time) AS total_spent
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name
ORDER BY total_spent DESC
LIMIT 10;
```

### Query 7: Most Recent 1000 Orders with Customer Info

```sql
SELECT o.order_id, o.order_date, cu.full_name, cu.email
FROM orders o
JOIN customers cu ON cu.customer_id = o.customer_id
ORDER BY o.order_date DESC
LIMIT 1000;
```

### Query 8: Products with Low Stock (< 10)

```sql
SELECT product_name, stock_quantity
FROM products
WHERE stock_quantity < 10;
```

### Query 9: Revenue per Product Category

```sql
SELECT c.category_name,
       SUM(oi.quantity * oi.price_at_time) AS revenue
FROM categories c
JOIN products p ON p.category_id = c.category_id
JOIN order_items oi ON oi.product_id = p.product_id
GROUP BY c.category_name;
```

---

## 4. Query Optimization using EXPLAIN ANALYZE

### Example Optimization (Query 6)

#### Before Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```

**Issue:** Sequential scans and expensive joins.

#### Optimization Technique

* Added indexes on foreign keys

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

#### After Optimization

```sql
EXPLAIN ANALYZE
SELECT cu.full_name, SUM(oi.quantity * oi.price_at_time)
FROM customers cu
JOIN orders o ON o.customer_id = cu.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY cu.full_name;
```



---

## 5. Performance Comparison Table

| Simple Query  | Time Before | Optimization | Rewrite Query | Time After |
| ------------- | ----------- | ------------ | ------------- | ---------- |
| Top Customers | High        | Indexing     | Same          | Lower      |


