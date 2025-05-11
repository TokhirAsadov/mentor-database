# PostgreSQL JSONB Ma'lumot Turi Misollari

PostgreSQL-da `JSONB` ma'lumot turi yarim tuzilgan ma'lumotlarni saqlash va so'rov qilish uchun ishlatiladi. U JSON formatidagi ma'lumotlarni ikkilik (binary) shaklda saqlaydi, bu esa tez so'rovlar va indekslash uchun optimallashtirilgan. Quyida `JSONB` ma'lumot turi bilan ishlashning muayyan misollari keltirilgan: oddiy `JSONB` ustuni yaratishdan tortib, `CHECK` cheklovi bilan ishlashgacha.
***
## 1. JSONB ustuni yaratish
**Tavsif**: Mahsulot ma'lumotlarini saqlash uchun `JSONB` ustuni yaratish.

**Misol**:
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    details JSONB
);

-- Ma'lumot kiritish
INSERT INTO products (name, details) VALUES ('Laptop', '{"brand": "Dell", "ram": 16, "color": "Silver"}');
INSERT INTO products (name, details) VALUES ('Phone', jsonb_build_object('brand', 'Apple', 'model', 'iPhone 14'));
INSERT INTO products (name, details) VALUES ('Tablet', NULL);

-- Natijani tekshirish
SELECT * FROM products;
```

**Natija**:
```
 id |  name  |                       details
----|--------|--------------------------------------------
  1 | Laptop | {"ram": 16, "brand": "Dell", "color": "Silver"}
  2 | Phone  | {"model": "iPhone 14", "brand": "Apple"}
  3 | Tablet | NULL
```

**Eslatma**:
- `jsonb_build_object` funksiyasi JSONB ob'ektlarini dinamik yaratishda qulay.
- `NULL` qiymat butunlay yo'q JSONB ob'ektini anglatadi.
***
## 2. JSONB operatorlari
**Tavsif**: JSONB bilan ishlash uchun maxsus operatorlar, masalan, kalit bo'yicha qiymat olish, tarkibni tekshirish va ichki kalitlarga murojaat qilish.

**Misol**:
```sql
-- Kalit bo'yicha qiymat olish (-> va ->>)
SELECT name, details -> 'brand' AS brand_json, details ->> 'brand' AS brand_text FROM products;
```
**Natija**:
```
 name  | brand_json | brand_text
-------|------------|------------
 Laptop | "Dell"     | Dell
 Phone  | "Apple"    | Apple
 Tablet | NULL       | NULL
```

```sql
-- Tarkibni tekshirish (@>)
SELECT name FROM products WHERE details @> '{"brand": "Dell"}';
```
**Natija**:
```
 name
-------
 Laptop
```

```sql
-- Ichki kalitga murojaat (#>)
INSERT INTO products (name, details)
VALUES ('Desktop', '{"brand": "HP", "features": {"cpu": "Intel"}}');

SELECT name, details #> '{features, cpu}' AS cpu FROM products
WHERE details @> '{"features": {"cpu": "Intel"}}';
```
**Natija**:
```
 name    |  cpu
---------|--------
 Desktop | "Intel"
```

**Eslatma**:
- `->`: JSON ob'ekti sifatida qaytaradi.
- `->>`: Matn sifatida qaytaradi.
- `@>`: JSONB ob'ekti muayyan tarkibni o'z ichiga olishini tekshiradi.
- `#>`: Ichki kalitlarga yo'l orqali murojaat qiladi.
***
## 3. JSONB funksiyalari
**Tavsif**: JSONB bilan ishlash uchun foydali funksiyalar, masalan, JSONB ob'ekti yaratish, qiymatlarni yangilash va massiv elementlarini qatorlarga aylantirish.

**Misol**:
```sql
-- jsonb_build_object: JSONB ob'ekti yaratish
INSERT INTO products (name, details)
VALUES ('Camera', jsonb_build_object('brand', 'Canon', 'price', 500));
```

```sql
-- jsonb_set: JSONB qiymatini yangilash
UPDATE products
SET details = jsonb_set(details, '{stock}', '10')
WHERE name = 'Laptop';

-- Natijani tekshirish
SELECT name, details FROM products WHERE name = 'Laptop';
```
**Natija**:
```
 name  |                       details
-------|--------------------------------------------
 Laptop | {"ram": 16, "brand": "Dell", "color": "Silver", "stock": "10"}
```

```sql
-- jsonb_array_elements: Massiv elementlarini qatorlarga aylantirish
INSERT INTO products (name, details)
VALUES ('TV', '{"brand": "Samsung", "features": ["4K", "Smart"]}');

SELECT name, jsonb_array_elements(details -> 'features') AS feature
FROM products
WHERE details @> '{"features": []}';
```
**Natija**:
```
 name | feature
------|---------
 TV   | "4K"
 TV   | "Smart"
```

**Eslatma**:
- `jsonb_build_object` dinamik JSONB ob'ektlarini yaratishda qulay.
- `jsonb_set` muayyan kalitni yangilash uchun ishlatiladi.
- `jsonb_array_elements` JSONB massivlarini qatorlarga aylantirishda foydali.
***
## 4. CHECK cheklovi bilan JSONB
**Tavsif**: JSONB ob'ekti ekanligini yoki muayyan shartlarga javob berishini cheklash uchun `CHECK` cheklovini qo'llash.

**Misol**:
```sql
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    metadata JSONB NOT NULL CHECK (jsonb_typeof(metadata) = 'object')
);

-- Muvaffaqiyatli kiritish
INSERT INTO items (name, metadata) VALUES ('Book', '{"category": "Literature"}');

-- Xato holati
INSERT INTO items (name, metadata) VALUES ('Pen', '["Blue"]');
```

**Natija**:
```
 id | name |           metadata
----|------|-------------------------
  1 | Book | {"category": "Literature"}
```

**Xato holati**:
```
ERROR: new row for relation "items" violates check constraint
```

**Eslatma**:
- `jsonb_typeof` funksiyasi JSONB qiymatining turini tekshiradi (`object`, `array`, `string`, `number`, `boolean`, `null`).
- `CHECK` cheklovi JSONB qiymatining tuzilishini cheklashda foydali.
