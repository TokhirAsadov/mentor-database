# PostgreSQL-da Matn (String) Funksiyalari

Quyida `ASCII`, `CHR`, `CONCAT`, `||`, `CONCAT_WS`, `FORMAT`, `INITCAP`, `LEFT`, `LENGTH`, `LOWER`, `UPPER`, `POSITION`, `TRIM`, `REPEAT`, `REVERSE`, `RTRIM`, `LTRIM`, va `SUBSTRING` funksiyalarining har biri uchun tushuntirish va misollar keltirilgan.

---

## 1. `ASCII`
**Maqsad**: Berilgan matnning birinchi belgisining ASCII kodini qaytaradi.  
**Sintaksis**:
```sql
ASCII(text)
```

**Misol 1**: Ismning birinchi harfi ASCII kodi
```sql
SELECT ASCII('Ali') AS ascii_code;
```
**Natija**: `65` (A harfi uchun ASCII kodi).

**Misol 2**: Boshqa belgi
```sql
SELECT ASCII('z') AS ascii_code;
```
**Natija**: `122` (z harfi uchun ASCII kodi).

---

## 2. `CHR`
**Maqsad**: Berilgan ASCII kodiga mos belgi qaytaradi.  
**Sintaksis**:
```sql
CHR(integer)
```

**Misol 1**: ASCII koddan belgi
```sql
SELECT CHR(65) AS character;
```
**Natija**: `A`.

**Misol 2**: Boshqa kod
```sql
SELECT CHR(97) AS character;
```
**Natija**: `a`.

---

## 3. `CONCAT`
**-Maqsad**: Bir yoki bir nechta matnni birlashtiradi. `NULL` qiymatlar e’tiborga olinmaydi.  
**Sintaksis**:
```sql
CONCAT(text1, text2, ...)
```

**Misol 1**: Ism va familiyani birlashtirish
```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employees
WHERE employee_id = 1;
```
**Natija**: `Ali Vohidov` (agar `first_name = 'Ali'`, `last_name = 'Vohidov'` bo‘lsa).

**Misol 2**: NULL bilan
```sql
SELECT CONCAT('Hello', NULL, 'World') AS result;
```
**Natija**: `HelloWorld`.

---

## 4. `|| (String Concatenation Operator)`
**Maqsad**: Ikki yoki undan ko‘p matnni birlashtiradi. `NULL` bo‘lsa, natija `NULL` bo‘ladi.  
**Sintaksis**:
```sql
text1 || text2
```

**Misol 1**: Ism va bo‘lim
```sql
SELECT first_name || ' - ' || department_id AS name_dept
FROM employees
WHERE employee_id = 2;
```
**Natija**: `Bobur - 10` (agar `first_name = 'Bobur'`, `department_id = 10` bo‘lsa).

**Misol 2**: NULL bilan
```sql
SELECT 'Hello' || NULL AS result;
```
**Natija**: `NULL`.

---

## 5. `CONCAT_WS`
**Maqsad**: Matnlarni berilgan ayirgich (separator) bilan birlashtiradi. `NULL` qiymatlar e’tiborga olinmaydi.  
**Sintaksis**:
```sql
CONCAT_WS(separator, text1, text2, ...)
```

**Misol 1**: Ism va familiyani vergul bilan birlashtirish
```sql
SELECT CONCAT_WS(', ', first_name, last_name) AS full_name
FROM employees
WHERE employee_id = 3;
```
**Natija**: `Sitora, Usmonova`.

**Misol 2**: Ko‘p qiymatlar
```sql
SELECT CONCAT_WS(' | ', 'A', 'B', NULL, 'C') AS result;
```
**Natija**: `A | B | C`.

---

## 6. `FORMAT`
**Maqsad**: Matnni shablon asosida formatlaydi, `%s` o‘rniga qiymatlar qo‘yiladi.  
**Sintaksis**:
```sql
FORMAT(format_string, arg1, arg2, ...)
```

