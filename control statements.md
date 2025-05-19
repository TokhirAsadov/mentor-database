# PL/pgSQL’da Boshqaruv Iboralari

PL/pgSQL’da **control statements** dastur oqimini boshqarish uchun ishlatiladi. Ular shartli tekshiruvlar, tsikllar va dastur oqimini o‘zgartirish imkonini beradi. Bu iboralar yordamida murakkab biznes logikasi, qaror qabul qilish va takroriy vazifalarni amalga oshirish mumkin.

## 1. PL/pgSQL’da Boshqaruv Iboralarining Turlari

PL/pgSQL’da quyidagi asosiy boshqaruv iboralari mavjud:
1. **Shartli iboralar**:
   - `IF ... THEN ... ELSE`
   - `CASE`
2. **Tsikllar**:
   - `LOOP`
   - `FOR`
   - `WHILE`
   - `FOREACH` (massivlar uchun)
3. **O‘qimni boshqarish**:
   - `EXIT`
   - `CONTINUE`
   - `RETURN`
   - `RAISE`

---

## 2. Shartli Iboralar

Shartli iboralar shartlarga asoslangan qaror qabul qilish uchun ishlatiladi.

### 2.1. `IF ... THEN ... ELSE`

`IF` iborasi shart to‘g‘ri bo‘lsa, ma’lum bir kod blokini bajaradi.

**Sintaksis:**
```sql
IF shart THEN
    -- Kod bloki
ELSIF boshqa_shart THEN
    -- Boshqa kod bloki
ELSE
    -- Aks holda kod bloki
END IF;
```

**Misol:**
```sql
DO $$
DECLARE
    user_age INTEGER := 20;
BEGIN
    IF user_age < 18 THEN
        RAISE NOTICE 'Yosh 18 dan kichik';
    ELSIF user_age BETWEEN 18 AND 65 THEN
        RAISE NOTICE 'Yoshli foydalanuvchi';
    ELSE
        RAISE NOTICE 'Katta yoshli foydalanuvchi';
    END IF;
END;
$$;
```
**Tushuntirish**: Foydalanuvchi yoshiga qarab turli xabarlar chiqariladi.

---

### 2.2. `CASE`

`CASE` bir nechta shartlarni solishtirish va mos keladigan kod blokini bajarish uchun ishlatiladi. Ikki turi mavjud: **oddiy CASE** va **qidiruv CASE**.

**Oddiy CASE sintaksisi:**
```sql
CASE ifoda
    WHEN qiymat1 THEN
        -- Kod bloki
    WHEN qiymat2 THEN
        -- Boshqa kod bloki
    ELSE
        -- Aks holda kod bloki
END CASE;
```

**Qidiruv CASE sintaksisi:**
```sql
CASE
    WHEN shart1 THEN
        -- Kod bloki
    WHEN shart2 THEN
        -- Boshqa kod bloki
    ELSE
        -- Aks holda kod bloki
END CASE;
```

**Misol (Oddiy CASE):**
```sql
DO $$
DECLARE
    user_role VARCHAR := 'admin';
BEGIN
    CASE user_role
        WHEN 'admin' THEN
            RAISE NOTICE 'Administrator';
        WHEN 'user' THEN
            RAISE NOTICE 'Oddiy foydalanuvchi';
        ELSE
            RAISE NOTICE 'Noma’lum rol';
    END CASE;
END;
$$;
```

**Misol (Qidiruv CASE):**
```sql
DO $$
DECLARE
    user_age INTEGER := 25;
BEGIN
    CASE
        WHEN user_age < 18 THEN
            RAISE NOTICE 'Yosh 18 dan kichik';
        WHEN user_age >= 18 AND user_age <= 65 THEN
            RAISE NOTICE 'Yoshli foydalanuvchi';
        ELSE
            RAISE NOTICE 'Katta yoshli foydalanuvchi';
    END CASE;
END;
$$;
```
**Tushuntirish**: `CASE` yordamida shartlarga asoslangan xabarlar chiqariladi.

---

## 3. Tsikllar

Tsikllar takroriy vazifalarni bajarish uchun ishlatiladi.

### 3.1. `LOOP`

`LOOP` oddiy tsikl bo‘lib, `EXIT` yoki `CONTINUE` bilan boshqariladi.

**Sintaksis:**
```sql
LOOP
    -- Kod bloki
    EXIT [WHEN shart];
END LOOP;
```

**Misol:**
```sql
DO $$
DECLARE
    counter INTEGER := 1;
BEGIN
    LOOP
        RAISE NOTICE 'Hisoblagich: %', counter;
        counter := counter + 1;
        EXIT WHEN counter > 5;
    END LOOP;
END;
$$;
```
**Tushuntirish**: Hisoblagich 1 dan 5 gacha oshirilib, har bir qadamda xabar chiqariladi.

---

### 3.2. `FOR`

`FOR` tsikli ma’lum bir diapazon yoki so‘rov qatorlari bo‘yicha aylanadi.

**Diapazon bo‘yicha FOR:**
```sql
FOR o'zgaruvchi IN [REVERSE] boshlanish..tugash LOOP
    -- Kod bloki
END LOOP;
```

**Misol:**
```sql
DO $$
BEGIN
    FOR i IN 1..5 LOOP
        RAISE NOTICE 'Raqam: %', i;
    END LOOP;
END;
$$;
```

**So‘rov bo‘yicha FOR:**
```sql
FOR o'zgaruvchi IN so'rov LOOP
    -- Kod bloki
END LOOP;
```

