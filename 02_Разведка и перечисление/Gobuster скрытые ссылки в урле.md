---
title: "Gobuster скрытые ссылки в урле"
id: note-05
created: "28.11.2025 17:03"
category: "02_Разведка и перечисление"
tags:
  - иб/разведка
  - recon
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `02_Разведка и перечисление` · Дата: `28.11.2025 17:03`

# Gobuster скрытые ссылки в урле

Gobuster скрытые ссылки в урле 

gobuster dir -u http://MACHINE_IP:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt

Для перечисления подоменов
 для обноружения типа
Сайт:www.site.com подбор будет типа:
Портал для сотрудников : staff.site.com, admin.site.com, internal.site.com 
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
