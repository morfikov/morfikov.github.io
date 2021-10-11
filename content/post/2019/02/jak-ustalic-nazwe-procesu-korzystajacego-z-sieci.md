---
author: Morfik
categories:
- Linux
date: "2019-02-08T20:10:41Z"
published: true
status: publish
tags:
- debian
- logi
- iptables
- nftables
- auditd
- debugfs
- kprobes
- ftrace
GHissueID: 308
title: Jak ustalić nazwę procesu korzystającego z sieci
---

Konfigurując filtr pakietów `iptables`/`nftables` na Debianie zwykle nie przykładamy większej wagi
do procesów, które chcą nawiązać połączenia wychodzące z naszego linux'owego hosta. Mamy przecież
"skonfigurowany" firewall w łańcuchach `INPUT` i `FORWARD` i wszelkie zagrożenia z sieci nie
powinny nas dotyczyć. Problem w tym, że jeśli jakiś złowrogi proces zostanie uruchomiony w naszym
systemie, to jest on w stanie komunikować się ze światem zewnętrznym praktycznie bez żadnych
ograniczeń za sprawą braku jakichkolwiek reguł w łańcuchu `OUTPUT` . Można oczywiście temu zaradzić
budując zaporę sieciową na bazie `cgroups` , gdzie każda aplikacja będzie miała oznaczone pakiety,
przez co będzie można je rozróżnić i zablokować albo przepuścić przez filter. W tym wpisie jednak
nie będziemy się zajmować konstrukcją tego typu FW, tylko spróbujemy sobie odpowiedzieć na pytanie
jak namierzyć proces, który komunikuje się z siecią (lub też próbuje), posiadając jedynie log
`iptables`/`nftables` .

<!--more-->
## Narzędzia netstat i ss

Zwykle, gdy poszukujemy informacji na temat procesów korzystających z sieci, to sięgamy po
narzędzia typu `netstat` albo `ss` . No i faktycznie są one w stanie powiedzieć nam sporo na temat
aktualnie nawiązanych przez procesy połączeń.

Poniżej przykład `netstat`:

	# netstat -napletu
	...
	tcp        0      0 192.168.0.135:54182     104.18.70.113:443       ESTABLISHED 1000       3406460    110338/firefox
	tcp        0      0 192.168.0.135:61156     95.101.72.207:80        ESTABLISHED 1000       3419968    110338/firefox
	tcp        0      0 192.168.0.135:53406     104.16.52.111:443       ESTABLISHED 1000       3405672    110338/firefox
	tcp        0      0 192.168.0.135:61032     151.101.114.165:443     ESTABLISHED 1000       3402793    110338/firefox
	tcp        0      0 192.168.0.135:54184     104.18.70.113:443       ESTABLISHED 1000       3406461    110338/firefox
	tcp        0      0 192.168.0.135:62966     198.252.206.25:443      ESTABLISHED 1000       3341379    110338/firefox

A niżej zaś przykład `ss`:

	# ss -tupan
	...
	tcp    ESTAB      0       0        192.168.0.135:54182     104.18.70.113:443     users:(("firefox",pid=110338,fd=198))
	tcp    ESTAB      0       0        192.168.0.135:61156     95.101.72.207:80      users:(("firefox",pid=110338,fd=83))
	tcp    ESTAB      0       0        192.168.0.135:53406     104.16.52.111:443     users:(("firefox",pid=110338,fd=211))
	tcp    ESTAB      0       0        192.168.0.135:61032   151.101.114.165:443     users:(("firefox",pid=110338,fd=175))
	tcp    ESTAB      0       0        192.168.0.135:54184     104.18.70.113:443     users:(("firefox",pid=110338,fd=212))
	tcp    ESTAB      0       0        192.168.0.135:62966    198.252.206.25:443     users:(("firefox",pid=110338,fd=62))

W obu przypadkach mamy informacje na temat nazwy procesu oraz jego PID'u, no i oczywiście szereg
danych na temat samego połączenia. W tym przypadku jest wszystko oczywiste i odnalezienie szukanego
procesu nie nastręcza żadnych trudności.

Co jednak w przypadku, gdy proces nawiązuje lub próbuje nawiązać połączenie i bardzo szybko się sam
unicestwi? W tych listingach wyżej takiego procesu już nie zobaczymy. Jak zatem ustalić, które
procesy w naszym systemie próbują się łączyć z siecią?

## Logi iptables/nftables

Gdy nie jesteśmy pewni, czy cały ruch sieciowy wychodzący z naszego linux'a został odpowiednio
sklasyfikowany przez dodane reguły `iptables`/`nftables` , to możemy wrzucić do filtra regułkę
logującą pakiety i zobaczyć czy coś zostanie zalogowane. Poniżej jest przykładowa reguła dla
`nftables` :

	add rule inet filter OUTPUT limit rate 30/minute burst 1 packets log flags all prefix "* OUTPUT * " counter

