# Описание

![Image:bacula\_pools\_volumes\_lables.png](Bacula_pools_volumes_lables.png
"Image:bacula_pools_volumes_lables.png")

  - **Пул (pool)** объединяет несколько томов так, чтобы отдельная
    резервная копия не была ограничена объёмом отдельного тома.
  - **Том (volume)** представляет из себя отдельную единицу в которую
    идёт запись, и может являться:
      -   
        физической лентой
        отдельным файлом
  - Прежде, чем bacula сможет использовать том (volume), он должен иметь
    **метку (label)**.
      -   
        В случае, когда том является файлом, файл создаётся при
        нанесении метки, и метка используется в качестве
        названия.
        Метки могут наноситься автоматически. Это определяется
        параметрами:
          - "LabelMedia = yes", в секции device конфигурации sd
          - "Label Format = LableName-" в конфигурации пула
        При этом, к заданному формату будет прибавляться цифровой
        индекс: LableName-0001. В формате метки можно использовать
        макросы, например:
          - "Label Format = LableName-${Year}\_${Month}\_${Day}"
        При этом, индекс добавляться не будет. И если, например при
        проверке, будет выполнено более одного резервного
        копирования в день, то будет использован один и тот же
        том, если позволит параметр Maximum Volume Jobs.
  - Для задания (Job) мы определяем не том а пул. При этом:
      -   
        Тома этого пула выбираются автоматически, используя новые по
        мере необходимости.
        При отсутствии возможности использования тома, будет запрос на
        подключение нужного.

Из [краткого пересказа официальной
документации](http://www.opennet.ru/opennews/art.shtml?num=7385):

  -   
    *Шаги по созданию Пула (Pool), добавлению Томов к нему, и запись
    программных меток на Тома, могут казаться утомительными сначала,
    но фактически они весьма просты и они позволяют Вам использовать
    множество Томов (вместо того, чтобы быть ограниченым размером
    одной единственной ленты). Пулы также дают вам существенную
    гибкость в вашем процессе резервирования. Например, вы можете
    иметь "Daily" (ежедневный) Пул Томов для инкрементальных
    (Incremental) резервных копий и "Weekly" (еженедельный) Пул Томов
    для полных (Full) резервных копий. Определяя соответствующий Пул
    в ежедневных и еженедельных резервных Заданиях (Job), вы таким
    образом обеспечиваете, чтобы никакое ежедневное Задание когда­
    либо не записало данные в Том в Еженедельном Пуле и наоборот, и
    кроме того Bacula скажет вам, какая лента необходима и когда.*

# Пробное использование

## Конфигурационные файлы

Что бы было проще ориентироваться в конфигурационных файлах, можно
выносить часть конфигурации в отдельные файлы.

Например вся последующая конфигурация (Pool, File set, Schedule и Job)
может быть вынесена в отдельный файл /etc/bacula/conf.d/test.conf,
который можно подключить в конфигурацию директора (bacula-dir.conf)
следующим образом:

    @/etc/bacula/conf.d/test.conf

## Pool

Создаём тестовый пул. Определяем параметры тома по умолчанию в этом
пуле, например:

    # Test Pool definition
    Pool {
      Name = TestPool
      Pool Type = Backup
      Recycle = yes                       # Bacula can automatically recycle Volumes
      AutoPrune = yes                     # Prune expired volumes
      Volume Retention = 5 days
      Maximum Volume Jobs = 1             # Если носитель не является файлом то возможно имеет смысл установить значение отличное от 1
      Maximum Volume Bytes = 50M          # Limit Volume size to something reasonable
      Maximum Volumes = 10                # Limit number of Volumes in Pool
      Label Format = "Test_volume-"
    }

Из статьи на
[ibm.com](http://www.ibm.com/developerworks/ru/library/l-Backup_4/):

  -   
    *Параметр Maximum Volume Jobs рекомендуется установить в значение 1.
    Это будет означать, что в рамках одного носителя данных могут быть
    размещены резервные данные, полученные в ходе выполнения только
    одного задания. \<...\> если мы говорим о файлах, то желательно
    придерживаться правила "один файл – одна копия", т.е. в одном
    файле Bacula должны храниться резервные данные, которые были
    сформированы в рамках выполнения одного задания. Для каждого
    последующего будут создаваться новые файлы.*

## File set

    # Test Set
    FileSet {
      Name = "TestFileSet"
      Include {
        Options {
          signature = MD5
        }
        File = /home/username/tmpfiles/ # заменить на актуальное значение
      }
    }

## Schedule

    Schedule {
      Name = "TestSchedule"
      Run = Full sun at 20:05
      Run = Incremental mon-sat at 23:05
    }

## Job

Теперь, описав пул (pool), набор файлов (fileset) и расписание
(schedule), мы можем описать задание (Job) в котором их используем.

Очевидно что описывать всё нужно не всегда, можно использовать в разных
заданиях одинаковые пулы, наборы файлов и расписания. Но в нашем
учебном примере имеет смысл сделать полное описание.

    # Test Backup
    Job {
      Name = "TestBackup"
      Type = Backup
      Level = Incremental
      Client = Client-fd            # заменить на актуальное значение
      FileSet = "TestFileSet"
      Schedule = "TestSchedule"
      Storage = File
      Messages = Standard
      Pool = TestPool
      Priority = 10
      Write Bootstrap = "/var/lib/bacula/%c.bsr"
      Level = Full
    }

## Тестирование

Готовим производство нужного количество файлов нужного объёма, например:

    #!/bin/bash
    #
    
    DIR=~/tmpfiles
    FILE="`date +%Y.%m.%d-%H.%M.%N`"
    
    # BS=1024&nbsp;; COUNT=10240
    BS=1M
    COUNT=10
    
    touch $DIR/$FILE
    dd if=/dev/urandom of=$DIR/$FILE bs=$BS count=$COUNT
    find $DIR/ -type f -ctime +3 -delete

Можно выполнить всё несколько раз вручную и удостовериться что правильно
разобрались с материалом, новые тома маркируются и создаются, и всё
работает соответственно нашему ожиданию.

После чего, на некоторое время создание тестового файлового материала
можно поставить в crontab а задания бакулы в расписание.

В настройках тестового пула использованы малые значения параметров, что
бы была возможность рассмотреть нештатные ситуации с нехваткой:

  - Volume Retention
  - Maximum Volumes
  - Maximum Volume Bytes
  - Места на ФС (если нет малого раздела, то можно создать файловую
    систему в файле и смонтировать как loop)

# Storage

Для распределения резервного хранения по разным точкам необходимо:

## bacula-sd.conf

В конфиге sd:

1.  Определить устройства (device), которые могут быть:
      - Ленточными накопителями
      - Каталогами на различных файловых системах (в случае если Media
        Type = File)

В конфиге sd, в секции определения себя, sd определяется как storage. Не
следует путать с последующими определениями storage в конфиге директора
с которыми мы далее будем работать.

## bacula-dir.conf

В конфиге директора:

1.  Определить storage, при описании задать:
      - Адрес sd к которому подключаться
      - Устройство (device) на данном sd
      - Название, по которому этот storage будет использоваться в
        заданиях (job)
2.  Описать пул (Pool)
3.  Использовать описанный storage в задании (job), указав:
      - Storage
      - Pool

Насколько я понимаю, пулы не привязаны к хранилищам (storage) и любой
пул может быть использован в любом хранилище. То есть, достаточно на
каждое устройство для записи (например каталог с файлами) определить
storage, после чего направлять произвольный пул на произвольное
устройство.

[Category:Bacula](Category:Bacula "wikilink")
