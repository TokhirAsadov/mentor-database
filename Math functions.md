# PostgreSQL-da Matematik Funksiyalar

Quyida `ABS`, `CBRT`, `CEIL`, `FLOOR`, `LN`, `LOG`, `PI`, `SQRT`, `RANDOM`, `TRUNC`, va `POWER` funksiyalarining har biri uchun tushuntirish va misollar keltirilgan.

---

## 1. `ABS`
**Maqsad**: Sonning mutlaq qiymatini (absolyut qiymatini) qaytaradi, ya’ni manfiy bo‘lsa musbat qiladi.  
**Sintaksis**:
```sql
ABS(number)
```

**Misol 1**: Xodim maoshidagi farqning mutlaq qiymati
```sql
SELECT first_name, ABS(salary - 5000) AS salary_difference
FROM employees
WHERE employee_id = 1;
```
**Natija**: Agar `salary = 4000` bo‘lsa, natija `1000` bo‘ladi.

**Misol 2**: Oddiy son bilan
```sql
SELECT ABS(-25.5) AS absolute_value;
```
**Natija**: `25.5`.

---

## 2. `CBRT`
**Maqsad**: Sonning kub ildizini qaytaradi.  
**Sintaksis**:
```sql
CBRT(number)
```

**Misol 1**: Kub ildiz hisoblash
```sql
SELECT CBRT(27) AS cube_root;
```
**Natija**: `3` (27 ning kub ildizi).

**Misol 2**: Maosh bilan
```sql
SELECT first_name, CBRT(salary) AS cube_root_salary
FROM employees
WHERE employee_id = 2;
```
**Natija**: Agar `salary = 8000` bo‘lsa, natija taxminan `20.0` bo‘ladi.

---

## 3. `CEIL`
**Maqsad**: Sonni eng yaqin katta butun songa yaxlitlaydi (tavanga).  
**Sintaksis**:
```sql
CEIL(number)
```

**Misol 1**: Maoshni yaxlitlash
```sql
SELECT first_name, CEIL(salary / 1000.0) AS salary_thousands
FROM employees
WHERE employee_id = 3;
```
**Natija**: Agar `salary = 5500` bo‘lsa, natija `6` bo‘ladi.

**Misol 2**: Oddiy son bilan
```sql
SELECT CEIL(4.3) AS ceiling_value;
```
**Natija**: `5`.

---

## 4. `FLOOR`
**Maqsad**: Sonni eng yaqin kichik butun songa yaxlitlaydi (porga).  
**Sintaksis**:
```sql
FLOOR(number)
```

**Misol 1**: Maoshni yaxlitlash
```sql
SELECT first_name, FLOOR(salary / 1000.0) AS salary_thousands
FROM employees
WHERE employee_id = 4;
```
**Natija**: Agar `salary = 5500` bo‘lsa, natija `5` bo‘ladi.

**Misol 2**: Oddiy son bilan
```sql
SELECT FLOOR(4.7) AS floor_value;
```
**Natija**: `4`.

---

## 5. `LN`
**Maqsad**: Sonning tabiiy logarifmini (e asosida) qaytaradi. Faqat musbat sonlar uchun ishlaydi.  
**Sintaksis**:
```sql
LN(number)
```

**Misol 1**: Maoshning tabiiy logarifmi
```sql
SELECT first_name, LN(salary) AS log_salary
FROM employees
WHERE employee_id = 5;
```
**Natija**: Agar `salary = 10000` bo‘lsa, natija taxminan `9.2103` bo‘ladi.

**Misol 2**: Oddiy son bilan
```sql
SELECT LN(2.71828) AS natural_log;
```
**Natija**: Taxminan `1.0` (e ning logarifmi).

---

## 6. `LOG`
**Maqsad**: Sonning berilgan asos bo‘yicha logarifmini qaytaradi (standart asos 10).  
**Sintaksis**:
```sql
LOG(base, number)
LOG(number) -- 10 asosida
```

**Misol 1**: 10 asosida logarifm
```sql
SELECT first_name, LOG(salary) AS log_salary
FROM employees
WHERE employee_id = 6;
```
**Natija**: Agar `salary = 10000` bo‘lsa, natija `4` bo‘ladi.

**Misol 2**: Maxsus asos bilan
```sql
SELECT LOG(2, 8) AS log_base_2;
```
**Natija**: `3` (2 asosida 8 ning logarifmi).

---

## 7. `PI`
**Maqsad**: Matematik konstanta π (pi, taxminan 3.14159) ni qaytaradi.  
**Sintaksis**:
```sql
PI()
```

**Misol 1**: Pi qiymatini ko‘rsatish
```sql
SELECT PI() AS pi_value;
```
**Natija**: `3.141592653589793`.

**Misol 2**: Doira yuzini hisoblash
```sql
SELECT PI() * POWER(radius, 2) AS circle_area
FROM shapes
WHERE shape_id = 1;
```
**Natija**: Agar `radius = 5` bo‘lsa, natija taxminan `78.5398` bo‘ladi.

---

## 8. `SQRT`
**Maqsad**: Sonning kvadrat ildizini qaytaradi. Faqat musbat sonlar uchun ishlaydi.  
**Sintaksis**:
```sql
SQRT(number)
```

**Misol 1**: Maoshning kvadrat ildizi
```sql
SELECT first_name, SQRT(salary) AS sqrt_salary
FROM employees
WHERE employee_id = 7;
```
**Natija**: Agar `salary = 2500` bo‘lsa, natija `50` bo‘ladi.

**Misol 2**: Oddiy son bilan
```sql
SELECT SQRT(16) AS square_root;
```
**Natija**: `4`.

---

## 9. `RANDOM`
**Maqsad**: 0 (shu jumladan) va 1 (bundan tashqari) orasidagi tasodifiy son qaytaradi.  
**Sintaksis**:
```sql
RANDOM()
```

**Misol 1**: Tasodifiy xodim tanlash
```sql
SELECT first_name
FROM employees
ORDER BY RANDOM()
LIMIT 1;
```
**Natija**: Tasodifiy xodim ismi, masalan, `Ali`.

**Misol 2**: Tasodifiy butun son (1-100)
```sql
SELECT FLOOR(RANDOM() * 100 + 1) AS random_number;
```
**Natija**: 1 dan 100 gacha tasodifiy son, masalan, `73`.

---

## 10. `TRUNC`
**Maqsad**: Sonning kasr qismini olib tashlaydi va butun qismini qaytaradi.  
**Sintaksis**:
```sql
TRUNC(number, [precision])
```

**Misol 1**: Maoshni butun qismga yaxlitlash
```sql
SELECT first_name, TRUNC(salary / 1000.0) AS salary_thousands
FROM employees
WHERE employee_id = 8;
```
**Natija**: Agar `salary = 5678.9` bo‘lsa, natija `5` bo‘ladi.

**Misol 2**: Aniqlik bilan
```sql
SELECT TRUNC(123.45678, 2) AS truncated_value;
```
**Natija**: `123.45`.

---

## 11. `POWER`
**Maqsad**: Sonni berilgan darajaga ko‘taradi (daraja).  
**Sintaksis**:
```sql
POWER(base, exponent)
```

**Misol 1**: Maoshni kvadratga ko‘tarish
```sql
SELECT first_name, POWER(salary / 1000.0, 2) AS salary_squared
FROM employees
WHERE employee_id = 9;
```
**Natija**: Agar `salary = 5000` bo‘lsa, natija `25` bo‘ladi.

**Misol 2**: Oddiy son bilan
```sql
SELECT POWER(2, 3) AS power_value;
```
**Natija**: `8` (2 ning 3-darajasi).
