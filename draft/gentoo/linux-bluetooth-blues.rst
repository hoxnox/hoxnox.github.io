Linux bluetooth blueZ
#####################

:date: 2012-08-08
:author: Merder Kim <hoxnox@gmail.com>
:slug: gentoo/linux_bluez
:tags: linux, bluetooth, gentoo, bluez
:summary: Всем давно известно, что с bluetooth'ом всё становится лучше. Вот и мы учимся общаться с
          девайсами по bluetooth в linux.

Включение bluetooth устройства
==============================

Как обычно ядро должно поддерживать нас в наших начинаниях.

.. code-block:: none

        [*] Networking Support  --->
            <*> Bluetooth subsystem support  --->
                <*> RFCOMM protocol support
                [*]   RFCOMM TTY support
                <*> BNEP protocol support
                [*]   Multicast filter support
                [*]   Protocol filter support
                <*> HIDP protocol support
                    Bluetooth device drivers  --->
                        <*> HCI USB driver
                        <*> HCI BCM203x USB driver


Теперь необходимо включить чип. В моём случае пишем "1" в
`/sys/devices/platform/thinkpad_acpi/bluetooth_enable`:

.. code-block:: none

        echo 1 | sudo tee /sys/devices/platform/thinkpad_acpi/bluetooth_enable >/dev/null

Если имеется лампочка на устройстве она должна загореться.

bluez
=====

