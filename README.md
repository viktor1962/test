# test
install test archlinux


Проверяем наличие интернет соединения:

# ping google.com


Если оно отсутствует, подключаемся к сети:

# wifi-menu


В случае проводного соединения, при загрузке с установочного образа подключение к сети происходит автоматически при помощи утилиты dhcpcd 




1. ========= Проверяем активность режима UEFI.

# ls /sys/firmware/efi/efivars

Если каталог efivars не пуст - продолжаем установку.

Достаточно распространненная проблема с загрузкой установочного образа в UEFI режиме - включенный UEFI Secure в биосе




2. ========= Смотрим доступные диски и разделы

# lsblk




3. ========= Разбивка диска 

# gdisk /dev/sda

Список доступных команд можно почмотреть, нажав ?

Создадим новую таблицу разделов нажатием клавишу o

Далее создадим разделы нажатием клавиши n  (см. видео)

boot 256-512M  EF00  (BIOS boot partition)
root 20-40G    8300  
swap RAM+1G    8200 (кол-во оперативной памяти + 1 Гб)
home           8300  (оставшееся место)

В конце не забываем сохранить изменения - w



4. ========= Форматирование разделов

# mkfs.vfat -F32 /dev/sda1 
# mkfs.ext4 /dev/sda2
# mkswap /dev/sda3
# swapon /dev/sda3
# mkfs.ext4 /dev/sda4


5. ========= Обновление системных часов

# timedatectl set-ntp true



6. ========= Выбор оптимального репозитория Arch

nano /etc/pacman.d/mirrorlist

Переносим подходящий сервер в начало списка.

=========
Нужные нам комбинации клавиш для текстового редактора nano
Ctrl + 6  - Выделить;
Alt + 6   - Копировать;
Ctrl + K  - Вырезать;
Ctrl + U  -Вставить;
Ctrl + X -  Выход с подтверждением сохранения - Y/N Enter;
F2 - Сохранить.
=========



7. ========= Монтирование разделов

# mount /dev/sda2 /mnt
# mkdir -p /mnt/{boot,home}
# mount /dev/sda1 /mnt/boot
# mount /dev/sda4 /mnt/home



8. ========= Установка

# pacstrap -i /mnt base base-devel

Генерируем файл fstab с информацией о разделах

# genfstab -U -p /mnt >> /mnt/etc/fstab



9. ========= Переходим в корневой каталог новой системы

# arch-chroot /mnt /bin/bash


10. ========= Включаем нужные локали

Включаем необходимые локали

# nano /etc/locale.gen   (раскоментируем нужные строки в кодировке UTF8)

# locale-gen

Поддержка кирилицы в консоли
создаем файл
# nano /etc/vconsole.conf

и добавляем в него

KEYMAP=ru
FONT=cyr-sun16




10. ========= Настройка часового пояса

# ln -s /usr/share/zoneinfo/Europe/Kiev /etc/localtime
(в каталоге zoneinfo присуствуют файлы временных зон для все регионов)

Если если выпадет ошибка, что якобы файл существует. Лучше его пересоздать.
# rm /etc/localtime
# ln -s /usr/share/zoneinfo/Europe/Kiev /etc/localtime


Синхронизация аппаратных часов

# hwclock --systohc --utc


11. ========= Зададим сетевое имя вашей машины

# echo Arch > /etc/hostname

# nano /etc/hosts

#
# /etc/hosts: static lookup table for host names
#
#<ip-address>   <hostname.domain.org>   <hostname>
127.0.0.1       localhost.localdomain   localhost   Arch
::1             localhost.localdomain   localhost   Arch
# End of file



12. ========= Устанавим необходимы пакеты для корректной работы сети после перезагрузки

# pacman -S iw wpa_supplicant dialog wpa_actiond net-tools mc bash-completion networkmanager

# systemctl enable NetworkManager.service



13. ========= Зададим пароль root

# passwd


14. ========= Создадим и добавим его в нужные группы
(работать под рутом небезопасно)

useradd -m username
passwd username
sudo usermod -G audio,games,lp,optical,power,scanner,storage,video,wheel username

15. ========= Делегируем новому пользователю подномочия администратора (sudo)

# nano /etc/sudoers

находим строку
root ALL=(ALL) ALL

ниже добавляем
username ALL=(ALL) ALL  (username - имя вашего пользователя)


16. ========= Установка UEFI загрузчика (systemd-boot)

# bootctl --path=/boot install

редактируем
# nano /boot/loader/loader.conf

default  arch
timeout  4

Создаем конфигурационный файл загрузки

# nano /boot/loader/entries/arch.conf

title          Arch Linux
linux          /vmlinuz-linux
initrd         /initramfs-linux.img
options        root=/dev/sda2 rw



16. ========= Выходим из корневого каталога и перезагружаемся

# exit
# reboot
