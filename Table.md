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
# `UPDATE` examples
### Oddiy `update`
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT,
    status VARCHAR(20) DEFAULT 'active'
);

INSERT INTO users (name, age) VALUES ('Ali', 25), ('Vali', 30), ('Nargiza', 22);

UPDATE users
SET age = 26
WHERE name = 'Ali';
```
Natija: `users` jadvalida `name = 'Ali'` bo‘lgan qatorning `age` ustuni `26` ga o‘zgartiriladi.
***
### `FROM` bilan Boshqa Jadvaldan Ma'lumot Ishlatish
```sql
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50),
    bonus_rate NUMERIC(4, 2)
);

INSERT INTO departments (dept_name, bonus_rate) VALUES ('HR', 1.10), ('IT', 1.15);

ALTER TABLE users ADD COLUMN dept_id INT REFERENCES departments(dept_id);
UPDATE users SET dept_id = 1 WHERE id = 1;
UPDATE users SET dept_id = 2 WHERE id IN (2, 3);

UPDATE users u
SET age = age * d.bonus_rate
FROM departments d
WHERE u.dept_id = d.dept_id;
```
### Murakkab `update`. Murakkab shartlarga asoslangan `update` uchun `CASE` ishlatiladi
```sql 
UPDATE users
SET status = CASE
    WHEN age < 25 THEN 'young'
    WHEN age >= 25 THEN 'senior'
    ELSE 'unknown'
END;
```
***
