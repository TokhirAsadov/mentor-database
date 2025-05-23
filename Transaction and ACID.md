## Tranzaksiya nima?

**Tranzaksiya** — ma'lumotlar bazasida bir yoki bir nechta operatsiyalarni bir butun sifatida bajarish jarayoni bo'lib, ular yaxlit va muvofiq tarzda amalga oshiriladi. Tranzaksiya ma'lumotlar bazasining holatini bir muvofiq holatdan boshqa muvofiq holatga o'tkazadi va agar xatolik yuz bersa, o'zgarishlar bekor qilinadi.

- **Misol**: Bankda bir hisobdan boshqa hisobga pul o'tkazish. Bu jarayon ikkita operatsiyani o'z ichiga oladi:
  1. Birinchi hisobdan pulni ayirish.
  2. Ikkinchi hisobga pulni qo'shish.
     Agar ikkala operatsiya ham muvaffaqiyatli bajarilmasa, tranzaksiya bekor qilinadi, aks holda ma'lumotlar bazasi notog'ri holatda qolishi mumkin.

Tranzaksiyalar ma'lumotlar bazasining ishonchliligi, muvofiqligi va xavfsizligini ta'minlash uchun ishlatiladi.

## Tranzaksiyalarning asosiy xususiyatlari (ACID)

## ACID xususiyatlari

### Atomicity
- **Ta'rifi**: Tranzaksiyadagi barcha operatsiyalar yaxlit tarzda bajarilishi kerak, ya'ni tranzaksiyaning barcha qismlari muvaffaqiyatli bajariladi yoki hech biri bajarilmaydi.
- **Misol**: Bank tranzaksiyasida bir hisobdan ikkinchi hisobga pul o'tkazishda, agar pul bir hisobdan chiqarilsa, lekin boshqa hisobga tushmasa, tranzaksiya to'liq bekor qilinadi.
- **PostgreSQL'da**: Tranzaksiyalar `BEGIN`, `COMMIT` va `ROLLBACK` buyruqlari orqali boshqariladi. Agar tranzaksiya muvaffaqiyatsiz bo'lsa, `ROLLBACK` barcha o'zgarishlarni bekor qiladi.

### Consistency
- **Ta'rifi**: Tranzaksiya ma'lumotlar bazasini bir muvofiq holatdan boshqa muvofiq holatga o'tkazadi. Bunda barcha ma'lumotlar integrallik qoidalariga (constraints), triggerlarga va boshqa qoidalarga rioya qilishi kerak.
- **Misol**: Agar ma'lumotlar bazasida hisob balansining manfiy bo'lmasligi qoidasi bo'lsa, tranzaksiya ushbu qoidani buzmaydi.
- **PostgreSQL'da**: Foreign key constraints, unique constraints va triggerlar orqali muvofiqlik ta'minlanadi. Qoida buzilsa, tranzaksiya `ROLLBACK` bilan bekor qilinadi.

### Isolation
- **Ta'rifi**: Tranzaksiyalar bir-biridan mustaqil ravishda ishlaydi. Bir tranzaksiya boshqa tranzaksiyalarning natijalariga ta'sir qilmasdan yoki ulardan ta'sirlanmasdan bajariladi.
- **Misol**: Ikkita foydalanuvchi bir vaqtning o'zida bir hisobni o'zgartirsa, izolyatsiya ularning o'zgarishlari bir-biriga aralashmasligini ta'minlaydi.
- **PostgreSQL'da**: MVCC (Multiversion Concurrency Control) mexanizmi ishlatiladi. Izolyatsiya darajalari (`READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`) tranzaksiyalar o'rtasidagi o'zaro ta'sirni boshqaradi.

### Durability
- **Ta'rifi**: Tranzaksiya muvaffaqiyatli yakunlanganidan so'ng (`COMMIT` qilinganidan keyin), o'zgarishlar doimiy ravishda saqlanadi va tizim nosozligi bo'lsa ham yo'qolmaydi.
- **Misol**: Pul o'tkazish tranzaksiyasi yakunlansa, tizim o'chib-qayta yoqilganda ham o'zgarishlar saqlanib qoladi.
- **PostgreSQL'da**: WAL (Write-Ahead Logging) mexanizmi orqali barqarorlik ta'minlanadi.

