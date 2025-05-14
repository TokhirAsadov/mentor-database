# PostgreSQL-da WHERE Bandida Ishlatiladigan Operatorlar

PostgreSQL-da `WHERE` bandi so‘rov natijasini filtrlash uchun ishlatiladi. Ushbu bandda turli operatorlar yordamida shartlar yaratiladi. Quyida `WHERE` bandida ishlatiladigan asosiy operatorlar turlari, ularning maqsadi va misollar keltirilgan.

## 1. Taqqoslash Operatorlari
Taqqoslash operatorlari qiymatlarni solishtirish uchun ishlatiladi.

### 1.1. `=` (Tenglik)
**Maqsad**: Bir qiymat boshqasiga teng ekanligini tekshiradi.
```sql
SELECT first_name, salary
FROM employees
WHERE department_id = 10;
```
**Tushuntirish**: 10-bo‘limdagi xodimlarning ismlari va maoshlari qaytariladi.

### 1.2. `!=` yoki `<>` (Teng emas)
**Maqsad**: Bir qiymat boshqasiga teng emasligini tekshiradi.
```sql
SELECT first_name, last_name
FROM employees
WHERE salary != 5000;
```
**Tushuntirish**: Maoshi 5000 ga teng bo‘lmagan xodimlar qaytariladi.

### 1.3. `>` (Kattaroq)
**Maqsad**: Bir qiymat boshqasidan katta ekanligini tekshiradi.
```sql
SELECT first_name, salary
FROM employees
WHERE salary > 10000;
```
**Tushuntirish**: Maoshi 10000 dan yuqori xodimlar qaytariladi.

### 1.4. `<` (Kichikroq)
**Maqsad**: Bir qiymat boshqasidan kichik ekanligini tekshiradi.
```sql
SELECT first_name, age
FROM employees
WHERE age < 30;
```
**Tushuntirish**: 30 yoshdan kichik xodimlar qaytariladi.

### 1.5. `>=` (Kattaroq yoki teng)
**Maqsad**: Bir qiymat boshqasidan katta yoki teng ekanligini tekshiradi.
```sql
SELECT first_name, salary
FROM employees
WHERE salary >= 5000;
```
**Tushuntirish**: Maoshi 5000 yoki undan yuqori xodimlar qaytariladi.

### 1.6. `<=` (Kichikroq yoki teng)
**Maqsad**: Bir qiymat boshqasidan kichik yoki teng ekanligini tekshiradi.
```sql
SELECT first_name, age
FROM employees
WHERE age <= 25;
```
**Tushuntirish**: 25 yosh yoki undan kichik xodimlar qaytariladi.

---

## 2. Mantiqiy Operatorlar
Mantiqiy operatorlar bir nechta shartlarni birlashtirish uchun ishlatiladi.

### 2.1. `AND` (Va)
**Maqsad**: Ikkala shart ham to‘g‘ri bo‘lishi kerak.
```sql
SELECT first_name, salary
FROM employees
WHERE department_id = 20 AND salary > 6000;
```
**Tushuntirish**: 20-bo‘limdagi va maoshi 6000 dan yuqori xodimlar qaytariladi.

### 2.2. `OR` (Yoki)
**Maqsad**: Kamida bitta shart to‘g‘ri bo‘lsa, natija qaytariladi.
```sql
SELECT first_name, department_id
FROM employees
WHERE department_id = 10 OR department_id = 30;
```
**Tushuntirish**: 10 yoki 30-bo‘limdagi xodimlar qaytariladi.

### 2.3. `NOT` (Inkor)
**Maqsad**: Shartni inkor qiladi (teskari holatni tekshiradi).
```sql
SELECT first_name, department_id
FROM employees
WHERE NOT department_id = 40;
```
**Tushuntirish**: 40-bo‘limdan tashqari barcha xodimlar qaytariladi.

---

## 3. Maxsus Operatorlar
Maxsus operatorlar ma’lumotlarning muayyan shartlarga mosligini tekshirishda qo‘llaniladi.

