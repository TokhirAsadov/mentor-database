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
# `INSERT` statement example
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT
);

INSERT INTO users (name, age) VALUES ('Ali', 25);
```
***
# `SELECT` statement examples
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
# `UPDATE` statement examples
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
# `DELETE` statement example
### Oddiy `DELETE`
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT,
    status VARCHAR(20) DEFAULT 'active'
);

INSERT INTO users (name, age) 
VALUES ('Ali', 25), ('Vali', 30), ('Nargiza', 22);

DELETE FROM users
WHERE name = 'Ali';
```
Natija: `users` jadvalida `name = 'Ali'` bo‘lgan qator o‘chiriladi.
### `USING` bilan Boshqa Jadvaldan Foydalanish. Murakkab `DELETE`
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    order_date DATE
);

INSERT INTO orders (user_id, order_date) 
VALUES (2, '2023-01-01'), (3, '2024-01-01');

DELETE FROM users
USING orders
WHERE users.id = orders.user_id AND orders.order_date < '2024-01-01';
```
***
# `UPSERT` statement examples
### `DO NOTHING` bilan `UPSERT`
Agar konflikt yuzaga kelsa, hech narsa qilmaydi, ya'ni qator qo‘shilmaydi.
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    name VARCHAR(100),
    age INT
);

INSERT INTO users (email, name, age)
VALUES ('ali@example.com', 'Ali', 25)
ON CONFLICT (email) DO NOTHING;
```
Natija:
- Agar email = 'ali@example.com' bo‘lgan qator mavjud bo‘lmasa, yangi qator qo‘shiladi.
- Agar mavjud bo‘lsa, hech narsa o‘zgarmaydi.
***
### `DO UPDATE` bilan `UPSERT`
Agar konflikt yuzaga kelsa, mavjud qatorni yangilaydi.
```sql
INSERT INTO users (email, name, age)
VALUES ('ali@example.com', 'Aliyor', 26)
ON CONFLICT (email) DO UPDATE
SET name = EXCLUDED.name,
    age = EXCLUDED.age
RETURNING id, email, name, age;
```
Tushuntirish:
- `EXCLUDED`: Qo‘shilmoqchi bo‘lgan, lekin konflikt kelib chiqqan qiymatlarni ifodalaydi.
- Agar `email = 'ali@example.com'` mavjud bo‘lsa, `name` va `age` yangi qiymatlarga o‘zgartiriladi.
- Agar mavjud bo‘lmasa, yangi qator qo‘shiladi.
Natija:
```txt
 id |       email       |  name  | age
----+-------------------+--------+-----
  1 | ali@example.com   | Aliyor |  26
```
***
