<p align="center">
  <img src="https://github.com/kamil1403/otus_bash/blob/main/screenshots/bash-logo-dark.jpg" alt="Banner" width="65%">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__bash-0A84FF?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-20.10.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание:   
- [ ] Написать скрипт для CRON, который раз в час будет формировать письмо и отправлять на заданную почту.
Необходимая информация в письме:   
- [ ] Список запрашиваемых URL (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;         
- [ ] Ошибки веб-сервера/приложения c момента последнего запуска;   
- [ ] Список всех кодов HTTP ответа с указанием их кол-ва с момента последнего запуска скрипта;
- [ ] Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.        

### ✅ Результат:   
- [x] Письмо формируется и отправляетя на заданную почту, со всей необходимой информацией из задания выше. Результат см. на скриншоте 🖼️ ["watchlog.timer"](https://github.com/kamil1403/otus_systemd/blob/main/screenshots/watchlog.timer.png)     

### 🧭 Оглавление
- [🧰 Команда №1 - "Топ IP по количеству запросов"](#one)
- [🧰 Команда №2 - "Топ запрашиваемых URL"](#two)
- [🧰 Команда №3 - "Ошибки веб-сервера / приложения (4xx/5xx)"](#three)
- [🧰 Команда №4 - "Все коды HTTP с их количеством"](#four) 
- [🚀 Краткий порядок действий (создание и запуск скрипта)(#start)
- [⏱️ Добавление скрипта в cron(#cron)
- [📝 Содержимое скрипта (#script)

---

<a id="unit"></a>
## 📋 Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова

```bash
# ----------------------------- STEP 1 -----------------------------
# Конфиг с переменными:
cd /etc/default/
nano watchlog

# ----------------------------- STEP 2 -----------------------------
# Вставить и сохранить:
# Configuration file for my watchlog service
# Place it to /etc/default
# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log

# ----------------------------- STEP 3 -----------------------------
# Лог-файл и тестовые строки (обязательно с ALERT):
cd /var/log/
nano watchlog.log

# ----------------------------- STEP 4 -----------------------------
# Скрипт:
bash
cd /opt
nano watchlog.sh

# ----------------------------- STEP 5 -----------------------------
# Вставить и сохранить:
#!/bin/bash
WORD=$1
LOG=$2
DATE=`date`
if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi

# ----------------------------- STEP 6 -----------------------------
# Сделать исполняемым:
chmod +x /opt/watchlog.sh

# ----------------------------- STEP 7 -----------------------------
# Юнит сервиса:
cd /etc/systemd/system
nano watchlog.service

# ----------------------------- STEP 8 -----------------------------
# ставить и сохранить:
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/default/watchlog
ExecStart=/opt/watchlog.sh $WORD $LOG

# ----------------------------- STEP 9 -----------------------------
# Юнит таймера:
nano watchlog.timer

# ----------------------------- STEP 10 ----------------------------
# Вставить и сохранить:
[Unit]
Description=Run watchlog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target

# ----------------------------- STEP 11 ----------------------------
# Запуск:
systemctl daemon-reload
systemctl start watchlog.service
systemctl start watchlog.timer

# ----------------------------- STEP 12 ----------------------------
# Проверка результата:
tail -n 1000 /var/log/syslog | grep word

# ----------------------------- RESULT -----------------------------
# Готово. Если увидим строку: "I found word, Master!" - значит сервис работает.
```

---

<a id="spawn"></a>
## ⚙️ Установить spawn-fcgi и создать unit-файл 

```bash
# ----------------------------- STEP 1 -----------------------------
# Устанавливаем spawn-fcgi и необходимые для него пакеты:
apt install spawn-fcgi php php-cgi php-cli \ apache2 libapache2-mod-fcgid -y

# ----------------------------- STEP 2 -----------------------------
# Скрипт:
cd /etc/spawn-fcgi
nano fcgi.conf

# ----------------------------- STEP 3 -----------------------------
# Вставить и сохранить:
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u www-data -g www-data -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"

# ----------------------------- STEP 3 -----------------------------
# Юнит сервиса:
cd /etc/systemd/system
nano spawn-fcgi.service

# ----------------------------- STEP 4 -----------------------------
# Вставить и сохранить:
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/spawn-fcgi/fcgi.conf
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

# ----------------------------- STEP 5 ----------------------------
# Запуск:
systemctl daemon-reload
systemctl start spawn-fcgi
systemctl start spawn-fcgi

# ----------------------------- STEP 12 ----------------------------
# Проверка результата:
tail -n 1000 /var/log/syslog | grep word

# ----------------------------- RESULT -----------------------------
# Готово. Если увидим строку: "active (running)" - значит сервис работает.

```

---

<a id="nginx"></a>
## 🌐 Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов

```bash
# ----------------------------- STEP 1 -----------------------------
# Установка  
apt install nginx -y

# ----------------------------- STEP 2 -----------------------------
# Шаблонный unit для инстансов:
nano /etc/systemd/system/nginx@.service

# ----------------------------- STEP 3 -----------------------------
# Вставить и сохранить:
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target
[Service]
Type=forking
PIDFile=/run/nginx-%I.pid
ExecStartPre=/usr/sbin/nginx -t -c /etc/nginx/nginx-%I.conf -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -c /etc/nginx/nginx-%I.conf -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx-%I.pid
TimeoutStopSec=5
KillMode=mixed
[Install]
WantedBy=multi-user.target

# ----------------------------- STEP 4 -----------------------------
# Два конфига из базового (/etc/nginx/nginx-first.conf, /etc/nginx/nginx-second.conf):
cd /etc/nginx/nginx.conf /etc/nginx/nginx-first.conf
nano /etc/nginx/nginx-first.conf
cp /etc/nginx/nginx.conf /etc/nginx/nginx-second.conf
nano /etc/nginx/nginx-second.conf

# ----------------------------- STEP 5 -----------------------------
# Минимально меняю (пример для nginx-first и nginx-second). У каждого инстанса свой PID-файл и свой порт:
pid /run/nginx-first.pid;
http {
    server {
        listen 9001;
    }
    # include /etc/nginx/sites-enabled/*;
    # (при необходимости — свои логи/root и т.д.)
}
pid /run/nginx-second.pid;
http {
    server {
        listen 9002;
    }
    # include /etc/nginx/sites-enabled/*;
}

# ----------------------------- STEP 6 -----------------------------
# Старт и проверка:
# Если мы видим две группы процессов Nginx, то все в порядке. 
# Если сервисы не стартуют, смотрим их статус, ищем ошибки, проверяем ошибки в /var/log/nginx/error.log, а также в journalctl -u nginx@first.
systemctl start nginx@first
systemctl start nginx@second
systemctl status nginx@first
systemctl status nginx@second

# ----------------------------- STEP 7 -----------------------------
# Проверяю порты и процессы:
ss -tnulp | grep nginx
ps afx | grep nginx

# ----------------------------- STEP 8 -----------------------------
# Если что-то не взлетело:
journalctl -u nginx@first -n 100 --no-pager
tail -n 100 /var/log/nginx/error.log
```

---