**Misol 1**: Xodim ma’lumotlarini formatlash
```sql
SELECT FORMAT('Xodim: %s, Bo‘lim: %s', first_name, department_id) AS info
FROM employees
WHERE employee_id = 4;
```
**Natija**: `Xodim: Jasur, Bo‘lim: 20`.

**Misol 2**: Tartibli format
```sql
SELECT FORMAT('1: %s, 2: %s', 'Bir', 'Ikki') AS result;
```
**Natija**: `1: Bir, 2: Ikki`.

---

## 7. `INITCAP`
**Maqsad**: Har bir so‘zning birinchi harfini katta, qolganlarini kichik harfga aylantiradi.  
**Sintaksis**:
```sql
INITCAP(text)
```

**Misol 1**: Ismni formatlash
```sql
SELECT INITCAP('ali vohidov') AS formatted_name;
```
**Natija**: `Ali Vohidov`.

**Misol 2**: Murakkab matn
```sql
SELECT INITCAP('hello world') AS formatted_text;
```
**Natija**: `Hello World`.

---

## 8. `LEFT`
**Maqsad**: Matnning chap tarafidan berilgan uzunlikdagi qismini qaytaradi.  
**Sintaksis**:
```sql
LEFT(text, length)
```

**Misol 1**: Ismning dastlabki 3 harfi
```sql
SELECT LEFT(first_name, 3) AS short_name
FROM employees
WHERE employee_id = 5;
```
**Natija**: `Zul` (agar `first_name = 'Zulfiya'` bo‘lsa).

**Misol 2**: Manfiy uzunlik
```sql
SELECT LEFT('PostgreSQL', -2) AS result;
```
**Natija**: `Postgre` (oxirgi 2 harf olib tashlanadi).

---

## 9. `LENGTH`
**Maqsad**: Matnning uzunligini (belgilar sonini) qaytaradi.  
**Sintaksis**:
```sql
LENGTH(text)
```

**Misol 1**: Ism uzunligi
```sql
SELECT first_name, LENGTH(first_name) AS name_length
FROM employees
WHERE employee_id = 6;
```
**Natija**: `Kamola, 6`.

**Misol 2**: Bo‘shliqli matn
```sql
SELECT LENGTH('Hello World') AS text_length;
```
**Natija**: `11`.

---

## 10. `LOWER`
**Maqsad**: Matnni kichik harflarga aylantiradi.  
**Sintaksis**:
```sql
LOWER(text)
```

**Misol 1**: Ismni kichik harfga aylantirish
```sql
SELECT LOWER(first_name) AS lower_name
FROM employees
WHERE employee_id = 7;
```
**Natija**: `shaxzod` (agar `first_name = 'Shaxzod'` bo‘lsa).

**Misol 2**: Aralash matn
```sql
SELECT LOWER('HeLLo WoRLD') AS result;
```
**Natija**: `hello world`.

---

## 11. `UPPER`
**Maqsad**: Matnni katta harflarga aylantiradi.  
**Sintaksis**:
```sql
UPPER(text)
```

**Misol 1**: Familiyani katta harfga aylantirish
```sql
SELECT UPPER(last_name) AS upper_surname
FROM employees
WHERE employee_id = 8;
```
**Natija**: `ABDULLAYEV`.

**Misol 2**: Aralash matn
```sql
SELECT UPPER('HeLLo WoRLD') AS result;
```
**Natija**: `HELLO WORLD`.

---

## 12. `POSITION`
**Maqsad**: Matnda berilgan qismning birinchi paydo bo‘lish pozitsiyasini qaytaradi.  
**Sintaksis**:
```sql
POSITION(substring IN string)
```

**Misol 1**: Harf pozitsiyasi
```sql
SELECT POSITION('lo' IN 'Hello') AS position;
```
**Natija**: `4` (lo 4-pozitsiyadan boshlanadi).

