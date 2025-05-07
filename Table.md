# Create Table Example
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT DEFAULT 18
);
```
***
# Constraints ishlatilinishiga example
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    total_amount NUMERIC CHECK (total_amount >= 0),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
***
# `INSERT` example
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT
);

INSERT INTO users (name, age) VALUES ('Ali', 25);
```
***
# `SELECT` examples
### Oddiy `select`
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    age INT
);

INSERT INTO users (name, age) VALUES ('Ali', 25), ('Vali', 30), ('Nargiza', 22);

SELECT * FROM users;
```
```txt
 id |  name   | age
----+---------+-----
  1 | Ali     |  25
  2 | Vali    |  30
  3 | Nargiza |  22
```
***
### Muayyan ustunlar:
```sql
SELECT name, age FROM users;
```
***
