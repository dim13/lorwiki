Из описания su стоит убрать

su \[-\] ...

так как опция "-" вводит в заблуждение и имеет ограничения. Лучге
указывать вместо нее стандартную форму -l.

## О типичных граблях с логином root'a

Плюсую [vurdalak](http://www.linux.org.ru/people/vurdalak/profile)'a:
Стоит перечислить грабли, связанные с логином рута, на которые не
стоит натыкаться:

    PermitRootLogin из sshd_config
    AllowRootLogin из kdmrc
    RootLogin в proftpd.conf

ну и так далее, кто что вспомнит. Чтобы когда дефолтный или
скопипащенный конфиг ставится, минимизировать потенциальные
угрозы в этом направлении. --[Андрей](User:adriano32 "wikilink")
26-Jul-2011 13:24 MSK