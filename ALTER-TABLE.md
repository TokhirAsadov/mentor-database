# PostgreSQL-da ALTER TABLE Buyrug'iga oid To'liq Qo'llanma

`ALTER TABLE` jadval tuzilmasini o'zgartirish uchun ishlatiladi: ustunlar qo'shish/o'chirish, ma'lumot turlarini o'zgartirish, cheklovlar qo'shish/o'chirish, nom o'zgartirish va boshqa operatsiyalar.

---

## 1. `ALTER TABLE` nima?

`ALTER TABLE` buyrug'i PostgreSQL-da mavjud jadvalning tuzilmasini yoki xususiyatlarini o'zgartirish uchun ishlatiladi. U jadvalning ustunlari, cheklovlari, nomlari, sxemasi va boshqa atributlarini o'zgartirish imkonini beradi.

### Umumiy sintaksis:
```sql
ALTER TABLE [IF EXISTS] table_name
    action [, ...];
```

- **`table_name`**: O'zgartiriladigan jadval nomi.
- **`IF EXISTS`**: Agar jadval mavjud bo'lmasa, xato chiqmasligini ta'minlaydi.
- **`action`**: Quyidagi operatsiyalardan biri yoki bir nechtasi:
  - Ustun qo'shish/o'chirish/o'zgartirish.
  - Cheklov qo'shish/o'chirish.
  - Nom o'zgartirish.
  - Sxema yoki egalikni o'zgartirish va hokazo.

---

## 2. `ALTER TABLE` ning asosiy holatlari va misollar

Quyida `ALTER TABLE` ning barcha mumkin bo'lgan operatsiyalari guruhlarga ajratilib, har biri uchun misollar keltirilgan.

### 2.1. Ustunlar bilan ishlash

#### 2.1.1. Ustun qo'shish (`ADD COLUMN`)
Yangi ustun qo'shish uchun.

```sql
-- Jadval yaratish
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Yangi ustun qo'shish
ALTER TABLE users
    ADD COLUMN email VARCHAR(255);

-- Ma'lumot kiritish
INSERT INTO users (name, email) VALUES ('Ali', 'ali@example.com');

-- Natijani ko'rish
SELECT * FROM users;
```

**Natija:**
```
 id | name |       email
----+------+-------------------
  1 | Ali  | ali@example.com
```

**Eslatma**: Agar `IF NOT EXISTS` ishlatilsa, ustun allaqachon mavjud bo'lsa xato chiqmaydi:
```sql
ALTER TABLE users
    ADD COLUMN IF NOT EXISTS email VARCHAR(255);
```

#### 2.1.2. Ustunni o'chirish (`DROP COLUMN`)
Mavjud ustunni o'chirish.

```sql
-- Email ustunini o'chirish
ALTER TABLE users
    DROP COLUMN email;

-- Natijani ko'rish
SELECT * FROM users;
```

**Natija:**
```
 id | name
----+------
  1 | Ali
```

**Eslatma**: `IF EXISTS` bilan xato oldini olish mumkin:
```sql
ALTER TABLE users
    DROP COLUMN IF EXISTS email;
```

#### 2.1.3. Ustun ma'lumot turini o'zgartirish (`ALTER COLUMN ... TYPE`)
Ustunning ma'lumot turini o'zgartirish.

```sql
-- Name ustunini TEXT ga o'zgartirish
ALTER TABLE users
    ALTER COLUMN name TYPE TEXT;

-- Natijani ko'rish
\d users
```

**Natija:**
```
 Column |  Type  | ...
--------+--------+ ...
 id     | integer| ...
 name   | text   | ...
```

**Eslatma**: Agar ma'lumotlar mos kelmasa, `USING` bilan konvertatsiya qilish kerak:
```sql
-- VARCHAR dan INTEGER ga o'zgartirish
ALTER TABLE users
    ADD COLUMN age VARCHAR(10);

INSERT INTO users (name, age) VALUES ('Vali', '25');

ALTER TABLE users
    ALTER COLUMN age TYPE INTEGER USING (age::INTEGER);

SELECT * FROM users;
```

**Natija:**
```
 id | name | age
----+------+-----
  1 | Ali  | NULL
  2 | Vali | 25
```

#### 2.1.4. Ustun uchun standart qiymat o'rnatish (`SET DEFAULT`)
Ustunga standart qiymat belgilash.

```sql
-- Age ustuniga standart qiymat o'rnatish
ALTER TABLE users
    ALTER COLUMN age SET DEFAULT 18;

-- Ma'lumot kiritish
INSERT INTO users (name) VALUES ('Bob');

SELECT * FROM users;
```

