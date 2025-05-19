# PL/pgSQL'da SELECT INTO Misollari

Bu hujjatda PL/pgSQL'da `SELECT INTO` iborasining foydalanishiga oid misollar keltirilgan. Har bir misol kod va uning tushuntirishini o'z ichiga oladi.

## 1. Bitta qiymatni o‘zgaruvchiga saqlash
So‘rov natijasini bitta o‘zgaruvchiga saqlash.

```sql
DO $$
DECLARE
    user_count INTEGER;
BEGIN
    SELECT COUNT(*) INTO user_count FROM users;
    RAISE NOTICE 'Foydalanuvchilar soni: %', user_count;
END;
$$;
```

**Tushuntirish**: `users` jadvalidagi qatorlar soni `user_count` o‘zgaruvchisiga saqlanadi va xabar sifatida chiqariladi.

---

## 2. Bir nechta ustunlarni o‘zgaruvchilarga saqlash
Bir qatorning bir nechta ustunlarini alohida o‘zgaruvchilarga saqlash.

```sql
DO $$
DECLARE
    user_id INTEGER;
    user_name VARCHAR;
BEGIN
    SELECT id, name INTO user_id, user_name 
    FROM users 
    WHERE id = 1;
    RAISE NOTICE 'ID: %, Ism: %', user_id, user_name;
END;
$$;
```

**Tushuntirish**: `id=1` bo‘lgan foydalanuvchining `id` va `name` qiymatlari mos ravishda `user_id` va `user_name` o‘zgaruvchilariga saqlanadi va xabar sifatida chiqariladi.

---

## 3. Yozuv (record) sifatida saqlash
Butun qatorni `RECORD` turidagi o‘zgaruvchiga saqlash.

```sql
DO $$
DECLARE
    user_rec RECORD;
BEGIN
    SELECT id, name, is_active INTO user_rec 
    FROM users 
    WHERE id = 1;
    RAISE NOTICE 'ID: %, Ism: %, Faol: %', user_rec.id, user_rec.name, user_rec.is_active;
END;
$$;
```

**Tushuntirish**: Qatorning barcha ustunlari `user_rec` yozuviga saqlanadi va ustunlarga nuqta (`.`) orqali murojaat qilinib, xabar sifatida chiqariladi.

---

## 4. STRICT bilan ishlash
`STRICT` so‘rovning roppa-rosa bitta qator qaytarishini talab qiladi.

```sql
DO $$
DECLARE
    user_id INTEGER;
    user_name VARCHAR;
BEGIN
    SELECT id, name INTO STRICT user_id, user_name 
    FROM users 
    WHERE id = 999;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RAISE NOTICE 'Foydalanuvchi topilmadi';
    WHEN TOO_MANY_ROWS THEN
        RAISE NOTICE 'Bir nechta qator qaytdi';
END;
$$;
```

**Tushuntirish**: Agar `id=999` bo‘lgan foydalanuvchi topilmasa yoki bir nechta qator qaytsa, mos xato xabari chiqariladi.

---

## 5. Dinamik SQL bilan SELECT INTO
Dinamik so‘rov natijasini saqlash.

```sql
DO $$
DECLARE
    table_name TEXT := 'users';
    user_name VARCHAR;
BEGIN
    EXECUTE 'SELECT name FROM ' || quote_ident(table_name) || ' WHERE id = 1' 
    INTO user_name;
    RAISE NOTICE 'Foydalanuvchi ismi: %', user_name;
END;
$$;
```

**Tushuntirish**: Jadval nomi dinamik ravishda kiritiladi va so‘rov natijasi `user_name` o‘zgaruvchisiga saqlanib, xabar sifatida chiqariladi.

---

**Qo'shimcha eslatmalar**:
- `SELECT INTO` faqat PL/pgSQL bloklari ichida ishlaydi.
- `STRICT` ishlatilganda, so‘rov 0 yoki 1’den ortiq qator qaytarsa, xato yuzaga keladi.
- Xatolarni ushlash uchun `EXCEPTION` bloki ishlatilishi tavsiya etiladi.
