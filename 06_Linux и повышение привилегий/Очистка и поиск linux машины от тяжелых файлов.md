---
title: "Очистка и поиск linux машины от тяжелых файлов"
id: note-63
created: "10.07.2026 14:42"
category: "06_Linux и повышение привилегий"
tags:
  - иб/linux
  - privesc
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `06_Linux и повышение привилегий` · Дата: `10.07.2026 14:42`

# Очистка и поиск linux машины от тяжелых файлов

Быстрый обзор смонтированных разделов
df -h
Топ самых тяжёлых папок первого уровня в корне (/)
sudo du -sh /* 2>/dev/null | sort -rh | head -20

Рекурсивный поиск самых больших файлов и папок (глубже)
sudo du -ah / 2>/dev/null | sort -rh | head -30

Найти большие файлы по конкретному пути (например, /var/log)
sudo find /var/log -type f -size +100M -exec ls -lh {} \; 2>/dev/null

Очистка кэша скачанных пакетов
sudo apt clean

Кэш пользовательских директорий (не системного root, а обычного юзера)
rm -rf ~/.cache/*

Общий совет: «генеральная уборка» за один прогон
sudo apt autoremove --purge -y
sudo apt clean
sudo journalctl --vacuum-size=100M
sudo rm -rf /tmp/* /var/tmp/*
