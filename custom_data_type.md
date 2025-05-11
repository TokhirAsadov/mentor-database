# PostgreSQL-da DOMAIN va TYPE Ma'lumot Turlari

PostgreSQL-da maxsus ma'lumot turlari foydalanuvchilarga ilovalariga mos ravishda yangi ma'lumot turlarini yaratish imkonini beradi. Ushbu hujjatda **DOMAIN** (cheklovli ma'lumot turi), **Composite TYPE** (murakkab tuzilma turi) va **Enumerated TYPE** (sanab o'tilgan qiymatlar turi) haqida tushuntirishlar va misollar keltirilgan.
***
## 1. `DOMAIN` Ma'lumot Turi
**Tavsif**: DOMAIN mavjud ma'lumot turiga qo'shimcha cheklovlar (constraints) qo'yib, yangi ma'lumot turi sifatida ishlatiladi. Bu ma'lumotlarning muayyan shartlarga mos kelishini ta'minlaydi, masalan, faqat ijobiy sonlar yoki maxsus formatdagi email manzillar.

**Sintaksis**:
```sql
CREATE DOMAIN domain_name AS base_type [CONSTRAINT constraint_name CHECK (condition)];
```

**Misol 1: Ijobiy sonlar uchun DOMAIN**
```sql
CREATE DOMAIN positive_integer AS INTEGER
    CHECK (VALUE > 0);

CREATE TABLE inventory (
    id SERIAL PRIMARY KEY,
    item VARCHAR(100),
    quantity positive_integer
);

-- Muvaffaqiyatli kiritish
INSERT INTO inventory (item, quantity) VALUES ('Book', 10);

-- Xato holati
INSERT INTO inventory (item, quantity) VALUES ('Pen', -5);
```

**Natija**:
```
 id | item | quantity
----|------|----------
  1 | Book | 10
```

**Xato holati**:
```
ERROR: value for domain positive_integer violates check constraint
```

**Misol 2: Email manzillari uchun DOMAIN**
```sql
CREATE DOMAIN email_address AS VARCHAR(100)
    CHECK (VALUE ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$');

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    email email_address
);

-- Muvaffaqiyatli kiritish
INSERT INTO users (username, email) VALUES ('Alice', 'alice@example.com');

-- Xato holati
INSERT INTO users (username, email) VALUES ('Bob', 'invalid_email');
```

**Natija**:
```
 id | username |       email
----|----------|------------------
  1 | Alice    | alice@example.com
```

**Xato holati**:
```
ERROR: value for domain email_address violates check constraint
```

**Eslatma**:
- `VALUE` domen qiymatini tekshirishda ishlatiladi.
- `CHECK` cheklovi ma'lumotning to'g'ri formatda ekanligini ta'minlaydi.
- `ALTER DOMAIN` bilan cheklovlarni o'zgartirish mumkin:
  ```sql
  ALTER DOMAIN positive_integer DROP CONSTRAINT positive_integer_check;
  ALTER DOMAIN positive_integer ADD CONSTRAINT positive_integer_check CHECK (VALUE >= 0);
  ```
***
## 2. Composite `TYPE` (Murakkab turi)
**Tavsif**: Composite TYPE bir nechta maydonlarni (fields) birlashtirgan tuzilma bo'lib, har bir maydon o'z ma'lumot turiga ega. Bu C tilidagi `struct` yoki Python'dagi `named tuple` ga o'xshaydi.

**Sintaksis**:
```sql
CREATE TYPE type_name AS (
    field_name1 data_type1,
    field_name2 data_type2,
    ...
);
```

**Misol 1: Manzil ma'lumotlari uchun Composite TYPE**
```sql
CREATE TYPE address AS (
    street VARCHAR(100),
    city VARCHAR(50),
    postal_code VARCHAR(10)
);

CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    home_address address
);

-- Ma'lumot kiritish
INSERT INTO employees (name, home_address)
VALUES ('Alice', ROW('123 Main St', 'New York', '10001'));

-- So'rov
SELECT name, (home_address).street AS street FROM employees;
```

