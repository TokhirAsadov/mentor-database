# PostgreSQL Vaqt Funksiyalari va Sozlamalari

PostgreSQL-da vaqt va vaqt mintaqasi bilan ishlash uchun bir qator funksiyalar va sozlamalar mavjud. Ushbu bo'limda `NOW()`, `CURRENT_TIMESTAMP`, `SHOW TIMEZONE`, `SET TIMEZONE`, `pg_timezone_names`, `TIMEOFDAY()`, va `TIMEZONE(tz, timestamp)` funksiyalarining maqsadi, sintaksisi va amaliy misollari keltirilgan.
***
## 1. NOW()
**Tavsif**: Joriy sana va vaqtni `TIMESTAMPTZ` (vaqt mintaqasi bilan timestamp) sifatida qaytaradi. Bu funksiya joriy vaqt mintaqasi sozlamalariga asoslanadi.

**Sintaksis**:
```sql
SELECT NOW();
```

**Misol**:
```sql
SELECT NOW() AS current_time;
```
**Natija** (2025-05-09, Asia/Tashkent mintaqasida):
```
          current_time
-------------------------------
 2025-05-09 14:30:00.123456+05
```

**Eslatma**:
- `NOW()` har doim vaqt mintaqasi bilan qaytariladi.
- Natija `CURRENT_TIMESTAMP` bilan bir xil, lekin `NOW()` ko'proq ishlatiladi.
***
## 2. CURRENT_TIMESTAMP
**Tavsif**: Joriy sana va vaqtni `TIMESTAMPTZ` sifatida qaytaradi. `NOW()` bilan funksional jihatdan bir xil, lekin SQL standartiga mos keladi.

**Sintaksis**:
```sql
SELECT CURRENT_TIMESTAMP;
```

**Misol**:
```sql
SELECT CURRENT_TIMESTAMP AS current_time;
```
**Natija**:
```
          current_time
-------------------------------
 2025-05-09 14:30:00.123456+05
```

**Misol (Aniqlikni cheklash)**:
```sql
SELECT CURRENT_TIMESTAMP(3) AS current_time; -- 3 o'nlik raqam (millisoniyalar)
```
**Natija**:
```
       current_time
--------------------------
 2025-05-09 14:30:00.123+05
```

**Eslatma**:
- `CURRENT_TIMESTAMP` tranzaksiya boshida qaytarilgan vaqtni fiksatsiya qiladi va tranzaksiya davomida o'zgarmaydi.
***
## 3. SHOW TIMEZONE
**Tavsif**: Joriy sessiyada ishlatilayotgan vaqt mintaqasini ko'rsatadi.

**Sintaksis**:
```sql
SHOW TIMEZONE;
```

**Misol**:
```sql
SHOW TIMEZONE;
```
**Natija**:
```
 TimeZone
------------
 Asia/Tashkent
```

**Eslatma**:
- Vaqt mintaqasi `SET TIMEZONE` yoki konfiguratsiya fayli (`postgresql.conf`) orqali o'rnatiladi.
- Agar vaqt mintaqasi o'rnatilmagan bo'lsa, tizimning standart mintaqasi ishlatiladi.
***
## 4. SET TIMEZONE
**Tavsif**: Joriy sessiya uchun vaqt mintaqasini o'rnatadi. Bu `TIMESTAMPTZ` qiymatlarining ko'rinishiga ta'sir qiladi.

**Sintaksis**:
```sql
SET TIMEZONE TO 'timezone_name';
```
yoki
```sql
SET TIME ZONE 'timezone_name';
```

**Misol**:
```sql
SET TIMEZONE TO 'UTC';
SELECT NOW() AS utc_time;
```
**Natija**:
```
         utc_time
--------------------------
 2025-05-09 09:30:00.123456+00
```

**Boshqa misol**:
```sql
SET TIMEZONE TO 'America/New_York';
SELECT CURRENT_TIMESTAMP AS ny_time;
```
**Natija**:
```
         ny_time
--------------------------
 2025-05-09 05:30:00.123456-04
```

**Eslatma**:
- Vaqt mintaqasi nomlari `pg_timezone_names` jadvalidan olinishi mumkin.
- Sessiya tugaganda vaqt mintaqasi standart qiymatga qaytadi.
***
## 5. pg_timezone_names
**Tavsif**: PostgreSQL-da mavjud bo'lgan barcha vaqt mintaqalari nomlari va ularning ma'lumotlarini ko'rsatuvchi tizim jadvali. U vaqt mintaqasi nomi, UTC ofseti va boshqa detallarni o'z ichiga oladi.