**Natija:**
```
 id | name | age
----+------+-----
  1 | Ali  | NULL
  2 | Vali | 25
  3 | Bob  | 18
```

#### 2.1.5. Standart qiymatni o'chirish (`DROP DEFAULT`)
Ustunning standart qiymatini olib tashlash.

```sql
ALTER TABLE users
    ALTER COLUMN age DROP DEFAULT;

-- Ma'lumot kiritish
INSERT INTO users (name) VALUES ('Jane');

SELECT * FROM users;
```

**Natija:**
```
 id | name | age
----+------+-----
  1 | Ali  | NULL
  2 | Vali | 25
  3 | Bob  | 18
  4 | Jane | NULL
```

#### 2.1.6. `NOT NULL` cheklovini qo'shish (`SET NOT NULL`)
Ustunning bo'sh bo'lmasligini talab qilish.

```sql
-- Name ustuniga NOT NULL qo'shish
ALTER TABLE users
    ALTER COLUMN name SET NOT NULL;

-- Xato: bo'sh qiymat kiritish
INSERT INTO users (name) VALUES (NULL); -- Xato: null value in column "name"
```

**Eslatma**: Agar ustunda allaqachon `NULL` qiymatlar bo'lsa, avval ularni to'ldirish kerak:
```sql
UPDATE users SET age = 0 WHERE age IS NULL;

ALTER TABLE users
    ALTER COLUMN age SET NOT NULL;
```

#### 2.1.7. `NOT NULL` cheklovini o'chirish (`DROP NOT NULL`)
Ustunning bo'sh qiymatlarni qabul qilishiga ruxsat berish.

```sql
ALTER TABLE users
    ALTER COLUMN age DROP NOT NULL;

-- Endi NULL kiritish mumkin
INSERT INTO users (name, age) VALUES ('Tom', NULL);

SELECT * FROM users;
```

**Natija:**
```
 id | name | age
----+------+-----
  1 | Ali  | 0
  2 | Vali | 25
  3 | Bob  | 18
  4 | Jane | 0
  5 | Tom  | NULL
```

---

### 2.2. Cheklovlar (Constraints) bilan ishlash

#### 2.2.1. Cheklov qo'shish (`ADD CONSTRAINT`)
Jadvalga yangi cheklov qo'shish (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`).

```sql
-- Jadval yaratish
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    amount NUMERIC
);

-- FOREIGN KEY qo'shish
ALTER TABLE orders
    ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users(id);

-- CHECK cheklovi qo'shish
ALTER TABLE orders
    ADD CONSTRAINT positive_amount CHECK (amount > 0);

-- Ma'lumot kiritish
INSERT INTO orders (user_id, amount) VALUES (1, 100.50);
INSERT INTO orders (user_id, amount) VALUES (2, -50); -- Xato: CHECK constraint
```

#### 2.2.2. Cheklovni o'chirish (`DROP CONSTRAINT`)
Mavjud cheklovni olib tashlash.

```sql
ALTER TABLE orders
    DROP CONSTRAINT positive_amount;

-- Endi salbiy amount kiritish mumkin
INSERT INTO orders (user_id, amount) VALUES (2, -50);

SELECT * FROM orders;
```

**Natija:**
```
 order_id | user_id | amount
----------+---------+--------
        1 |       1 | 100.50
        2 |       2 | -50
```

**Eslatma**: `IF EXISTS` bilan xato oldini olish:
```sql
ALTER TABLE orders
    DROP CONSTRAINT IF EXISTS positive_amount;
```

#### 2.2.3. `UNIQUE` cheklov qo'shish
Ustunga yoki ustunlar guruhiga `UNIQUE` cheklov qo'shish.

```sql
ALTER TABLE users
    ADD COLUMN email VARCHAR(255);

ALTER TABLE users
    ADD CONSTRAINT unique_email UNIQUE (email);

-- Ma'lumot kiritish
INSERT INTO users (name, email) VALUES ('Sam', 'sam@example.com');
INSERT INTO users (name, email) VALUES ('Max', 'sam@example.com'); -- Xato: duplicate key
```

---

### 2.3. Nom o'zgartirish

#### 2.3.1. Jadval nomini o'zgartirish (`RENAME TO`)
Jadvalning nomini o'zgartirish.

```sql
ALTER TABLE users
    RENAME TO customers;

-- Yangi nom bilan so'rov
SELECT * FROM customers;
```