**Misol:**
```sql
DO $$
DECLARE
    rec RECORD;
BEGIN
    FOR rec IN SELECT id, name FROM users ORDER BY id LOOP
        RAISE NOTICE 'ID: %, Ism: %', rec.id, rec.name;
    END LOOP;
END;
$$;
```
**Tushuntirish**: `users` jadvalidagi qatorlar bo‘yicha tsikl yuritiladi.

---

### 3.3. `WHILE`

`WHILE` tsikli shart to‘g‘ri bo‘lguncha kodni takrorlaydi.

**Sintaksis:**
```sql
WHILE shart LOOP
    -- Kod bloki
END LOOP;
```

**Misol:**
```sql
DO $$
DECLARE
    counter INTEGER := 1;
BEGIN
    WHILE counter <= 5 LOOP
        RAISE NOTICE 'Hisoblagich: %', counter;
        counter := counter + 1;
    END LOOP;
END;
$$;
```
**Tushuntirish**: Hisoblagich 5 gacha oshirilib, xabarlar chiqariladi.

---

### 3.4. `FOREACH` (Massivlar uchun)

`FOREACH` massiv elementlari bo‘yicha tsikl yuritish uchun ishlatiladi (PostgreSQL 9.1+).

**Sintaksis:**
```sql
FOREACH o'zgaruvchi IN ARRAY massiv LOOP
    -- Kod bloki
END LOOP;
```

**Misol:**
```sql
DO $$
DECLARE
    numbers INTEGER[] := ARRAY[1, 2, 3, 4, 5];
    num INTEGER;
BEGIN
    FOREACH num IN ARRAY numbers LOOP
        RAISE NOTICE 'Raqam: %', num;
    END LOOP;
END;
$$;
```
**Tushuntirish**: Massivdagi har bir element xabar sifatida chiqariladi.

---

## 4. O‘qimni Boshqarish Iboralari

Bu iboralar tsikllar yoki funksiyalar ichidagi oqimni boshqaradi.

### 4.1. `EXIT`

`EXIT` tsikldan chiqish uchun ishlatiladi.

**Sintaksis:**
```sql
EXIT [WHEN shart];
```

**Misol:**
```sql
DO $$
DECLARE
    counter INTEGER := 1;
BEGIN
    LOOP
        RAISE NOTICE 'Hisoblagich: %', counter;
        counter := counter + 1;
        EXIT WHEN counter > 3;
    END LOOP;
END;
$$;
```
**Tushuntirish**: Hisoblagich 3 dan oshganda tsikl to‘xtaydi.

---

### 4.2. `CONTINUE`

`CONTINUE` tsiklning joriy iteratsiyasini o‘tkazib yuborib, keyingi iteratsiyaga o‘tadi.

**Sintaksis:**
```sql
CONTINUE [WHEN shart];
```

**Misol:**
```sql
DO $$
BEGIN
    FOR i IN 1..5 LOOP
        IF i % 2 = 0 THEN
            CONTINUE;
        END IF;
        RAISE NOTICE 'Toq raqam: %', i;
    END LOOP;
END;
$$;
```
**Tushuntirish**: Faqat toq raqamlar chiqariladi, juft raqamlar o‘tkazib yuboriladi.

---

### 4.3. `RETURN`

`RETURN` funksiya yoki protseduradan natija qaytarish yoki uni to‘xtatish uchun ishlatiladi.

**Sintaksis:**
```sql
RETURN qiymat;
```

**Misol:**
```sql
CREATE FUNCTION get_user_name(user_id INTEGER)
RETURNS VARCHAR AS $$
DECLARE
    user_name VARCHAR;
BEGIN
    SELECT name INTO user_name FROM users WHERE id = user_id;
    IF NOT FOUND THEN
        RETURN 'Topilmadi';
    END IF;
    RETURN user_name;
END;
$$ LANGUAGE plpgsql;
```
**Tushuntirish**: Foydalanuvchi nomi qaytariladi, agar topilmasa, xabar qaytariladi.

---

### 4.4. `RAISE`

`RAISE` xabar chiqarish yoki xato yuzaga keltirish uchun ishlatiladi.

**Sintaksis:**
```sql
RAISE [LEVEL] 'xabar' [, o'zgaruvchi, ...];
```

**LEVEL turlari:**
- `NOTICE`: Oddiy xabar.
- `WARNING`: Ogohlantirish.
- `EXCEPTION`: Xato yuzaga keltiradi va dasturni to‘xtatadi.

**Misol:**
```sql
DO $$
DECLARE
    user_id INTEGER := 999;
BEGIN
    IF NOT EXISTS (SELECT 1 FROM users WHERE id = user_id) THEN
        RAISE EXCEPTION 'Foydalanuvchi % topilmadi', user_id;
    END IF;
    RAISE NOTICE 'Foydalanuvchi topildi';
END;
$$;
```
**Tushuntirish**: Agar foydalanuvchi topilmasa, xato chiqariladi.

---

## 5. Muhim Jihatlar

- **Shartlarning aniqligi**: `IF` va `CASE` shartlari aniq bo‘lishi kerak, aks holda xatolar yuzaga keladi.
- **Tsikl cheksizligi**: `LOOP` yoki `WHILE` ishlatganda cheksiz tsikldan qochish uchun `EXIT` shartini qo‘ying.
- **Performans**: Katta ma'lumotlar bilan ishlashda tsikllar o‘rniga SQL so‘rovlaridan foydalanish samaraliroq bo‘lishi mumkin.
- **Xato boshqaruvi**: `RAISE EXCEPTION` bilan xatolarni aniq boshqaring.
- **Moslashuvchanlik**: `FOR` va `FOREACH` ma'lumotlar tuzilishiga qarab moslashadi.
