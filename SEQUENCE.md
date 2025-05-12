# PostgreSQL-da Sequence haqida to'liq qo'llanma

Bu hujjat PostgreSQL-da `Sequence` (tartib raqami generatori) haqida to'liq ma'lumot beradi, uning tuzilmasi, funksiyalari va amaliy misollar keltiriladi. Sequence ma'lumotlar bazasida avtomatik ketma-ket raqamlar ishlab chiqarish uchun ishlatiladi, odatda `PRIMARY KEY` yoki unikal identifikatorlar uchun.

---

## 1. Sequence nima?

**Sequence** PostgreSQL-da mustaqil ob'ekt bo'lib, ketma-ket raqamlar (butun sonlar) ishlab chiqarish uchun mo'ljallangan. U ko'pincha `SERIAL` ma'lumot turi bilan ishlatiladi, lekin mustaqil ravishda ham qo'llanilishi mumkin.

### Asosiy xususiyatlar:
- Avtomatik o'suvchi yoki kamayuvchi raqamlar ishlab chiqaradi.
- Bir nechta jadvallar yoki so'rovlar tomonidan ishlatilishi mumkin.
- Tranzaksiyalardan mustaqil: `nextval` ishlatilganda qiymatlar rollback qilinsa ham qaytarilmaydi.
- Moslashuvchan: boshlang'ich qiymat, qadam, chegaralar sozlanadi.

### Umumiy foydalanish holatlari:
- Jadval qatorlari uchun unikal `id` yaratish.
- Tartib raqamlari yoki ketma-ket qiymatlar ishlab chiqarish.

---

## 2. Sequence tuzilmasi (Structure)

Sequence yaratish uchun `CREATE SEQUENCE` buyrug'i ishlatiladi. Quyida uning sintaksisi va parametrlar tushuntiriladi.

### Sintaksis:
```sql
CREATE SEQUENCE [IF NOT EXISTS] sequence_name
    [AS { SMALLINT | INTEGER | BIGINT }]
    [START WITH start_value]
    [INCREMENT BY increment_value]
    [MINVALUE min_value | NO MINVALUE]
    [MAXVALUE max_value | NO MAXVALUE]
    [CYCLE | NO CYCLE]
    [CACHE cache_value | NO CACHE]
    [OWNED BY table_name.column_name];
```

