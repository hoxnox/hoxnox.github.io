
HTPC на Acer Revo с gentoo на борту
###################################

:date: 2012-09-28
:author: Kim Merder <hoxnox@gmail.com>
:tags: htpc, gentoo, acer aspire revo
:category: gentoo
:slug: gentoo/htpc_revo
:lang: ru

:summary: Собираем HTPC на основе gentoo.

#. Переразбиваем диск (Partition Magic)
   [boot - 250M (ext2)][swap - (linux-swap) - 4096M]{extended [/ 50G][/home остальное]}[win - 50G]
#. Создаем загрузочную флешку как сказано тут: http://www.gentoo.org/doc/en/liveusb.xml, грузимся с
   неё

   :note: Для того, чтобы избежать зависаний при попытке смонтировать расширенный раздел указываем:

   .. code-block:: none

        boot: gentoo cdroot=/dev/sdb1

#. Не будем пока заморачиваться с WiFi, подключаем машинку к Интернет через Ethernet кабель.

  .. code-block:: none

        cp /etc/init.d/net.lo /etc/init.d/net.eth0
        /etc/init.d/net.eth0 start
        dhcpcd eth0

#. Устанавливаем дату (date MMDDhhmmYYYY)

   .. code-block:: none

       date 310813342012

#. Монтируем наши разделы

   .. code-block:: none

      mount /dev/sda5 /mnt/gentoo
      mkdir /mnt/gentoo/boot
      mkdir /mnt/gentoo/home
      mount /dev/sda2 /mnt/gentoo/boot
      mount /dev/sda6 /mnt/gentoo/homt

#. Качаем и распаковываем stage3

   .. code-block:: none

      cd /media/gentoo
      wget http://mirror.yandex.ru/gentoo-distfiles/releases/x86/current-stage3/stage3-i686-20120710.tar.bz2
      tar xvjpf stage3-i686-20120710.tar.bz2
      rm stage3-i686-20120710.tar.bz2

#. Устанавливаем портежи

   .. code-block:: none

      wget http://mirror.yandex.ru/gentoo-distfiles/snapshots/portage-latest.tar.bz2
      tar xvjf /mnt/gentoo/portage-latest.tar.bz2 -C /mnt/gentoo/usr
      rm portage-latest.tar.bz2

#. Опции компиляции (/mnt/gentoo/etc/make.conf)

   .. code-block:: none

     CFLAGS="-O9 -march=native -pipe -fomit-frame-pointer"
     CXXFLAGS="${CFLAGS}"
     MAKEOPTS="-j5"

#. Выбираем зеркало

   .. code-block:: none

      mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
      mirrorselect -i -r -o >> /mnt/gentoo/etc/portage/make.conf

#. DNS

   .. code-block:: none 

           cp -L /etc/resolv.conf /mnt/gentoo/etc/


#. chroot

   .. code-block:: none

         mount -t proc none /mnt/gentoo/proc
         mount --rbind /sys /mnt/gentoo/sys
         mount --rbind /dev /mnt/gentoo/dev
         chroot /mnt/gentoo /bin/bash
         env-update
         source /etc/profile
         export PS1="(chroot) $PS1"

#. Portage

   .. code-block:: none

      emerge --sync
      eselect profile list
      eselect profile set 2

#. USE

   доступные флаги можно посмотреть

   .. code-block:: none

      less /usr/portage/profiles/use.desc

#. Timezone

   .. code-block:: none 

      ls /usr/share/zoneinfo

#. Ядро

   .. code-block:: none 

        emerge gentoo-sources
        ls -l /usr/src/linux

#. Локализация

	Генерируем локализацию
                <code>
                echo "ru_RU.UTF-8 UTF-8" | sudo tee /etc/locale.gen >/dev/null
                locale-gen
                </code>
	Выставляем LINUAS в /etc/make.conf
		<file name="/etc/make.conf">
		LINGUAS="ru:en"
		</file>
	Выставляем LANG в профиле (можно в ~/.bash_login):
		<file name="/etc/env.d/02locale">
		LC_ALL=""
		LANG="ru_RU.UTF-8"
		LC_NUMERIC="C"
		</file>
	Генерируем новый профиль:
		<code>
		sudo env-update
		source /etc/profile
		</code>
	Ставим свободные шрифты из проекта GNOME и Terminus
		<code>
		sudo emerge -av media-fonts/dejavu
		sudo emerge -av media-fonts/terminus-font
		</code>
	Указываем настройки консоли:
		Шрифт
			<file name="/etc/conf.d/consolefont">
			# выбираем из /usr/share/consolefonts
			CONSOLEFONT="ter-k14n"
			</file>
		Метод переключения
			<file name="/etc/conf.d/keymaps">
			# выбираем из /usr/share/keymaps/i386/qwerty
			KEYMAP="ruwin_ct_sh-UTF-8.map.gz"
			</file>
	Пере запускаем службы
		<code>
		sudo /etc/init.d/consolefont restart
		sudo /etc/init.d/keymaps restart
		</code>
	Добавляем скрипты загрузки в default runlevel
		<code>
		sudo rc-update add consolefont default
		sudo rc-update add keymaps default
		<code>


