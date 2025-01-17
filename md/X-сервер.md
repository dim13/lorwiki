## Как настроить переключение раскладок средствами Xorg?

Для примера возьмем переключение по Ctrl+Shift двух языков (английский,
русский) с включением индикатора Scroll Lock на русской раскладке.
Открываем текстовым редактором /etc/X11/xorg.conf и ищем секцию
InputDevice (Если ее нет, то нужно создать. Последние версии Xorg по
умолчанию работают с практически пустым конфигом):

    Section "InputDevice"
    Identifier  "Keyboard0"
    Driver      "kbd"
    Option      "XkbModel" "pc105"
    Option      "XkbLayout" "us,ru"
    Option      "XkbOptions" "grp:ctrl_shift_toggle,grp_led:scroll"
    EndSection

Пример с переключением трех языков (английский, русский, украинский):

    Section "InputDevice"
    Identifier  "Keyboard0"
    Driver      "kbd"
    Option      "XkbModel" "pc105"
    Option      "XkbLayout" "us,ru,ua"
    Option      "XkbOptions" "grp:ctrl_shift_toggle,grp_led:scroll"
    EndSection

Для старых версий Xorg (где xkeyboard-config \< 1.2) XkbLayout нужно
указывать в виде "us,ru(winkeys)".

Теперь непосредственно рассмотрим переключатель. Во всех примерах по
умолчанию стоит английский. Переключатель описывается в последней
строке. Для того, чтобы переключаться по Ctrl+Shift указывается
параметр ctrl\_shift\_toggle, по Alt+Shift - alt\_shift\_toggle.
Значение grp\_led:scroll говорит о том, что после переключения будет
загораться индикатор Scroll Lock.

