# PostgreSQL'da Triggerlar haqida ma'lumot

## 1. Triggerlar haqida umumiy ma'lumot

Triggerlar PostgreSQL'da jadvalda ma'lum bir hodisa (event) sodir bo'lganda avtomatik ravishda chaqiriladigan funksiyalardir. Ular quyidagi holatlarda ishlatiladi:

- **Ma'lumotlar yaxlitligini ta'minlash**: Masalan, ma'lum bir ustundagi qiymatlarning to'g'riligini tekshirish.
- **Loglash**: O'zgartirishlar tarixini saqlash.
- **Avtomatlashtirish**: Masalan, yangi yozuv qo'shilganda boshqa jadvalni yangilash.
- **Qoidalar qo'llash**: Masalan, foydalanuvchi ma'lumotlarini yangilashda qo'shimcha tekshiruvlar o'tkazish.

Triggerlar ikki turga bo'linadi:
1. **Row-level triggers**: Har bir qator uchun alohida ishlaydi (`FOR EACH ROW`).
2. **Statement-level triggers**: Operatsiya (masalan, bir nechta qatorlarni o'chirish) uchun bir marta ishlaydi (`FOR EACH STATEMENT`).

Triggerlar quyidagi hodisalarga javob berishi mumkin:
- `INSERT`: Yangi yozuv qo'shilganda.
- `UPDATE`: Mavjud yozuv yangilanganda.
- `DELETE`: Yozuv o'chirilganda.
- `TRUNCATE`: Jadval tozalananda (faqat statement-level triggerlar uchun).

## 2. Trigger sintaksisi

Trigger yaratish uchun ikki bosqich kerak:
1. Trigger funksiyasini yaratish (`CREATE FUNCTION`).
2. Triggerning o'zini yaratish (`CREATE TRIGGER`).

### Trigger funksiyasi sintaksisi
```sql
CREATE OR REPLACE FUNCTION trigger_function_name()
RETURNS TRIGGER AS $$
BEGIN
  -- Trigger logikasi
  RETURN NEW; -- yoki RETURN OLD; yoki NULL;
END;
$$ LANGUAGE plpgsql;
```

- **`RETURNS TRIGGER`**: Funksiya trigger sifatida ishlatilishi uchun maxsus tur.
- **`NEW`**: `INSERT` yoki `UPDATE` operatsiyalarida yangi qator ma'lumotlari.
- **`OLD`**: `UPDATE` yoki `DELETE` operatsiyalarida eski qator ma'lumotlari.
- **`RETURN`**: Trigger funksiyasi `NEW`, `OLD` yoki `NULL` qaytarishi kerak. `NULL` qaytarsa, operatsiya bekor qilinadi.

### Trigger yaratish sintaksisi
```sql
CREATE [OR REPLACE] TRIGGER trigger_name
{BEFORE | AFTER | INSTEAD OF} {INSERT | UPDATE [OF column_name [, ...]] | DELETE | TRUNCATE}
ON table_name
[FOR EACH {ROW | STATEMENT}]
[WHEN (condition)]
EXECUTE FUNCTION trigger_function_name();
```

- **`trigger_name`**: Triggerning nomi.
- **`BEFORE`**: Operatsiyadan oldin ishlaydi (masalan, ma'lumotlarni tekshirish yoki o'zgartirish uchun).
- **`AFTER`**: Operatsiyadan keyin ishlaydi (masalan, loglash yoki boshqa jadvallarni yangilash uchun).
- **`INSTEAD OF`**: `VIEW` jadvallarida ishlatiladi, operatsiyani o'zgartirish uchun.
- **`INSERT | UPDATE | DELETE | TRUNCATE`**: Trigger qaysi operatsiyaga javob berishini belgilaydi.
- **`OF column_name`**: Faqat muayyan ustunlar yangilanganda triggerni ishga tushiradi (`UPDATE` uchun).
- **`FOR EACH ROW`**: Har bir qator uchun alohida ishlaydi.
- **`FOR EACH STATEMENT`**: Butun operatsiya uchun bir marta ishlaydi.
- **`WHEN (condition)`**: Trigger faqat shart bajarilganda ishga tushadi.
- **`EXECUTE FUNCTION`**: Chaqiriladigan trigger funksiyasi.

## 3. Trigger funksiyalarida maxsus o'zgaruvchilar

Trigger funksiyalarida quyidagi maxsus o'zgaruvchilar ishlatiladi:
- **`NEW`**: `INSERT` yoki `UPDATE` da yangi qator ma'lumotlari (row-level triggerlarda).
- **`OLD`**: `UPDATE` yoki `DELETE` da eski qator ma'lumotlari (row-level triggerlarda).
- **`TG_OP`**: Operatsiya turi (`INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`).
- **`TG_TABLE_NAME`**: Trigger ishlayotgan jadval nomi.
- **`TG_WHEN`**: Trigger qachon ishga tushgani (`BEFORE`, `AFTER`, `INSTEAD OF`).
- **`TG_LEVEL`**: Trigger turi (`ROW` yoki `STATEMENT`).
- **`TG_ARGV`**: Triggerga uzatilgan qo'shimcha argumentlar (`CREATE TRIGGER` da ko'rsatilgan).