Всю сложную работу по реализации стека протоколов bluetooth берет на себя пакет bluez (если кому
интересно как это работает см. http://www.bluetooth.org). Необходимо поставить его в
систему (используйте свой пакетный менеджер). Кроме-того полезно поставить и почитать набор текстовых скриптов,
которые идут с пакетом (USE += test_programs в gentoo).

.. code-block:: none

        sudo emerge -av bluez

Запускаем демон `bluetoothd`. Данный демон общается с внешним миром посредством `dbus`. Все скрипты,
которые идут в пакете `bluez` лишь посылают `dbus` сообщения данному демону. Чтобы получить полный
контроль над устройством можно использовать утилиту `dbus-send` для отправки команд `bluetoothd`.
Кому интересно - см. ниже.

Настройки самого демона можно поковырять в `/etc/bluetooth/main.conf`.

.. code-block:: none

        sudo /etc/init.d/bluetooth start

Изучение окружающих bluetooth устройств
=======================================

После включения поддержки `bluetooth` устройства в ядре и установки пакета `bluez` можно
приступать к работе с девайсами. Для начала оглядимся по сторонам и найдём нужный девайс (на нём должен
быть активирован режим обнаружения) с помощью `hcitool`.

.. code-block:: none

        hoxnox@noxbook:/$ hcitool scan
        Scanning ...
                58:C3:8B:E5:65:8F       hoxnox-sgs3

А вот и мой телефон. Посмотрим что он умеет. Для этих целей разработан `Service Discovery Protocol`
(кратко SDP). Для общения в рамках данного протокола в `bluez` имеется утилита `spdtool`.

.. code-block:: none

        spdtool browse 58:C3:8B:E5:65:8F

Программа выдает список служб, поддерживаемых телефоном и некоторые их свойства. Так, например,
Samsung Galaxy S III поддерживает

- Audio Source
- AVRCP TG
- Voice Gateway
- OBEX Object Push
- OBEX Phonebook Access Server
- Voice Gateway
- Network service
- Android SMS

в свою очередь SonyErricsson W880i умеет

-
-
-
-
-

Работу с некоторыми службами мы рассмотрим далее.

Спаривание с bluetooth девайсом
-------------------------------

Для того, чтобы иметь возможность подключаться к девайсу, необходимо спариться с ним, как бы п'ошло
это ни звучало. Для этих целей нужен специальный агент. В состав пакета входит простенькая утилита
simple-agent. Запускаем с двумя параметрами - наш интерфейс и адрес девайса (для новых), а
для повторного спаривания добавляем третий параметр (значение не важно). Вводим пин-код и
подтверждаем его на девайсе.

.. code-block:: none

        sudo simple-agent hci0 58:C3:8B:E5:65:8F foo

:note: Ключики `bluez` сохраняет в /var/lib/bluetooth/hh:hh:hh:hh:hh:hh/linkkeys. Кстати в этой
       папочке можно найти еще много полезной информации.

Используем функции телефона
===========================

Предварительный этап закончен. Теперь можно поиграться с устройством. Мы уже знаем что оно умеет,
попробуем это на практике. Мы потрогаем лишь несколько служб. Ознакомится с остальными можно аналогично
(пляшем от протокола в рамках которого предоставляется служба).

Соединение с интернетом
-----------------------

Одна из самых распространенных фич bluetooth - выход в Интернет с компьютера через телефон. Данная
возможность в Samsung Galaxy S III реализуется службой Network Service:

.. code-block:: none

        Service Name: Network service
        Service Description: Network service
        Service RecHandle: 0x10007
        Service Class ID List:
          "Network Access Point" (0x1116)
        Protocol Descriptor List:
          "L2CAP" (0x0100)
            PSM: 15
          "BNEP" (0x000f)
            Version: 0x0100
            SEQ16: 800 806
        Language Base Attr List:
          code_ISO639: 0x656e
          encoding:    0x6a
          base_offset: 0x100
        Profile Descriptor List:
          "Network Access Point" (0x1116)
            Version: 0x0100

Общение с данной службой реализуется в рамках протокола BNEP (на более низком уровне L2CAP). Для
доступа в Интернет построим `PAN` сеть (`Personal Area Network`) с телефоном на базе `BNEP`. Такие
сети часто называют также `piconet`. Ставим пакет pan и натягиваем сеть:

.. code-block:: none

	sudo emerge -av pan

TODO:

Передача файлов
---------------

.. code-block:: none

      Service Name: Audio Source
      Service RecHandle: 0x10001
      Service Class ID List:
        "Audio Source" (0x110a)
      Protocol Descriptor List:
        "L2CAP" (0x0100)
          PSM: 25
        "AVDTP" (0x0019)
          uint16: 0x102
      Profile Descriptor List:
        "Advanced Audio" (0x110d)
          Version: 0x0102

dbus
====

С помощью `dbus` вы можете делать абсолютно всё, но придётся размять пальцы - синтаксис довольно
многословный. Для ускорения, можно воспользоваться Python. API неплохо документирован. В исходниках,
`доступных на официальном сайте`_, в папочке doc можно найти всё необходимое. В пакете `dbus` идет
утилита `dbus-send`, которая позволяет работать с `dbus`.

.. _`доступных на официальном сайте` : http://kernel.org/pub/linux/bluetooth

:note: DBus tutorial: http://dbus.freedesktop.org/doc/dbus-tutorial.html

Первое, что необходимо сделать - получить переменный путь к таким объектам как Adapter, Device и т.п. Это позволяет сделать метод
GetProperties объекта Manager:

.. code-block:: none

	hoxnox@noxbook:~$ dbus-send --system --print-reply --type=method_call --dest=org.bluez / org.bluez.Manager.GetProperties
	method return sender=:1.16 -> dest=:1.27 reply_serial=2
	   array [
	      dict entry(
	         string "Adapters"
	         variant             array [
	               object path "/org/bluez/6221/hci0"
	            ]
	      )
	   ]

Для того, чтобы получить список девайсов, находящихся в поле действия адаптера, необходимо
инициировать процесс поиска и обрабатывать сигналы DeviceFound объекта Adapter. Вместо обработки
сигналов (не хочу захламлять статью кодом) будем мониторить `dbus` для этого запускаем в отдельном
терминале `dbus-monitor`:

.. code-block:: none

	dbus-monitor --monitor --system

теперь в начальном терминале инициируем процесс обнаружения:

.. code-block:: none

	dbus-send --system --print-reply --type=method_call --dest=org.bluez /org/bluez/6221/hci0 org.bluez.Adapter.StartDiscovery

Монитор выплюнет состояние dbus, в котором можно увидеть следующие строки:

.. code-block:: none

	signal sender=:1.36 -> dest=(null destination) serial=115 path=/org/bluez/6221/hci0; interface=org.bluez.Adapter; member=DeviceFound
	   string "58:C3:8B:E5:65:8F"
	   array [
	      dict entry(
	         string "Address"
	         variant             string "58:C3:8B:E5:65:8F"
	      )
	      dict entry(
	         string "Class"
	         variant             uint32 5898764
	      )
	      dict entry(
	         string "Icon"
	         variant             string "phone"
	      )
	      dict entry(
	         string "RSSI"
	         variant             int16 -45
	      )
	      dict entry(
	         string "Name"
	         variant             string "hoxnox-sgs3"
	      )
	      dict entry(
	         string "Alias"
	         variant             string "hoxnox-sgs3"
	      )
	      dict entry(
	         string "LegacyPairing"
	         variant             boolean false
	      )
	      dict entry(
	         string "Paired"
	         variant             boolean false
	      )
	      dict entry(
	         string "Trusted"
	         variant             boolean false
	      )
	   ]

К сожалению процесс спаривания требует наличие агента (который примет от пользователя пин-код и
передаст в `dbus`). Агентов по-умолчанию не существует, поэтому
голым `dbus-send` тут не отделаешься. Придется спаривать как это делалось выше (с помощью
simple-agent).

После спаривания наш девайс должен быть в списке Devices объекта Adapter:

.. code-block:: none

	hoxnox@noxbook:~$ dbus-send --system --print-reply --type=method_call --dest=org.bluez /org/bluez/6221/hci0 org.bluez.Adapter.GetProperties
	...
	        string "Devices"
	        variant             array [
	              object path "/org/bluez/6221/hci0/dev_00_1C_A4_A2_47_1D"
	              object path "/org/bluez/6221/hci0/dev_F4_9F_54_7D_D5_55"
	              object path "/org/bluez/6221/hci0/dev_58_C3_8B_E5_65_8F"
	           ]
	...

Подключаемся к NAP сети девайса:

.. code-block:: none

	hoxnox@noxbook:~$ dbus-send --system --print-reply --type=method_call --dest=org.bluez /org/bluez/6221/hci0/dev_58_C3_8B_E5_65_8F org.bluez.Network.Connect string:nap
	method return sender=:1.36 -> dest=:1.84 reply_serial=2
	   string "bnep0"

TODO: сразу же выключается устройство

Пример удаления девайса:

.. code-block:: none

	hoxnox@noxbook:~$ dbus-send --system --print-reply --type=method_call --dest=org.bluez /org/bluez/6221/hci0 org.bluez.Adapter.RemoveDevice objpath:/org/bluez/6221/hci0/dev_58_C3_8B_E5_65_8F
	method return sender=:1.36 -> dest=:1.49 reply_serial=2
