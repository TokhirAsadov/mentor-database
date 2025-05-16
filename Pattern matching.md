# PostgreSQL-da SIMILAR TO va POSIX Regular Expression Shablon Belgilari

PostgreSQL-da `SIMILAR TO` va `POSIX Regular Expressions` matnni qidirish va filtrlash uchun shablon belgilardan foydalanadi. `SIMILAR TO` SQL standartiga asoslangan oddiy shablonlarni qo‘llab-quvvatlaydi, `POSIX Regular Expressions` esa ko‘proq moslashuvchan va an’anaviy regular expression sintaksisiga ega. Quyida har bir shablon belgisi uchun tushuntirish va misollar keltirilgan.

---

## 1. SIMILAR TO Shablon Belgilari

`SIMILAR TO` operatori `LIKE`ning kengaytirilgan shakli bo‘lib, quyidagi shablon belgilarni qo‘llab-quvvatlaydi.

---

### 1.1. `%` (Har qanday uzunlikdagi belgilar)
**Maqsad**: Nol yoki undan ko‘p har qanday belgilarni ifodalaydi.  
**Sintaksis**: `text SIMILAR TO 'pattern%'`

**Misol 1**: A bilan boshlanadigan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO 'A%';
```
**Natija**: `Ali`, `Alim`, `Aziza`.

**Misol 2**: .com bilan tugaydigan e-pochtalar
```sql
SELECT email
FROM users
WHERE email SIMILAR TO '%.com';
```
**Natija**: `user1@example.com`, `test@site.com`.

---

### 1.2. `_` (Bitta belgi)
**Maqsad**: Faqat bitta har qanday belgi.  
**Sintaksis**: `text SIMILAR TO 'pattern_'`

**Misol 1**: Uch harfli ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO '___';
```
**Natija**: `Ali`, `Zul`, `Bob`.

**Misol 2**: Ikkinchi harfi “l” bo‘lgan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO '_l_';
```
**Natija**: `Ali`, `Elm`.

---

### 1.3. `|` (Alternativa, yoki)
**Maqsad**: Bir nechta shablonlardan biriga moslikni tekshiradi.  
**Sintaksis**: `text SIMILAR TO 'pattern1|pattern2'`

**Misol 1**: Phone yoki Tablet bilan boshlanadigan mahsulotlar
```sql
SELECT product_name
FROM products
WHERE product_name SIMILAR TO '(Phone|Tablet)%';
```
**Natija**: `Phone X`, `Tablet Pro`.

**Misol 2**: A yoki B bilan boshlanadigan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO 'A|B%';
```
**Natija**: `Ali`, `Bobur`.

---

### 1.4. `()` (Guruhlash)
**Maqsad**: Shablonlarni guruhlaydi, odatda `|` bilan ishlatiladi.  
**Sintaksis**: `text SIMILAR TO '(pattern1|pattern2)'`

**Misol 1**: Muayyan so‘zlar bilan boshlanadigan kodlar
```sql
SELECT product_code
FROM products
WHERE product_code SIMILAR TO '(AB|CD)-[0-9]%';
```
**Natija**: `AB-123`, `CD-456`.

**Misol 2**: Guruhlangan alternativa
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO '(Al|Zu)%';
```
**Natija**: `Ali`, `Zulfiya`.

---

### 1.5. `[]` (Belgilar to‘plami)
**Maqsad**: Belgilar oralig‘idagi yoki ro‘yxatdagi belgilardan biriga moslik.  
**Sintaksis**: `text SIMILAR TO '[range]'`

**Misol 1**: Kichik harf bilan boshlanadigan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO '[a-z]%';
```
**Natija**: `ali`, `bobur`.

**Misol 2**: Muayyan harflar bilan boshlanadiganlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO '[ABC]%';
```
**Natija**: `Ali`, `Bobur`, `Cem`.

---

### 1.6. `*` (Nol yoki ko‘p takror)
**Maqsad**: Oldingi belgi yoki guruhning nol yoki undan ko‘p takrorlanishi.  
**Sintaksis**: `text SIMILAR TO 'pattern*'`

**Misol 1**: A harfi takrorlanishi mumkin bo‘lgan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO 'A[a-z]*';
```
**Natija**: `Ali`, `Aziza`.

**Misol 2**: Raqamlar takrorlanishi
```sql
SELECT product_code
FROM products
WHERE product_code SIMILAR TO 'P[0-9]*';
```
**Natija**: `P`, `P123`, `P456789`.

---

### 1.7. `+` (Bir yoki ko‘p takror)
**Maqsad**: Oldingi belgi yoki guruhning kamida bir marta takrorlanishi.  
**Sintaksis**: `text SIMILAR TO 'pattern+'`