**Misol 2**: Topilmasa
```sql
SELECT POSITION('xyz' IN 'Hello') AS position;
```
**Natija**: `0` (topilmadi).

---

## 13. `TRIM`
**Maqsad**: Matnning boshidan, oxiridan yoki ikkalasidan berilgan belgilar (odatda bo‘shliq) olib tashlanadi.  
**Sintaksis**:
```sql
TRIM([LEADING | TRAILING | BOTH] [characters] FROM text)
```

**Misol 1**: Bo‘shliqlarni olib tashlash
```sql
SELECT TRIM('   Hello   ') AS trimmed;
```
**Natija**: `Hello`.

**Misol 2**: Belgilarni olib tashlash
```sql
SELECT TRIM(BOTH '*' FROM '***Hello***') AS trimmed;
```
**Natija**: `Hello`.

---

## 14. `REPEAT`
**Maqsad**: Matnni berilgan miqdorda takrorlaydi.  
**Sintaksis**:
```sql
REPEAT(text, number)
```

**Misol 1**: Matnni takrorlash
```sql
SELECT REPEAT('Hi ', 3) AS repeated;
```
**Natija**: `Hi Hi Hi `.

**Misol 2**: Bo‘lim identifikatori
```sql
SELECT REPEAT('0', 5) || department_id AS padded_id
FROM departments
WHERE department_id = 10;
```
**Natija**: `0000010`.

---

## 15. `REVERSE`
**Maqsad**: Matnni teskari tartibda qaytaradi.  
**Sintaksis**:
```sql
REVERSE(text)
```

**Misol 1**: Ismni teskari qilish
```sql
SELECT REVERSE(first_name) AS reversed_name
FROM employees
WHERE employee_id = 9;
```
**Natija**: `rusaJ` (agar `first_name = 'Jasur'` bo‘lsa).

**Misol 2**: Oddiy matn
```sql
SELECT REVERSE('Hello') AS result;
```
**Natija**: `olleH`.

---

## 16. `RTRIM`
**Maqsad**: Matnning o‘ng tarafidan (oxiridan) berilgan belgilar olib tashlanadi.  
**Sintaksis**:
```sql
RTRIM(text, characters)
```

**Misol 1**: Bo‘shliqlarni olib tashlash
```sql
SELECT RTRIM('Hello   ') AS trimmed;
```
**Natija**: `Hello`.

**Misol 2**: Belgilarni olib tashlash
```sql
SELECT RTRIM('Hello!!!', '!') AS trimmed;
```
**Natija**: `Hello`.

---

## 17. `LTRIM`
**Maqsad**: Matnning chap tarafidan (boshidan) berilgan belgilar olib tashlanadi.  
**Sintaksis**:
```sql
LTRIM(text, characters)
```

**Misol 1**: Bo‘shliqlarni olib tashlash
```sql
SELECT LTRIM('   Hello') AS trimmed;
```
**Natija**: `Hello`.

**Misol 2**: Belgilarni olib tashlash
```sql
SELECT LTRIM('###Hello', '#') AS trimmed;
```
**Natija**: `Hello`.

---

## 18. `SUBSTRING`
**Maqsad**: Matnning ma’lum bir qismini chiqaradi, boshlang‘ich pozitsiya va uzunlik yoki shablon asosida.  
**Sintaksis**:
```sql
SUBSTRING(text FROM start FOR length)
SUBSTRING(text, pattern)
```

**Misol 1**: Muayyan qismni chiqarish
```sql
SELECT SUBSTRING(first_name FROM 1 FOR 3) AS short_name
FROM employees
WHERE employee_id = 10;
```
**Natija**: `Ali` (agar `first_name = 'Aliyor'` bo‘lsa).

**Misol 2**: Shablon bilan
```sql
SELECT SUBSTRING('Phone: 123-456-7890' FROM '\d{3}-\d{3}-\d{4}') AS phone_number;
```
**Natija**: `123-456-7890`.
