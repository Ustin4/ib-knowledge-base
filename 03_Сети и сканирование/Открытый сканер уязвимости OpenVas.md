---
title: "Открытый сканер уязвимости OpenVas"
id: note-11
created: "19.12.2025 21:07"
category: "03_Сети и сканирование"
tags:
  - иб/сети
  - сканирование
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `03_Сети и сканирование` · Дата: `19.12.2025 21:07`

# Открытый сканер уязвимости OpenVas

Открытый сканер уязвимости OpenVas

Установка через докер чтоб меньше хлопот с уязвимостями 

sudo apt install docker.io
sudo docker run -d -p 443:443 --name openvas immauss/openvas

Запуск 
docker start openvas
Подключение через https://127.0.0.1/login/login.html
Стандартный учетка для входа 
Username: admin
Password: admin
