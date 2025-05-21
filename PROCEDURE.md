# PostgreSQL'da CREATE PROCEDURE haqida ma'lumot

## 1. Protseduralar haqida umumiy ma'lumot

PostgreSQL'da protseduralar (`CREATE PROCEDURE`) ma'lumotlar bazasida tranzaksiyalarni boshqarish, ma'lumotlarni o'zgartirish yoki muayyan operatsiyalarni bajarish uchun ishlatiladi. Protseduralar PostgreSQL 11 versiyasidan boshlab qo'llab-quvvatlanadi va ular funksiyalardan (`CREATE FUNCTION`) farqli o'laroq, tranzaksiyalarni to'g'ridan-to'g'ri boshqarish imkonini beradi (masalan, `COMMIT` va `ROLLBACK`). 

Protseduralar quyidagi maqsadlarda ishlatiladi:
- Ma'lumotlarni o'zgartirish (`INSERT`, `UPDATE`, `DELETE`).
- Tranzaksiyalarni boshqarish (`COMMIT`, `ROLLBACK`).
- Murakkab biznes logikasini amalga oshirish.
- Bir nechta operatsiyalarni ketma-ket bajarish (masalan, bir nechta jadvallarni yangilash).

**Protseduralar va funksiyalar o'rtasidagi asosiy farqlar**:
- **Protseduralar**:
  - Qiymat qaytarmaydi (odatda `void` sifatida ishlaydi).
  - `CALL` bayonoti bilan chaqiriladi.
  - Tranzaksiyalarni to'g'ridan-to'g'ri boshqarish imkoniyati mavjud (`COMMIT`, `ROLLBACK`).
- **Funksiyalar**:
  - Qiymat qaytaradi (`RETURNS` bilan).
  - `SELECT` yoki boshqa SQL so'rovlarida ishlatiladi.
  - Tranzaksiya boshqaruvi cheklangan.

## 2. CREATE PROCEDURE sintaksisi

```sql
CREATE [OR REPLACE] PROCEDURE procedure_name ([parameter_name [IN | OUT | INOUT] parameter_type [, ...]])
LANGUAGE plpgsql
AS $$
DECLARE
  -- O'zgaruvchilar deklaratsiyasi (ixtiyoriy)
BEGIN
  -- Protsedura logikasi
  [COMMIT | ROLLBACK];
END;
$$;
```

### Tushuntirish:
- **`CREATE OR REPLACE`**: Agar protsedura allaqachon mavjud bo'lsa, uni yangilaydi.
- **`procedure_name`**: Protseduraning nomi (odatda schema bilan, masalan, `public.my_procedure`).
- **`parameter_name`**: Protseduraga uzatiladigan parametrlar:
  - `IN`: Kirish parametri (standart).
  - `OUT`: Chiqish parametri (natija sifatida qaytariladi).
  - `INOUT`: Kirish va chiqish sifatida ishlatiladi.
