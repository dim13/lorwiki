Здесь подразумевается использование файлов в качестве носителя.

В случае с лентами картина, по видимому, будет отличаться.

# Ключевые параметры

Расписание:

  - Schedule

И параметры, определяющие использование накопителей:

  - В настройках пула:
      - Maximum Volume Bytes
      - Maximum Volumes
      - Volume Retention
  - В настройках клиента:
      - File Retention
      - Job Retention

# Schedule

Время хранения резервных копий будет определяться расписанием и
политикой хранения, где:

  -   
    расписание будет определять минимальное время хранения (период
    хранения очевидно имеет смысл делать кратным расписанию).
    а политика - максимальное.

Вот пример расписания из конфигурации по умолчанию:

    Schedule {
      Name = "WeeklyCycle"
      Run = Full 1st sun at 23:05
      Run = Differential 2nd-5th sun at 23:05
      Run = Incremental mon-sat at 23:05
    }

Здесь описано полное копирование каждое первое воскресенье месяца и
далее дифференциальные и инкрементальные.

В [неофициальном переводе официальной
документации](http://www.opennet.ru/opennews/art.shtml?num=7385)
рекомендуется устанавливать период хранения томов в два раза больше
интервала выполнения полных резервных копий. То есть, при
использовании данного расписания хранить резервные копии
следует не менее двух месяцев, а насколько дольше - определится
политикой.

# Объём тома (volume)

Максимальный объём тома. Необязательный параметр.

При использовании файлов в качестве носителей, и правиле "одно задание -
один том", максимальный размер тома должен соответствовать самому
большому возможному заданию.

Для определения необходимого значения можно замерить, какой объем займет
полный бэкап файлсета (см. estimate в Bacula Console and Operator's
Guide, а также прогнать пробное задание, если используется сжатие).

    Maximum Volume Bytes

# Количество томов (volume)

Максимальное количество используемых в пуле томов.

Если хранится N томов, то нужно иметь в пуле как минимум на два тома
больше. Один - на текущий бэкап, чтобы не затирать самый старый из
предыдущих на случай ошибки бэкапа, и как минимум один - для свободы
маневра в случае ошибок.

    Maximum Volumes

# Периоды хранения (retention)

Определение периодов хранения.

Имеет смысл установить такие сроки хранения, чтобы "резервные тома" не
понадобились при нормальной работе; например: Volume Retention на
неделю дольше максимального срока хранения по политике, если между
бэкапами проходит примерно месяц. В таком случае тома будут нормально
ротироваться без привлечения резерва.

## Пул

Минимальное время хранения Тома перед его повторным исипользованием.

Должен быть в вдвое больше интервала полных резервных копий. Это
означает, что, если полная резервная копия выполняется один раз
в месяц, то минимальный период Volume Retention должен быть два месяца.

Этот параметр должен быть как минимум не менее определённого политикой
резервного копирования периода хранения резервных копий.

    Volume Retention

## Клиент

Определяют период хранения информации о задании, и отдельных файлах в
нём, в базе данных бакулы (каталог). Не относятся непосредственно к
хранению резервных копий.

Очевидно должны быть не менее времени хранения резервной копии. Хранить
устаревшие записи нецелесообразно, чтобы не раздувать базу данных.

    File Retention

    Job Retention

# Пример использования

Пусть нам надо выполнять резервную копию сайта site\_001 так, что бы:

1.  Неделю хранить ежедневную копию
2.  Месяц хранить еженедельную
3.  Полгода хранить ежемесячную

Пример взят из официальной документации (Bacula Main Reference Guide,
глава 27, Automated Disk Backup):

  -   
    Если мы хотим также распределить эти резервные копии на разные
    накопители (например полный на ленту а инкрементальные и
    дифференциальные на диск), то storage нужно определять не в
    описании задания (job) а в каждом пуле. Если storage указан и там
    и там, то будет использован тот что указан в задании, у него
    приоритет выше.

<!-- end list -->

    Job {
     Name = client
     Type = Backup
     Client = client-fd
     FileSet = "Site_001"
     Schedule = "WeeklyCycle"
     Storage = File
     Messages = Standard
     Pool = Default
     Full Backup Pool = Full-Pool
     Incremental Backup Pool = Inc-Pool
     Differential Backup Pool = Diff-Pool
     Write Bootstrap = "/home/bacula/working/client.bsr"
     Priority = 10
    }
    
    FileSet {
     Name = "Site_001"
     Include = { Options { signature=SHA1; compression=GZIP9 }
      File = /var/www/html/site_001
     }
    }
    
    Schedule {
     Name = "WeeklyCycle"
     Run = Level=Full 1st sun at 2:05
     Run = Level=Differential 2nd-5th sun at 2:05
     Run = Level=Incremental mon-sat at 2:05
    }
    
    Client {
     Name = client-fd
     Address = client
     FDPort = 9102
     Catalog = MyCatalog
     Password = " *** CHANGE ME ***"
     AutoPrune = yes
     # Prune expired Jobs/Files
     Job Retention = 6 months
     File Retention = 60 days
    }
    
    Pool {
     Name = Full-Pool
     Pool Type = Backup
     Recycle = yes
     # automatically recycle Volumes
     AutoPrune = yes
     # Prune expired volumes
     Volume Retention = 6 months
     Maximum Volume Jobs = 1
     Label Format = Full-
     Maximum Volumes = 9
    }
    
    Pool {
     Name = Inc-Pool
     Pool Type = Backup
     Recycle = yes
     # automatically recycle Volumes
     AutoPrune = yes
     # Prune expired volumes
     Volume Retention = 20 days
     Maximum Volume Jobs = 1
     Label Format = Inc-
     Maximum Volumes = 7
    }
    
    Pool {
     Name = Diff-Pool
     Pool Type = Backup
     Recycle = yes
     AutoPrune = yes
     Volume Retention = 40 days
     Maximum Volume Jobs = 1
     Label Format = Diff-
     Maximum Volumes = 10
    }

# Мониторинг свободного места

По материалам ru-bacula:

При использовании дисков в качестве устройств хранения, что бы не
допустить возникновения проблемы при нехватке свободного места на
дисковом пространстве, можно использовать скрипт мониторинга:

В секции Job:

``` 
 RunBeforeJob = "/path/to/diskmon.sh"
```

Скрипт:

    #!/bin/sh
    
    # (c) tim4dev
    
    # *** start setup ***
    ADMIN="admin-backup@gmail.com"
    
    HOST="BACKUP"
    
    # свободный размер диска в Гб
    ERR_LIMIT=1
    WARN_LIMIT=3
    # *** end setup ***
    
    ERR=0
    
    
    # если надо измените следующую строку для grep (какие FS _НЕ_ нужно мониторить)
    for STR in `df -k | grep -vE '^Filesystem|tmpfs|sda6|sda3|sda2|sda1' | awk '{ print $4 "___" $1 }'`
    do
      avail_str=$(echo $STR | awk -F "___" '{ print $1}' | cut -d'%' -f1  )
      avail=$(echo "scale=0; $avail_str/1048576" | bc)
      partition=$(echo $STR | awk -F "___"  '{ print $2 }' )
    
      if [ "$avail" -le "$ERR_LIMIT" ]; then
         msg="ERROR: $HOST Very low disk space \"$partition (${avail}Gb)\" on $(hostname) as on $(date)"
         echo $msg
         echo $msg | mail -s "ERROR: $HOST Very low disk space ${avail}Gb" $ADMIN
         ERR=1
      else
         if [ ${avail} -le ${WARN_LIMIT} ]; then
            msg="WARNING: $HOST Low disk space \"$partition (${avail}Gb)\" on $(hostname) as on $(date)"
            echo $msg
            echo $msg | mail -s "WARNING: $HOST Low disk space ${avail}Gb" $ADMIN
         fi
      fi
    
    done
    
    if [ "${ERR}" == "0" ]; then
       exit 0;
    else
       exit 1;
    fi

[Источник](http://tim4dev.blogspot.com/2011/08/bacula-and-disk-free-space.html)

# Источники

По материалам
[ru-bacula](https://groups.google.com/forum/#!forum/ru-bacula).

[Category:Bacula](Category:Bacula "wikilink")