## 4. Trigger misollari

### Oddiy BEFORE INSERT trigger
Yangi foydalanuvchi qo'shilganda emailni tekshiruvchi trigger:
```sql
-- Trigger funksiyasi
CREATE OR REPLACE FUNCTION check_email()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' THEN
    RAISE EXCEPTION 'Noto''g''ri email formati: %', NEW.email;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger yaratish
CREATE TRIGGER validate_email
BEFORE INSERT ON users
FOR EACH ROW
EXECUTE FUNCTION check_email();

-- Test
INSERT INTO users (username, email) VALUES ('alexpeter', 'invalid_email');
```

**Natija**:
```
ERROR:  Noto'g'ri email formati: invalid_email
```

### AFTER UPDATE trigger bilan loglash
Foydalanuvchi ma'lumotlari yangilanganda log jadvliga yozuvchi trigger:
```sql
-- Log jadvali
CREATE TABLE user_log (
  log_id SERIAL PRIMARY KEY,
  user_id integer,
  old_email text,
  new_email text,
  change_time timestamp DEFAULT CURRENT_TIMESTAMP
);

-- Trigger funksiyasi
CREATE OR REPLACE FUNCTION log_email_change()
RETURNS TRIGGER AS $$
BEGIN
  IF OLD.email != NEW.email THEN
    INSERT INTO user_log (user_id, old_email, new_email)
    VALUES (OLD.id, OLD.email, NEW.email);
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger yaratish
CREATE TRIGGER email_change_log
AFTER UPDATE OF email ON users
FOR EACH ROW
EXECUTE FUNCTION log_email_change();

-- Test
UPDATE users SET email = 'new.email@example.com' WHERE id = 1;
```

**Natija** (`user_log` jadvalida):
```
 log_id | user_id | old_email          | new_email              | change_time
--------+---------+--------------------+-----------------------+------------------------
 1      | 1       | old.email@example.com | new.email@example.com | 2025-05-21 12:23:00
```

### BEFORE DELETE trigger
Foydalanuvchi o'chirilishidan oldin arxiv jadvliga saqlovchi trigger:
```sql
-- Arxiv jadvali
CREATE TABLE user_archive (
  id integer,
  username text,
  email text,
  deleted_at timestamp DEFAULT CURRENT_TIMESTAMP
);

-- Trigger funksiyasi
CREATE OR REPLACE FUNCTION archive_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO user_archive (id, username, email)
  VALUES (OLD.id, OLD.username, OLD.email);
  RETURN OLD;
END;
$$ LANGUAGE plpgsql;

-- Trigger yaratish
CREATE TRIGGER archive_before_delete
BEFORE DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION archive_user();

-- Test
DELETE FROM users WHERE id = 1;
```

**Natija** (`user_archive` jadvalida):
```
 id | username   | email                | deleted_at
----+------------+---------------------+------------------------
 1  | alexpeter  | alex@example.com    | 2025-05-21 12:23:00
```

### Statement-level trigger
Jadvaldan yozuvlar o'chirilganda umumiy sonini loglash:
```sql
-- Log jadvali
CREATE TABLE delete_log (
  log_id SERIAL PRIMARY KEY,
  table_name text,
  deleted_rows integer,
  log_time timestamp DEFAULT CURRENT_TIMESTAMP
);

-- Trigger funksiyasi
CREATE OR REPLACE FUNCTION log_delete_count()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO delete_log (table_name, deleted_rows)
  VALUES (TG_TABLE_NAME, (SELECT COUNT(*) FROM deleted));
  RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Trigger yaratish
CREATE TRIGGER log_delete
AFTER DELETE ON users
FOR EACH STATEMENT
EXECUTE FUNCTION log_delete_count();

-- Test
DELETE FROM users WHERE status = 'inactive';
```

**Natija** (`delete_log` jadvalida):
```
 log_id | table_name | deleted_rows | log_time
--------+------------+--------------+------------------------
 1      | users      | 5            | 2025-05-21 12:23:00
```

