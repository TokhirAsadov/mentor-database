# PostgreSQL-da IDENTITY Ustunlarga oid Qo'llanma

Bu hujjat PostgreSQL-da **IDENTITY** ustunlar haqida to'liq ma'lumot va amaliy misollar beradi. IDENTITY ustunlar avtomatik ravishda ketma-ket raqamlar (identifikatorlar) ishlab chiqarish uchun ishlatiladi, odatda `PRIMARY KEY` yoki unikal `id` lar uchun. IDENTITY `SERIAL` ga zamonaviy alternativa bo'lib, SQL standartiga (ANSI SQL) mos keladi.

---

## 1. IDENTITY nima?

**IDENTITY** ustun PostgreSQL-da jadvalda avtomatik o'suvchi raqamlar ishlab chiqarish uchun ishlatiladi. U ichki Sequence ob'ektiga asoslanadi, lekin foydalanuvchidan mustaqil ravishda boshqariladi. IDENTITY `SERIAL` dan farqli o'laroq, SQL standartiga mos va ko'proq xavfsiz boshqaruv imkonini beradi.

### Asosiy xususiyatlar:
- **SQL standartiga moslik**: ANSI SQL (SQL:2003 va undan keyingi) ga mos, boshqa DBMS-larga ko'chma.
- **Avtomatik boshqaruv**: Ichki Sequence-ni PostgreSQL o'zi yaratadi va boshqaradi.
- **Moslashuvchanlik**: Boshlang'ich qiymat, qadam, chegaralar sozlanishi mumkin.
- **Xavfsizlik**: `GENERATED ALWAYS` bilan tasodifiy qiymat kiritishni oldini oladi.

---

## 2. IDENTITY sintaksisi

IDENTITY ustun `CREATE TABLE` buyrug'ida quyidagi shaklda belgilanadi:

```sql
CREATE TABLE table_name (
    column_name data_type GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY [( sequence_options )],
    ...
);
```

### Parametrlar:
- **`column_name`**: Ustun nomi (masalan, `id`).
- **`data_type`**: `SMALLINT`, `INTEGER`, `BIGINT` (odatda `INTEGER` yoki `BIGINT`).
- **`GENERATED ALWAYS AS IDENTITY`**: Qiymatlar faqat Sequence tomonidan ishlab chiqariladi.
- **`GENERATED BY DEFAULT AS IDENTITY`**: Foydalanuvchi o'z qiymatini kirita oladi.
- **`sequence_options`**:
  - `START WITH start_value`: Boshlang'ich qiymat.
  - `INCREMENT BY increment_value`: Qadam.
  - `MINVALUE min_value`: Eng kichik qiymat.
  - `MAXVALUE max_value`: Eng katta qiymat.
  - `CYCLE | NO CYCLE`: Maksimal qiymatdan keyin boshidan boshlash yoki xato.
  - `CACHE cache_value`: Oldindan saqlanadigan qiymatlar soni.

---

## 3. IDENTITY misollari

Quyida IDENTITY-ning turli foydalanish holatlari bo'yicha amaliy misollar keltirilgan.

### Misol 1: Oddiy IDENTITY ustun (`GENERATED ALWAYS`)
`GENERATED ALWAYS` bilan foydalanuvchi qiymat kiritishi taqiqlanadi.

```sql
CREATE TABLE users (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- Ma'lumot kiritish
INSERT INTO users (name) VALUES ('Ali');
INSERT INTO users (name) VALUES ('Vali');

-- Natijani ko'rish
SELECT * FROM users;
```

**Natija:**
```
 id | name
----+------
  1 | Ali
  2 | Vali
```

**Eslatma**: `id` ga qiymat kiritishga urinish xato beradi:
```sql
INSERT INTO users (id, name) VALUES (5, 'Bob'); -- Xato: cannot insert into generated column
```

Maxsus qiymat kiritish uchun:
```sql
INSERT INTO users (id, name) OVERRIDING SYSTEM VALUE VALUES (5, 'Bob');

SELECT * FROM users;
```

**Natija:**
```
 id | name
----+------
  1 | Ali
  2 | Vali
  5 | Bob
```

---

### Misol 2: GENERATED BY DEFAULT
Foydalanuvchiga o'z qiymatini kiritishga ruxsat beriladi.

```sql
CREATE TABLE products (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- Ma'lumot kiritish
INSERT INTO products (name) VALUES-analyses('Laptop'); -- id avtomatik = 1
INSERT INTO products (id, name) VALUES (100, 'Phone'); -- id = 100
INSERT INTO products (name) VALUES ('Tablet'); -- id avtomatik = 2

-- Natijani ko'rish
SELECT * FROM products;
```

**Natija:**
```
 id  |  name
-----+--------
   1 | Laptop
 100 | Phone
   2 | Tablet
```

**Eslatma**: `GENERATED BY DEFAULT` foydalanuvchiga moslashuvchanlik beradi, lekin `PRIMARY KEY` yoki `UNIQUE` cheklovlariga rioya qilish kerak.

---

### Misol 3: Maxsus Sequence sozlamalari
Boshlang'ich qiymat, qadam va chegaralarni sozlash.

