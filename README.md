# SQL cheat sheet

(Youtube Tutorial - MySQL Tutorial for Beginners)[https://www.youtube.com/watch?v=7S_tz1z_5bA&ab_channel=ProgrammingwithMosh]

### LIKE
Return customers born between 1/1/1990 and 1/1/2000
```
SELECT *
FROM customers
WHERE birth_date BETWEEN '1990-01-01' AND '2000-01-01'
```

### REGEXP
```
SELECT *
FROM customers
WHERE first_name REGEXP 'ELKA|AMBUR'
WHERE last_name REGEXP 'EY$|ON$'
WHERE last_name REGEXP '^MY|SE'
WHERE last_name REGEXP 'B[RU]'
WHERE last_name REGEXP 'BR|BU'
```

### ORDER BY
order rows by largest total price
```
SELECT *, quantity * unit_price AS total_price
FROM order_items
WHERE order_id = 2
ORDER BY total_price DESC
```

### Inner Join
Inner Join - Join 2 tables and set the customer ids from orders and customers table to be matching on the same row
Inner is optional, it can be written either `JOIN` or `INNER JOIN`
```
SELECT
  order_id,
  o.customer_id,
  first_name,
  last_name,
  c.customer_id
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id  
```

```
SELECT order_id, oi.product_id, name, quantity, oi.unit_price
FROM order_items oi
JOIN products p
ON oi.product_id = p.product_id
```

### Joining tables across databases
```
USE sql_inventory;
SELECT * FROM sql_store.order_items oi
JOIN products p
ON oi.product_id = p.product_id
```

### self joins
useful when we want to join tables by itself to display info from the same table showing only columns we want
```
USE sql_hr;
SELECT
  e.employee_id,
  e.first_name,
  e.last_name,
  m.employee_id,
  m.first_name AS manager
FROM employees e
JOIN employees m
ON e.reports_to = m.employee_id
```

### oin more than 2 tables
```
USE sql_store;
SELECT
  o.order_id,
  o.order_date,
  c.first_name,
  c.last_name,
  os.name AS status
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id
JOIN order_statuses os
  ON o.status = os.order_status_id
```

```
-- date, invoice_id, amoutn, client name, payment method name
USE sql_invoicing;
SELECT date, p.invoice_id, i.payment_total, c.name, pm.name
FROM payments p
JOIN clients c
  ON p.client_id = c.client_id
JOIN payment_methods pm
 ON pm.payment_method_id = p.payment_method
JOIN invoices i
  ON p.invoice_id = i.invoice_id
```
  
### Compound join conditions
This doesnt work :/ Need to find another example
```
USE sql_store;
SELECT *
FROM order_items oi
JOIN order_items_notes oin
  ON oi.order_id = oin.order_id
  AND oi.product_id = oin.product_id
```
  
### Implicit join syntax
combine tables in FROM, and add WHERE instead of ON
```
SELECT *
FROM orders o, customers c
WHERE o.customer_id = c.customer_id
```

### Outer joins - LEFT, RIGHT
Cn be written either `LEFT JOIN` or `RIGHT JOIN`.
Useful when you need to return records that doesn't completely match the ON condition.
2 types of OUTER joins, RIGHT and LEFT.
LEFT JOIN returns all the records from the left table (ie customers) regardless of the ON condition. It will return all customers even if they don't have order_id 
RIGHT JOIN returns all the records from the right table (ie orders) regardless of the ON condition.

```
SELECT 
  c.customer_id,
  c.first_name,
  o.order_id
-- FROM customers c LEFT JOIN orders o
-- FROM orders o RIGHT JOIN customers c
FROM customers c RIGHT JOIN orders o
  ON o.customer_id = c.customer_id
ORDER BY c.customer_id
```

```
SELECT
  p.product_id,
  p.name,
  oi.quantity
FROM products p
LEFT JOIN order_items oi
  ON p.product_id = oi.product_id
ORDER BY p.product_id
```

### OUTER joins between multiple tables
```
SELECT
  c.customer_id,
  c.first_name,
  o.order_id,
  sh.name AS shipper
FROM customers c
LEFT JOIN orders o
  ON c.customer_id = o.customer_id
LEFT JOIN shippers sh
  ON o.shipper_id = sh.shipper_id
ORDER BY c.customer_id
```

```
SELECT
  o.order_id,
  o.order_date,
  c.first_name,
  sh.name AS shipper,
  os.name AS status
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id
LEFT JOIN shippers sh
  ON o.shipper_id = sh.shipper_id
JOIN order_statuses os
  ON o.status = os.order_status_id
ORDER BY o.order_id
```

### Self outer joins
LEFT join here will get EVERY employee from the `employees e` table whether they have a manager or not.
```
USE sql_hr;
SELECT 
  e.employee_id,
  e.first_name,
  m.first_name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.reports_to = m.employee_id
```

### USING clause
Useful when column names are exactly the same across tables
`USING (customer_id)` is the same as `ON o.customer_id = c.customer_id`
```
USE sql_store;
SELECT
  o.order_id,
  c.first_name,
  sh.name AS shipper
FROM orders o
JOIN customers c
-- ON o.customer_id = c.customer_id
  USING (customer_id)
LEFT JOIN shippers sh
  USING (shipper_id)
```

```
USE sql_invoicing;
SELECT 
  p.date,
  c.name AS client,
  p.amount,
  pm.name AS 'payment method'
FROM payments p
JOIN payment_methods pm
  ON p.payment_method = pm.payment_method_id
JOIN clients c
  USING (client_id)
```

### Natural JOINS
This is dangerous because we are letting MySQL to work out the columns itself.
Natural joins will join them based on common columns (columns with same name)
```
USE sql_store;
SELECT 
  o.order_id,
  c.first_name
FROM orders o
NATURAL JOIN customers c
```

### CROSS join
Explicit syntax - `CROSS JOIN`
Implicit syntax - multiple tables in `FROM`
Combine every record in the first table with ever record in the second table.
Useful when you have table of sizes ('small', 'medium', 'large') and table of colours ('red', 'green', 'yellow') and want to combine them.
```
USE sql_store;
SELECT 
  c.first_name AS customer,
  p.name AS product
FROM customers c
CROSS JOIN products p
ORDER BY c.first_name
```

```
USE sql_store;
SELECT 
  sh.name AS shipper,
  p.name AS product
-- implicit
FROM shippers sh, products p
-- explicit
-- FROM shippers sh
-- CROSS JOIN products p
ORDER BY sh.name
```

### UNIONS
Combine records/results from multiple queries
```
USE sql_store;
SELECT 
  order_id,
  order_date,
  'Active' AS status
FROM orders o
WHERE order_date >= '2019-01-01'
UNION
SELECT 
  order_id,
  order_date,
  'Archived' AS status
FROM orders o
WHERE order_date < '2019-01-01'
```

```
USE sql_store;
SELECT 
  customer_id,
  first_name,
  points,
  'Bronze' AS type
FROM customers
WHERE points < 2000
UNION
SELECT 
  customer_id,
  first_name,
  points,
  'Silver' AS type
FROM customers
WHERE points BETWEEN 2000 AND 3000
UNION
SELECT 
  customer_id,
  first_name,
  points,
  'Gold' AS type
FROM customers
WHERE points > 3000
ORDER BY first_name
```