**Natija:**
```
 id | name | age |       email
----+------+-----+--------------------
  1 | Ali  | 0   | NULL
  2 | Vali | 25  | NULL
  3 | Bob  | 18  | NULL
  4 | Jane | 0   | NULL
  5 | Tom  | NULL| NULL
  6 | Sam  | 0   | sam@example.com
```

#### 2.3.2. Ustun nomini o'zgartirish (`RENAME COLUMN`)
Ustunning nomini o'zgartirish.

```sql
ALTER TABLE customers
    RENAME COLUMN name TO full_name;

-- Natijani ko'rish
\d customers
```

**Natija:**
```
 Column   |  Type  | ...
----------+--------+ ...
 id       | integer| ...
 full_name| text   | ...
 age      | integer| ...
 email    | varchar| ...
```

---

### 2.4. IDENTITY bilan ishlash

#### 2.4.1. IDENTITY sozlamalarini o'zgartirish
IDENTITY ustun sozlamalarini o'zgartirish.

```sql
-- IDENTITY jadval
CREATE TABLE employees (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(100)
);

-- Sozlamalarni o'zgartirish
ALTER TABLE employees
    ALTER COLUMN id
    SET GENERATED ALWAYS
    SET START WITH 100
    SET INCREMENT BY 5;

-- Ma'lumot kiritish
INSERT INTO employees (name) VALUES ('John');

SELECT * FROM employees;
```

**Natija:**
```
 id  | name
-----+------
 100 | John
```

#### 2.4.2. IDENTITY-ni o'chirish
IDENTITY xususiyatini olib tashlash.

```sql
ALTER TABLE employees
    ALTER COLUMN id DROP IDENTITY;

-- Endi id oddiy INTEGER
INSERT INTO employees (id, name) VALUES (200, 'Jane');

SELECT * FROM employees;
```

**Natija:**
```
 id  | name
-----+------
 100 | John
 200 | Jane
```

---

### 2.5. Sxema va egalik bilan ishlash

#### 2.5.1. Jadvalni boshqa sxemaga ko'chirish (`SET SCHEMA`)
Jadvalni boshqa sxemaga o'tkazish.

```sql
-- Yangi sxema yaratish
CREATE SCHEMA archive;

-- Jadvalni archive sxemasiga ko'chirish
ALTER TABLE customers
    SET SCHEMA archive;

-- Yangi sxemada so'rov
SELECT * FROM archive.customers;
```

#### 2.5.2. Jadval egaligini o'zgartirish (`OWNER TO`)
Jadvalning egasini o'zgartirish.

```sql
ALTER TABLE archive.customers
    OWNER TO new_user;
```

---

### 2.6. Boshqa operatsiyalar

#### 2.6.1. Jadvalni faqat o'qish rejimiga o'tkazish (`SET ...`)
Jadvalni faqat o'qish (read-only) rejimiga o'tkazish.

```sql
ALTER TABLE archive.customers
    SET (autovacuum_enabled = false, toast.autovacuum_enabled = false);

-- Sozlamalarni tekshirish
SELECT reloptions FROM pg_class WHERE relname = 'customers';
```

#### 2.6.2. Klasterlashni o'zgartirish (`CLUSTER ON`)
Jadvalning klasterlash indeksini o'zgartirish.

```sql
-- Indeks yaratish
CREATE INDEX idx_customers_email ON archive.customers(email);

-- Klasterlashni o'rnatish
ALTER TABLE archive.customers
    CLUSTER ON idx_customers_email;
```

#### 2.6.3. Jadvalni qayta klasterlash (`SET WITHOUT CLUSTER`)
Klasterlashni bekor qilish.

```sql
ALTER TABLE archive.customers
    SET WITHOUT CLUSTER;
```

---

## 3. Bir nechta operatsiyalarni birlashtirish

Bir `ALTER TABLE` buyrug'ida bir nechta operatsiyalarni bajarish mumkin.

```sql
ALTER TABLE archive.customers
    ADD COLUMN phone VARCHAR(20),
    ALTER COLUMN email SET NOT NULL,
    RENAME COLUMN full_name TO username,
    ADD CONSTRAINT unique_phone UNIQUE (phone);

-- Natijani ko'rish
\d archive.customers
```

**Natija:**
```
 Column  |  Type  | ...
---------+--------+ ...
 id      | integer| ...
 username| text   | ...
 age     | integer| ...
 email   | varchar| not null
 phone   | varchar| ...
```
