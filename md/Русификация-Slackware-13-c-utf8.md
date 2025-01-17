### Консоль

  - Раскладка по Ctrl+Shift

/etc/rc.d/rc.keymap

    #!/bin/sh
    # Load the keyboard map.  More maps are in /usr/share/kbd/keymaps.
    if [ -x /usr/bin/loadkeys ]; then
     /usr/bin/loadkeys /usr/share/kbd/keymaps/i386/qwerty/ruwin_ct_sh-UTF-8.map.gz
    fi

  - Шрифт, отображающий кириллицу

/etc/rc.d/rc.font

    #!/bin/sh
    setfont Cyr_a8x16.psfu.gz

  - Локаль

/etc/profile.d/lang.sh

    #!/bin/sh
     
    # en_US is the Slackware default locale:
    #export LANG=en_US
    
    # There is also support for UTF-8 locales, but be aware that
    # some programs are not yet able to handle UTF-8 and will fail to
    # run properly.  In those cases, you can set LANG=C before
    # starting them.  Still, I'd avoid UTF unless you actually need it.
    #export LANG=en_US.UTF-8
    export LANG=ru_RU.UTF-8
     
    # One side effect of the newer locales is that the sort order
    # is no longer according to ASCII values, so the sort order will
    # change in many places.  Since this isn't usually expected and
    # can break scripts, we'll stick with traditional ASCII sorting.
    # If you'd prefer the sort algorithm that goes with your $LANG
    # setting, comment this out.
    export LC_COLLATE=C
    
    # End of /etc/profile.d/lang.sh

Не забываем убедиться, что на выше приведенных файлах (rc.font,
rc.keymap, lang.sh) установлен атрибут выполнения.

Поставить же его можно следующей командой:

    chmod +x полный_путь_к_файлу

Или же, разом все файлы:

    chmod +x /etc/rc.d/rc.keymap /etc/rc.d/rc.font /etc/profile.d/lang.sh

  - Lilo

В /etc/lilo.conf нужно исправить строчку:

    append=" vt.default_utf8=0"

на:

    append=" vt.default_utf8=1"

и выполнить команду:

    lilo

### HAL, udev и X'ы

  - Раскладка через HAL (Slackware 13.0, 13.1)

<!-- end list -->

    cp /usr/share/hal/fdi/policy/10osvendor/10-keymap.fdi /etc/hal/fdi/policy/10-keymap.fdi

Правим строки файла /etc/hal/fdi/policy/10-keymap.fdi с input.xkb, а
именно options, layout, variant, задаем в них примерно следующее:

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <deviceinfo version="0.2">
      <device>
        <match key="info.capabilities" contains="input.keymap">
          <append key="info.callouts.add" type="strlist">hal-setup-keymap</append>
        </match>
    
        <match key="info.capabilities" contains="input.keys">
          <merge key="input.xkb.rules"   type="string">base</merge>
          <merge key="input.xkb.model"   type="string">evdev</merge>
          <merge key="input.xkb.layout"  type="string">us,ru</merge>
          <merge key="input.xkb.variant" type="string">,winkeys</merge>
          <merge key="input.xkb.options" type="string">terminate:ctrl_alt_bksp,grp:ctrl_shift_toggle,grp_led:scroll</merge>
        </match>
      </device>
    </deviceinfo>

  - Раскладка через udev (Slackware 13.37 и Current)

в Slackware Current используется более новая версия xorg-server в
которой наконец то избавились от поддержки HAL, поэтому
переключение раскладок снова настраивается как и раньше,
только с небольшим отличием

Правим файл /etc/X11/xorg.conf.d/90-keyboard-layout.conf, (если этого
файла нет, то создаём) так, как нам нужно. Конечный результат должен
выглядеть примерно так:

    Section "InputClass"
        Identifier "keyboard-all"
        MatchIsKeyboard "on"
        Driver "evdev"
        Option "XkbLayout" "us,ru(winkeys)"
        Option "XkbOptions" "terminate:ctrl_alt_bksp,grp:alt_shift_toggle,grp_led:scroll"
    EndSection

Перезапускаем иксы, и переключаем раскладку по Alt+Shift, с подсветкой
лампочки "Scroll Lock" на клаве.

  - Типографская раскладка

Если вы часто-часто готовите тексты, в, упаси боже, OpenOffice и он
тупит с автозаменой — то вам пригодится «типографская раскладка»,
такая как [тут](http://sapegin.ru/typolayout)(раскладка Артёма
Сапегина) или вот
[здесь](http://ilyabirman.ru/typography-layout/)(раскладка Ильи
Бирмана), например.

Тогда файл /etc/X11/xorg.conf.d/90-keyboard-layout.conf должен выглядить
следующим образом:

    Section "InputClass"
        Identifier "keyboard-all"
        MatchIsKeyboard "on"
        Driver "evdev"
        Option "XkbLayout" "us+typo,ru(winkeys):2+typo" 
        Option "XkbOptions" "lv3:ralt_switch,terminate:ctrl_alt_bksp,grp:alt_shift_toggle,grp_led:scroll"
    EndSection

Перезапускаем иксы, и радуемся, «типографские» символы набираются через
правый Alt, и правый Alt+Shift.

Раскладка немного отличается от двух приведенныйх выше, но
поэкспериметировать и выяснить что-как не составит труда в
том же OpenOffice’е. Да и во всяких жабберах можно выпендриться.

### Русские man-страницы

[1](http://www.slackware.ru/forum/viewtopic.php?f=8&t=234)

Если кратко, то в /usr/lib/man.conf заменить строку

    NROFF /usr/bin/nroff -Tlatin1 -mandoc

на

    NROFF iconv -f utf8 -t koi8r -c | /usr/bin/nroff -Tlatin1 -mandoc -c | iconv -f koi8r -t utf8 -c
