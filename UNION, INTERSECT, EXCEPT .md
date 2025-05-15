# PostgreSQL To‘plam Operatorlari: `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT`

## Jadvallar
**employees**:
| employee_id | name       | department |
|------------|------------|------------|
| 1          | Ali        | IT         |
| 2          | Valijon    | HR         |
| 3          | Hasan      | Sales      |

**contractors**:
| contractor_id | name       | department |
|---------------|------------|------------|
| 1             | Ali        | IT         |
| 2             | Hasan      | Sales      |
| 3             | Zafar      | Marketing  |

---

## 1. `UNION`

Tushuntirish: Ikkala so‘rovning barcha noyob qatorlarini birlashtiradi, dublikatlar olib tashlanadi.

**Misol 1**: Barcha noyob ismlarni olish  
So‘rov:  
SELECT name FROM employees UNION SELECT name FROM contractors;  

| name       |
|------------|
| Ali        |
| Valijon    |
| Hasan      |
| Zafar      |

**Misol 2**: Noyob ism va bo‘limlarni olish  
So‘rov:  
SELECT name, department FROM employees UNION SELECT name, department FROM contractors;  

| name       | department |
|------------|------------|
| Ali        | IT         |
| Valijon    | HR         |
| Hasan      | Sales      |
| Zafar      | Marketing  |

---

## 2. `UNION ALL`

Tushuntirish: Ikkala so‘rovning barcha qatorlarini birlashtiradi, dublikatlar saqlanadi.

**Misol 1**: Barcha ismlarni olish (dublikatlar bilan)  
So‘rov:  
SELECT name FROM employees UNION ALL SELECT name FROM contractors;  

| name       |
|------------|
| Ali        |
| Valijon    |
| Hasan      |
| Ali        |
| Hasan      |
| Zafar      |

**Misol 2**: Barcha ism va bo‘limlarni olish  
So‘rov:  
SELECT name, department FROM employees UNION ALL SELECT name, department FROM contractors;  

| name       | department |
|------------|------------|
| Ali        | IT         |
| Valijon    | HR         |
| Hasan      | Sales      |
| Ali        | IT         |
| Hasan      | Sales      |
| Zafar      | Marketing  |

---

## 3. `INTERSECT`

Tushuntirish: Faqat ikkala so‘rovda umumiy bo‘lgan noyob qatorlarni qaytaradi.

**Misol 1**: Umumiy ismlarni topish  
So‘rov:  
SELECT name FROM employees INTERSECT SELECT name FROM contractors;  

| name       |
|------------|
| Ali        |
| Hasan      |

**Misol 2**: Umumiy ism va bo‘limlarni topish  
So‘rov:  
SELECT name, department FROM employees INTERSECT SELECT name, department FROM contractors;  

| name       | department |
|------------|------------|
| Ali        | IT         |
| Hasan      | Sales      |

---

## 4. `EXCEPT`

Tushuntirish: Birinchi so‘rovda bor, lekin ikkinchisida yo‘q noyob qatorlarni qaytaradi.

**Misol 1**: Faqat employees’da bor ismlarni topish  
So‘rov:  
SELECT name FROM employees EXCEPT SELECT name FROM contractors;  

| name       |
|------------|
| Valijon    |

**Misol 2**: Faqat employees’da bor ism va bo‘limlarni topish  
So‘rov:  
SELECT name, department FROM employees EXCEPT SELECT name, department FROM contractors;  

| name       | department |
|------------|------------|
| Valijon    | HR         |

---
- `UNION`: Noyob qatorlarni birlashtiradi.
- `UNION ALL`: Barcha qatorlarni, shu jumladan dublikatlarni birlashtiradi.
- `INTERSECT`: Faqat umumiy qatorlarni qaytaradi.
- `EXCEPT`: Birinchi so‘rovda bor, lekin ikkinchisida yo‘q qatorlarni qaytaradi.
