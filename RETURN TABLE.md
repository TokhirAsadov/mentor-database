# PostgreSQL'da RETURNS TABLE haqida ma'lumot

## 1. RETURNS TABLE haqida umumiy ma'lumot

**RETURNS TABLE** PostgreSQL funksiyalarida natija sifatida jadval shaklidagi ma'lumotlarni qaytarish uchun ishlatiladi. Bu funksiyalar SQL so'rovlarida (`SELECT`, `FROM` va boshqa joylarda) ishlatilishi mumkin va natija to'plami (result set) sifatida qaytariladi. **RETURNS TABLE** odatda quyidagi holatlarda ishlatiladi:

- Murakkab so'rovlar natijasini qaytarish.
- Ma'lumotlarni qayta ishlash va filtrlangan natijalarni jadval shaklida taqdim etish.
- Bir nechta ustun va qatorlardan iborat ma'lumotlarni qaytarish.
- Dinamik yoki shartli ma'lumotlar to'plamini yaratish.

**RETURNS TABLE** bilan yaratilgan funksiyalar `RETURNS SETOF` bilan solishtirganda ancha qulay, chunki u aniq ustun nomlari va turlarini belgilash imkonini beradi.

## 2. RETURNS TABLE sintaksisi

```sql
CREATE [OR REPLACE] FUNCTION function_name ([parameter_name [IN | OUT | INOUT] parameter_type [, ...]])
RETURNS TABLE (column_name column_type [, ...])
AS $$
DECLARE
  -- O'zgaruvchilar deklaratsiyasi (ixtiyoriy)
BEGIN
  -- Funksiya logikasi
  RETURN QUERY [query];
  -- yoki
  RETURN QUERY EXECUTE [dynamic_query];
END;
$$ LANGUAGE plpgsql
[ VOLATILE | STABLE | IMMUTABLE ]
[ STRICT ]
[ SECURITY DEFINER | SECURITY INVOKER ];
```

### Tushuntirish:
- **`RETURNS TABLE (column_name column_type [, ...])`**: Funksiya qaytaradigan jadvalning tuzilishini belgilaydi. Har bir ustun nomi va ma'lumot turi aniq ko'rsatiladi.
- **`RETURN QUERY`**: Funksiyada SQL so'rov natijasini qaytarish uchun ishlatiladi. Bu statik so'rovlar uchun ishlatiladi.
- **`RETURN QUERY EXECUTE`**: Dinamik SQL so'rovlarining natijasini qaytarish uchun ishlatiladi.
- **Atributlar**:
  - `VOLATILE` (standart): Funksiya har safar har xil natija qaytarishi mumkin.
  - `STABLE`: Funksiya tranzaksiya ichida barqaror natija qaytaradi.
  - `IMMUTABLE`: Funksiya har doim bir xil kirish uchun bir xil natija qaytaradi.
  - `STRICT`: Agar kirish parametrlaridan biri `NULL` bo'lsa, funksiya `NULL` qaytaradi.
  - `SECURITY DEFINER`: Funksiyani yaratuvchining ruxsatlari bilan ishlaydi.
  - `SECURITY INVOKER` (standart): Funksiyani chaqiruvchining ruxsatlari bilan ishlaydi.

## 3. RETURNS TABLE vs. RETURNS SETOF

PostgreSQL'da jadval qaytaruvchi funksiyalar yaratishning ikki asosiy usuli mavjud: **RETURNS TABLE** va **RETURNS SETOF**. Ular o'rtasidagi farqlar:

| Xususiyat              | **RETURNS TABLE**                              | **RETURNS SETOF**                              |
|------------------------|-----------------------------------------------|-----------------------------------------------|
| **Ustunlar deklaratsiyasi** | Aniqlangan ustun nomlari va turlari bilan ishlaydi | Mavjud jadval yoki kompozit turga bog'lanadi |
| **Foydalanish qulayligi** | Ustun tuzilishini aniq belgilash imkoniyati | Jadval yoki turga bog'langan bo'lishi kerak |
| **Moslashuvchanlik**    | Ko'proq moslashuvchan, chunki har qanday ustun tuzilishi yaratiladi | Faqat mavjud tuzilmalarga moslashadi |
| **Misal**               | `RETURNS TABLE (id integer, name text)`      | `RETURNS SETOF users`                        |

**RETURNS TABLE** odatda ko'proq afzal qilinadi, chunki u aniqroq va moslashuvchan.

## 4. RETURNS TABLE misollari