**Sintaksis**:
```sql
SELECT * FROM pg_timezone_names;
```

**Misol**:
```sql
SELECT name, abbrev, utc_offset, is_dst FROM pg_timezone_names WHERE name LIKE 'Asia%';
```
**Natija** (qisqa namunaviy natija):
```
      name       | abbrev | utc_offset | is_dst
-----------------|--------|------------|--------
 Asia/Tashkent   | +05    | 05:00:00   | f
 Asia/Tokyo      | JST    | 09:00:00   | f
 Asia/Dubai      | +04    | 04:00:00   | f
```

**Misol (Muayyan mintaqa)**:
```sql
SELECT name, utc_offset FROM pg_timezone_names WHERE name = 'Asia/Tashkent';
```
**Natija**:
```
      name       | utc_offset
-----------------|------------
 Asia/Tashkent   | 05:00:00
```

**Eslatma**:
- `is_dst` ustuni yozgi vaqt (Daylight Saving Time) qo'llanilayotganini ko'rsatadi (`t` yoki `f`).
- Bu jadval vaqt mintaqasi nomlarini tanlashda foydali.
***
## 6. TIMEOFDAY()
**Tavsif**: Joriy sana va vaqtni matn (`TEXT`) sifatida qaytaradi, odatda tizim soati va vaqt mintaqasi bilan birga. U `NOW()` yoki `CURRENT_TIMESTAMP`dan farqli o'laroq, aniqroq vaqt ma'lumotini matn shaklida beradi.

**Sintaksis**:
```sql
SELECT TIMEOFDAY();
```

**Misol**:
```sql
SELECT TIMEOFDAY() AS current_time;
```
**Natija**:
```
              current_time
---------------------------------------
 Fri May 09 14:30:00.123456 2025 +05
```

**Eslatma**:
- `TIMEOFDAY()` tranzaksiya vaqtini emas, real vaqtni qaytaradi.
- Natija matn sifatida qaytariladi, shuning uchun arifmetik amallar uchun `NOW()` yoki `CURRENT_TIMESTAMP` afzalroq.
***
## 7. TIMEZONE(tz, timestamp)
**Tavsif**: Berilgan `TIMESTAMP` yoki `TIMESTAMPTZ` qiymatini muayyan vaqt mintaqasiga aylantiradi. Bu funksiya vaqt mintaqasi o'zgarishlarini boshqarishda foydali.

**Sintaksis**:
```sql
SELECT TIMEZONE('timezone_name', timestamp);
```

**Misol**:
```sql
SELECT TIMEZONE('America/New_York', TIMESTAMP '2025-05-09 14:30:00') AS ny_time;
```
**Natija**:
```
         ny_time
--------------------------
 2025-05-09 14:30:00-04
```

**Misol (TIMESTAMPTZ bilan)**:
```sql
SELECT TIMEZONE('Asia/Tokyo', TIMESTAMPTZ '2025-05-09 14:30:00+05:00') AS tokyo_time;
```
**Natija**:
```
       tokyo_time
------------------------
 2025-05-09 18:30:00+09
```

**Eslatma**:
- `timezone_name` `pg_timezone_names` jadvalidan olish mumkin.
- Agar kiritilgan qiymat `TIMESTAMPTZ` bo'lsa, u birinchi UTC-ga aylantiriladi, so'ngra yangi mintaqaga moslashtiriladi.
***
## Foydalanish bo'yicha maslahatlar
- **Vaqt mintaqasi boshqaruvi**: Xalqaro ilovalar uchun `TIMESTAMPTZ` va `SET TIMEZONE` ishlatishni tavsiya qilamiz.
- **Aniqlik**: `CURRENT_TIMESTAMP` yoki `NOW()` bilan aniqlikni cheklash uchun `(precision)` qo'llanilishi mumkin (masalan, `CURRENT_TIMESTAMP(3)`).
- **Matn chiqishi**: `TIMEOFDAY()` loglar uchun qulay, lekin hisob-kitoblar uchun `NOW()` afzal.
- **Vaqt mintaqasi ro'yxati**: `pg_timezone_names` yordamida to'g'ri vaqt mintaqasi nomlarini tanlang.
- **Indekslash**: Katta jadvallarda vaqt ustunlari uchun indekslar qo'llang:
  ```sql
  CREATE INDEX idx_logs_time ON logs(log_time);
  ```


***
## Qo'shimcha resurslar
- [PostgreSQL rasmiy hujjatlari: Date/Time Functions](https://www.postgresql.org/docs/current/functions-datetime.html)
- [PostgreSQL rasmiy hujjatlari: Time Zones](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-TIMEZONES)
