# PostgreSQL DATE Funksiyalari

PostgreSQL-da `DATE` ma'lumot turi bilan ishlash uchun bir qator o'rnatilgan funksiyalar mavjud. 

## 1. EXTRACT
**Tavsif**: Sana yoki vaqt ma'lumotlaridan muayyan qismni (yil, oy, kun va hokazo) olish uchun ishlatiladi.

**Sintaksis**:
```sql
EXTRACT(field FROM source)
```

**Misol**:
```sql
SELECT EXTRACT(YEAR FROM DATE '2025-05-09') AS year;
```
**Natija**: `2025`

**Boshqa misol**:
```sql
SELECT EXTRACT(MONTH FROM DATE '2025-05-09') AS month;
```
**Natija**: `5`

**Qo'llaniladigan fieldlar**:
- `YEAR`, `MONTH`, `DAY`, `WEEK`, `QUARTER`, `DOW` (hafta kuni), `DOY` (yil kuni) va boshqalar.
***
## 2. TO_CHAR
**Tavsif**: Sana yoki vaqtni belgilangan formatda matn sifatida formatlash uchun ishlatiladi.

**Sintaksis**:
```sql
TO_CHAR(date, 'format_pattern')
```

**Misol**:
```sql
SELECT TO_CHAR(DATE '2025-05-09', 'DD Mon YYYY') AS formatted_date;
```
**Natija**: `09 May 2025`

**Boshqa misol**:
```sql
SELECT TO_CHAR(DATE '2025-05-09', 'Day, DD-MM-YYYY') AS full_date;
```
**Natija**: `Friday, 09-05-2025`

**Umumiy formatlar**:
- `DD`: Kun (01-31)
- `MM`: Oy (01-12)
- `YYYY`: Yil (4 raqamli)
- `Mon`: Oy nomi qisqa (Jan, Feb, ...)
- `Month`: To'liq oy nomi (January, February, ...)
- `Day`: Hafta kuni nomi (Monday, Tuesday, ...)
***
## 3. CURRENT_DATE
**Tavsif**: Joriy sanani qaytaradi (vaqt mintaqasiga bog'liq holda).

**Sintaksis**:
```sql
CURRENT_DATE
```

**Misol**:
```sql
SELECT CURRENT_DATE AS today;
```
**Natija** (2025-05-09 bo'lsa): `2025-05-09`
***
## 4. AGE
**Tavsif**: Ikki sana o'rtasidagi farqni `INTERVAL` sifatida qaytaradi yoki berilgan sanadan joriy sanagacha bo'lgan farqni hisoblaydi. Bu funksiya yoshni yoki vaqt oralig'ini aniqlashda foydali.

**Sintaksis**:
```sql
AGE(date1, date2) -- Ikki sana o'rtasidagi farq
AGE(date) -- Joriy sanagacha bo'lgan farq
```

**Misol (Ikki sana o'rtasidagi farq)**:
```sql
SELECT AGE(DATE '2025-05-09', DATE '1990-05-15') AS age_difference;
```
**Natija**: `34 years 11 months 24 days`

**Misol (Joriy sanagacha)**:
```sql
SELECT AGE(DATE '1990-05-15') AS age;
```
**Natija** (2025-05-09 bo'lsa): `34 years 11 months 24 days`

**Eslatma**:
- `AGE` funksiyasi natijani `INTERVAL` sifatida qaytaradi.
- Agar faqat yillarni olish kerak bo'lsa, `EXTRACT` bilan birgalikda ishlatilishi mumkin:
  ```sql
  SELECT EXTRACT(YEAR FROM AGE(DATE '1990-05-15')) AS years_old;
  ```
  **Natija**: `34`
***
## 5. INTERVAL
**Tavsif**: Vaqt oralig'ini ifodalaydi va sanalarga qo'shish yoki ayirish uchun ishlatiladi. `INTERVAL` `DATE` bilan birgalikda sana hisob-kitoblarida keng qo'llaniladi.

**Sintaksis**:
```sql
date + INTERVAL 'quantity unit'
date - INTERVAL 'quantity unit'
```

**Misol (Sanaga interval qo'shish)**:
```sql
SELECT DATE '2025-05-09' + INTERVAL '30 days' AS due_date;
```
**Natija**: `2025-06-08`

**Misol (Sanadan interval ayirish)**:
```sql
SELECT DATE '2025-05-09' - INTERVAL '1 year' AS last_year;
```
**Natija**: `2024-05-09`

**Misol (Interval bilan ishlash)**:
```sql
SELECT DATE '2025-05-09' + INTERVAL '2 months 15 days' AS future_date;
```
**Natija**: `2025-07-24`

**Umumiy interval birliklari**:
- `year`, `month`, `day`, `week`, `hour`, `minute`, `second`.
- Kombinatsiyalar: `1 year 3 months`, `2 days 12 hours`.

**Eslatma**:
- `INTERVAL` `AGE` funksiyasi natijasi sifatida ham qaytariladi.
- Intervalni ko'paytirish yoki bo'lish mumkin:
  ```sql
  SELECT INTERVAL '1 day' * 7 AS one_week;
  ```
  **Natija**: `7 days`
***
## 6. DATE_TRUNC
**Tavsif**: Sanani belgilangan birlikka (yil, oy, kun va hokazo) qisqartiradi, undan keyingi ma'lumotlarni nolga aylantiradi.

**Sintaksis**:
```sql
DATE_TRUNC('field', source)
```

**Misol**:
```sql
SELECT DATE_TRUNC('month', DATE '2025-05-09') AS start_of_month;
```
**Natija**: `2025-05-01`

**Boshqa misol**:
```sql
SELECT DATE_TRUNC('year', DATE '2025-05-09') AS start_of_year;
```
**Natija**: `2025-01-01`

**Qo'llaniladigan fieldlar**:
- `year`, `quarter`, `month`, `week`, `day`, `hour`, `minute`, `second`.

## Foydalanish bo'yicha maslahatlar
- **Formatlash**: `TO_CHAR` yordamida sanalarni mijoz ilovasiga mos formatda chiqaring.
- **Yosh hisoblash**: `AGE` va `EXTRACT` funksiyalarini birgalikda ishlatib, aniq yoshni olish mumkin.
- **Interval operatsiyalari**: `INTERVAL` bilan sanalarni osonlikcha o'zgartiring, masalan, muddatlar yoki kelajak sanalarini hisoblashda.
- **Performans**: Katta jadvallarda `DATE` ustunlarida so'rovlar uchun indekslar qo'llang:
  ```sql
  CREATE INDEX idx_events_date ON events(event_date);
  ```

## Qo'shimcha resurslar
- [PostgreSQL rasmiy hujjatlari: Date/Time Functions](https://www.postgresql.org/docs/current/functions-datetime.html)
- `DATE`, `AGE` va `INTERVAL` bilan tajriba o'tkazish uchun PostgreSQL Playground-dan foydalaning.
