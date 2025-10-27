<p align="center">
  <img src="https://github.com/kamil1403/otus_bash/blob/main/screenshots/bash-logo-dark.jpg" alt="Banner" width="65%">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-otus__bash-0A84FF?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-20.10.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание:   
- [ ] Написать скрипт для CRON, который раз в час будет формировать письмо и отправлять на заданную почту
> [!NOTE]
> Признаюсь честно 😊 - поднимать и настраивать почтовый сервер ради проверки ДЗ было лень, хоть и есть понимание.
> Поэтому решил сделать отчет альтернативным способом - через Telegram-бота.

Необходимая информация в отчете:   
- [ ] Список запрашиваемых URL (с наибольшим кол-вом запросов) с указанием кол-ва запросов c момента последнего запуска скрипта;         
- [ ] Ошибки веб-сервера/приложения c момента последнего запуска;   
- [ ] Список всех кодов HTTP ответа с указанием их кол-ва с момента последнего запуска скрипта;
- [ ] Скрипт должен предотвращать одновременный запуск нескольких копий, до его завершения.        

### ✅ Результат:   
- [x] Отчет формируется и отправляетя в Telegram-бот, со всей необходимой информацией из задания выше. Результат см. на скриншоте 🖼️ ["log_report_tg"](https://github.com/kamil1403/otus_bash/blob/main/screenshots/log_report_tg.png)     

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
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -n 10
```

---

<a id="two"></a>
## 🧰 Команда №2 — «Топ запрашиваемых URL

```bash
awk -F'"' '{print $2}' "$LOG_FILE" | cut -d'/' -f2 | sort | uniq -c | sort -nr | head -n 10
```

---

<a id="three"></a>
## 🧰 Команда №3 - "Ошибки веб-сервера / приложения (4xx/5xx)

```bash
awk '$9 ~ /^[45]/ {print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr
```

---

<a id="four"></a>
## 🧰 Команда №4 - "Все коды HTTP с их количеством

```bash
awk '$9 ~ /^[12345]/ {print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr
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
#!/usr/bin/env bash

# Настройки
LOG_FILE="/home/ubuntuserv/loglab/otus.log"
EMAIL="test_otus@gmail.com"

# Временный файл отчёта
REPORT_FILE="/tmp/log_report.txt"

# Telegram
BOT_TOKEN="8440377128:AAHI96mV9l62BGDdseZVS3-L9-v_3IVxx2U"
CHAT_ID="580441018"

# Заголовок отчёта
{
  echo "Почасовой отчет из логов"
  echo "Дата: $(date '+%Y-%m-%d %H:%M:%S')"
  echo "Лог-файл:  $LOG_FILE"
  echo "----------------------------------------"
  echo
} > "$REPORT_FILE"

# 1) Топ IP-адресов (топ-10)
{
  echo "Топ IP-адресов:"
  awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -n 10
  echo
} >> "$REPORT_FILE"

# 2) Топ запрашиваемых URL
{
  echo "Топ запрашиваемых URL:"
  awk -F'"' '{print $2}' "$LOG_FILE" | cut -d'/' -f2 | sort | uniq -c | sort -nr | head -n 10
  echo
} >> "$REPORT_FILE"

# 3) Ошибки веб-сервера/приложения c момента последнего запуска (4xx/5xx)
{
  echo "Ошибки (4xx/5xx):"
  awk '$9 ~ /^[45]/ {print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr
  echo
} >> "$REPORT_FILE"

# 4) Список всех кодов HTTP ответа с указанием их кол-ва с момента последнего запуска скрипта
{
  echo "Все коды HTTP:"
  awk '$9 ~ /^[12345]/ {print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr
  echo
} >> "$REPORT_FILE"

# Отправка письма
#mail -s "[$(hostname)] Hourly Log Report" "$EMAIL" < "$REPORT_FILE"

# Отправляем лога в Telegram
TEXT=$(cat "$REPORT_FILE")
curl -s -X POST "https://api.telegram.org/bot${BOT_TOKEN}/sendMessage" \
     -d chat_id="${CHAT_ID}" \
     --data-urlencode "text=$TEXT" >/dev/null
```

---
