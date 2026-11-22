---
title: "Что делает waybackurls?"
id: note-04
created: "27.11.2025 20:07"
category: "02_Разведка и перечисление"
tags:
  - иб/разведка
  - recon
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `02_Разведка и перечисление` · Дата: `27.11.2025 20:07`

# Что делает waybackurls?

Что делает waybackurls?
Эта утилита получает все URL, которые когда-либо были сохранены в архиве Интернета (Wayback Machine) для заданного домена.

# Простой запрос для домена
waybackurls example.com

# Сохранение результатов в файл
waybackurls example.com > urls.txt

# Фильтрация уникальных URL
waybackurls example.com | sort -u


# Поиск определенных параметров
waybackurls target.com | grep "?id="

# Поиск API endpoints
waybackurls api.target.com | grep "/api/"

# Поиск файлов определенного типа
waybackurls target.com | grep "\.pdf"

# Анализ на интересные endpoints
cat historical_urls.txt | grep -E "(admin|login|api|config)"

# Поиск старых версий файлов
waybackurls vuln-site.com | grep "wp-config.php"

# Поиск параметров
waybackurls target.com | grep "?user="

waybackurls blog-site.com | grep "/2020/"

# 1. Найти старую админку
waybackurls victim.com | grep "admin"

# 2. Проверить жива ли она СЕЙЧАС
curl -I https://victim.com/old-admin/

# 3. Если жива - тестировать (с разрешения!)
