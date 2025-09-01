# Д.З. 9 Инициализация системы. Systemd.
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

## 1. Написать service...

Задаем пароль для root и в дальнейшем все действия выполняем под ним

		vagrant@sysd:~$ sudo passwd root
		New password:
		Retype new password:
		passwd: password updated successfully
		vagrant@sysd:~$ su -
		Password:

#### Конфигурирем сервис

создаём файл с конфигурацией для сервиса в директории /etc/default

		root@sysd:/vagrant# cat > /etc/default/log-monitor << 'END'
		# Конфигурационный файл для сервиса мониторинга логов
		LOG_FILE="/var/log/watchlog.log"
		KEYWORD="ALERT"
		END


Генерим лог-файл [watchlog.log](./watchlog.log)  и кладем его в папку /var/log

Создадим скрипт [log-monitor.sh ](./log-monitor.sh)

		root@sysd:/vagrant# cat > /usr/local/bin/log-monitor.sh << 'END'
		#!/bin/bash

		# Добавляем дату к строке вывода
		log_message() {
				logger "$(date '+%d.%m.%Y-%H:%M:%S') - $1"
		}

		# Проверим существование параметров
		if [ -z "$LOG_FILE" ] || [ -z "$KEYWORD" ]; then
				log_message "ОШИБКА: LOG_FILE или KEYWORD не заданы"
				exit 1
		fi

		# Проверим существования файла лога
		if [ ! -f "$LOG_FILE" ]; then
				log_message "ОШИБКА: Файл лога $LOG_FILE не найден"
				exit 1
		fi

		# Поиск ключевого слова
		matches=$(grep -c "$KEYWORD" "$LOG_FILE" 2>/dev/null)

		if [ "$matches" -gt 0 ]; then
				log_message "НАЙДЕНО: $matches вхождений ключевого слова '$KEYWORD' в $LOG_FILE"
				exit 0
		else
				log_message "Ключевое слово '$KEYWORD' не найдено"
				exit 0
		fi

		END



И добавляем права на запуск

		root@sysd:/vagrant# chmod +x /usr/local/bin/log-monitor.sh

Создадим юнит для сервиса [log-monitor.service ](./log-monitor.service):

		root@sysd:~# cat > /etc/systemd/system/log-monitor.service << 'END'
		[Unit]
		Description=Log Monitor Service
		After=network.target

		[Service]
		Type=oneshot
		User=root
		EnvironmentFile=/etc/default/log-monitor
		ExecStart=/usr/local/bin/log-monitor.sh

		# Настройки журналирования
		SyslogIdentifier=log-monitor
		SyslogFacility=local0
		SyslogLevel=info

		[Install]
		WantedBy=multi-user.target
		END

Создадим юнит для таймера [log-monitor.timer](./log-monitor.timer):

		root@sysd:~#cat > /etc/systemd/system/log-monitor.timer << 'END'
		[Unit]
		Description=Run log monitor every 30 seconds
		Requires=log-monitor.service

		[Timer]
		# Запускать каждые 30 секунд
		OnCalendar=*:*:0/30

		# Запускать через 30 секунд после загрузки системы
		OnBootSec=30s
		OnUnitActiveSec=30s

		# Настройки точности
		AccuracySec=1s

		[Install]
		WantedBy=timers.target
		END


#### Проверка работы

Перезагружаем демон systemd

	sudo systemctl daemon-reload

Включаем и запускаем таймер

	sudo systemctl enable log-monitor.timer
	sudo systemctl start log-monitor.timer

Проверяем статус таймера

		sudo systemctl status log-monitor.timer


Смотрим логи на предмет нужных нам сообщений

		tail -n 1000 /var/log/syslog  | grep слов

		Sep  1 12:00:30 ubuntu-jammy root: 01.09.2025-12:00:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:01:00 ubuntu-jammy root: 01.09.2025-12:01:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:01:30 ubuntu-jammy root: 01.09.2025-12:01:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:02:00 ubuntu-jammy root: 01.09.2025-12:02:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:02:30 ubuntu-jammy root: 01.09.2025-12:02:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:03:00 ubuntu-jammy root: 01.09.2025-12:03:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:03:30 ubuntu-jammy root: 01.09.2025-12:03:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:04:00 ubuntu-jammy root: 01.09.2025-12:04:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:04:30 ubuntu-jammy root: 01.09.2025-12:04:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:05:00 ubuntu-jammy root: 01.09.2025-12:05:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:05:30 ubuntu-jammy root: 01.09.2025-12:05:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:06:00 ubuntu-jammy root: 01.09.2025-12:06:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:06:30 ubuntu-jammy root: 01.09.2025-12:06:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:07:00 ubuntu-jammy root: 01.09.2025-12:07:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:07:30 ubuntu-jammy root: 01.09.2025-12:07:30 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
		Sep  1 12:08:00 ubuntu-jammy root: 01.09.2025-12:08:00 - НАЙДЕНО: 21 вхождений ключевого слова 'ALERT' в /var/log/watchlog.log