**Misol 1**: Kamida bitta raqamli kodlar
```sql
SELECT product_code
FROM products
WHERE product_code SIMILAR TO 'P[0-9]+';
```
**Natija**: `P123`, `P456789` (lekin `P` emas).

**Misol 2**: Harf takrorlanadigan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO 'A[a-z]+';
```
**Natija**: `Ali`, `Aziza` (lekin `A` emas).

---

### 1.8. `?` (Nol yoki bir marta takror)
**Maqsad**: Oldingi belgi yoki guruhning ixtiyoriy (nol yoki bir marta) takrorlanishi.  
**Sintaksis**: `text SIMILAR TO 'pattern?'`

**Misol 1**: Ixtiyoriy harfli ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name SIMILAR TO 'Al[i]?';
```
**Natija**: `Ali`, `Al`.

**Misol 2**: Ixtiyoriy raqamli kod
```sql
SELECT product_code
FROM products
WHERE product_code SIMILAR TO 'P[0-9]?';
```
**Natija**: `P`, `P1`.

---

## 2. POSIX Regular Expression Shablon Belgilari

POSIX regular expressions an’anaviy regular expression sintaksisiga asoslanadi va `~`, `~*`, `!~`, `!~*` operatorlari bilan ishlaydi. Quyida har bir shablon belgisi uchun misollar keltirilgan.

---

### 2.1. `.` (Har qanday bitta belgi)
**Maqsad**: Har qanday bitta belgi (yangi qator bundan mustasno).  
**Sintaksis**: `text ~ 'pattern.'`

**Misol 1**: Ikkinchi pozitsiyada har qanday belgi
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^A.';
```
**Natija**: `Ali`, `Az`.

**Misol 2**: E-pochtada nuqta oldin belgi
```sql
SELECT email
FROM users
WHERE email ~ '@.\.com';
```
**Natija**: `user1@x.com`, `test@y.com`.

---

### 2.2. `*` (Nol yoki ko‘p takror)
**Maqsad**: Oldingi belgi yoki guruhning nol yoki undan ko‘p takrorlanishi.  
**Sintaksis**: `text ~ 'pattern*'`

**Misol 1**: Raqamlar takrorlanishi
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^P\d*';
```
**Natija**: `P`, `P123`, `P456789`.

**Misol 2**: Harf takrorlanishi
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^A[a-z]*';
```
**Natija**: `A`, `Ali`, `Aziza`.

---

### 2.3. `+` (Bir yoki ko‘p takror)
**Maqsad**: Oldingi belgi yoki guruhning kamida bir marta takrorlanishi.  
**Sintaksis**: `text ~ 'pattern+'`

**Misol 1**: Kamida bitta raqam
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^P\d+';
```
**Natija**: `P123`, `P456789` (lekin `P` emas).

**Misol 2**: Harf takrorlanishi
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^A[a-z]+';
```
**Natija**: `Ali`, `Aziza` (lekin `A` emas).

---

### 2.4. `?` (Nol yoki bir marta takror)
**Maqsad**: Oldingi belgi yoki guruhning ixtiyoriy takrorlanishi.  
**Sintaksis**: `text ~ 'pattern?'`

**Misol 1**: Ixtiyoriy raqam
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^P\d?';
```
**Natija**: `P`, `P1`.

**Misol 2**: Ixtiyoriy harf
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^Al[i]?';
```
**Natija**: `Ali`, `Al`.

---

### 2.5. `|` (Alternativa, yoki)
**Maqsad**: Bir nechta shablonlardan biriga moslik.  
**Sintaksis**: `text ~ 'pattern1|pattern2'`

**Misol 1**: Phone yoki Tablet
```sql
Select product_name
FROM products
WHERE product_name ~ '^(Phone|Tablet)';
```
**Natija**: `Phone X`, `Tablet Pro`.

**Misol 2**: A yoki B bilan boshlanadiganlar
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^A|B';
```
**Natija**: `Ali`, `Bobur`.

---

### 2.6. `()` (Guruhlash)
**Maqsad**: Shablonlarni guruhlash, odatda `|` yoki takrorlash bilan ishlatiladi.  
**Sintaksis**: `text ~ '(pattern)'`

**Misol 1**: Guruhlangan kodlar
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^(AB|CD)-\d{3}';
```
**Natija**: `AB-123`, `CD-456`.

**Misol 2**: Guruhlangan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^(Al|Zu)';
```
**Natija**: `Ali`, `Zulfiya`.

---

