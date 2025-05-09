# PostgreSQL INTERVAL Sintaksisi: SQL Standart Formati

PostgreSQL-da `INTERVAL` ma'lumot turi vaqt oralig'ini ifodalash uchun ishlatiladi. `INTERVAL` qiymatlari turli formatlarda kiritilishi mumkin, ulardan biri SQL standart formati hisoblanadi. Quyida SQL standart formati, ya'ni `'1-2'` (1 yil 2 oy) gacha bo'lgan sintaksis tavsiflanadi.

## SQL Standart Formati: `'1-2'` (1 yil 2 oy)

**Tavsif**: SQL standart formati `INTERVAL` qiymatini yillar va oylar sifatida ifodalash uchun ishlatiladi. Bu formatda qiymat `yil-oy` shaklida kiritiladi, masalan, `'1-2'` 1 yil va 2 oyni anglatadi.

**Sintaksis**:
```sql
'yil-oy'
```

- `yil`: Butun son, vaqt oralig'idagi yillar sonini ifodalaydi (masalan, `1`, `10`).
- `oy`: Butun son, vaqt oralig'idagi oylar sonini ifodalaydi (0 dan 11 gacha, yoki undan ko'p bo'lsa, yillarga aylantiriladi).
- Belgilash: Yil va oy o'rtasida chiziqcha (`-`) ishlatiladi.

**Misol**:
```sql
SELECT INTERVAL '1-2' AS interval_example;
```
**Natija**: `1 year 2 months`

**Jadvalda ishlatish**:
```sql
CREATE TABLE projects (
    project_id SERIAL PRIMARY KEY,
    project_name VARCHAR(100),
    duration INTERVAL NOT NULL
);

INSERT INTO projects (project_name, duration) VALUES ('App Development', '1-2');
```

**Natijani tekshirish**:
```sql
SELECT * FROM projects;
```
```
 project_id | project_name      | duration
------------|-------------------|----------------
          1 | App Development   | 1 year 2 mons
```

**Boshqa misollar**:
```sql
SELECT INTERVAL '5-0' AS five_years; -- 5 yil
SELECT INTERVAL '0-6' AS six_months; -- 6 oy
SELECT INTERVAL '10-11' AS ten_years_eleven_months; -- 10 yil 11 oy
```
**Natijalar**:
- `5 years`
- `6 months`
- `10 years 11 months`

## E'tibor beriladigan jihatlar
- **Cheklovlar**: SQL standart formati faqat yillar va oylarni ifodalaydi. Kunlar, soatlar, daqiqalar yoki soniyalar bu formatda kiritilmaydi. Buning uchun boshqa formatlar (masalan, `'2 days'`, `'P1DT2H'`) ishlatiladi.
- **Normallashtirish**: Agar oylar soni 12 yoki undan ko'p bo'lsa, PostgreSQL avtomatik ravishda yillarga aylantiradi:
  ```sql
  SELECT INTERVAL '1-14' AS interval;
  ```
  **Natija**: `2 years 2 months`
- **Manfiy qiymatlar**: Manfiy intervallar kiritilishi mumkin:
  ```sql
  SELECT INTERVAL '-1-2' AS negative_interval;
  ```
  **Natija**: `-1 year -2 months`
- **Sana bilan integratsiya**: SQL standart formatidagi `INTERVAL` `DATE` yoki `TIMESTAMP` bilan ishlatilishi mumkin:
  ```sql
  SELECT DATE '2025-05-09' + INTERVAL '1-2' AS future_date;
  ```
  **Natija**: `2026-07-09`

## Foydalanish bo'yicha maslahatlar
- SQL standart formati (`'yil-oy'`) yillar va oylar bilan ishlashda qulay, lekin kunlar yoki soatlar kabi boshqa birliklar kerak bo'lsa, boshqa formatlardan foydalaning.
- `INTERVAL` qiymatlarini aniq kiritish uchun ISO 8601 formatini (`P1Y2M`) ham ko'rib chiqing, chunki u ko'proq moslashuvchan.
- `CHECK` cheklovlari bilan intervallarni cheklash mumkin:
  ```sql
  CREATE TABLE projects (
      duration INTERVAL CHECK (duration >= INTERVAL '0-0')
  );
  ```

## Qo'shimcha resurslar
- [PostgreSQL rasmiy hujjatlari: Date/Time Types](https://www.postgresql.org/docs/current/datatype-datetime.html)
- [PostgreSQL rasmiy hujjatlari: Interval Input](https://www.postgresql.org/docs/current/datatype-datetime.html#DATATYPE-INTERVAL-INPUT)
