Hibernate
#########

:date: 2012-08-02
:tags: gentoo, hibernate
:category: gentoo
:slug: gentoo/hibernate
:lang: ru

:summary: засыпаем на закрытии крышки ноутбука

Ставим pm-utils. С помощью скриптов, идущих в комплекте данного портежа будем засыпать.

..code-block: none

        sudo emerge -av --quiet pm-utils

Управление событием "закрытие/открытие" крышки ноутбука управляется через acpi. Если быть точным:

<file name="/etc/acpi/actions/lm_lid.sh">
#! /bin/sh

test -f /usr/sbin/laptop_mode || exit 0

# lid button pressed/released event handler

/usr/sbin/laptop_mode auto

if grep -q open /proc/acpi/button/lid/LID/state
then
	ACTION="on"
else
	#ACTION="off"
	/usr/sbin/pm-suspend
fi
</file>

Нам также интересно блокировать экран после сна. Для этих целей корректируем скрипты pm-utils:

<file name="/etc/pm/sleep.d/00_lock">
#!/bin/sh
#
# 00screensaver-lock: lock workstation on hibernate or suspend

DBUS=$(ps aux | grep 'dbus-launch' | grep -v root)
if [[ ! -z $DBUS ]];then
 USER=$(echo $DBUS | awk '{print $1}')
 USERHOME=$(getent passwd $USER | cut -d: -f6)
 export XAUTHORITY="$USERHOME/.Xauthority"
 for x in /tmp/.X11-unix/*; do
   DISPLAYNUM=$(echo $x | sed s#/tmp/.X11-unix/X##)
   if [[ -f "$XAUTHORITY" ]]; then
           export DISPLAY=":$DISPLAYNUM"
   fi
 done
else
 USER=hoxnox
 USERHOME=/home/hoxnox
 export XAUTHORITY="$USERHOME/.Xauthority"
 export DISPLAY=":0"
fi

case "$1" in
   hibernate|suspend)
      su $USER -c "/usr/bin/xkb-switch -s us"
      su $USER -c "/usr/bin/xtrlock" & # or any other such as /usr/bin/xscreensaver-command -lock
      ;;
   thaw|resume)
      ;;
   *) exit $NA
      ;;
esac
</file>

В данном случае мы лочим с помощью xtrlock.

:note: Внимание на строку

        ..code-block:: none

                su $USER -c "/usr/bin/xkb-switch -s us"

        Дело в том, что после блокирования экрана с помощью xtrlock сменить язык возможности уже не
        будет (в той, конфигурации, которой пользуюсь я), поэтому нужная раскладка устанавливается
        заранее.