### Parametrlar:
- **`sequence_name`**: Sequence nomi (masalan, `user_id_seq`).
- **`AS data_type`**: Qiymatlar turi (`SMALLINT`, `INTEGER`, `BIGINT`). Standart: `BIGINT`.
- **`START WITH`**: Boshlang'ich qiymat (standart: 1).
- **`INCREMENT BY`**: Har bir qadamning o'sishi/kamayishi (standart: 1).
- **`MINVALUE`**: Eng kichik qiymat (standart: turi bo'yicha minimal).
- **`MAXVALUE`**: Eng katta qiymat (standart: turi bo'yicha maksimal).
- **`CYCLE`**: Maksimal qiymatga yetganda boshidan boshlash.
- **`NO CYCLE`**: Maksimal qiymatda xato chiqarish (standart).
- **`CACHE`**: Oldindan saqlanadigan qiymatlar soni (tezlik uchun). Standart: 1.
- **`OWNED BY`**: Sequence-ni jadval ustuniga bog'lash (jadval o'chirilganda Sequence ham o'chadi).

### Misol: Oddiy Sequence yaratish
```sql
CREATE SEQUENCE my_sequence
    START WITH 1
    INCREMENT BY 1
    MINVALUE 1
    MAXVALUE 1000
    NO CYCLE;
```

---

## 3. Sequence bilan ishlash funksiyalari

Sequence-ni boshqarish uchun maxsus funksiyalar mavjud:

| Funksiya                     | Tavsif                                                                 |
|------------------------------|----------------------------------------------------------------------|
| `nextval('sequence_name')`   | Keyingi qiymatni oladi va Sequence-ni bir qadam oshiradi.            |
| `currval('sequence_name')`   | Joriy sessiyada oxirgi olingan qiymatni qaytaradi.                   |
| `setval('sequence_name', n)` | Sequence-ning joriy qiymatini `n` ga o'rnatadi.                      |
| `lastval()`                  | Joriy sessiyada oxirgi ishlatilgan Sequence qiymatini qaytaradi.     |

### Misol: Funksiyalardan foydalanish
```sql
-- Sequence yaratish
CREATE SEQUENCE test_seq START WITH 10 INCREMENT BY 5;

-- Keyingi qiymatlarni olish
SELECT nextval('test_seq'); -- 10
SELECT nextval('test_seq'); -- 15
SELECT nextval('test_seq'); -- 20

-- Joriy qiymatni ko'rish
SELECT currval('test_seq'); -- 20

-- Qiymatni qayta o'rnatish
SELECT setval('test_seq', 100);
SELECT nextval('test_seq'); -- 100
```

---

## 4. Sequence va SERIAL

`SERIAL` ma'lumot turi ichki Sequence ob'ektini yaratadi va uni ustunga avtomatik bog'laydi. `SERIAL` turlari:
- `SMALLSERIAL`: 1 dan 32767 gacha.
- `SERIAL`: 1 dan 2147483647 gacha.
- `BIGSERIAL`: 1 dan 9223372036854775807 gacha.

### Misol: SERIAL bilan jadval
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

#### Ichki jarayon:
1. `users_id_seq` nomli Sequence yaratiladi.
2. `id` ustuni `INTEGER` turi sifatida belgilanadi.
3. `DEFAULT` qiymati sifatida `nextval('users_id_seq')` o'rnatiladi.

#### Ma'lumot kiritish:
```sql
INSERT INTO users (name) VALUES ('Ali'); -- id = 1
INSERT INTO users (name) VALUES ('Vali'); -- id = 2
SELECT * FROM users;
```
**Natija:**
```
 id | name
----+------
  1 | Ali
  2 | Vali
```

---

## 5. Sequence-ni boshqarish

### 5.1. Sequence-ni o'zgartirish (`ALTER SEQUENCE`)
```sql
ALTER SEQUENCE sequence_name
    [RESTART WITH new_start_value]
    [INCREMENT BY new_increment]
    [MINVALUE new_min_value | NO MINVALUE]
    [MAXVALUE new_max_value | NO MAXVALUE]
    [CYCLE | NO CYCLE];
```

#### Misol: Sequence-ni qayta sozlash
```sql
ALTER SEQUENCE test_seq RESTART WITH 50;
SELECT nextval('test_seq'); -- 50
```

### 5.2. Sequence-ni o'chirish (`DROP SEQUENCE`)
```sql
DROP SEQUENCE IF EXISTS test_seq;
```

---

## 6. Amaliy misollar

### Misol 1: Maxsus Sequence bilan buyurtmalar jadvali
Maxsus Sequence yaratib, uni buyurtmalar jadvaliga bog'laymiz.

```sql
-- Sequence yaratish
CREATE SEQUENCE order_seq
    START WITH 1000
    INCREMENT BY 10
    MINVALUE 1000
    MAXVALUE 9999
    NO CYCLE;

-- Jadval yaratish
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY DEFAULT nextval('order_seq'),
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

### Misol 2: Bir Sequence-ni bir nechta jadvallar uchun ishlatish
Bitta Sequence-ni ikki jadvalda ishlatamiz.

```sql
-- Sequence yaratish
CREATE SEQUENCE global_seq START WITH 1 INCREMENT BY 1;

-- Birinchi jadval
CREATE TABLE products (
    id INTEGER PRIMARY KEY DEFAULT nextval('global_seq'),
    name VARCHAR(100)
);

-- Ikkinchi jadval
CREATE TABLE categories (
    id INTEGER PRIMARY KEY DEFAULT nextval('global_seq'),
    description TEXT
);

-- Ma'lumot kiritish
INSERT INTO products (name) VALUES ('Laptop'); -- id = 1
INSERT INTO categories (description) VALUES ('Electronics'); -- id = 2
INSERT INTO products (name) VALUES ('Phone'); -- id = 3

-- Natijani ko'rish
SELECT * FROM products;
SELECT * FROM categories;
```

**Natija:**
```
 id |  name
----+--------
  1 | Laptop
  3 | Phone

 id | description
----+-------------
  2 | Electronics
```

### Misol 3: Sequence-ni CYCLE bilan ishlatish
Maksimal qiymatga yetganda boshidan boshlaydigan Sequence.

```sql
CREATE SEQUENCE cycle_seq
    START WITH 1
    INCREMENT BY 1
    MINVALUE 1
    MAXVALUE 5
    CYCLE;

SELECT nextval('cycle_seq'); -- 1
SELECT nextval('cycle_seq'); -- 2
SELECT nextval('cycle_seq'); -- 3
SELECT nextval('cycle_seq'); -- 4
SELECT nextval('cycle_seq'); -- 5
SELECT nextval('cycle_seq'); -- 1 (CYCLE tufayli qayta boshlanadi)
```

### Misol 4: Sequence-ni tranzaksiyada ishlatish
Sequence tranzaksiyadan mustaqil ekanligini ko'rsatamiz.

```sql
CREATE SEQUENCE temp_seq START WITH 1;

BEGIN;
SELECT nextval('temp_seq'); -- 1
SELECT nextval('temp_seq'); -- 2
ROLLBACK;

SELECT nextval('temp_seq'); -- 3 (ROLLBACK bo'lsa ham qiymatlar yo'qoladi)
```

---

## 7. E'tibor beriladigan jihatlar

- **Bo'shliqlar (Gaps)**: Tranzaksiya `ROLLBACK` qilinsa, `nextval` ishlatilgan qiymatlar qaytarilmaydi, bu raqamlarda bo'shliqlar hosil qiladi.
- **CACHE sozlamasi**: `CACHE` qiymatini oshirish (masalan, `CACHE 100`) so'rovlar tezligini oshiradi, lekin xato yuz bersa ko'proq raqam yo'qotiladi.
- **CYCLE xavfi**: `CYCLE` ishlatilsa, eski qiymatlar qayta ishlatilishi mumkin, bu unikal identifikatorlarda muammo keltirib chiqaradi.
- **UUID alternativi**: Agar global unikal identifikator kerak bo'lsa, `UUID` ma'lumot turini ko'rib chiqing.
- **Cheklovlar**: Sequence qiymatlari `UNIQUE` yoki `PRIMARY KEY` cheklovlariga mos kelishi kerak.

---

## 8. Xulosa

PostgreSQL-da Sequence unikal identifikatorlar ishlab chiqarish uchun samarali va moslashuvchan vositadir. U `SERIAL` bilan avtomatik ishlatilishi yoki maxsus ehtiyojlar uchun mustaqil sozlanishi mumkin. `nextval`, `currval`, `setval` kabi funksiyalar Sequence-ni boshqarishda muhim rol o'ynaydi. Yuqoridagi misollar turli holatlarda Sequence-ni qanday qo'llash mumkinligini ko'rsatadi.
