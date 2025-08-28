# Д.З. 8 Загрузка системы
1. Включить отображение меню Grub.
2. Попасть в систему без пароля несколькими способами.
3. Установить систему с LVM, после чего переименовать VG.


## 1. Включить отображение меню Grub

#### Редактирование файла конфигурации grub
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

#### Обновление конфигурации файла загрузки и перезагрузка машины

        debugger@ubuntu-learn:~$ sudo update-grub
        Sourcing file `/etc/default/grub'
        Sourcing file `/etc/default/grub.d/kdump-tools.cfg'
        Generating grub configuration file ...
        Found linux image: /boot/vmlinuz-6.11.0-25-generic
        Found initrd image: /boot/initrd.img-6.11.0-25-generic
        Found linux image: /boot/vmlinuz-6.11.0-24-generic
        Found initrd image: /boot/initrd.img-6.11.0-24-generic
        Warning: os-prober will not be executed to detect other bootable partitions.
        Systems on them will not be added to the GRUB boot configuration.
        Check GRUB_DISABLE_OS_PROBER documentation entry.
        Adding boot menu entry for UEFI Firmware Settings ...
        done
        debugger@ubuntu-learn:~$ sudo reboot

#### После перезагрузки виден экран выбора вариантов загрузки

![grub_boot](grub-menu.jpg).