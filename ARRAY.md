# PostgreSQL ARRAY Ma'lumot Turi Misollari
***
## 1. Oddiy TEXT ARRAY ustuni
**Tavsif**: Foydalanuvchi teglarini saqlash uchun oddiy `TEXT[]` massiv ustuni yaratish.

**Misol**:
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    tags TEXT[]
);

-- Ma'lumot kiritish
INSERT INTO users (username, tags) VALUES ('Alice', ARRAY['admin', 'active']);
INSERT INTO users (username, tags) VALUES ('Bob', '{user, inactive}');
INSERT INTO users (username, tags) VALUES ('Charlie', NULL);

-- Natijani tekshirish
SELECT * FROM users;
```

**Natija**:
```
 id | username |       tags
----|---------|-------------------
  1 | Alice   | {admin,active}
  2 | Bob     | {user,inactive}
  3 | Charlie | NULL
```

**Eslatma**:
- `ARRAY['value1', 'value2']` yoki `'{value1, value2}'` formatlari ishlatilishi mumkin.
- `NULL` massiv butunlay yo'q qiymatni anglatadi.
***
## 2. INTEGER ARRAY bilan ishlash
**Tavsif**: Talaba baholarini saqlash uchun `INTEGER[]` massiv ustuni yaratish va elementlarga indeks orqali murojaat qilish.

**Misol**:
```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    grades INTEGER[]
);

-- Ma'lumot kiritish
INSERT INTO students (name, grades) VALUES ('Alice', ARRAY[85, 90, 92]);
INSERT INTO students (name, grades) VALUES ('Bob', '{78, 65, 70}');

-- Birinchi bahoga murojaat
SELECT name, grades[1] AS first_grade FROM students;
```

**Natija**:
```
 name  | first_grade
-------|-------------
 Alice | 85
 Bob   | 78
```

**Eslatma**:
- Indekslar PostgreSQL-da odatda 1 dan boshlanadi.
- Massiv elementlariga `array_name[index]` orqali murojaat qilinadi.
***
## 3. Ko'p o'lchovli ARRAY
**Tavsif**: Matritsani saqlash uchun ikki o'lchovli `INTEGER[][]` massiv ustuni yaratish.

**Misol**:
```sql
CREATE TABLE matrices (
    id SERIAL PRIMARY KEY,
    matrix INTEGER[][]
);

-- Ma'lumot kiritish
INSERT INTO matrices (matrix) VALUES (ARRAY[[1, 2], [3, 4]]);

-- Natijani tekshirish
SELECT * FROM matrices;
```

**Natija**:
```
 id |     matrix
----|----------------
  1 | {{1,2},{3,4}}
```

**Eslatma**:
- Ko'p o'lchovli massivlar murakkab tuzilmalarni saqlash uchun foydali.
- Har bir o'lcham uchun `array_name[i][j]` sintaksisi ishlatiladi.
***
## 4. ARRAY funksiyalari va operatorlari
**Tavsif**: PostgreSQL-da `ARRAY` bilan ishlash uchun maxsus funksiyalar va operatorlar, masalan, uzunlikni olish, elementlarni qatorlarga aylantirish, muayyan qiymatni qidirish, `unnest` funksiyasi va arraylarni birlashtirish.

**Misol**:
```sql
-- array_length: Massiv uzunligini olish
SELECT name, array_length(grades, 1) AS grade_count FROM students;
```
**Natija**:
```
 name  | grade_count
-------|-------------
 Alice | 3
 Bob   | 3
```
***
```sql
-- unnest: Massiv elementlarini alohida qatorlarga aylantirish
SELECT name, unnest(grades) AS grade FROM students;
```
**Natija**:
```
 name  | grade
-------|-------
 Alice | 85
 Alice | 90
 Alice | 92
 Bob   | 78
 Bob   | 65
 Bob   | 70
```
***
```sql
-- ANY: Massivda muayyan qiymat borligini tekshirish
SELECT name FROM students WHERE 90 = ANY(grades);
```
**Natija**:
```
 name
-------
 Alice
```
***
```sql
-- Array birlashtirish (|| operatori)
SELECT ARRAY[1, 2] || ARRAY[3, 4] AS combined_array;
```
**Natija**:
```
 combined_array
----------------
 {1,2,3,4}
```
***
```sql
-- Array birlashtirish (ustunlar bilan)
SELECT username, tags || ARRAY['new'] AS updated_tags FROM users WHERE username = 'Alice';
```
**Natija**:
```
 username |       updated_tags
----------|------------------------
 Alice    | {admin,active,new}
```
***
```sql
-- unnest bilan ko'p massivlarni birlashtirish
SELECT unnest(ARRAY['a', 'b'] || ARRAY['c', 'd']) AS element;
```
**Natija**:
```
 element
---------
 a
 b
 c
 d
```

**Eslatma**:
- `unnest` massiv elementlarini qatorlarga aylantirishda foydali, ayniqsa katta massivlar bilan ishlashda.
- `||` operatori ikki yoki undan ko'p massivlarni birlashtiradi.
- `GIN` indekslari `ANY` va o'xshash so'rovlar uchun samarali:
  ```sql
  CREATE INDEX idx_users_tags ON users USING GIN(tags);
  ```
***
## 5. CHECK cheklovi bilan ARRAY
**Tavsif**: Massivning uzunligini yoki tarkibini cheklash uchun `CHECK` cheklovini qo'llash.

**Misol**:
```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    categories TEXT[] NOT NULL CHECK (array_length(categories, 1) >= 1)
);

-- Muvaffaqiyatli kiritish
INSERT INTO products (name, categories) VALUES ('Laptop', ARRAY['electronics', 'computers']);

-- Xato holati
INSERT INTO products (name, categories) VALUES ('Phone', '{}');
```

**Natija**:
```
 id |  name  |      categories
----|--------|----------------------
  1 | Laptop | {electronics,computers}
```
***
**Xato holati**:
```
ERROR: new row for relation "products" violates check constraint
```

**Eslatma**:
- `CHECK` cheklovi massivning bo'sh bo'lmasligini yoki muayyan shartlarga javob berishini ta'minlaydi.
- Masalan, `CHECK (array_length(categories, 1) <= 5)` 5 tagacha cheklaydi.

## 6. ALTER TABLE bilan ARRAY qo'shish
**Tavsif**: Mavjud jadvalga `ARRAY` ustuni qo'shish va standart qiymat o'rnatish.

**Misol**:
```sql
ALTER TABLE users
ADD COLUMN roles TEXT[] DEFAULT ARRAY['user'];
```

**Tushuntirish**:
- `roles` yangi qatorlar uchun standart `["user"]` qiymatiga ega bo'ladi.

**Misol (Ma'lumot kiritish)**:
```sql
INSERT INTO users (username) VALUES ('David');

-- Natijani tekshirish
SELECT username, roles FROM users WHERE username = 'David';
```

**Natija**:
```
 username | roles
----------|--------
 David    | {user}
```

**Eslatma**:
- `DEFAULT` qiymati massiv sifatida to'g'ri formatda bo'lishi kerak.
- Agar `NOT NULL` qo'shilsa, eski qatorlar uchun standart qiymat talab qilinadi.