### 2.7. `[]` (Belgilar to‘plami)
**Maqsad**: Belgilar oralig‘i yoki ro‘yxatdagi belgilardan biriga moslik.  
**Sintaksis**: `text ~ '[range]'`

**Misol 1**: Harf bilan boshlanadiganlar
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^[a-zA-Z]';
```
**Natija**: `Ali`, `Bobur`, `Zulfiya`.

**Misol 2**: Muayyan raqamlar
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^P[0-2]';
```
**Natija**: `P0`, `P1`, `P2`.

---

### 2.8. `^` (Matn boshi)
**Maqsad**: Matnning boshlanishini belgilaydi.  
**Sintaksis**: `text ~ '^pattern'`

**Misol 1**: A bilan boshlanadigan ismlar
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^A';
```
**Natija**: `Ali`, `Aziza`.

**Misol 2**: Kodning boshlanishi
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^P';
```
**Natija**: `P123`, `P456`.

---

### 2.9. `$` (Matn oxiri)
**Maqsad**: Matnning tugashini belgilaydi.  
**Sintaksis**: `text ~ 'pattern$'`

**Misol 1**: .com bilan tugaydiganlar
```sql
SELECT email
FROM users
WHERE email ~ '\.com$';
```
**Natija**: `user1@example.com`, `test@site.com`.

**Misol 2**: Raqam bilan tugaydiganlar
```sql
SELECT product_code
FROM products
WHERE product_code ~ '[0-9]$';
```
**Natija**: `P123`, `AB-456`.

---

### 2.10. `\` (Maxsus belgilarni ekransizlantirish)
**Maqsad**: Maxsus belgilarning literal ma’nosini ishlatish.  
**Sintaksis**: `text ~ '\special_char'`

**Misol 1**: Nuqta bilan moslashish
```sql
SELECT email
FROM users
WHERE email ~ '\.';
```
**Natija**: `user1@example.com`, `test@site.org`.

**Misol 2**: Yulduz belgisi
```sql
SELECT description
FROM products
WHERE description ~ '\*';
```
**Natija**: `Item * Special`.

---

### 2.11. `\d` (Raqam)
**Maqsad**: Har qanday raqam (`[0-9]`).  
**Sintaksis**: `text ~ '\d'`

**Misol 1**: Raqamli kodlar
```sql
SELECT product_code
FROM products
WHERE product_code ~ '^\d{3}$';
```
**Natija**: `123`, `456`.

**Misol 2**: Telefon raqami
```sql
SELECT phone_number
FROM contacts
WHERE phone_number ~ '^\+\d{2}-\d{3}-\d{7}$';
```
**Natija**: `+99-123-4567890`.

---

### 2.12. `\w` (Harf yoki raqam)
**Maqsad**: Harf, raqam yoki pastki chiziq (`[a-zA-Z0-9_]`).  
**Sintaksis**: `text ~ '\w'`

**Misol 1**: Foydalanuvchi nomi
```sql
SELECT email
FROM users
WHERE email ~ '^\w+@';
```
**Natija**: `user1@example.com`, `test123@site.com`.

**Misol 2**: So‘zlar
```sql
SELECT first_name
FROM employees
WHERE first_name ~ '^\w+$';
```
**Natija**: `Ali`, `Bobur`.

---

### 2.13. `\s` (Bo‘shliq belgisi)
**Maqsad**: Bo‘shliq, tab, yangi qator kabi bo‘shliq belgilari.  
**Sintaksis**: `text ~ '\s'`

**Misol 1**: Bo‘shliq bor matn
```sql
SELECT description
FROM products
WHERE description ~ '\s';
```
**Natija**: `Phone X Pro`, `Tablet with Case`.

**Misol 2**: So‘zlar orasidagi bo‘shliq
```sql
SELECT full_name
FROM employees
WHERE full_name ~ '^\w+\s\w+$';
```
**Natija**: `Ali Vohidov`, `Sitora Usmonova`.

---

## Xulosa
`SIMILAR TO` shablon belgilari (`%`, `_`, `|`, `()`, `[]`, `*`, `+`, `?`) oddiy va tushunarli filtrlash uchun qulay, lekin cheklangan imkoniyatlarga ega. POSIX regular expressions shablonlari (`.`, `*`, `+`, `?`, `|`, `()`, `[]`, `^`, `$`, `\`, `\d`, `\w`, `\s`) esa ko‘proq moslashuvchanlik va aniqlik beradi, ayniqsa murakkab shablonlar va funksiyalar (`SUBSTRING`, `REGEXP_REPLACE`) bilan ishlaganda. To‘g‘ri shablonni tanlash so‘rovning samaradorligi va aniqligini oshiradi.