**Natija**:
```
 name  |     street
-------|---------------
 Alice | 123 Main St
```

**Misol 2: Shaxs ma'lumotlari uchun Composite TYPE**
```sql
CREATE TYPE person AS (
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    age INTEGER
);

CREATE TABLE contacts (
    id SERIAL PRIMARY KEY,
    info person
);

-- Ma'lumot kiritish
INSERT INTO contacts (info)
VALUES (ROW('John', 'Doe', 30));

-- So'rov
SELECT (info).first_name AS first_name, (info).age AS age FROM contacts;
```

**Natija**:
```
 first_name | age
------------|-----
 John       | 30
```

**Eslatma**:
- `ROW` konstruktsiyasi composite type qiymatini yaratish uchun ishlatiladi.
- Maydonlarga `(column_name).field_name` sintaksisi bilan murojaat qilinadi.
- `ALTER TYPE` bilan maydon qo'shish mumkin:
  ```sql
  ALTER TYPE address ADD ATTRIBUTE country VARCHAR(50);
  ```
***
## 3. `Enumerated` TYPE (Enum turi)
**Tavsif**: Enumerated TYPE oldindan belgilangan qiymatlar ro'yxatini saqlash uchun ishlatiladi. Bu ma'lumot turi faqat belgilangan qiymatlardan birini qabul qiladi, masalan, holatlar yoki ro'yxatlar.

**Sintaksis**:
```sql
CREATE TYPE type_name AS ENUM ('value1', 'value2', ...);
```

**Misol 1: Buyurtma holati uchun Enum TYPE**
```sql
CREATE TYPE order_status AS ENUM ('pending', 'shipped', 'delivered', 'cancelled');

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    product VARCHAR(100),
    status order_status
);

-- Ma'lumot kiritish
INSERT INTO orders (product, status) VALUES ('Laptop', 'pending');
INSERT INTO orders (product, status) VALUES ('Phone', 'shipped');

-- So'rov
SELECT * FROM orders WHERE status = 'pending';
```

**Natija**:
```
 id | product | status
----|---------|---------
  1 | Laptop  | pending
```

**Misol 2: Haftaning kunlari uchun Enum TYPE**
```sql
CREATE TYPE weekday AS ENUM ('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday');

CREATE TABLE schedule (
    id SERIAL PRIMARY KEY,
    event VARCHAR(100),
    day weekday
);

-- Ma'lumot kiritish
INSERT INTO schedule (event, day) VALUES ('Meeting', 'Monday');
INSERT INTO schedule (event, day) VALUES ('Workshop', 'Friday');

-- So'rov
SELECT * FROM schedule WHERE day IN ('Monday', 'Friday');
```

**Natija**:
```
 id |  event   |   day
----|----------|---------
  1 | Meeting  | Monday
  2 | Workshop | Friday
```

**Eslatma**:
- Enum qiymatlari kiritilgan tartibda saqlanadi va taqqoslashda shu tartib ishlatiladi.
- Yangi enum qiymatini qo'shish uchun:
  ```sql
  ALTER TYPE order_status ADD VALUE 'returned';
  ```
- Enum qiymatlarni o'chirish mumkin emas, faqat yangilar qo'shilishi mumkin.

## Umumiy Eslatmalar
- **DOMAIN** ma'lumotlarning muayyan shartlarga mos kelishini ta'minlaydi, lekin yangi saqlash mexanizmini yaratmaydi.
- **Composite TYPE** bog'langan ma'lumotlarni guruhlash uchun foydali, masalan, manzil yoki shaxs ma'lumotlari.
- **Enumerated TYPE** cheklangan qiymatlar ro'yxati uchun ideal, masalan, holatlar yoki ro'yxatlar.
- Turlarni o'chirish uchun `DROP TYPE` ishlatiladi, lekin bog'liqliklarni tekshirish kerak:
  ```sql
  DROP TYPE address CASCADE;
  ```
- Indekslashni katta jadvallarda unutmang, masalan:
  ```sql
  CREATE INDEX idx_employees_address ON employees USING GIN(home_address);
  ```