### Oddiy RETURNS TABLE funksiyasi
Muayyan yoshdan katta foydalanuvchilarni qaytaruvchi funksiya:
```sql
CREATE OR REPLACE FUNCTION get_users_older_than(min_age integer)
RETURNS TABLE (id integer, username text, age integer) AS $$
BEGIN
  RETURN QUERY
    SELECT u.id, u.username, EXTRACT(YEAR FROM AGE(CURRENT_DATE, u.birth_date))::integer
    FROM users u
    WHERE EXTRACT(YEAR FROM AGE(CURRENT_DATE, u.birth_date)) > min_age;
END;
$$ LANGUAGE plpgsql STABLE;

-- Chaqirish
SELECT * FROM get_users_older_than(30);
```

**Natija** (agar `users` jadvali mavjud bo'lsa):
```
 id | username    | age
----+-------------+-----
 1  | alexpeter   | 35
 2  | john.doe    | 32
```

### Dinamik SQL bilan RETURNS TABLE
Jadval nomini parametr sifatida oluvchi funksiya:
```sql
CREATE OR REPLACE FUNCTION get_table_data(table_name text)
RETURNS TABLE (row_id integer, row_data text) AS $$
BEGIN
  RETURN QUERY EXECUTE
    format('SELECT id, name FROM %I', table_name);
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT * FROM get_table_data('users');
```

**Natija**:
```
 row_id | row_data
--------+-----------
 1      | alexpeter
 2      | john.doe
```

**Eslatma**: Dinamik SQL ishlatganda SQL injection xavfini oldini olish uchun `format` funksiyasi va `%I` ishlatiladi.

### Xato boshqaruvi bilan RETURNS TABLE
Xato ushlovchi va filtrlangan natijalarni qaytaruvchi funksiya:
```sql
CREATE OR REPLACE FUNCTION get_active_users(min_age integer)
RETURNS TABLE (id integer, username text, status text) AS $$
BEGIN
  IF min_age < 0 THEN
    RAISE EXCEPTION 'Yosh salbiy bo''lmasligi kerak: %', min_age;
  END IF;

  RETURN QUERY
    SELECT u.id, u.username, u.status
    FROM users u
    WHERE EXTRACT(YEAR FROM AGE(CURRENT_DATE, u.birth_date)) > min_age
    AND u.status = 'active';
EXCEPTION
  WHEN undefined_table THEN
    RAISE NOTICE 'Jadval topilmadi';
    RETURN QUERY SELECT 0 AS id, 'no data' AS username, 'inactive' AS status;
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT * FROM get_active_users(25);
```

**Natija**:
```
 id | username    | status
----+-------------+--------
 1  | alexpeter   | active
 3  | jane.smith  | active
```

Agar jadval topilmasa:
```
NOTICE:  Jadval topilmadi
 id | username | status
----+---------+---------
 0  | no data | inactive
```

### Bir nechta RETURN QUERY ishlatish
Bir funksiyada bir nechta so'rov natijalarini birlashtirish:
```sql
CREATE OR REPLACE FUNCTION get_users_by_status(status_filter text)
RETURNS TABLE (id integer, username text, status text) AS $$
BEGIN
  RETURN QUERY
    SELECT u.id, u.username, u.status
    FROM users u
    WHERE u.status = status_filter;

  RETURN QUERY
    SELECT u.id, u.username, u.status
    FROM users u
    WHERE u.status = 'pending'
    AND status_filter = 'all';
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT * FROM get_users_by_status('active');
```

**Natija**:
```
 id | username    | status
----+-------------+--------
 1  | alexpeter   | active
 3  | jane.smith  | active
```

### OUT parametrlari bilan kombinatsiya
`RETURNS TABLE` o'rniga `OUT` parametrlari ishlatilishi mumkin, ammo `RETURNS TABLE` aniqroq:
```sql
CREATE OR REPLACE FUNCTION get_user_details(user_id integer, OUT id integer, OUT username text, OUT email text)
RETURNS SETOF record AS $$
BEGIN
  RETURN QUERY
    SELECT u.id, u.username, u.email
    FROM users u
    WHERE u.id = user_id;
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT * FROM get_user_details(1);
```

**Natija**:
```
 id | username   | email
----+------------+------------------
 1  | alexpeter  | alex@example.com
```

**Eslatma**: `RETURNS TABLE` `OUT` parametrlarga nisbatan aniqroq, chunki ustun nomlari va turlari to'g'ridan-to'g'ri belgilanadi.

## 5. Foydalanish usullari

1. **So'rovlar ichida ishlatish**:
   `RETURNS TABLE` funksiyalari oddiy jadvallar kabi SQL so'rovlarida ishlatilishi mumkin:
   ```sql
   SELECT u.username, u.age
   FROM get_users_older_than(30) u
   WHERE u.age < 40;
   ```

2. **Dinamik so'rovlar**:
   `RETURN QUERY EXECUTE` yordamida dinamik SQL so'rovlarini qaytarish mumkin, bu moslashuvchanlikni oshiradi.

3. **Kursorlar bilan ishlash**:
   Funksiya ichida kursorlardan foydalanib, katta ma'lumotlar to'plamini qatorlarga bo'lib qaytarish mumkin:
   ```sql
   CREATE OR REPLACE FUNCTION get_users_cursor()
   RETURNS TABLE (id integer, username text) AS $$
   DECLARE
     cur CURSOR FOR SELECT id, username FROM users;
     rec RECORD;
   BEGIN
     FOR rec IN cur LOOP
       id := rec.id;
       username := rec.username;
       RETURN NEXT;
     END LOOP;
   END;
   $$ LANGUAGE plpgsql;

   -- Chaqirish
   SELECT * FROM get_users_cursor();
   ```

4. **Agregat funksiyalar bilan**:
   `RETURNS TABLE` funksiyalari agregat natijalarni qaytarishda ishlatilishi mumkin:
   ```sql
   CREATE OR REPLACE FUNCTION get_stats_by_department()
   RETURNS TABLE (department text, avg_age numeric) AS $$
   BEGIN
     RETURN QUERY
       SELECT u.department, AVG(EXTRACT(YEAR FROM AGE(CURRENT_DATE, u.birth_date)))
       FROM users u
       GROUP BY u.department;
   END;
   $$ LANGUAGE plpgsql;

   -- Chaqirish
   SELECT * FROM get_stats_by_department();
   ```

**Natija**:
```
 department | avg_age
------------+---------
 IT         | 32.5
 HR         | 28.7
```

## 6. Foydali maslahatlar

1. **Ustun turlarini aniq belgilash**:
   `RETURNS TABLE` da har bir ustun turi aniq ko'rsatilishi kerak. Agar noto'g'ri tur ishlatilsa, xato yuzaga keladi.

2. **RETURN QUERY dan foydalanish**:
   `RETURN QUERY` statik so'rovlar uchun, `RETURN QUERY EXECUTE` esa dinamik so'rovlar uchun ishlatiladi. Dinamik so'rovlar uchun `format` funksiyasidan foydalanib, SQL injection xavfini oldini oling.

3. **Xato boshqaruvi**:
   `EXCEPTION` bloklaridan foydalanib, xatolarni ushlang va foydalanuvchiga tushunarli xabarlar qaytaring:
   ```sql
   EXCEPTION
     WHEN undefined_column THEN
       RAISE NOTICE 'Ustun topilmadi';
       RETURN QUERY SELECT 0 AS id, 'error' AS username;
   ```

4. **Optimallashtirish**:
   - `STABLE` yoki `IMMUTABLE` atributlarini to'g'ri ishlatish orqali so'rov optimallashtiruvchisiga yordam bering.
   - Katta ma'lumotlar to'plamlari bilan ishlashda indekslardan foydalaning.
   - `COST` va `ROWS` atributlarini to'g'ri belgilang, masalan:
     ```sql
     CREATE FUNCTION get_users_older_than(min_age integer)
     RETURNS TABLE (id integer, username text, age integer)
     AS $$ ... $$
     LANGUAGE plpgsql STABLE
     COST 100
     ROWS 1000;
     ```

5. **Xavfsizlik**:
   - `SECURITY DEFINER` ishlatganda, funksiya yaratuvchining ruxsatlari bilan ishlaydi, shuning uchun ehtiyotkor bo'ling.
   - Dinamik SQL ishlatganda, `format` va `%I`/`%L` dan foydalaning.

6. **Loglash**:
   - Funksiya xatolarini logga yozish uchun `RAISE LOG` yoki `INSERT` ishlatish mumkin:
     ```sql
     RAISE LOG 'Funksiya % chaqirildi, parametr: %', function_name, min_age;
     ```

7. **Kursorlardan foydalanish**:
   Katta ma'lumotlar to'plamlari bilan ishlashda `RETURN NEXT` bilan kursorlardan foydalanish xotirani tejashga yordam beradi.

## 7. Xulosa

**RETURNS TABLE** PostgreSQL'da PL/pgSQL funksiyalarida jadval shaklidagi natijalarni qaytarish uchun ishlatiladi. U aniq ustun nomlari va turlari bilan moslashuvchanlikni ta'minlaydi. `RETURN QUERY` va `RETURN QUERY EXECUTE` yordamida statik yoki dinamik so'rov natijalari qaytariladi. Xato boshqaruvi (`EXCEPTION`), optimallashtirish (`STABLE`, `COST`, `ROWS`) va xavfsizlik (`SECURITY DEFINER`, `format`) masalalariga e'tibor berish muhim. **RETURNS TABLE** `RETURNS SETOF` ga nisbatan ancha qulay va zamonaviy hisoblanadi.
