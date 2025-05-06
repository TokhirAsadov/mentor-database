# PostgreSQL Commands

Quyidagi jadvalda PostgreSQL (`psql`) da eng ko‘p ishlatiladigan buyruqlar va ularning qisqacha tavsifi keltirilgan. Ushbu ma'lumotlar README.md fayli uchun moslashtirilgan bo‘lib, loyihangizda PostgreSQL buyruqlarini hujjatlashtirish uchun foydali bo‘ladi.

| **Buyruq**                     | **Tavsif**                                                                 |
|--------------------------------|----------------------------------------------------------------------------|
| `\l`                          | Barcha ma'lumotlar bazalarini ro‘yxatini ko‘rsatadi.                       |
| `\c database_name`            | Belgilangan ma'lumotlar bazasiga ulanadi.                                  |
| `\dt`                         | Joriy sxemadagi barcha jadvallarni ro‘yxatini ko‘rsatadi.                  |
| `\d table_name`               | Belgilangan jadvalning tuzilishini (ustunlar, turlar) ko‘rsatadi.          |
| `\d+ table_name`              | Jadval tuzilishi va qo‘shimcha ma'lumotlarni (indekslar, cheklovlar) ko‘rsatadi. |
| `\du`                         | Barcha foydalanuvchilar va ularning rollarini ko‘rsatadi.                  |
| `\dn`                         | Joriy ma'lumotlar bazasidagi sxemalarni ro‘yxatini ko‘rsatadi.             |
| `\i file_name`                | Belgilangan fayldagi SQL buyruqlarini bajaradi.                            |
| `\o file_name`                | So‘rov natijalarini belgilangan faylga yozadi.                             |
| `\q`                          | psql seansidan chiqadi.                                                   |
| `\h command`                  | Belgilangan SQL buyruq haqida yordam ma'lumotini ko‘rsatadi (masalan, `\h SELECT`). |
| `\timing`                     | So‘rovlarning bajarilish vaqtini ko‘rsatishni yoqadi/yoki o‘chiradi.       |
| `\x`                          | Kengaytirilgan jadval formatini yoqadi/yoki o‘chiradi (vertikal ko‘rinish).|
| `\copy table_name FROM 'file'`| Fayldan jadvalga ma'lumotlarni nusxalaydi.                                 |
| `\copy table_name TO 'file'`  | Jadvaldan faylga ma'lumotlarni nusxalaydi.                                 |
| `CREATE DATABASE db_name;`    | Yangi ma'lumotlar bazasini yaratadi.                                      |
| `CREATE TABLE table_name (...);`| Yangi jadval yaratadi (ustunlar va turlarni belgilash bilan).             |
| `INSERT INTO table_name (...) VALUES (...);`| Jadvalga yangi yozuv qo‘shadi.                              |
| `SELECT * FROM table_name;`   | Jadvaldagi barcha yozuvlarni tanlaydi.                                    |
| `UPDATE table_name SET column = value WHERE condition;`| Jadvaldagi ma'lumotlarni yangilaydi.                    |
| `DELETE FROM table_name WHERE condition;`| Jadvaldagi yozuvlarni o‘chiradi.                              |
| `ALTER TABLE table_name ADD COLUMN column_name data_type;`| Jadvalga yangi ustun qo‘shadi.             |
| `DROP TABLE table_name;`      | Belgilangan jadvalni o‘chiradi.                                           |
| `DROP DATABASE db_name;`      | Belgilangan ma'lumotlar bazasini o‘chiradi.                               |
| `GRANT privilege ON table_name TO user;`| Foydalanuvchiga jadval uchun ruxsat beradi.                |
| `REVOKE privilege ON table_name FROM user;`| Foydalanuvchidan jadval ruxsatini olib qo‘yadi.         |
| `VACUUM table_name;`          | Jadvalni optimallashtiradi va bo‘sh joyni tozalaydi.                      |
| `EXPLAIN query;`              | So‘rovning bajarilish rejasini ko‘rsatadi.                                |
| `pg_dump -t table_name db_name > file.sql`| Jadvalni zaxiralash uchun SQL faylga eksport qiladi.     |

## Qo‘shimcha eslatmalar
- **SQL buyruqlari**: Yuqoridagi SQL buyruqlari (`CREATE`, `SELECT`, `INSERT` va h.k.) PostgreSQLning asosiy so‘rov tili bo‘lib, ularni `psql` yoki boshqa mijozlar orqali ishlatish mumkin.
- **Meta-buyruqlar**: `\l`, `\dt`, `\d` kabi buyruqlar `psql` ichidagi meta-buyruqlar bo‘lib, faqat `psql` terminalida ishlaydi.
- **Yordam**: Har qanday buyruq haqida qo‘shimcha ma'lumot olish uchun `\h` yoki PostgreSQL rasmiy hujjatlariga murojaat qiling: [PostgreSQL Documentation](https://www.postgresql.org/docs/).
- **Fayllar bilan ishlash**: `\copy` fayllardan ma'lumot import/eksport qilish uchun ishlatiladi, lekin fayl yo‘llari server yoki mijoz joylashuviga bog‘liq bo‘lishi mumkin.
