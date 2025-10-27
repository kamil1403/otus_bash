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
- [🚀 Краткий порядок действий (создание и запуск скрипта)](#start)
- [⏱️ Добавление скрипта в cron](#cron)
- [📝 Содержимое скрипта](#script)

---

<a id="one"></a>
## 🧰 Команда №1 - "Топ IP по количеству запросов

```bash
awk '{print $1}' otus.log | sort | uniq -c | sort -nr
```

---

<a id="two"></a>
## 🧰 Команда №2 — «Топ первых сегментов URL

```bash
awk -F'"' '{print $2}' otus.log | cut -d'/' -f2 | sort | uniq -c | sort -nr | head -n 11
```

---

<a id="three"></a>
## 🧰 Команда №3 - "Ошибки веб-сервера / приложения (4xx/5xx)

```bash
awk '$9 ~ /^[45]/ {print $9}' otus.log | sort | uniq -c | sort -nr
```

---

<a id="four"></a>
## 🧰 Команда №4 - "Все коды HTTP с их количеством

```bash
awk '$9 ~ /^[12345]/ {print $9}' otus.log | sort | uniq -c | sort -nr
```

---

<a id="start"></a>
## 🚀 Краткий порядок действий (создание и запуск скрипта)

```bash
# ----------------------------- STEP 1 -----------------------------
# Создаем файл скрипта:
touch ~/loglab/log_report.sh

# ----------------------------- STEP 2 -----------------------------
# Создаем временный файл для отчета:
touch /tmp/log_report.txt

# ----------------------------- STEP 3 -----------------------------
# Делаем скрипт исполняемым:
chmod +x ~/loglab/log_report.sh

# ----------------------------- STEP 4 -----------------------------
# Запускаем вручную для проверки:
./log_report.sh
```

---

<a id="cron"></a>
## ⏱️ Добавление в cron

```bash
# ----------------------------- STEP 1 -----------------------------
# Сделать скрипт исполняемым:
chmod +x /home/ubuntuserv/loglab/log_report.sh

# ----------------------------- STEP 2 -----------------------------
# Открыть cron:
crontab -e

# ----------------------------- STEP 3 -----------------------------
# Добавить строку:
0 * * * * /home/ubuntuserv/loglab/log_report.sh >> /home/ubuntuserv/loglab/cron.log 2>&1
```

---

<a id="script"></a>
## 📝 Содержимое скрипта

```bash
awk '$9 ~ /^[45]/ {print $9}' otus.log | sort | uniq -c | sort -nr
```

---