Na razie nam nie zależy na tym, by cokolwiek jeszcze blokować i lepiej póki co jeszcze tego nie
robić. Mając dodaną regułę, zaglądamy do logu systemowego. W tym przypadku został zalogowany
poniższy komunikat:

    Feb 07 18:42:35 morfikownia kernel: * IPTABLES:OUTPUT * IN= OUT=bond0 SRC=192.168.0.135
    DST=104.81.106.31 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=2989 DF PROTO=TCP SPT=63798
    DPT=443 SEQ=850951390 ACK=0 WINDOW=64240 RES=0x00 SYN URGP=0 OPT
    (020405B40402080AA8F3EBDA0000000001030309) UID=1000 GID=1000

To co się rzuca od razu w oczy, to fakt, że proces, który wysłał ten pakiet, miał `UID=1000` i
`GID=1000` . No to zawęża nieco krąg poszukiwań ale wciąż odnalezienie tego procesu na podstawie
powyższego komunikatu nie jest zbytnio możliwe.

## Narzędzie auditd

Władając informacją na temat czasu zalogowania tego pakietu, można odszukać z dużą dokładnością
proces, który chciał ten pakiet wysłać. Potrzebne nam jednak będzie dodatkowe oprogramowanie, które
musimy zainstalować w swoim systemie. W Debianie instalujemy paczkę `auditd` . Zawiera ona demona
audytu, który jest w stanie zalogować dosłownie wszystko. My nie będziemy logować co popadnie, a
jedynie skupimy się na zalogowaniu wywołań systemowych typu `connect` .

Wszystko czego nam teraz trzeba to reguły, która zaloguje pożądane przez nas wywołanie systemowe.
Tą regułę możemy na sztywno dodać do pliku `/etc/audit/rules.d/audit.rules` lub też stosowne opcje
możemy podać w `auditctl` . Gdy dodajemy reguły do pliku, to trzeba przeładować demona. Poniżej
jest reguła, która zaloguje wywołania `connect` :

    # auditctl -a exit,always -F arch=b64 -S connect -k MYCONNECT

Wartość parametru `-k` może być dowolna. Jest to w zasadzie nic innego jak TAG, który oznaczy
wszystkie zalogowane komunikaty tym co sobie tutaj ustawimy -- to ma więcej sensu, gdy mamy nieco
więcej reguł. Aktualnie załadowane reguły można podejrzeć w poniższy sposób:

    # auditctl -l
    -a always,exit -F arch=b64 -S connect -F key=MYCONNECT

Wszystkie komunikaty audytu są logowane do pliku `/var/log/audit/audit.log` .

Czasami może się zdarzyć tak, że jakiś proces będzie próbował nawiązywać połączenia ale w logu
audytu żadnych informacji na jego temat nie znajdziemy. Być może trzeba będzie dodać również
wywołania `socket` i `bind` do tej powyższej reguły:

    auditctl -a exit,always -F arch=b64 -S socket,connect,bind -k MYCONNECT

### Logi w czytelnej dla człowieka formie (ausearch)