```sql
CREATE TABLE orders (
    order_id BIGINT GENERATED ALWAYS AS IDENTITY (
        START WITH 1000
        INCREMENT BY 10
        MINVALUE 1000
        MAXVALUE 9999
        NO CYCLE
    ) PRIMARY KEY,
    order_date DATE DEFAULT CURRENT_DATE
);

-- Ma'lumot kiritish
INSERT INTO orders (order_date) VALUES ('2025-05-12');
INSERT INTO orders (order_date) VALUES ('2025-05-13');

-- Natijani ko'rish
SELECT * FROM orders;
```

**Natija:**
```
 order_id | order_date
----------+------------
     1000 | 2025-05-12
     1010 | 2025-05-13
```

**Eslatma**: Agar `MAXVALUE` (9999) ga yetilsa, keyingi `INSERT` xato beradi, chunki `NO CYCLE` belgilangan.

---

### Misol 4: IDENTITY-ni o'zgartirish
Mavjud IDENTITY ustun sozlamalarini `ALTER TABLE` bilan o'zgartirish.

```sql
-- Jadval yaratish
CREATE TABLE employees (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(100)
);

-- Ma'lumot kiritish
INSERT INTO employees (name) VALUES ('John');

-- IDENTITY sozlamalarini o'zgartirish
ALTER TABLE employees
    ALTER COLUMN id
    SET GENERATED ALWAYS
    SET START WITH 500
    SET INCREMENT BY 5;

-- Ma'lumot kiritish
INSERT INTO employees (name) VALUES ('Jane');

-- Natijani ko'rish
SELECT * FROM employees;
```

**Natija:**
```
 id  | name
-----+------
   1 | John
 500 | Jane
```

**Eslatma**: `SET START WITH` faqat keyingi qiymatlarga ta'sir qiladi, eski qatorlar o'zgarmaydi.

---

### Misol 5: SERIAL-dan IDENTITY-ga migratsiya
Mavjud `SERIAL` ustunni `IDENTITY` ga aylantirish.

```sql
-- Eski SERIAL jadval
CREATE TABLE old_customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Ma'lumot kiritish
INSERT INTO old_customers (name) VALUES ('Ali');
INSERT INTO old_customers (name) VALUES ('Vali');

-- Yangi IDENTITY jadval
CREATE TABLE new_customers (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    name VARCHAR(100)
);

-- Ma'lumotlarni ko'chirish
INSERT INTO new_customers (id, name)
SELECT id, name FROM old_customers;

-- Natijani ko'rish
SELECT * FROM new_customers;

-- Eski jadvalni o'chirish
DROP TABLE old_customers;
```

**Natija:**
```
 id | name
----+------
  1 | Ali
  2 | Vali
```

**Eslatma**: Migratsiya paytida `UNIQUE` yoki `PRIMARY KEY` cheklovlariga rioya qilish muhim.

---

### Misol 6: IDENTITY-ni o'chirish
IDENTITY xususiyatini ustundan olib tashlash.

```sql
-- Jadval yaratish
CREATE TABLE test_table (
    id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    data VARCHAR(100)
);

-- IDENTITY-ni o'chirish
ALTER TABLE test_table
    ALTER COLUMN id DROP IDENTITY;

-- Endi id oddiy INTEGER ustun
INSERT INTO test_table (id, data) VALUES (1, 'Test');

-- Natijani ko'rish
SELECT * FROM test_table;
```

**Natija:**
```
 id | data
----+------
  1 | Test
```

**Eslatma**: IDENTITY o'chirilganda ichki Sequence ham o'chadi, ustun oddiy `INTEGER` ga aylanadi.

---

## 4. IDENTITY-ni boshqarish

IDENTITY ustun ichki Sequence-ga asoslangan, uni quyidagi buyruqlar bilan boshqarish mumkin:

- **Sozlamalarni o'zgartirish**:
```sql
ALTER TABLE table_name
    ALTER COLUMN column_name
    [SET GENERATED { ALWAYS | BY DEFAULT }]
    [SET sequence_option | RESTART [WITH new_value]];
```

- **IDENTITY-ni o'chirish**:
```sql
ALTER TABLE table_name
    ALTER COLUMN column_name
    DROP IDENTITY [IF EXISTS];
```

- **Jadvalni o'chirish**: IDENTITY ustun jadvalga bog'langan, shuning uchun `DROP TABLE` ichki Sequence-ni ham o'chiradi.

---

## 5. E'tibor beriladigan jihatlar

- **Tranzaksiya xavfsizligi**: IDENTITY ichki Sequence-ga asoslanadi, shuning uchun `nextval` tranzaksiyadan mustaqil. `ROLLBACK` bo'lsa, ishlatilgan qiymatlar qaytarilmaydi, bu `id` larda bo'shliqlar (gaps) hosil qiladi.
- **GENERATED ALWAYS vs BY DEFAULT**:
  - `ALWAYS`: Tasodifiy qiymat kiritishni oldini oladi, xavfsizroq.
  - `BY DEFAULT`: Moslashuvchan, lekin cheklovlarga e'tibor berish kerak.
- **Ko'chmalilik**: IDENTITY boshqa SQL DBMS-larida (masalan, Oracle, SQL Server) ishlaydi, shuning uchun katta loyihalarda afzal.
- **Chegaralar**: `MINVALUE`, `MAXVALUE` va boshqa sozlamalar ma'lumot turi chegaralariga mos bo'lishi kerak.
- **UUID alternativi**: Agar bo'shliqlar muammo bo'lsa yoki global unikal identifikator kerak bo'lsa, `UUID` ni ko'rib chiqing.
