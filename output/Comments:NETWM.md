```
    qDebug("Desktop name: %s", names + ind);

    // использовать strlen для UTF-8 строк не совсем корректно, т.к. внутри строки может быть управляющая
    // последовательность с нулём, но для данного случая мы сделаем допущение, что таких последовательностей
    // нам не встретится
    ind += (strlen(names+ind) + 1);
```

Почему бы вместо такого извращения не сконвертировать names в QString,
который потом разбить по нуль-символу. В ассистанте говорится, что так
делать можно.

[Alexander Galanin](User:gaa "wikilink") 09-Mar-2009 18:23 MSK

  -
    Можно и так, но для данного случая это будет излишеством. --[Alex
    Custov](User:alex_custov "wikilink") 09-Mar-2009 18:35 MSK

Напоминаю, при всём уважении к авторам статьи, правильно пишется [Net
WM](http://blackboxwm.sourceforge.net/NetWM) или на худой конец NetWM.
Несолидно, две ссылки в топе гугла, одна на оффсайт, вторая на Вики
ЛОРа, причём вторая противоречит первой.

Я к тому, что надо исправить как название статьи, так и упоминание в
содержимом. --[adriano32](User:adriano32 "wikilink") 06.06.2011
04:45