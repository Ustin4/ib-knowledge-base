---
title: "Взлом хешей John"
id: note-07
created: "06.12.2025 20:54"
category: "08_Пароли и криптография"
tags:
  - иб/крипто
  - пароли
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `08_Пароли и криптография` · Дата: `06.12.2025 20:54`

# Взлом хешей John

Базовый синтексис взлома в John
john --wordlist=[path to wordlist] [path to file]
Пример: 
john --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
 
Взлом, специфичный для формата
john --format=[format] --wordlist=[path to wordlist] [path to file]
Пример: 
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt

Взлом хешей из /etc/shadow
unshadow [path to passwd] [path to shadow]
Пример:
unshadow local_passwd  local_shadow > unshadowed.txt
Взлом:
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt

Взлом  защищеного zip  и rar файла
zip2john [options] [zip file] > [output file]
rar2john [rar file] > [output file]
Пример:
zip2john zipfile.zip > zip_hash.txt
/opt/john/rar2john rarfile.rar > rar_hash.txt
Взлом:
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

Пользовательские правила
john --wordlist=[path to wordlist] --rule=PoloPassword [path to file]


Взлом паролей SSH -ключей
ssh2john [id_rsa private key file] > [output file]
Пример:
/opt/john/ssh2john.py id_rsa > id_rsa_hash.txt
Взлом:
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