Gdy zajrzy się w plik `/var/log/audit/audit.log` , to informacje, które tam się znajdują nie są
zbyt zrozumiałe czy też czytelne dla człowieka. Można oczywiście ręcznie zdekodować te logi ale
zamiast się trudzić, to lepiej skorzystać z narzędzia `ausearch` . Jeśli mamy do czynienia z całą
masą wywołań systemowych, to dobrze jest określić `--start` i `--end` podając w nich czas, który
nas interesuje:

	# cat /var/log/audit/audit.log | ausearch -i
	...
	----
	type=PROCTITLE msg=audit(07/02/19 18:42:35.345:98867) : proctitle=wget --quiet -U firefox -o /dev/null -O /home/morfik/.conky/...
	type=PATH msg=audit(07/02/19 18:42:35.345:98867) : item=0 name=/var/run/nscd/socket nametype=UNKNOWN cap_fp=none cap_fi=none cap_fe=0 cap_fver=0
	type=CWD msg=audit(07/02/19 18:42:35.345:98867) : cwd=/media/Android/
	type=SOCKADDR msg=audit(07/02/19 18:42:35.345:98867) : saddr={ fam=local path=/var/run/nscd/socket }
	type=SYSCALL msg=audit(07/02/19 18:42:35.345:98867) : arch=x86_64 syscall=connect success=no exit=ENOENT(No such file or directory) a0=0x5 a1=0x7ffd307b3a50 a2=0x6e a3=0x6 items=1 ppid=68429 pid=68753 auid=morfik uid=morfik gid=morfik euid=morfik suid=morfik fsuid=morfik egid=morfik sgid=morfik fsgid=morfik tty=pts11 ses=19 comm=wget exe=/usr/bin/wget subj==conky (enforce) key=MYCONNECT
	----
	type=PROCTITLE msg=audit(07/02/19 18:42:35.345:98868) : proctitle=wget --quiet -U firefox -o /dev/null -O /home/morfik/.conky/...
	type=PATH msg=audit(07/02/19 18:42:35.345:98868) : item=0 name=/var/run/nscd/socket nametype=UNKNOWN cap_fp=none cap_fi=none cap_fe=0 cap_fver=0
	type=CWD msg=audit(07/02/19 18:42:35.345:98868) : cwd=/media/Android/
	type=SOCKADDR msg=audit(07/02/19 18:42:35.345:98868) : saddr={ fam=local path=/var/run/nscd/socket }
	type=SYSCALL msg=audit(07/02/19 18:42:35.345:98868) : arch=x86_64 syscall=connect success=no exit=ENOENT(No such file or directory) a0=0x5 a1=0x7ffd307b3c10 a2=0x6e a3=0x6 items=1 ppid=68429 pid=68753 auid=morfik uid=morfik gid=morfik euid=morfik suid=morfik fsuid=morfik egid=morfik sgid=morfik fsgid=morfik tty=pts11 ses=19 comm=wget exe=/usr/bin/wget subj==conky (enforce) key=MYCONNECT
	----
	type=PROCTITLE msg=audit(07/02/19 18:42:35.347:98869) : proctitle=wget --quiet -U firefox -o /dev/null -O /home/morfik/.conky/...
	type=SOCKADDR msg=audit(07/02/19 18:42:35.347:98869) : saddr={ fam=inet laddr=127.0.2.1 lport=53 }
	type=SYSCALL msg=audit(07/02/19 18:42:35.347:98869) : arch=x86_64 syscall=connect success=yes exit=0 a0=0x5 a1=0x7e5745179914 a2=0x10 a3=0x55c4af2df010 items=0 ppid=68429 pid=68753 auid=morfik uid=morfik gid=morfik euid=morfik suid=morfik fsuid=morfik egid=morfik sgid=morfik fsgid=morfik tty=pts11 ses=19 comm=wget exe=/usr/bin/wget subj==conky (enforce) key=MYCONNECT
	----
	type=PROCTITLE msg=audit(07/02/19 18:42:35.348:98875) : proctitle=wget --quiet -U firefox -o /dev/null -O /home/morfik/.conky/...
	type=SOCKADDR msg=audit(07/02/19 18:42:35.348:98875) : saddr={ fam=inet laddr=104.81.106.31 lport=443 }
	type=SYSCALL msg=audit(07/02/19 18:42:35.348:98875) : arch=x86_64 syscall=connect success=yes exit=0 a0=0x5 a1=0x7ffd307b4550 a2=0x10 a3=0x0 items=0 ppid=68429 pid=68753 auid=morfik uid=morfik gid=morfik euid=morfik suid=morfik fsuid=morfik egid=morfik sgid=morfik fsgid=morfik tty=pts11 ses=19 comm=wget exe=/usr/bin/wget subj==conky (enforce) key=MYCONNECT

Z tego logu wyżej wynika, że `conky` (oprofilowany przez AppArmor) wywołał sobie `wget` , który
próbował połączyć się z adresem `104.81.106.31` na porcie `443` ale wcześniej posłał zapytanie DNS
do `dnscrypt-proxy` , który nasłuchuje na adresie `127.0.2.1` i na porcie `53` . Komunikacja DNS
została przepuszczona wcześniej na zaporze sieciowej ale brakło stosownej reguły od `conky`/`wget` .
W podobny sposób można namierzyć praktycznie każdy inny proces, który z jakiegoś powodu próbuje się
komunikować ze światem zewnętrznym, a mając informacje o procesie, w tym też i ścieżkę do pliku
binarnego, to możemy już sobie ten proces zablokować lub przepuścić na zaporze bez większego
problemu.

## Ustalenie nazwy procesu i numeru PID via FTrace (kprobes)

Inną metodą, która jest w stanie nam pomóc w ustaleniu jaki proces chce komunikować się z siecią
jest [FTrace][1] ([kprobes][2]), który korzysta z systemu plików `tracefs` montowanego w
`/sys/kernel/tracing/` (we wersjach kernela do 4.1, był wykorzystywany system plików `debugfs`
montowany w `/sys/kernel/debug/tracing/` ) . Możemy nakazać kernelowi by logował każde nowe
połączenie, które dowolny program będzie otwierał, przykładowo:

    # echo 'p:m security_socket_connect' >> /sys/kernel/tracing/kprobe_events
    # echo 1 > /sys/kernel/tracing/events/kprobes/m/enable
    # cat /sys/kernel/tracing/trace_pipe

By ten mechanizm wyłączyć, wpisujemy w terminal poniższe polecenia:

    # echo 0 > /sys/kernel/tracing/events/kprobes/m/enable
    # echo '-:kprobes/m' >>  /sys/kernel/tracing/kprobe_events


[1]: https://www.kernel.org/doc/html/latest/trace/ftrace.html
[2]: https://www.kernel.org/doc/html/latest/trace/kprobetrace.html
