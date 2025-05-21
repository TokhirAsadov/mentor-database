# PostgreSQL'da CREATE FUNCTION haqida ma'lumot

## 1. CREATE FUNCTION sintaksisi

```sql
CREATE [OR REPLACE] FUNCTION function_name ([parameter_name [IN | OUT | INOUT] parameter_type [, ...]])
RETURNS return_type
AS $$
DECLARE
  -- O'zgaruvchilar deklaratsiyasi (ixtiyoriy)
BEGIN
  -- Funksiya logikasi
END;
$$ LANGUAGE plpgsql
[ VOLATILE | STABLE | IMMUTABLE ]
[ STRICT ]
[ SECURITY DEFINER | SECURITY INVOKER ]
[ COST cost ]
[ ROWS rows ];
```

### Tushuntirish:
- **`CREATE OR REPLACE`**: Agar funksiya allaqachon mavjud bo'lsa, uni yangilaydi.
- **`function_name`**: Funksiyaning nomi (odatda schema bilan birga, masalan, `public.my_function`).
- **`parameter_name`**: Funksiyaga uzatiladigan parametrlar. Har bir parametr quyidagi rejimlarga ega bo'lishi mumkin:
  - `IN`: Faqat kirish parametri (standart).
  - `OUT`: Chiqish parametri (funksiya qiymat qaytarmaydi, lekin OUT parametri orqali ma'lumot qaytaradi).
  - `INOUT`: Kirish va chiqish sifatida ishlatiladi.
- **`RETURNS return_type`**: Funksiya qaytaradigan ma'lumot turi (masalan, `integer`, `text`, `table(column_name type)`).
- **`AS $$ ... $$`**: Funksiya tanasini o'z ichiga oladi.
- **`DECLARE`**: O'zgaruvchilar, kursorlar yoki boshqa ob'ektlar deklaratsiya qilinadi.
- **`BEGIN ... END`**: Funksiya logikasi joylashadi.
- **`LANGUAGE plpgsql`**: Funksiya tili (PL/pgSQL, SQL, PL/Python va hokazo).
- **Atributlar**:
  - `VOLATILE` (standart): Funksiya har safar har xil natija qaytarishi mumkin (ma'lumotlarni o'zgartirishi mumkin).
  - `STABLE`: Funksiya bir tranzaksiya ichida barqaror natija qaytaradi.
  - `IMMUTABLE`: Funksiya har doim bir xil kirish uchun bir xil natija qaytaradi (masalan, matematik hisob-kitoblar).
  - `STRICT`: Agar har qanday kirish parametri `NULL` bo'lsa, funksiya avtomatik ravishda `NULL` qaytaradi.
  - `SECURITY DEFINER`: Funksiyani chaqiruvchining emas, funksiyani yaratuvchining ruxsatlari bilan ishlaydi.
  - `SECURITY INVOKER` (standart): Funksiyani chaqiruvchining ruxsatlari bilan ishlaydi.
  - `COST`: Funksiyaning taxminiy ishlash narxi (optimallashtirish uchun).
  - `ROWS`: Agar funksiya jadvallar qaytarsa, taxminiy qatorlar soni.

## 2. Funksiya turlari

1. **Skalyar funksiyalar**: Yagona qiymat qaytaradi (masalan, `integer`, `text`).
2. **Jadval funksiyalari**: Bir nechta qator va ustunlardan iborat natija to'plamini qaytaradi (`RETURNS TABLE` yoki `RETURNS SETOF`).
3. **Protsedural funksiyalar**: Ma'lumotlarni o'zgartirish yoki tranzaksiyalarni boshqarish uchun ishlatiladi (odatda `RETURNS void`).
4. **Agregat funksiyalar**: Maxsus agregat operatsiyalarni bajarish uchun (masalan, o'z agregat funksiyasini yaratish).

## 3. CREATE FUNCTION misollari

### Oddiy skalyar funksiya
Foydalanuvchi yoshini hisoblovchi funksiya:
```sql
CREATE OR REPLACE FUNCTION calculate_age(birth_date date)
RETURNS integer AS $$
BEGIN
  RETURN EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date));
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Chaqirish
SELECT calculate_age('1990-05-21');
```

**Natija** (2025-yil 21-may holatiga):
```
calculate_age
-------------
35
```

### Jadval funksiyasi
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

### OUT parametrlari bilan funksiya
Foydalanuvchi ma'lumotlarini qaytaruvchi funksiya:
```sql
CREATE OR REPLACE FUNCTION get_user_info(user_id integer, OUT username text, OUT email text)
AS $$
BEGIN
  SELECT u.username, u.email
  INTO username, email
  FROM users u
  WHERE u.id = user_id;
  
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Foydalanuvchi ID % topilmadi', user_id;
  END IF;
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT * FROM get_user_info(1);
```

**Natija**:
```
 username   | email
------------+------------------
 alexpeter  | alex@example.com
```

### Xato boshqaruvi bilan funksiya
Nolga bo'lish xatosini ushlovchi funksiya:
```sql
CREATE OR REPLACE FUNCTION safe_divide(a integer, b integer)
RETURNS integer AS $$
BEGIN
  ASSERT b != 0, 'Nolga bo''lish mumkin emas!';
  RETURN a / b;
EXCEPTION
  WHEN division_by_zero THEN
    RAISE NOTICE 'Xato: Nolga bo''lish urinildi';
    RETURN NULL;
  WHEN assertion_failure THEN
    RAISE NOTICE 'Xato: %', SQLERRM;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT safe_divide(10, 0);
```

**Natija**:
```
NOTICE:  Xato: Nolga bo'lish mumkin emas!
safe_divide
-------------
NULL
```

### Dinamik SQL bilan funksiya
Jadval nomini parametr sifatida oluvchi funksiya:
```sql
CREATE OR REPLACE FUNCTION count_rows(table_name text)
RETURNS bigint AS $$
DECLARE
  row_count bigint;
BEGIN
  EXECUTE format('SELECT COUNT(*) FROM %I', table_name) INTO row_count;
  RETURN row_count;
EXCEPTION
  WHEN undefined_table THEN
    RAISE NOTICE 'Jadval % topilmadi', table_name;
    RETURN 0;
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT count_rows('users');
```

**Natija**:
```
count_rows
------------
100
```

### Protsedural funksiya (VOID)
Ma'lumotlarni yangilovchi funksiya:
```sql
CREATE OR REPLACE FUNCTION update_user_email(user_id integer, new_email text)
RETURNS void AS $$
BEGIN
  UPDATE users
  SET email = new_email
  WHERE id = user_id;
  
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Foydalanuvchi ID % topilmadi', user_id;
  END IF;
END;
$$ LANGUAGE plpgsql;

-- Chaqirish
SELECT update_user_email(1, 'new.email@example.com');
```

## 4. Funksiyalarni boshqarish

1. **Funksiyani o'chirish**:
```sql
DROP FUNCTION function_name(parameter_types);
```
Masalan:
```sql
DROP FUNCTION calculate_age(date);
```

2. **Funksiyani yangilash**:
`CREATE OR REPLACE FUNCTION` yordamida funksiyani qayta yaratish.

3. **Funksiya ma'lumotlarini ko'rish**:
Funksiya haqida ma'lumot olish uchun:
```sql
SELECT * FROM information_schema.routines
WHERE routine_name = 'calculate_age';
```
Yoki:
```sql
\df calculate_age
```

## 5. Foydali maslahatlar

1. **Atributlardan to'g'ri foydalanish**:
   - `IMMUTABLE` ni faqat doim bir xil natija qaytaradigan funksiyalar uchun ishlating (masalan, matematik hisoblar).
   - `STABLE` ni tranzaksiya ichida barqaror natija qaytaradigan funksiyalar uchun ishlating.
   - `VOLATILE` ni ma'lumotlarni o'zgartiradigan yoki tasodifiy natijalar qaytaradigan funksiyalar uchun ishlating.

2. **Xavfsizlik**:
   - `SECURITY DEFINER` ni ehtiyotkorlik bilan ishlating, chunki u funksiyani yaratuvchining ruxsatlari bilan ishlaydi.
   - Dinamik SQL ishlatganda (`EXECUTE`), SQL injection xavfini oldini olish uchun `format` funksiyasi va `%I` (identifikatorlar uchun) ishlatishni unutmang.

3. **Xato boshqaruvi**:
   - `EXCEPTION` bloklaridan foydalanib, xatolarni ushlang va foydalanuvchiga tushunarli xabarlar qaytaring.
   - `ASSERT` bayonotlarini debugging uchun ishlating.

4. **Optimallashtirish**:
   - Funksiyaning ishlash narxini (`COST`) va qatorlar sonini (`ROWS`) to'g'ri belgilang, bu so'rov optimallashtiruvchisiga yordam beradi.
   - Katta jadvallarga dinamik so'rovlar yozganda indekslardan foydalaning.

5. **Loglash**:
   - Funksiya xatolarini logga yozish uchun `RAISE LOG` yoki `INSERT` ishlatish mumkin.