## PostgreSQL'da tranzaksiyalar

PostgreSQL'da tranzaksiyalar `BEGIN`, `COMMIT` va `ROLLBACK` buyruqlari orqali boshqariladi. PLpgSQL'da tranzaksiyalar protseduralar va funksiyalar ichida ham qo'llaniladi.

### Asosiy tranzaksiya buyruqlari
1. **BEGIN**: Tranzaksiyani boshlaydi.
2. **COMMIT**: Tranzaksiyani muvaffaqiyatli yakunlaydi va o'zgarishlarni doimiy saqlaydi.
3. **ROLLBACK**: Tranzaksiyani bekor qiladi va barcha o'zgarishlarni qaytaradi.
4. **SAVEPOINT**: Tranzaksiya ichida oraliq nuqtalar yaratadi, ularga qaytish mumkin.
5. **ROLLBACK TO SAVEPOINT**: Belgilangan savepointgacha o'zgarishlarni bekor qiladi.

### Misol: Oddiy tranzaksiya
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;
```
Agar xatolik yuz bersa:
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Xatolik yuz berdi deylik
ROLLBACK;
```

### SAVEPOINT misoli
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
-- Xatolik yuz berdi, faqat ikkinchi yangilanishni bekor qilamiz
ROLLBACK TO my_savepoint;
COMMIT;
```

## PLpgSQL'da tranzaksiyalar

PLpgSQL'da tranzaksiyalar funksiyalar va saqlanadigan protseduralar ichida boshqariladi. Tranzaksiyalar odatda avtomatik boshlanadi va protsedura yakunlanganda `COMMIT` yoki `ROLLBACK` qilinadi.

### PLpgSQL'da tranzaksiya misoli
```sql
CREATE OR REPLACE PROCEDURE transfer_funds(
    sender_id INT,
    receiver_id INT,
    amount NUMERIC
)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Tranzaksiyani boshlash (avtomatik)
    UPDATE accounts SET balance = balance - amount 
    WHERE account_id = sender_id AND balance >= amount;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'Yuboruvchida yetarli mablag'' yo''q!';
    END IF;

    UPDATE accounts SET balance = balance + amount 
    WHERE account_id = receiver_id;

    -- Agar xatolik bo'lmasa, tranzaksiya avtomatik commit qilinadi
EXCEPTION
    WHEN OTHERS THEN
        -- Xatolik yuz bersa, tranzaksiya bekor qilinadi
        RAISE NOTICE 'Xatolik yuz berdi: %', SQLERRM;
        ROLLBACK;
END;
$$;

-- Protsedurani chaqirish
CALL transfer_funds(1, 2, 100);
```

### Tushuntirish
- **Tranzaksiya boshlanishi**: Protsedura ichida tranzaksiya avtomatik boshlanadi.
- **Xatolik boshqaruvi**: `EXCEPTION` bloki yordamida xatolar ushlanadi va `ROLLBACK` qilinadi.
- **Muvaffaqiyat**: Xatolik bo'lmasa, tranzaksiya avtomatik `COMMIT` qilinadi.


## WAL (Write-Ahead Logging) va barqarorlik
- Barcha o'zgarishlar avval log fayliga yoziladi.
- Tizim nosozligi yuz bersa, WAL loglari yordamida ma'lumotlar tiklanadi.
- `synchronous_commit` sozlamasi tranzaksiyaning diskka yozilishini boshqaradi:
  - `on` (standart): Tranzaksiya diskka yozilgandan keyin tasdiqlanadi.
  - `off`: Tezroq ishlaydi, lekin nosozlikda ma'lumot yo'qotilishi mumkin.


## Xulosa
PostgreSQL va PLpgSQL tranzaksiyalarni ACID xususiyatlariga muvofiq boshqarish uchun kuchli vositalarni taqdim etadi:
- **Atomarlik**: `BEGIN`, `COMMIT`, `ROLLBACK` va `SAVEPOINT` orqali.
- **Muvofiqlik**: Integrallik qoidalari va triggerlar orqali.
- **Izolyatsiya**: MVCC va izolyatsiya darajalari orqali.
- **Barqarorlik**: WAL mexanizmi orqali.
