---
title: "Аналитика сетевой безопасности: фильтры Wireshark и NetworkMiner"
id: note-47
created: "18.02.2026 10:03"
category: "03_Сети и сканирование"
tags:
  - иб/сети
  - сканирование
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `03_Сети и сканирование` · Дата: `18.02.2026 10:03`

# Аналитика сетевой безопасности: фильтры Wireshark и NetworkMiner

Wireshark (display filters)
Назначение 
SYN-скан (окно ≤1024)
Фильтр 
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size<=1024
Назначение 
Connect-скан
Фильтр 
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size>1024
Назначение 
NULL-скан
Фильтр 
tcp.flags==0
Назначение 
Xmas-скан
Фильтр 
tcp.flags.fin==1 and tcp.flags.urg==1 and tcp.flags.push==1
Назначение 
ARP-спуфинг
Фильтр 
arp.duplicate-address-detected
Назначение 
DNS-туннель
Фильтр 
dns.qry.name.len > 40
Назначение 
HTTP-методы
Фильтр 
http.request.method in {"PUT","DELETE","CONNECT","TRACE"}
Назначение 
User-Agent (sqlmap/nmap)
Фильтр 
http.user_agent contains "sqlmap" or http.user_agent contains "nmap"
Назначение 
ICMP-туннель
Фильтр 
icmp.type==8 and data.len>48
Назначение 
Подозрительные файлы
Фильтр 
http.request.uri matches "\.(exe|dll|zip|rar|pdf)$"
Назначение 
FTP-пароли
Фильтр 
ftp.request.command=="PASS"
Назначение 
IP-адрес
Фильтр 
ip.addr == 192.168.1.1
Назначение 
Порт
Фильтр 
tcp.port==80 или udp.port==53
Назначение 
Сеть
Фильтр 
ip.addr==192.168.1.0/24
NetworkMiner (быстрый анализ)
Вкладка
Что смотрим
Hosts
Все хосты, ОС, открытые порты. Щелчок ПКМ → Filter on this IP
Files
Извлечённые файлы (фильтр по расширению .exe, .pdf и т.д.)
Credentials
Логины/пароли в открытом виде (HTTP Basic, FTP, SMTP)
Sessions
Все TCP/UDP сессии (сортировка по размеру/длительности)
DNS
DNS-запросы (ищи длинные имена)
Images
Картинки из HTTP
Запуск в Linux
Wireshark: wireshark & или sudo wireshark -i eth0 -k
NetworkMiner: mono NetworkMiner.exe [file.pcap] (из папки инструмента)
tshark: tshark -r file.pcap -Y "фильтр" -T fields -e поле
Capture filters (при захвате)
host 192.168.1.10 — трафик только с этого IP
port 80 — только порт 80
tcp port 22 — только SSH
not host 10.0.0.1 — исключить IP
tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0 — только SYN-пакеты