### 3.1. `IN`
**Maqsad**: Qiymat berilgan ro‘yxatda bo‘lishini tekshiradi.
```sql
SELECT first_name, department_id
FROM employees
WHERE department_id IN (10, 20, 30);
```
**Tushuntirish**: 10, 20 yoki 30-bo‘limdagi xodimlar qaytariladi.

### 3.2. `BETWEEN`
**Maqsad**: Qiymat ma’lum oraliqda bo‘lishini tekshiradi (chegaralar kiritiladi).
```sql
SELECT first_name, salary
FROM employees
WHERE salary BETWEEN 4000 AND 8000;
```
**Tushuntirish**: Maoshi 4000 dan 8000 gacha bo‘lgan xodimlar qaytariladi.

### 3.3. `LIKE`
**Maqsad**: Matn shabloniga mos kelishini tekshiradi (`%` — har qanday belgilar, `_` — bitta belgi).
```sql
SELECT first_name
FROM employees
WHERE first_name LIKE 'A%';
```
**Tushuntirish**: Ismi A harfi bilan boshlanadigan xodimlar qaytariladi.

### 3.4. `ILIKE`
**Maqsad**: Katta-kichik harflarga sezgir bo‘lmagan `LIKE` (PostgreSQL-ga xos).
```sql
SELECT first_name
FROM employees
WHERE first_name ILIKE '%john%';
```
**Tushuntirish**: Ismida “john” so‘zi bo‘lgan xodimlar qaytariladi (katta-kichik harflarga e’tibor berilmaydi).

### 3.5. `IS NULL`
**Maqsad**: Qiymat `NULL` ekanligini tekshiradi.
```sql
SELECT first_name, manager_id
FROM employees
WHERE manager_id IS NULL;
```
**Tushuntirish**: Menejer ID’si `NULL` bo‘lgan xodimlar qaytariladi.

### 3.6. `IS NOT NULL`
**Maqsad**: Qiymat `NULL` emasligini tekshiradi.
```sql
SELECT first_name, manager_id
FROM employees
WHERE manager_id IS NOT NULL;
```
**Tushuntirish**: Menejer ID’si `NULL` bo‘lmagan xodimlar qaytariladi.

---

## 4. Subquery Operatorlari
Subquery operatorlari ichki so‘rov natijalari bilan taqqoslash uchun ishlatiladi.

### 4.1. `EXISTS`
**Maqsad**: Subquery’da kamida bitta qator mavjud bo‘lsa, natija qaytariladi.
```sql
SELECT first_name, department_id
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.department_id = e.department_id
      AND d.location = 'Paris'
);
```
**Tushuntirish**: Parijdagi bo‘limlarda ishlaydigan xodimlar qaytariladi.

### 4.2. `IN` (Subquery bilan)
**Maqsad**: Qiymat subquery natijalarida bo‘lishini tekshiradi.
```sql
SELECT first_name, department_id
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location = 'London'
);
```
**Tushuntirish**: Londondagi bo‘limlarda ishlaydigan xodimlar qaytariladi.

### 4.3. `ANY` / `SOME`
**Maqsad**: Qiymat subquery natijalaridan kamida bittasiga mos kelishini tekshiradi.
```sql
SELECT first_name, salary
FROM employees
WHERE salary > ANY (
    SELECT avg_salary
    FROM salary_stats
);
```
**Tushuntirish**: `salary_stats`dagi o‘rtacha maoshlardan kamida bittasidan yuqori maoshli xodimlar qaytariladi.

### 4.4. `ALL`
**Maqsad**: Qiymat subquery natijalarining hammasiga mos kelishini tekshiradi.
```sql
SELECT first_name, salary
FROM employees
WHERE salary > ALL (
    SELECT avg_salary
    FROM salary_stats
);
```
**Tushuntirish**: `salary_stats`dagi barcha o‘rtacha maoshlardan yuqori maoshli xodimlar qaytariladi.
