Переключатель\отображатель текущей раскладки в Awesome
######################################################

Как и у многих раскладка в моих X-ах настроена в конфигурации Xorg. В моем случае это выглядит как
файлик /etc/X11/xorg.conf.d/10-evdev.conf с содержимым:

.. code-block:: none 

        Section "InputClass" 
                Identifier "evdev keyboard catchall" 
                MatchIsKeyboard "on" 
                MatchDevicePath "/dev/input/event*" 
                Driver "evdev" 
                Option "xkb_layout" "us,ru" 
                Option "XkbOptions" "grp:ctrl_shift_toggle,grp_led:scroll"
        EndSection

Естественно grp_led:scroll при такой раскладке отображает состояние scroll, а не раскладки. Да и
привычней видеть текущую раскладку где-нибудь в notification area. Придется немножко допилить
awesome. Напишем простенький widget, 

...

С ходу не нашёл простого способа добиться от X-ов текущего состояния раскладки. Такие функции
имеются в API. Можно написать свою утилитку, а можно воспользоваться проектом xkb-switch:
http://github.com/ierton/xkb-switch.

