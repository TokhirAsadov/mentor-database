# PostgreSQL-da Sana va Vaqt Funksiyalari

---

## 1. AGE
**Maqsad**: Ikki sana o‘rtasidagi farqni yil, oy va kun sifatida qaytaradi yoki berilgan sanadan joriy sanani ayirib, yoshni hisoblaydi.  
**Sintaksis**:
```sql
AGE(timestamp, timestamp)
AGE(timestamp)
```
- Birinchi shaklda ikki sana o‘rtasidagi farqni qaytaradi.
- Ikkinchi shaklda joriy sanadan berilgan sanani ayiradi.

**Misol 1**: Xodimning yoshini hisoblash
```sql
SELECT first_name, AGE(birth_date) AS age
FROM employees
WHERE employee_id = 1;
```
**Natija**: Masalan, `birth_date` 1990-05-14 bo‘lsa va joriy sana 2025-05-14 bo‘lsa, natija `35 years` bo‘ladi.

**Misol 2**: Ikki sana o‘rtasidagi farq
```sql
SELECT AGE('2025-05-14'::date, '2020-01-01'::date) AS time_difference;
```
**Natija**: `5 years 4 mons 13 days`.

---

## 2. EXTRACT
**Maqsad**: Sana yoki vaqtning muayyan qismini (yil, oy, kun, soat va hokazo) chiqaradi.  
**Sintaksis**:
```sql
EXTRACT(field FROM source)
```
- `field`: `YEAR`, `MONTH`, `DAY`, `HOUR`, `MINUTE`, `SECOND` va boshqalar.
- `source`: `TIMESTAMP`, `DATE` yoki `INTERVAL`.

**Misol 1**: Xodimning ishga qabul yilini chiqarish
```sql
SELECT first_name, EXTRACT(YEAR FROM hire_date) AS hire_year
FROM employees
WHERE employee_id = 2;
```
**Natija**: Agar `hire_date` 2020-03-15 bo‘lsa, natija `2020` bo‘ladi.

**Misol 2**: Joriy oyni chiqarish
```sql
SELECT EXTRACT(MONTH FROM CURRENT_DATE) AS current_month;
```
**Natija**: Agar joriy sana 2025-05-14 bo‘lsa, natija `5` bo‘ladi.

---

## 3. CURRENT_DATE
**Maqsad**: Joriy sanani (yil-oy-kun) qaytaradi, vaqt qismini o‘z ichiga olmaydi.  
**Sintaksis**:
```sql
CURRENT_DATE
```

**Misol 1**: Bugungi sanani ko‘rsatish
```sql
SELECT CURRENT_DATE AS today;
```
**Natija**: `2025-05-14` (joriy sana).

**Misol 2**: Bugundan keyin ishga qabul qilingan xodimlarni topish
```sql
SELECT first_name, hire_date
FROM employees
WHERE hire_date > CURRENT_DATE;
```
**Natija**: Kelajakda ishga qabul qilingan xodimlarni qaytaradi (odatda bo‘sh natija).

---

## 4. CURRENT_TIME(precision)
**Maqsad**: Joriy vaqtni (soat, daqiqa, soniya) qaytaradi, sana qismini o‘z ichiga olmaydi. Aniqlik darajasini belgilash mumkin.  
**Sintaksis**:
```sql
CURRENT_TIME(precision)
```
- `precision`: Soniyadan keyingi kasr qismining aniqligi (0-6).

**Misol 1**: Joriy vaqtni ko‘rsatish
```sql
SELECT CURRENT_TIME AS now;
```
**Natija**: Masalan, `09:56:23.123456+05`.

**Misol 2**: Aniqlik bilan
```sql
SELECT CURRENT_TIME(2) AS precise_time;
```
**Natija**: `09:56:23.12+05` (soniyadan keyin 2 raqam).

---