Остальные варианты значений XkbOptions можно посмотреть
[тут](http://ru.gentoo-wiki.com/wiki/XkbOptions).

## А как настроить переключение раскладок если нет доступа к конфигу или нужно сменить настройки переключения на лету?

    user@linux$ setxkbmap -layout 'us,ru' -option 'grp:ctrl_shift_toggle,grp_led:scroll'

## Редактирование xorg.conf не помогло. Что делать?

Начиная с Xorg 7.4 по умолчанию конфигурация устройств ввода
осуществляется не через файл xorg.conf, а через HAL. Это
позволяет подключать различные клавиатуры и мышки "на лету" и они
будут работать без необходимости перезапуска иксов. Но при этом,
например для настройки раскладки, возникает необходимость правки
xml конфигов HAL вместо простого и удобного файла xorg.conf.

Есть три пути решения проблемы:

1\. Настройка через файл политики HAL.

Копируем fdi-файл, содержащий конфигурацию клавиатуры:

    cp /usr/share/hal/fdi/policy/10osvendor/10-keymap.fdi /etc/hal/fdi/policy

Дальнейшие настройки необходимо вносить именно в скопированный файл.

В зависимости от того, какой драйвер обслуживает вашу клавиатуру,
используются разные конфигурационные данные.

Для того, чтобы определить какой драйвер используется в вашей системе,
необходимо выполнить следующую команду:

    hal-device | grep -iA 10 input.keyboard

И поискать в её выводе упоминание о драйвере.

Если используется драйвер kbd:

    <?xml version="1.0" encoding="ISO-8859-1"?> 
    <deviceinfo version="0.2">
        <match key="info.capabilities" contains="input.keyboard">
          <merge key="input.xkb.rules" type="string">base</merge>
          <merge key="input.xkb.layout" type="string">us,ru</merge>
          <merge key="input.xkb.options" type="string">grp:ctrl_shift_toggle,grp_led:scroll</merge>
          <merge key="input.xkb.variant" type="string">,winkeys</merge> 
        </match>
      </device>
    </deviceinfo>

Если используется драйвер evdev:

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <deviceinfo version="0.2">
      <device>
        <match key="info.capabilities" contains="input.keyboard">
          <merge key="input.x11_driver" type="string">evdev</merge>
          <merge key="input.x11_options.XkbRules" type="string">evdev</merge>
          <merge key="input.x11_options.XkbLayout" type="string">us,ru</merge>
          <merge key="input.x11_options.XkbOptions" type="strlist">grp:ctrl_shift_toggle,grp_led:scroll</merge>
          <merge key="input.x11_options.XkbVariant" type="strlist">,winkeys</merge>
        </match>
      </device>
    </deviceinfo>

После внесения изменений необходимо перезапустить HAL

    /etc/init.d/hal restart (для debian-based дистрибутивов)

В некоторых случаях придётся перезапустить Xorg.

2\. Если первый способ чем то не устраивает, то можно просто добавить
следующие строчки в xorg.conf:

    Section "ServerFlags"
    Option "AutoAddDevices" "False"
    EndSection

и настраивать переключение привычным способом.

3\. В debian-based дистрибутивах можно отредактировать файл
/etc/default/console-setup записав в переменные XKBMODEL, XKBLAYOUT,
XKBVARIANT и XKBOPTIONS подходящие значения:

    XKBMODEL="pc105"
    XKBLAYOUT="us,ru"
    XKBVARIANT=","
    XKBOPTIONS="grp:ctrl_shift_toggle,grp_led:scroll"

## Как сделать одну кириллическую раскладку вместо двух-трех?

Поскольку раскладки трех языков (русского, украинского, беларусского)
отличаются на небольшое число символов, то идея объединить все
раскладки в одну появляется сама собой. Мне известно два пути
решения: через level3 и через compose.

### Первый вариант. level3

level3 позволяет набирать альтернативные символы с помощью удержания
level3 модификатора (к примеру, кнопки Win).

1\. Создаем на основе, например, русской раскладки свою:

    root@linux# cd /usr/share/X11/xkb/symbols
    root@linux# cp ru cyr

2\. Редактируем в любимом текстовом редакторе. Модифицируем секцию
xkb\_symbols "basic":

    key <TLDE> {        [        apostrophe,        asciitilde,               Cyrillic_io,               Cyrillic_IO    ]       };
        
    key <AD09> {        [    Cyrillic_shcha,    Cyrillic_SHCHA,       Byelorussian_shortu,       Byelorussian_SHORTU    ]       };
        
    key <AD12> {        [ Cyrillic_hardsign, Cyrillic_HARDSIGN,              Ukrainian_yi,              Ukrainian_YI    ]       };
    key <BKSL> {        [         backslash,               bar, Ukrainian_ghe_with_upturn, Ukrainian_GHE_WITH_UPTURN    ]       };
    
    key <AC02> {        [     Cyrillic_yeru,     Cyrillic_YERU,               Ukrainian_i,               Ukrainian_I    ]       };
        
    key <AC11> {        [        Cyrillic_e,        Cyrillic_E,              Ukrainian_ie,              Ukrainian_IE    ]       };

В конце секции вставляем:

    include "level3(lwin_switch)"

Т.е. символы будут получаться при зажатой левой кнопке Win. В остальных
секциях меняем первую команду с `include "ru(`<что-то>`)"` на `include
"cyr(`<что-то>`)"`.

3\. Добавляем нашу раскладку в список раскладок
(/usr/share/X11/xkb/symbols.dir):

    -dp----- a------- ru(basic)
    --p----- a------- ru(winkeys)
    --p----- a------- ru(typewriter)

Строчек может быть больше, в зависимости от того, какие модификации были
оставлены в файле раскладок.

4\. Включаем, тестируем:

    setxkbmap -layout 'us,cyr'

Если все работает, добавляем в конфиг и наслаждаемся.

### Второй вариант. compose

1\. Создаем файл \~/.XCompose с таким содержимым:

    include "/usr/share/X11/locale/en_US.UTF-8"
    
    <Multi_key> <Cyrillic_yeru> <Cyrillic_yeru> : "і" U0456
    <Multi_key> <Cyrillic_YERU> <Cyrillic_YERU> : "І" U0406
    
    <Multi_key> <Cyrillic_hardsign> <Cyrillic_hardsign> : "ї" U0457
    <Multi_key> <Cyrillic_HARDSIGN> <Cyrillic_HARDSIGN> : "Ї" U0407
    
    <Multi_key> <Cyrillic_e> <Cyrillic_e> : "є" U0454
    <Multi_key> <Cyrillic_E> <Cyrillic_E> : "Є" U0404
    
    <Multi_key> <backslash> <backslash> : "ґ" U0491
    <Multi_key> <bar> <bar> : "Ґ" U0490
    
    <Multu_key> <Cyrillic_shcha> <Cyrillic_shcha> : "ў" 045E
    <Multu_key> <Cyrillic_SHCHA> <Cyrillic_SHCHA> : "Ў" 040E

Таким образом, для того, чтобы набрать нужный символ, нужно нажать
сначала compose(обычно menu-кнопка), а потом дважды на кнопку
требуемого символа.

2\. Перегружаем X-сервер, наслаждаемся.

## Как настроить нормальную частоту и разрешение? Как рассчитать modeline?

Вообще в век ЖК мониторов и ноутбуков проблема с разрешениями и частотой
развертки экрана должна уже исчезнуть. Но на всякий случай на ней
следует остановиться поподробнее.

Для начала можно попробовать добавить в xorg.conf нечто такое:

    Section "Monitor"
    Identifier  "Monitor0"
    HorizSync   31.5 - 79.0
    VertRefresh 50-90
    EndSection

Значения HorizSync и VertRefresh нужно взять из книжки к монитору.

Если это способ по каким-либо причинам не устраивает, можно вычислить
нужную modeline и прописать ее. Узнать нужную modeline можно с
помощью стандартной утилиты gtf или онлайн
[калькулятора](http://www.arachnoid.com/modelines/). В любом
случае результат должен быть таким:

    Modeline "1024x768_100.00"  113.31  1024 1096 1208 1392  768 769 772 814  -HSync +Vsync

Его и вписываем в в xorg.conf в раздел Monitor, чтобы получилось
примерно так:

    Section "Monitor"
    Identifier   "Monitor0"
    HorizSync    31.5 - 79.0
    VertRefresh  50-90
    Modeline     "1024x768"  113.31  1024 1096 1208 1392  768 769 772 814  -HSync +Vsync
    Option       "dpms"
    EndSection
    
    Section "Screen"
    Identifier "Screen0"
    Device "Card0"
    Monitor "Monitor0"
    SubSection "Display"
         Viewport 0 0
         Depth 32
         Modes "1024x768" "800x600"
    EndSubSection
    EndSection

После перезапуска X-сервера можно провести тонкую настройку (чтобы края
не вылезали и т.п.) программой xvidtune - открываем терминал, запускаем
в нем xvidtune, когда нам все понравится, жмем apply и получаем в
консоли исправленную строчку для modeline. Ее записываем вместо
первоначальной.

## Что делать, если частоты прописал, а частота обновления экрана по прежнему 60 герц?

Х-сервер перезапустили? Тогда читаем [вот этот
вопрос](http://www.linux.org.ru/wiki/en/Games#Многие_игры_выводят_изображение_с_частотой_60Гц._Как_это_исправить%3F),
точнее его конец об именовании моделайнов.

## Почему не выставляется 32-битная палитра?

Зато есть 24-битная, по восемь бит на каждый канал из RGB, а больше всё
равно ваш монитор не умеет. И четвёртый канал для значения прозрачности
не поможет вашему дисплею отобразить кусок стенки за ним. 32-битная
адресация включается на уровне драйверов автоматически.

## Где взять драйвер для монитора?

Нигде. Если вы не в курсе, в "драйверах для монитора" для Windows обычно
пишутся его рабочие частоты (которые современные мониторы и так отдают
операционной системе с помощью EDID) иногда рабочую температуру цвета,
цветовые профили.

Кстати, если вы думаете, что у вас нельзя поставить 100Гц вместо 85Гц
из-за того, что у вас отсутствуют данные драйверы, то вы ошибаетесь.
Виноват драйвер видеокарты (например, такое наблюдается на картах S3).

## Как правильно настроить шрифты?

/\* FIXME: Написать про cleartype патчи и тд \*/

## Как остановить или перезапустить Х-сервер?

Убить иксы можно комбинацией Ctrl+Alt+Backspace.

-----

Информация немного устарела, Ctrl+Alt+Backspace работает сейчас не везде
(практически во всех дистрах отключено), нужно настраивать отдельно.
Если не работает с дефолтными настройками, советуют сделать
следующее:

  - Поискать соответствующий пункт в настройках DE (помню, в Ubuntu
    10.10 в настройках клавиатуры была "Комбинация клавиш для прерывания
    работы X-сервера", должно быть актуально для любого Gnome)
  - Выполнить `setxkbmap -option terminate:ctrl_alt_bksp` или прописать
    этот `terminate:ctrl_alt_bksp` где-то в иксовых конфигах
  - (не проверял) Дописать следующее в xorg.conf

<code>

`Section “ServerFlags”`  
`   Option “DontZap” “false”`  
`EndSection`</code>

  - (не проверял) Установить пакет dontzap и выполнить `dontzap
    --disable` или `dontzap -d`
  - Использовать альтернативное сочетание клавиш Alt+SysRq+K

-----

Если же используется Display Manager (например KDM или GDM), то
достаточно остановить соответствующий демон - `/etc/init.d/kdm
stop` в случае с KDM или `/etc/init.d/gdm stop` в случае с GDM. В
некоторых дистрибутивах расположение init-скриптов может
отличаться (например /etc/init.d/xdm в Gentoo и производных).

Альтернативный способ это редактирование /etc/inittab. Чтобы при
загрузке система не загружалась в графический режим, нужно
выбрать другой сценарий загрузки (runlevel). Графический режим -
это 5 runlevel (в Slackware - 4), а текстовый - 3 (в Debian - 2):

    id:N:initdefault:

где N - номер режима загрузки и меняем на 3 или 2, в зависимости от
дистрибутива. В случае с Fedora или RedHat-подобным дистрибутиво не
забудьте, что после этого будут грузиться сервисы, указанные в
/etc/rc3.d, а не /etc/rc5.d

## Как предотвратить отключение монитора по истечению определенного времени?

Добавить в секцию "Monitor" файла /etc/X11/xorg.conf опцию:

    Section "Monitor"
      ...
      Option "DPMS" "false"
    EndSection

Либо, можно выставить время задержки перед выключением в секции
"ServerFlags":

    Section "ServerFlags"
      ...
      Option "BlankTime"      "0"
      Option "StandbyTime"    "0"
      Option "SuspendTime"    "0"
      Option "OffTime"        "0"
    EndSection

Для текущей сессии можно воспользоваться командой xset:

    user@linux$ xset -dpms

## Как принудительно отключить монитор?

    user@linux$ xset dpms force off

## Как запустить второй X-сервер?

    user@linux$ startx -- :N

где N - номер сервера. Нумерация начинается с нуля и если один сервер
уже запущен, то он, скорее всего, имеет нулевой номер.

## Как запустить "иксовую" программу удаленно с другого компьютера?

### Linux

В случае с Linux все просто. На обоих компьютерах ставим openssh. На
удалённом компьютере в /etc/ssh/sshd\_config должно быть:

    X11Forwarding yes

На своём в /etc/ssh/ssh\_config:

    ForwardX11 yes

После внесения изменений перезапускаем на удалённом компьютере openssh и
пробуем к нему подключиться и запустить какое нибудь приложение:

    user@linux$ ssh -X user@host
    user@host's password: 
    user@host$ xterm &
    [1] 11146
    user@host$

Также могут быть полезны опции ssh -C (включить компрессию траффика) и
-Y (если Вы получаете ошибки xauth).

### Windows

Для начала придется установить Х-сервер на Windows, я (JB) предпочитаю
[Xming](http://xming.sourceforge.net/) и опишу все действия с его
использованием. Если кто то использует иксы из состава Cygwin, то
пусть дополнит. В качестве ssh клиента подойдет
[PuTTy](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

1\. Запускаем XLaunch. В первом шаге выбираем Multiple windows, во
втором Start no client, на третьем указываем параметры запуска
`-dpi 96 -xkblayout us,ru -xkbvariant basic,winkeys -xkboptions
grp:ctrl_shift_toggle`. Сохраняем настройки кнопкой "Save configuration"
и запускаем сервер кнопкой "Готово".

2\. Запускаем PuTTy. Открываем Connection -\> SSH -\> X11 и ставим
галочку напротив "Enable X11 forwarding". Возвращаемся в Session,
вводим адрес удалённого компьютера, порт, выбираем протокол SSH, даем
название сессии и не забываем нажать кнопку "Save", это позволит
сохранить настройки и не вводить их каждый раз заново.

3\. Подключаемся, вводим логин и пароль и запускаем то, что хотели.

## Как запустить X-сервер с другой машины по сети? (XDMCP)

  - [1](http://www.opennet.ru/base/X/xdmcp_windows.txt.html)

<!-- end list -->

  - [2](http://wiki.linuxformat.ru/index.php/LXF83:XDMCP)

<!-- end list -->

  - [3](http://linux.mkrovlya.ru/book/xdmcp-клиенты)

/\* FIXME: Все это нужно описать, не ссылаясь на другие ресурсы. \*/

## Можно ли запустить внутри иксов еще одни иксы?

Можно -- воспользуйтесь Xnest или Xephyr и переопределите для новых
иксов переменную DISPLAY - `DISPLAY=":1"`. Для чего это нужно?
Например, для запуска игр, не работающих в оконном режиме.

## Как подключить к компьютеру несколько комплектов клавиатура+мышь+монитор?

[Настройка мультимониторной
конфигурации](http://www.klv.lg.ua/~vadim/multihead.html).
Автор Вадим Лихота. Оно уже устарело, но при желании можно
адаптировать под современные дистрибутивы. /\* FIXME: Не
знаю, есть ли другие решения. Вроде бы в Xorg 7.5 обещают что то
похожее. \*/

[На базе Kubuntu Linux 10.04 LTS](http://habrahabr.ru/post/112534/).

## Где хранятся настройки стандартных X-овых программ?

В /etc/X11/app-defaults

Для того, чтобы настроить их под конкретного пользователя нужно в файле
\~/.Xdefaults или \~/.Xresources прописать свои параметры.

## Alt в xterm не работает. Как исправить?

Добавить в \~/.Xresources:

    XTerm*eightBitInput: false
    XTerm*metaSendsEscape: true 

## У меня под root'ом 3D-ускорение работает, а под обычным пользователем - нет.

Нужно добавить в /etc/X11/xorg.conf такие строчки:

    Section "DRI"
            Mode 0666
    EndSection

Если надо ограничить доступ только одной группе (users в данном случае):

    Section "DRI"
            Group        video
            Mode         0660
    EndSection

Примечание - на многих современных дистрибутивах используется группа
video, естественно пользователь должен принадлежать к этой группе.

## glxgears почему то выдаёт только 60 fps. В чем дело?

Необходимо отключить вертикальную синхронизацию. Можно воспользоваться
утилитой driconf или самостоятельно добавить в \~/.drirc следующее
(пример для драйвера radeon):

    <driconf>
        <device screen="0" driver="radeon">
            <application name="Default">
              <option name="vblank_mode" value="0" />
              </application>
        </device>
    </driconf>

Для новых ядер (\>= 2.6.35) необходимо использовать параметры драйвера
dri2 (работает для radeon, intel, nouveau):

    <driconf>
        <device screen="0" driver="dri2">
            <application name="Default">
                <option name="vblank_mode" value="0" />
            </application>
        </device>
    </driconf>

Также можно добавить параметров по вкусу к своему драйверу, например,
для intel i915:

    <driconf>
          <device screen="0" driver="dri2">
              <application name="Default">
                  <option name="vblank_mode" value="0" />
              </application>
          </device>
          <device screen="0" driver="i915">
              <application name="Default">
                  <option name="force_s3tc_enable" value="false" />
                  <option name="no_rast" value="false" />
                  <option name="always_flush_cache" value="false" />
                  <option name="stub_occlusion_query" value="false" />
                  <option name="always_flush_batch" value="false" />
                  <option name="bo_reuse" value="1" />
                  <option name="texture_tiling" value="true" />
                  <option name="early_z" value="false" />
                  <option name="allow_large_textures" value="2" />
                  <option name="fragment_shader" value="false" />
              </application>
          </device>
    </driconf>

Это сработает только при использовании открытых драйверов. Проприетарные
nvidia и fglrx используют собственную реализацию DRI.

Владельцам видеокарточек Intel на старых ядрах (\< 2.6.35) стоит
попробовать такой вариант:

    <driconf>
         <device screen="0" driver="i915">
             <application name="Default">
                 <option name="force_s3tc_enable" value="false" />
                 <option name="no_rast" value="false" />
                 <option name="fthrottle_mode" value="2" />
                 <option name="always_flush_cache" value="false" />
                 <option name="always_flush_batch" value="false" />
                 <option name="bo_reuse" value="1" />
                 <option name="vblank_mode" value="0" />
                 <option name="allow_large_textures" value="2" /> 
             </application>
         </device>
     </driconf>

Это немного ускорит рендер. У меня прирост в районе 80%. Наслаждайтесь\!

## Как настроить двухколесную мышь? Как настроить многокнопочную мышь?

ищем мышь

    ls -l /dev/input/by-id/*event*

получим что-то похожее на

    lrwxrwxrwx 1 root 0 9 Апр 27 10:02 usb-A4Tech_USB_Mouse-event-mouse -> ../event3

в /etc/X11/xorg.conf находим что-то похожее на

    Section "InputDevice"
        Identifier     "Mouse0"
        Driver         "mouse"
        Option         "Protocol"
        Option         "Device" "/dev/input/mice"
        Option         "Emulate3Buttons" "no"
        Option         "ZAxisMapping" "4 5"
    EndSection

и изменяем строку

``` 
    Option         "Device" "/dev/input/mice"
```

на

``` 
    Option         "Device" "/dev/input/event3"
```

-----

перезапускаем иксы или перезагружаемся

-----

ищем номера кнопок

    xev | grep button

в появившемся окне нажимаем кнопки и получаем их номера

``` 
    state 0x10, button 1, same_screen YES
    state 0x2110, button 1, same_screen YES
    state 0x10, button 1, same_screen YES
    state 0x2110, button 1, same_screen YES
    state 0x10, button 12, same_screen YES
    state 0x2010, button 12, same_screen YES
    state 0x10, button 10, same_screen YES
    state 0x2010, button 10, same_screen YES
и т.д.
```

(если на одно нажатие номер показывается 4/6 раз - значит кнопка
эмулирует, соответственно, 2-х/3-х кратное нажатие левой, в
данном случае, кнопки)

-----

ставим xbindkeys и создаём файл настроек

    xbindkeys -d > ~/.xbindkeysrc

и пишем в него желаемые сочетания в виде

``` 
"команда"
b:x   
```

где x=номер кнопки например

    "qmmp -t "
      b:10

или сочетание кнопок

    "qmmp --volume-dec"
      control + b:5

для эмуляции нажатия комбинации кнопок клавиатуры ставим xvkbd и в
конфиге пишем например

    "/usr/bin/xvkbd -text "\[Control_L]\[r]""  
         b:12

теперь запускаем xbindkeys и проверяем

-----

после окончания настройки прописываем в автозапуск например скриптом
вида

    #!/bin/bash
    xbindkeys

делаем его исполняемым и кладём в автозапуск (в KDE4 это
\~/.kde4/Autostart)

так же для xbindkeys есть GUI - xbindkeys\_config

## Как настроить multitouch на тачпаде?

Прочитайте
[эту](http://www.ibm.com/developerworks/opensource/library/os-touchpad/)
статью.

## Как регулировать скорость мыши без KDE/GNOME/XFCE?

**1. Простое ускорение указателя мыши.**

    xset m Х

где Х - желаемая скорость (обычно 4-6).

**2. Динамическая акселерация скорости указателя мыши**

Она настраивается с помощью указания двух чисел. Первое число ускорение,
а второе число это порог. Особенностью динамической акселерации состоит
в том что ускорение мыши появляется только после превышении мышью
определенного порога. Попробуйте например так.

    xset m 3 10

Пока вы будете двигать мышью медленно ускорения нету. И это позволяет
удобно навести мышь на какую-нибудь мелкую кнопку. Но если вы начнете
быстро двигать мышью, то сразу появится заданное ускорение. Так можно
быстро перевести мышь на другую часть экрана.

**3. Динамическая акселерация скорости указателя мыши по экспоненте.**

В этом случае скорость меняется гибко в зависимости от скорости движения
мышью. То есть скорость ускорения в некой степени пропорциональна
скорости движения мышью. В качестве второго значения укажите
ноль.

    xset m 3/2 0

Рекомендуемые значения для первого числа от 3/2 до 2.

Этот вариант весьма похож на тот что используется в ОС Windows XP.

## Как отключить Composite?

Нужно добавить в xorg.conf:

    Section "Extensions"
        Option      "Composite" "false"
    EndSection

## Как установить драйвер Nvidia? Где скачать драйвер?

В первую очередь почитать как **правильно** устанавливать драйвер в
своем дистрибутиве. Если же ничего не нашли, то можно
воспользоваться ручной установкой (имейте ввиду что
официальный драйвер состоит из ядерной компоненты и X11
компоненты, если Вы обновили ядро, то без компиляции модуля
ядра (nvidia.ko) драйвер перестанет работать).

1\. Скачиваем драйвер с сайта,
[4](http://www.nvidia.com/object/linux.html)

ВНИМАНИЕ\! Если ваша карта младше чем GeForce 6 ( 6xxx ) , то текущая
серия драйверов вам не подойдет, посмотрите
[ниже](http://www.linux.org.ru/wiki/en/X-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80#%D0%9A%D0%B0%D0%BA_%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B8%D1%82%D1%8C_%D0%BA%D0%B0%D0%BA%D0%B0%D1%8F_%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D1%8F_%D0%B4%D1%80%D0%B0%D0%B9%D0%B2%D0%B5%D1%80%D0%BE%D0%B2_%D1%82%D1%80%D0%B5%D0%B1%D1%83%D0%B5%D1%82%D1%81%D1%8F_%D0%B4%D0%BB%D1%8F_%D0%BC%D0%BE%D0%B5%D0%B9_%D0%BA%D0%B0%D1%80%D1%82%D1%8B_GeForce_%3F)
по поводу того , какую версию драйверов вам нужно выбрать.

2\. Устанавливаем все необходимое для сборки. Как минимум это компилятор
gcc и заголовочные файлы ядра (пакет kernel-headers, в разных
дистрибутивах может называться слегка иначе)

3\. Запускаем инсталлятор, на вопрос о скачивании готовых модулей
отвечаем отрицательно, после чего начнется сборка модуля. Если
компиляция оканчивается ошибкой, то открываем
/var/log/nvidia-installer.log и смотрим в чем дело.

4\. Проверяем загружается ли модуль - modprobe nvidia. В dmesg должно
быть что то вроде такого:

    [    6.952637] nvidia: module license 'NVIDIA' taints kernel.
    [    7.203821] NVRM: loading NVIDIA UNIX x86_64 Kernel Module  180.44  Tue Mar 24 05:46:32 PST 2009

5\. Открываем /etc/X11/xorg.conf и меняем Driver "nv" на Driver
"nvidia". Если вам не нравится выскакивающее лого нвидии, то добавляем
туда же параметр **Option "NoLogo" "True"**.

6\. Запускаем иксы и проверяем наличие ускорения:

    glxinfo |grep direct
    direct rendering: Yes

## Как определить какая версия драйверов требуется для моей карты GeForce ?

Проприетарные драйвера Nvidia делятся на несколько веток:

1\) Текущая ветка, в которую добавляются все новые возможности ,
исправления и поддержка для новых карт: поддерживает все новые
карты начиная с GeForce 6 (6xxx), нумерация версий 177.xx.xx -
195.xx.xx , на июль 2010 - 256.xx.xx

2\) Старые , неразрабатываемые, но обновляемые для поддержки новых ядер
и xorg-server:

GeForce 5xxx : версии 173.xx.xx

GeForce 2 MX и GeForce 4: версии 96.xx.xx

Поддержка карт GeForce 3, GeForce 256, TNT / TNT2, Riva 128, Vanta и
Quadro 2 Pro (драйвера версии 71.xx.xx) была прекращена в июне 2010, тем
кто использует новые версии ядра и/или xorg-server рекомендуется
воспользоваться открытыми драйверами nouveau.

Посмотреть последние версии драйверов и к каким GPU они подходят можно
[по этой
ссылке](http://www.nvnews.net/vbulletin/showthread.php?t=122606)

## Чем отличаются пакеты драйверов Nvidia -pkg0.run -pkg1.run и -pkg2.run ?

Для 32-битных архитектур имеются 2 варианта пакетирования драйверов pkg0
и pkg1, pkg0 содержит исходные коды для модуля ядра (вам потребуется
самим собрать модуль nvidia.ko или использовать включеный в ваш
дистрибутив механизм
[DKMS](http://ru.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) ,
драйвер для Xorg, библиотеки
[OpenGL](http://ru.wikipedia.org/wiki/OpenGL),
[vdpau](http://en.wikipedia.org/wiki/VDPAU),
[CUDA](http://ru.wikipedia.org/wiki/CUDA) /
[OpenCL](http://ru.wikipedia.org/wiki/OpenCL), pkg1 еще дополнительно
содержит набор заранее собраных модулей для ядер наиболее популярных
дистрибутивов, тем не менее этот набор часто оказывается устаревшим и
ядерный модуль все равно приходится собирать самостоятельно, так
скорее всего pkg1 не стоит потраченных на него 8-10 мегабайт
лишнего траффика.

Для 64 битных архитектур предоставляется дополнительный пакет pkg2,
содержащий 32 битные библиотеки совместимости, вам потребуется этот
пакет, если вы собираетесь использовать 32-битные OpenGL и 3D приложения
в 64-битной системе ( сюда например относятся и игры под WINE ).

Начиная с серии 256.xx драйвер больше не пакетируется на pkg0, pkg1,
pkg2 , для 32 битных архитектур существует 1 вариант пакета, для 64
битных - 2 варианта (полный и -no-compat32 , который не содержит
32-битных библиотек).

## Как разогнать видеокарту nVidia?

Добавляем в xorg.conf в Section "Device" параметр **Option "CoolBits"
"1"**, после чего в nvidia-settings появится возможность оверклокинга.
Еще для этого существует утилита
[NVClock](http://www.linuxhardware.org/nvclock/).

## Как удалить драйвер установленый посредством NVIDIA\*.run ?

Если вы решили использовать драйвера поставляемые с вашим дистрибутивом
или установить открытый драйвер, то вам потребуется удалить драйвер
установленый инсталлером NVidia, для этого нужно запустить
инсталятор \*.run (которым вы устанавливали драйвер) с
параметром --uninstall от root:

    % sh NVIDIA-Linux-x86-XYZ.AB.pkg1.run --uninstall

## Как установить драйвера ATI?

Проприетарные драйвера ATI - fglrx поддерживают новейшие графические
чипсеты а также имеют расширенные в сравнении со свободными
драйверами возможности. Процесс подготовки к установке и сама
установка зависит от используемого дистрибутива. В пакетных
дистрибутивах установка обычно происходит по стандартной
схеме: подключение соответствующего репозитария с драйверами,
обновления, выбора пакетов драйвера. Иначе можно воспользоваться
стандартным пакетом-инсталлятором от разработчиков ATI,
предназначенного для широкого диапазона дистрибутивов.
Пакет скачиваются с официального сайта [5](http://ati.amd.com) и
запускается с указанием необходимого дистрибутива.

Пример для Debian Lenny

Устанавливаем средства сборки

    apt-get install build-essential linux-headers-$(uname -r) module-assistant

начинаем сборку

    sh ati-driver-installer-9-8-x86.x86_64.run --buildpkg Debian/lenny

..и далее по инструкции

Для Debian также можно воспользоваться скриптом **sgfxi** ―
универсальным установщиком драйверов nvidia, ati/amd fglrx
и свободных драйверов xorg [6](http://code.google.com/p/sgfxi/)

Примечание: AMD/ATI прекратили поддержку карт с GPU R3xx-R5xx в своих
драйверах после Catalyst 9.3 , если у Вас такая карта, то можно
использовать старый Catalyst с ядрами включительно до 2.6.28
(патчи для более новых ядер существуют, но из за закрытости
драйвера могут работать не совсем стабильно) или использовать
открытые драйвера.

## Как включить UXA и DRI2 на видеоадаптерах Intel?

Для поддержки этого счастья нам потребуется следующее: xorg-server \>=
1.6, Mesa \>= 7.4, xf86-video-intel \>= 2.6 и ядро начиная с 2.6.28
включительно.

Попробуем написать следующее в наш конфиг (/etc/X11/xorg.conf, не
забываем о бэкапах\!):

    Section "Module"
        Load      "glx"  //модуль Opengl
        Load      "dri2" //модуль интерфейса с Direct Rendering Manager'ом
    EndSection

и

    Section "Device"
        Identifier      "Card0"
        Option          "AccelMethod"    "UXA"   //указываем какой тип ускорения использовать
        Option          "Tiling"         "False" //иногда помогает избавиться от мелких артефактов
        Option          "XvMC"           "True"  //обработка видео средствами видеоадаптера
        Driver          "intel"
    EndSection

Если связка UXA+DRI2 даёт низкую производительность, либо артефакты, то
можно попробовать заменить dri2 на dri или UXA на EXA и попробовать это
в разных комбинациях. В некоторых случаях это помогает.

## Как включить перезагрузку X-сервера по Control+Alt+Backspace

В новых версиях X.org была заблокирована возможность перезагружать иксы
комбинацией ctrl+alt+backspace. Чтобы снова включить эту возможность,
добавьте следующую секцию в /etc/X11/xorg.conf:

    Section "ServerFlags"
      Option "DontZap"  "off"
    EndSection

Есть одна тонкость — если добавить эту секцию в конец файла, то X-сервер
не запускается. Поэтому я вписал эту секцию вслед за секцией "Module".

Способ, кажется, устарел. По крайней мере для xserver-xorg-7.4 решение
таково:

    Section "InputDevice"
        Identifier "Default Keyboard"
        ...
        Option     "XkbOptions" "grp:ctrl_shift_toggle,compose:ralt,terminate:ctrl_alt_bksp"
        ...
    EndSection

## Как избавиться от артефактов в Blender на видеоадаптерах Intel

Для новых, глюкавых, версий драйверов подходит этот хак:

    LIBGL_ALWAYS_SOFTWARE=1 blender

Да-да, у нас появился аж GLSL. Правда... силами процессора :)

## Как включить поддержку DXT в Mesa?

Многие Windows-игры, будучи запущенными в WINE, могут требовать
поддержки сжатия текстур, некоторые ведут себя достаточно
прилично, если поддержки DXT нет (или включают собственную
поддержку), некоторые позволяют настраивать это в опциях,
некоторые же просто некорректно изображают текстуры.
Проприетарные драйверы Nvidia и ATI Catalyst содержат в
себе расширения для работы с такими текстурами, в Mesa же ввиду
патентных ограничений на реализацию сжатия текстур DXT (OpenGL
расширения GL\_EXT\_texture\_compression\_s3tc , GL\_S3\_s3tc)
поддержка DXT была вынесена в внешнюю библиотеку, которую
предлагается установить самостоятельно в случае если вы берете
на себя ответственность за возможное нарушение патентов S3 (как минимум
на территории США это будет незаконно).

Исходный код libtxc\_dxtn можно взять тут
<http://homepage.hispeed.ch/rscheidegger/dri_experimental/s3tc_index.html>
, для сборки потребуется пакет с заголовками libGL (mesa-dev).

Собираете и устанавливаете libtxc\_dxtn (от root):

*make*

*make install*

Устанавливаете утилиту **driconf** , запускаете её (от пользователя), на
вкладке Image Quality включаете опцию "Enable S3c texture compression
even if software support is not available"

Проверяете насколько успешно вы сделали установку

*glxinfo | grep -i s3*

Должны появиться вот такие расширения:

GL\_EXT\_texture\_compression\_s3tc, GL\_S3\_s3tc

Всё, можете запускать вашу игру\!

## Какую версию Mesa лучше выбрать для i915 (GMA900, GMA950)?

mesa 7.5.1 - высокая производительность, шейдеры отсутствуют, T\&L не
работает. Протестирована на kernel-2.6.31, intel-2.9.1.

mesa 7.7.1 - низкая производительность, шейдеры и T\&L работают без
артефактов. kernel-2.6.35, kernel-2.6.32, intel-2.12.0 и
intel-2.9.1.

mesa 7.8.2 - производительность высокая, быстрее чем в mesa 7.7 в 2-5
раз, шейдеры и T\&L работают без артефактов. kernel-2.6.35,
intel-2.12.0.

mesa 7.9-git20100914 - производительность высокая, сплошные артефакты.
kernel-2.6.35, intel-2.12.0.

mesa 7.9 - производительность высокая, без артефактов, по умолчанию
включен vsync. kernel-2.6.36, intel-2.13.0

gallium неработоспособен, сплошные сегфолты, артефакты, очень низкая
производительность. kernel-2.6.35, intel-2.12.

## Multiseat на одной видеокарте

1\) Для начала, создайте отдельный конфиг для двух мониторов.
Отредактируйте его примерно таким образом:

    Section "ServerLayout"
        Identifier     "Layout0"
        Screen      0  "Screen0" 3000 0
        Screen      1  "Screen1" 0 0 #RightOf "Screen0"
        InputDevice    "Keyboard0" "CoreKeyboard"
        InputDevice    "Mouse0" "CorePointer"
        Option         "Xinerama" "0"
    EndSection

2\) Запустите ещё один X server командой:

''

    user@linux$ xinit /usr/bin/xterm -- :1 vt8 -config [путь-к-конфигу] 

''

3\) Создайте второй курсор и клаву этим скриптом:
<http://pastebin.com/c7FGBxk4>

4\) Идём сюда
<http://digamma.cs.unm.edu/trac.dmohr/wiki/DualscreenMouseUtils>,
скачиваем dualscreen-mouse-utils и перекидываем один курсор
программой mouse-switchscreen

5\) Для запуска 2-х экземпляров игры можно сделать

''

    user@linux$ cp -R .wine .wine2 

''

И перед запуском игры на втором мониторе достаточно выполнить:

''

    user@linux$ export WINEPREFIX=$HOME/.wine2/ 

''

[Category:X Window System](Category:X_Window_System "wikilink")
