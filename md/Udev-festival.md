Небольшой скрипт, который молвит человеческим голосом при подключении
какого-либо usb устройства или компакт диска. Необходим festival и,
естественно, udev.

Скрипт boltalka.sh помещаем куда-нибудь в PATH, а 62-festival.rules - в
/etc/udev/rules.d (в зависимости от дистрибутива это место может
отличаться). Можно добавлять свои фразы для других устройств,
для этого надо смотреть на вывод

    udevinfo -a -p `udevinfo -q path -n /dev/устройство`

и писать соответствующие правила в 62-festival.rules Есть хороший
[мануал](http://reactivated.net/writing_udev_rules.html) по
настройке udev.

## boltalka.sh

    #!/bin/bash
    
    export PATH=/bin:/sbin:/usr/bin:/usr/sbin
    
    FESTIVAL="festival --tts"
    
    DEVICE=$1
    UDEVINFO="udevadm info"
    
    [ -z "$DEVICE" ] && exit
    [ -z "$ACTION" ] && exit
    
    function get_device_attr ()
    {
        path=`find /sys/devices -name $1`
        echo `$UDEVINFO --attribute-walk   --path=$path | grep $2 -m1 | cut -f 2 -d '"'`
    }
    
    function get_device_name ()
    {
        device=$1
    
        case $device in
            [0-9]-[0-9])
                s=`get_device_attr $device "product"`
                [ -z "$s" ] && echo "device" || echo "$s"       
            ;;
            sr*)
                echo "optical drive"
            ;;
            [sh]d*)
                s=`get_device_attr $device "KERNEL"`
                echo "$s drive"
            ;;
            *)
                exit
            ;;
        esac
        
    }
    function say ()
    {
        echo "$1 $2" | $FESTIVAL
        exit
    }
    
    name=`get_device_name $DEVICE`
    
    if [ -n "$name" ]; then 
        case "$ACTION" in
            add)
            say "$name" "was found"
            ;;
            remove)
            say "device" "has been removed"     
            ;;
            change)
            say "$name" "was changed"
            ;;
        esac
    fi

## 62-festival.rules

    SUBSYSTEMS=="usb", RUN+="/usr/bin/boltalka.sh %k"
    SUBSYSTEMS=="block", RUN+="/usr/bin/boltalka.sh %b"