## 5. CURRENT_TIMESTAMP(precision)
**Maqsad**: Joriy sana va vaqtni (yil-oy-kun soat:daqiqa:soniya) qaytaradi. Aniqlik darajasini belgilash mumkin.  
**Sintaksis**:
```sql
CURRENT_TIMESTAMP(precision)
```
- `precision`: Soniyadan keyingi kasr qismining aniqligi (0-6).

**Misol 1**: Joriy sana va vaqtni ko‘rsatish
```sql
SELECT CURRENT_TIMESTAMP AS now;
```
**Natija**: `2025-05-14 09:56:23.123456+05`.

**Misol 2**: Aniqlik bilan
```sql
SELECT CURRENT_TIMESTAMP(0) AS rounded_timestamp;
```
**Natija**: `2025-05-14 09:56:23+05` (kasrsiz).

---

## 6. LOCALTIME(precision)
**Maqsad**: Joriy vaqtni mahalliy vaqt mintaqasida qaytaradi, sana qismini o‘z ichiga olmaydi.  
**Sintaksis**:
```sql
LOCALTIME(precision)
```
- `precision`: Soniyadan keyingi kasr qismining aniqligi (0-6).

**Misol 1**: Mahalliy vaqtni ko‘rsatish
```sql
SELECT LOCALTIME AS local_time;
```
**Natija**: `09:56:23.123456`.

**Misol 2**: Aniqlik bilan
```sql
SELECT LOCALTIME(3) AS precise_local_time;
```
**Natija**: `09:56:23.123`.

---

## 7. LOCALTIMESTAMP(precision)
**Maqsad**: Joriy sana va vaqtni mahalliy vaqt mintaqasida qaytaradi.  
**Sintaksis**:
```sql
LOCALTIMESTAMP(precision)
```
- `precision`: Soniyadan keyingi kasr qismining aniqligi (0-6).

**Misol 1**: Mahalliy sana va vaqtni ko‘rsatish
```sql
SELECT LOCALTIMESTAMP AS local_timestamp;
```
**Natija**: `2025-05-14 09:56:23.123456`.

**Misol 2**: Aniqlik bilan
```sql
SELECT LOCALTIMESTAMP(1) AS rounded_local_timestamp;
```
**Natija**: `2025-05-14 09:56:23.1`.

---

## 8. TO_DATE(text, pattern)
**Maqsad**: Matnni berilgan shablon asosida `DATE` turiga aylantiradi.  
**Sintaksis**:
```sql
TO_DATE(text, pattern)
```
- `text`: Sana sifatida o‘zgartiriladigan matn.
- `pattern`: Matn formatini tasvirlaydigan shablon (masalan, `YYYY-MM-DD`).

**Misol 1**: Matnni sanaga aylantirish
```sql
SELECT TO_DATE('2025-05-14', 'YYYY-MM-DD') AS converted_date;
```
**Natija**: `2025-05-14`.

**Misol 2**: Boshqa format
```sql
SELECT TO_DATE('14/05/2025', 'DD/MM/YYYY') AS converted_date;
```
**Natija**: `2025-05-14`.

---

## 9. TO_TIMESTAMP(text, pattern)
**Maqsad**: Matnni berilgan shablon asosida `TIMESTAMP` turiga aylantiradi.  
**Sintaksis**:
```sql
TO_TIMESTAMP(text, pattern)
```
- `text`: Sana va vaqt sifatida o‘zgartiriladigan matn.
- `pattern`: Matn formatini tasvirlaydigan shablon.

**Misol 1**: Matnni timestamp’ga aylantirish
```sql
SELECT TO_TIMESTAMP('2025-05-14 09:56:23', 'YYYY-MM-DD HH24:MI:SS') AS converted_timestamp;
```
**Natija**: `2025-05-14 09:56:23`.

**Misol 2**: Boshqa format
```sql
SELECT TO_TIMESTAMP('14-May-2025 09:56', 'DD-Mon-YYYY HH24:MI') AS converted_timestamp;
```
**Natija**: `2025-05-14 09:56:00`.