### INSTEAD OF trigger (VIEW uchun)
`VIEW` jadvalida `INSERT` operatsiyasini boshqarish:
```sql
-- VIEW yaratish
CREATE VIEW active_users AS
SELECT id, username, email
FROM users
WHERE status = 'active';

-- Trigger funksiyasi
CREATE OR REPLACE FUNCTION insert_active_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO users (id, username, email, status)
  VALUES (NEW.id, NEW.username, NEW.email, 'active');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger yaratish
CREATE TRIGGER insert_active
INSTEAD OF INSERT ON active_users
FOR EACH ROW
EXECUTE FUNCTION insert_active_user();

-- Test
INSERT INTO active_users (id, username, email)
VALUES (10, 'newuser', 'newuser@example.com');
```

**Natija**: `users` jadvalida yangi yozuv `status = 'active'` bilan qo'shiladi.

## 5. Triggerlardan foydalanish usullari

1. **Ma'lumotlar yaxlitligi**:
   - Triggerlar yordamida ma'lumotlarning to'g'riligini tekshirish mumkin (masalan, email formati, musbat qiymatlar).
   - `BEFORE` triggerlar `NEW` ma'lumotlarini o'zgartirib, noto'g'ri kiritishlarni tuzatishi mumkin.

2. **Loglash va audit**:
   - `AFTER` triggerlar o'zgartirishlarni log jadvallariga yozish uchun ishlatiladi.
   - Masalan, foydalanuvchi harakatlari yoki o'chirilgan ma'lumotlarni saqlash.

3. **Avtomatlashtirish**:
   - Boshqa jadvallarni yangilash (masalan, balans hisoblash, statistika yig'ish).
   - Triggerlar yordamida bog'liq jadvallarni sinxronlashtirish.

4. **VIEW bilan ishlash**:
   - `INSTEAD OF` triggerlar `VIEW` jadvallarida `INSERT`, `UPDATE`, `DELETE` operatsiyalarini boshqarish uchun ishlatiladi.

5. **Statement-level triggerlar**:
   - Katta hajmdagi operatsiyalarda (masalan, `TRUNCATE` yoki ommaviy `DELETE`) umumiy statistikani yozish uchun ishlatiladi.

## 6. Foydali maslahatlar

1. **Ishlashga ta'sir**:
   - Triggerlar har bir qator yoki operatsiya uchun ishlaydi, shuning uchun katta jadvallarda ishlashni sekinlashtirishi mumkin. Optimallashtirish uchun faqat kerakli operatsiyalarga trigger qo'llang.
   - Indekslardan foydalanish triggerlar ichidagi so'rovlar samaradorligini oshiradi.

2. **Xavfsizlik**:
   - `SECURITY DEFINER` funksiyalar bilan triggerlarda ehtiyotkor bo'ling, chunki ular yaratuvchining ruxsatlari bilan ishlaydi.
   - Trigger funksiyalarida SQL injection xavfini oldini olish uchun `format` ishlatish tavsiya etiladi.

3. **Xato boshqaruvi**:
   - Trigger funksiyalarida `EXCEPTION` bloklaridan foydalanib, xatolarni ushlang:
     ```sql
     EXCEPTION
       WHEN unique_violation THEN
         RAISE NOTICE 'Noyob kalit qoidasi buzildi: %', SQLERRM;
         RETURN NULL;
     ```

4. **Cheklovlar**:
   - Triggerlar cheksiz tsikllarga olib kelishi mumkin (masalan, bir trigger boshqa jadvalni yangilasa, u yana trigger chaqirishi mumkin). Buni oldini olish uchun `TG_TABLE_NAME` yoki shartlardan foydalaning.
   - `BEFORE` triggerlarda `NEW` ni o'zgartirish orqali operatsiyani moslashtirish mumkin, lekin `AFTER` triggerlarda bu imkonsiz.

5. **Loglash**:
   - Trigger xatolarini yoki harakatlarini log jadvallariga yozish uchun:
     ```sql
     INSERT INTO trigger_log (table_name, operation, log_time)
     VALUES (TG_TABLE_NAME, TG_OP, CURRENT_TIMESTAMP);
     ```

6. **Test qilish**:
   - Triggerlarni sinov muhitida sinab ko'ring va tranzaksiyalarni `ROLLBACK` bilan bekor qilib, ma'lumotlarni himoya qiling:
     ```sql
     BEGIN;
     INSERT INTO users (username, email) VALUES ('test', 'test@example.com');
     ROLLBACK;
     ```

7. **Triggerlarni o'chirish/yangilash**:
   - Triggerni o'chirish:
     ```sql
     DROP TRIGGER trigger_name ON table_name;
     ```
   - Triggerni yangilash uchun `CREATE OR REPLACE TRIGGER` ishlatiladi (PostgreSQL 14+).
