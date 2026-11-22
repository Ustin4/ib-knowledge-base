---
title: "Поиск вех уязвимостей для эскалации привилегий в linux после смотреть заметки для эксплуатации"
id: note-41
created: "03.02.2026 09:08"
category: "06_Linux и повышение привилегий"
tags:
  - иб/linux
  - privesc
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `06_Linux и повышение привилегий` · Дата: `03.02.2026 09:08`

# Поиск вех уязвимостей для эскалации привилегий в linux после смотреть заметки для эксплуатации

 КЛЮЧЕВЫЕ КОМАНДЫ ОДНОЙ СТРОКОЙ
Полная разведка за 30 секунд
id; sudo -l; find / -perm -4000 -type f 2>/dev/null; cat /etc/crontab; cat /etc/exports 2>/dev/null

1. РАЗВЕДКА СИСТЕМЫ
1.1 Информация о системе
uname -a                                                               # Версия ядра
cat /etc/os-release                                              # Дистрибутив и версия
hostname                                                             # Имя хоста

1.2 Информация о пользователях
id                                                                           # Текущий пользователь и группы
whoami                                                                # Текущий пользователь
cat /etc/passwd                                                  # Все пользователи
cat /etc/shadow 2>/dev/null                             # Хэши паролей (если доступно)
sudo -l                                                                # Что можно запускать через sudo
2. ПОИСК SUID/SGID ФАЙЛОВ
2.1 Все SUID/SGID файлы
find / -type f -perm -4000 -ls 2>/dev/null        # SUID
find / -type f -perm -2000 -ls 2>/dev/null        # SGID
find / -type f -perm -6000 -ls 2>/dev/null        # Оба

2.2 Проверка на GTFOBins
Для каждого найденного SUID файла проверяем:
https://gtfobins.github.io/#+suid
Примеры опасных: find, nano, vim, bash, more, less, awk
3. ПОИСК НЕБЕЗОПАСНЫХ РАЗРЕШЕНИЙ ФАЙЛОВ
3.1 Файлы, доступные для записи всем
find / -type f -perm -o+w 2>/dev/null

3.2 Директории, доступные для записи всем
find / -type d -perm -o+w 2>/dev/null

3.3 Файлы, принадлежащие текущему пользователю
find / -type f -user $(whoami) 2>/dev/null

3.4 Исполняемые файлы в домашней директории
find /home -type f -executable 2>/dev/null

4. CRON JOBS И АВТОМАТИЧЕСКИЕ ЗАДАЧИ
4.1 Системные cron задачи
cat /etc/crontab
ls -la /etc/cron.d/
cat /etc/cron.d/* 2>/dev/null

4.2 Пользовательские cron задачи
ls -la /var/spool/cron/crontabs/
cat /var/spool/cron/crontabs/* 2>/dev/null 2>/dev/null

4.3 Ежедневные/еженедельные задачи
ls -la /etc/cron.hourly/ /etc/cron.daily/ /etc/cron.weekly/ /etc/cron.monthly/

4.4 Проверка writable скриптов в cron
find /etc/cron* -type f -writable 2>/dev/null

5. СЕТЕВЫЕ СЕРВИСЫ И NFS
5.1 Активные соединения
netstat -tulpn
ss -tulpn

5.2 Проверка NFS уязвимостей
cat /etc/exports | grep no_root_squash
showmount -e localhost 2>/dev/null
showmount -e 127.0.0.1 2>/dev/null

5.3 Смонтированные NFS shares
mount | grep nfs
cat /proc/mounts | grep nfs

6. ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ И PATH
6.1 Текущие переменные окружения
env
echo $PATH

6.2 Проверка на PATH hijacking
Ищем SUID программы, вызывающие команды без полного пути
for file in $(find / -perm -4000 -type f 2>/dev/null); do
    echo "=== $file ==="
    strings $file 2>/dev/null | grep -E "system\(|exec\(|popen\(" | grep -v "/"
done

6.3 Проверка LD_PRELOAD и LD_LIBRARY_PATH
echo $LD_PRELOAD
echo $LD_LIBRARY_PATH
7. ПРОЦЕССЫ И СЕРВИСЫ
7.1 Запущенные процессы
ps aux
ps aux | grep root

7.2 Сервисы, запущенные от root
systemctl list-units --type=service --state=running
service --status-all 2>/dev/null

7.3 Сетевые сервисы на localhost
netstat -tulpn | grep 127.0.0.1
ss -tulpn | grep 127.0.0.1
8. ЯДРО И МОДУЛИ
8.1 Информация о ядре
cat /proc/version
uname -r

8.2 Загруженные модули ядра
lsmod
cat /proc/modules

8.3 Параметры ядра
sysctl -a 2>/dev/null | grep -v "denied"
cat /etc/sysctl.conf 2>/dev/null
9. ЛОГИ И ИСТОРИЯ
9.1 История команд
history
cat ~/.bash_history

9.2 Логи авторизации
cat /var/log/auth.log 2>/dev/null
cat /var/log/secure 2>/dev/null

9.3 Другие логи
ls -la /var/log/

АВТОМАТИЧЕСКИЕ ИНСТРУМЕНТЫ
12.1 LinPEAS (скачать и запустить)
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

12.2 Linux Exploit Suggester
# https://github.com/mzet-/linux-exploit-suggester

12.3 Linux Smart Enumeration
curl https://github.com/diego-treitos/linux-smart-enumeration/raw/master/lse.sh | sh

## Ссылки

- [GitHub - The-Z-Labs/linux-exploit-suggester: Linux privilege escalation auditing tool](https://github.com/The-Z-Labs/linux-exploit-suggester)
- [GTFOBins](https://gtfobins.org/)
