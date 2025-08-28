# Д.З. 8 Загрузка системы
1. Включить отображение меню Grub.
2. Попасть в систему без пароля несколькими способами.
3. Установить систему с LVM, после чего переименовать VG.


## 1. Редактирование файла конфигурации grub

Приводим файл к виду:

        GRUB_DEFAULT=0
        GRUB_TIMEOUT_STYLE=menu
        GRUB_TIMEOUT=10
        GRUB_DISTRIBUTOR=`( . /etc/os-release; echo ${NAME:-Ubuntu} ) 2>/dev/null || echo Ubuntu`
        GRUB_CMDLINE_LINUX_DEFAULT=""
        GRUB_CMDLINE_LINUX=""

так же расскоментируем строку

        #GRUB_DISABLE_RECOVERY="true"
и изменим ее сл. образом:

        GRUB_DISABLE_RECOVERY="false"

