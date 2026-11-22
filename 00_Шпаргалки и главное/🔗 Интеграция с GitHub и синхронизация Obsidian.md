---
title: "Интеграция с GitHub и синхронизация Obsidian"
tags:
  - obsidian
  - github
  - синхронизация
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]]

# 🔗 Интеграция с GitHub и синхронизация Obsidian

Локальный Git-репозиторий уже инициализирован в папке `D:\Конспект по ИБ\Конспект`, создан `.gitignore` (чтобы не засорять репозиторий временными файлами рабочего пространства Obsidian), и сделан первый коммит.

---

## Шаг 1. Создай приватный репозиторий на GitHub
1. Открой [github.com/new](https://github.com/new).
2. Укажи название, например: `ib-knowledge-base`.
3. Выбери **Private** (приватный), чтобы твои заметки не были доступны публично.
4. **Не** добавляй README, `.gitignore` или лицензию (они уже есть локально).
5. Нажми **Create repository**.

---

## Шаг 2. Привяжи локальное хранилище к GitHub
Открой терминал (PowerShell) в папке `D:\Конспект по ИБ\Конспект` и выполни (подставив свой логин и имя репозитория):

```powershell
git branch -M main
git remote add origin https://github.com/<ТВОЙ_ЛОГИН>/ib-knowledge-base.git
git push -u origin main
```

---

## Шаг 3. Настрой автосинхронизацию в Obsidian (плагин Obsidian Git)
1. В Obsidian открой **Настройки (Settings)** → **Сторонние плагины (Community plugins)**.
2. Нажми **Обзор (Browse)**, найди **Obsidian Git** (автор: Vinzent03) и нажми **Установить** → **Включить**.
3. В настройках плагина **Obsidian Git** рекомендуемые параметры:
   - **Auto backup interval (minutes)**: `10` (автоматический коммит и push каждые 10 минут)
   - **Auto pull interval (minutes)**: `10` (автоматически подтягивать изменения с других устройств)
   - **Pull on startup**: `Вкл` (подтягивать свежие изменения при открытии Obsidian)
   - **Commit message**: `vault backup: {{date}}`

---

## Полезные команды Git вручную
```powershell
# Проверить статус изменений
git status

# Сохранить и отправить изменения вручную
git add .
git commit -m "Обновление конспекта"
git push
```