- **`LANGUAGE plpgsql`**: Protsedura tili (odatda PL/pgSQL, lekin SQL, PL/Python va boshqalar ham bo'lishi mumkin).
- **`AS $$ ... $$`**: Protsedura tanasi (logikasi).
- **`DECLARE`**: O'zgaruvchilar, kursorlar yoki boshqa ob'ektlar deklaratsiya qilinadi.
- **`BEGIN ... END`**: Protsedura logikasi joylashadi.
- **`COMMIT` / `ROLLBACK`**: Tranzaksiyani yakunlash yoki bekor qilish uchun ishlatiladi.

**Eslatma**: Protseduralar `RETURNS` kalit so'zini ishlatmaydi, lekin `OUT` yoki `INOUT` parametrlari orqali ma'lumot qaytarishi mumkin.

## 3. Protseduralarni chaqirish

Protseduralar `CALL` bayonoti yordamida chaqiriladi:
```sql
CALL procedure_name(argument1, argument2, ...);
```

Agar protsedura `OUT` yoki `INOUT` parametrlarga ega bo'lsa, natijalar quyidagicha olinadi:
```sql
CALL procedure_name(arg1, arg2, out_param);
```

## 4. CREATE PROCEDURE misollari

### Oddiy protsedura
Foydalanuvchi emailini yangilovchi protsedura:
```sql
CREATE OR REPLACE PROCEDURE update_user_email(user_id integer, new_email text)
LANGUAGE plpgsql
AS $$
BEGIN
  UPDATE users
  SET email = new_email
  WHERE id = user_id;
  
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Foydalanuvchi ID % topilmadi', user_id;
  END IF;
  
  COMMIT;
END;
$$;

-- Chaqirish
CALL update_user_email(1, 'new.email@example.com');
```

**Tushuntirish**: Bu protsedura `users` jadvalidagi foydalanuvchi emailini yangilaydi va tranzaksiyani `COMMIT` bilan yakunlaydi. Agar foydalanuvchi topilmasa, xato chiqaradi.

### OUT parametri bilan protsedura
Foydalanuvchi ma'lumotlarini yangilash va yangilangan qator sonini qaytaruvchi protsedura:
```sql
CREATE OR REPLACE PROCEDURE update_user_status(user_id integer, new_status text, OUT updated_rows integer)
LANGUAGE plpgsql
AS $$
BEGIN
  UPDATE users
  SET status = new_status
  WHERE id = user_id
  RETURNING 1 INTO updated_rows;
  
  IF updated_rows = 0 THEN
    RAISE EXCEPTION 'Foydalanuvchi ID % topilmadi', user_id;
  END IF;
  
  COMMIT;
END;
$$;

-- Chaqirish
CALL update_user_status(1, 'active', NULL);
```

**Natija**: Agar foydalanuvchi topilsa, `updated_rows` ga `1` yoziladi va tranzaksiya yakunlanadi. Aks holda, xato chiqariladi.

### Xato boshqaruvi bilan protsedura
Nolga bo'lish xatosini ushlovchi protsedura:
```sql
CREATE OR REPLACE PROCEDURE safe_divide(a integer, b integer, OUT result integer)
LANGUAGE plpgsql
AS $$
BEGIN
  result := a / b;
  COMMIT;
EXCEPTION
  WHEN division_by_zero THEN
    RAISE NOTICE 'Xato: Nolga bo''lish urinildi';
    result := NULL;
    ROLLBACK;
END;
$$;

-- Chaqirish
CALL safe_divide(10, 0, NULL);
```

**Natija**:
```
NOTICE:  Xato: Nolga bo'lish urinildi
 result
--------
 NULL
```

### Dinamik SQL bilan protsedura
Jadval nomini parametr sifatida oluvchi protsedura:
```sql
CREATE OR REPLACE PROCEDURE delete_old_records(table_name text, days_old integer)
LANGUAGE plpgsql
AS $$
BEGIN
  EXECUTE format('DELETE FROM %I WHERE created_at < CURRENT_DATE - INTERVAL ''%s days''', table_name, days_old);
  COMMIT;
EXCEPTION
  WHEN undefined_table THEN
    RAISE NOTICE 'Jadval % topilmadi', table_name;
    ROLLBACK;
END;
$$;

-- Chaqirish
CALL delete_old_records('users', 365);
```

**Tushuntirish**: Bu protsedura berilgan jadvaldan 365 kundan eski yozuvlarni o'chiradi. Agar jadval topilmasa, tranzaksiya bekor qilinadi.

### Tranzaksiya boshqaruvi bilan protsedura
Bir nechta jadvallarni yangilovchi protsedura:
```sql
CREATE OR REPLACE PROCEDURE transfer_funds(from_account integer, to_account integer, amount numeric)
LANGUAGE plpgsql
AS $$
BEGIN
  -- Balansni kamaytirish
  UPDATE accounts
  SET balance = balance - amount
  WHERE account_id = from_account;
  
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Hisob % topilmadi', from_account;
  END IF;
  
  -- Balansni oshirish
  UPDATE accounts
  SET balance = balance + amount
  WHERE account_id = to_account;
  
  IF NOT FOUND THEN
    RAISE EXCEPTION 'Hisob % topilmadi', to_account;
  END IF;
  
  COMMIT;
EXCEPTION
  WHEN others THEN
    RAISE NOTICE 'Xato yuz berdi: %', SQLERRM;
    ROLLBACK;
END;
$$;

-- Chaqirish
CALL transfer_funds(1, 2, 100.00);
```

**Tushuntirish**: Bu protsedura bir hisobdan boshqasiga pul o'tkazadi. Agar xato yuz bersa, tranzaksiya bekor qilinadi.

## 5. Protseduralarni boshqarish

1. **Protsedurani o'chirish**:
```sql
DROP PROCEDURE procedure_name(parameter_types);
```
Masalan:
```sql
DROP PROCEDURE update_user_email(integer, text);
```

2. **Protsedurani yangilash**:
`CREATE OR REPLACE PROCEDURE` yordamida protsedurani qayta yaratish.

3. **Protsedura ma'lumotlarini ko'rish**:
Protsedura haqida ma'lumot olish uchun:
```sql
SELECT * FROM information_schema.routines
WHERE routine_name = 'update_user_email' AND routine_type = 'PROCEDURE';
```
Yoki:
```sql
\dp update_user_email
```
