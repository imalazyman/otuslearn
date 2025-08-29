# Д.З. 9 Инициализация системы. Systemd.
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/default).
2. Установить spawn-fcgi и создать unit-файл (spawn-fcgi.sevice) с помощью переделки init-скрипта (https://gist.github.com/cea2k/1318020).
3. Доработать unit-файл Nginx (nginx.service) для запуска нескольких инстансов сервера с разными конфигурационными файлами одновременно.

## 1. Написать service...

#### Конфигурация сервиса

создаём файл с конфигурацией для сервиса в директории /etc/default

		cat > watchlog << EOF
		# Configuration file for my watchlog service
		# Place it to /etc/default

		# File and word in that file that we will be monit
		WORD="ALERT"
		LOG=/var/log/watchlog.log

		EOF

Копируем его в папку /etc/default

		sudo cp ./watchlog /etc/default

Генерим лог-файл whatchlog.log [here](./whatchlog.log)  и кладем его в папку /var/log