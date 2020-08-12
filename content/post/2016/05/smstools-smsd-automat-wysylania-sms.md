---
author: Morfik
categories:
- Linux
date: "2016-05-06T20:33:30Z"
date_gmt: 2016-05-06 18:33:30 +0200
published: true
status: publish
tags:
- debian
- sms
- modem
- huawei
- e3372
title: SMStools i smsd, czyli automat do wysyłania SMS
---

Pod linux'em jest całe mnóstwo oprogramowania, które może realizować zadanie odbierania i wysyłania
wiadomości SMS. Są również narzędzia, dzięki którym cały proces związany z przetwarzaniem SMS'ów
można zautomatyzować. Jakiś czas temu opisywałem tego typu funkcjonalność na przykładzie
[gammu-smsd]({{< baseurl >}}/post/gammu-smsd-czyli-wysylanie-odbieranie-sms/). Nadal uważam, że
jest to przyzwoite narzędzie ale jakby nie patrzeć wymaga ono wielu zależności. Właśnie przez nie
`gammu-smsd` nie nadaje się do zastosowań, gdzie ma się do dyspozycji niewiele miejsca. Niemniej
jednak, w przypadku OpenWRT mamy tam możliwość zainstalowania pakietu `smstool3` , w którym jest
dostępny demon `smsd` . Tak się też składa, że debian również ma swoich repozytoriach to narzędzie
posiada, z tym, że w pakiecie [smstools](http://smstools3.kekekasvi.com/index.php?p=blacklist). W
tym wpisie skonfigurujemy sobie działającą bramkę SMS, która będzie automatycznie odbierać
wiadomości SMS i podejmować stosowne działanie w zależności od numeru czy treści otrzymanego
komunikatu.

<!--more-->
## Usługa dla systemd i reguła dla udev'a

Instalujemy zatem pakiet `smstools` . W nim dostarczany jest min. demon `smsd` i to on będzie
zajmował się przetwarzaniem SMS'ów. Trzeba wspomnieć o fakcie braku natywnej usługi dla systemd.
Zatem tego demona trzeba albo odpalać przez skrypt init, albo też można się pokusić o stworzenie
pliku usługi. Niżej mamy zawartość, którą musimy umieścić w pliku
`/etc/systemd/system/smstools.service` :

    [Unit]
    Description=SMS daemon
    Documentation=man:smsd(1)
    After=network-online.target
    Before=multi-user.target
    ConditionPathExists=/dev/huawei-E3372-1

    [Service]
    User=smsd
    Group=dialout
    Type=forking
    ExecStart=/usr/sbin/smsd
    PIDFile=/var/run/smstools/smsd.pid

    [Install]
    WantedBy=multi-user.target

Widoczny wyżej warunek `ConditionPathExists` można usunąć. Niemniej jednak, przydaje się on w
momencie, gdy do komputera nie jest podpięty modem. Standardowo ta [nazwa modemu wskazuje na
/dev/ttyUSB0 ale można ją zmienić]({{< baseurl >}}/post/zmiana-nazwy-interfejsu-modemu-ttyusb0/).
Tak czy inaczej, jako, że demon `smsd` będzie operował na skryptach, to musimy wziąć pod uwagę jakie
operacje w tym skrypcie są wykonywane. Standardowo ten demon pracuje z uprawnieniami użytkownika
`smsd` i grupy `dialout` . Jeśli jednak nasz skrypt wymaga uprawnień administratora, musimy
przepisać powyższe parametry.

Mając z głowy usługę dla systemd, napiszmy także regułę dla udev'a, która będzie startować i
stopować demona `smsd` w zależności od tego czy modem USB został podpięty do portu czy też z niego
wyciągnięty. Tworzymy zatem plik `/etc/udev/rules.d/80-modem.rules` i wrzucamy do niego ten poniższy
kod:

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
        ENV{ID_VENDOR_ID}=="12d1", \
        ENV{ID_MODEL_ID}=="15b6", \
        ENV{ID_USB_INTERFACE_NUM}=="00", \
        SYMLINK+="huawei-E3372-0", \
        ENV{REMOVE_CMD}="/bin/systemctl stop smstools.service" , \
        RUN+="/bin/systemctl start smstools.service"

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
        ENV{ID_VENDOR_ID}=="12d1", \
        ENV{ID_MODEL_ID}=="15b6", \
        ENV{ID_USB_INTERFACE_NUM}=="01", \
        SYMLINK+="huawei-E3372-1"

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
        ENV{ID_VENDOR_ID}=="12d1", \
        ENV{ID_MODEL_ID}=="15b6", \
        ENV{ID_USB_INTERFACE_NUM}=="02", \
        SYMLINK+="huawei-E3372-2"

Oczywiście te powyższe reguły tworzą także dowiązania symboliczne dla modemu. Wszystkie użyte w nich
opcje zostały dokładnie omówione przy okazji zmiany nazwy modemu, zatem odsyłam do podlinkowanego
powyżej artykułu.

## Konfiguracja demona smsd

Mamy z głowy uruchamianie demona `smsd` . Musimy jednak jeszcze skonfigurować to narzędzie. Do tego
celu służy plik `/etc/smsd.conf` . Na dobrą sprawę potrafi on być bardzo obszerny i składać się z
całej masy parametrów, przez co nie ma możliwości dokładnie opisać wszystkich możliwych do
określenia rzeczy. Poniżej zostanie wykorzystana tylko niewielka część opcji ale wszystkie
pozostałe przyjmą wartości domyślne. Jeśli chcemy się zagłębić nieco bardziej w strukturę
konfiguracji `smsd` , to odsyłam do [manuala znajdującego się na stronie
projektu](http://smstools3.kekekasvi.com/index.php?p=configure).

Trzeba mieć na uwadze, że modemy czy telefony są różne i nie wszystkie użyte poniżej parametry
znajdą zastosowanie przy każdym możliwym sprzęcie. W tym przypadku mamy do czynienia z modemem
Huawei E3372 (E3372s-153) w wersji NON-HiLink. Dla niego plik `/etc/smsd.conf` wygląda mniej więcej
tak (aktualna wersja pliku zawsze na [moim
github'ie](https://github.com/morfikov/files/blob/master/configs/etc/smsd.conf)):

    devices = GSM1
    outgoing = /var/spool/sms/outgoing
    checked = /var/spool/sms/checked
    incoming = /var/spool/sms/incoming
    failed = /var/spool/sms/failed
    sent = /var/spool/sms/sent
    incoming_utf8 = yes
    #logfile = /var/log/smstools/smsd.log
    loglevel = 5
    delaytime = 15
    errorsleeptime = 30
    executable_check = yes
    eventhandler = /opt/skrypty/lte-modem/lte


    [GSM1]
    pre_init = yes
    #init =
    #init2 =
    device = /dev/huawei-E3372-1
    pin = ignore
    baudrate = 115200
    check_network = 2
    check_memory_method = 2
    primary_memory = SM
    secondary_memory = SM
    secondary_memory_max = 25
    memory_start = 1
    device_open_retries = -1
    keep_open = no

Przede wszystkim, mamy tutaj dwie sekcje: globalną i urządzenia. Globalna sekcja zawiera opcje,
które są aplikowane do każdego urządzenia. Natomiast w przypadku sekcji urządzenia (określona za
pomocą `[GSM1]` ) możemy ustawić szereg specyficznych opcji dla modemu czy telefonu. Przy pomocy tej
sekcji jesteśmy w stanie także przepisać szereg globalnych opcji.

Na początek omówmy sobie parametry globalne. W `devices` umieszczamy listę nazw urządzeń. To na
podstawie tych nazw są nazywane sekcje urządzeń. W opcjach `outgoing` , `checked` , `incoming` ,
`failed` oraz `sent` mamy określone ścieżki do katalogów, w których będą przechowywane SMS. Jeśli
chcemy coś robić z wiadomościami, które do nas przyjdą, np. korzystać z `grep` , to musimy
zatroszczyć się, aby te pliki z formy binarnej przekonwertować do formy tekstowej o kodowaniu UTF-8.
Za to właśnie odpowiada opcja `incoming_utf8` . Jeśli chcemy by logi szły do syslog'a, a nie tak jak
`smsd` ma w standardzie do pliku, to pomijamy opcję `logfile` . Warto też sobie dostosować poziom
logowania przy pomocy `loglevel` . W opcji `delaytime` jest określony przedział czasu w sekundach,
przez który demon `smsd` zostanie uśpiony po wykonaniu wszystkich zdań. Podobnie sprawa wygląda w
przypadku parametru `errorsleeptime` , gdzie demon zostanie uśpiony przez jakiś czas po wystąpieniu
błędu. Opcja `executable_check` sprawdza czy skrypt określony w `eventhandler` jest w stanie się
wykonać (o tym będzie w dalszej części artykułu).

W sekcji urządzenia mamy na początku opcję `pre_init` , która przesyła polecenia AT do modemu przed
wystartowaniem demona `smsd` . Są to dwa polecenia `ATE0` oraz `AT+CMEE=1` . W `init` oraz `init2`
jesteśmy w stanie określić dodatkowe polecenia AT. Zwykle jednak nie będą nam te pozycji potrzebne.
Dalej mamy `device` i tutaj podajemy interfejs modemu. Jeśli nie chcemy aby `smsd` zajmował
urządzenie dla siebie przez cały czas swojego działania, to możemy ustawić również `keep_open` . W
takim przypadku może się czasem zdarzyć tak, że `smsd` będzie chciał używać chwilowo niedostępnego
modemu. Dlatego też powinniśmy ustawić opcję `device_open_retries` , która sprawi, że `smsd` będzie
mógł się próbować podłączyć w nieskończoność. Jeśli nie korzystamy z kodu PIN, to najlepiej jest
dezaktywować cały mechanizm za niego odpowiedzialny. W `baudrate` ustawiamy prędkość komunikacji z
modemem. Obecnie modemy są w stanie działać poprawnie z `115200` ale czasem trzeba zjechać do
`19200` czy nawet `9600` . Następnie mamy opcję `check_network` , która odpowiada za sprawdzanie czy
sieć GSM jest dostępna. Spowalnia to trochę działanie `smsd` . Dlatego też ustawiona została tutaj
wartość `2` , która sprawdza dostępność sieci tylko w przypadku wysyłania wiadomości SMS. Dalej mamy
parametr `check_memory_method` określający polecenia AT, które są wykorzystywane do sprawdzania
nowych wiadomości SMS (w tym przypadku `CMGD` ). Opcje `primary_memory` , `secondary_memory` oraz
`secondary_memory_max` odpowiadają za wybór pamięci do sprawdzania pod kątem nowych SMS'ów. Ostatnia
opcja, tj. `memory_start` odpowiada za pozycję, od której zacznie się czytanie wiadomości. Zwykle
jest to `1` ale niektóre urządzenia wymagają, by tutaj ustawić `0` .

## Skrypt obsługujący wiadomości SMS

Mamy zatem przygotowaną konfigurację dla demona `smsd` . Wyżej jednak skorzystaliśmy z parametru
`eventhandler` , który, jak nazwa wskazuje, odpowiada za obsługę zdarzeń. Mamy tutaj określony
skrypt shell'owy. W nim znajdują się akcje, które są wywoływane, o ile określone warunki zostaną
spełnione. Można skorzystać z [mojego
skryptu](https://github.com/morfikov/files/blob/master/scripts/smstools-lte-script.sh). Taki skrypt
z grubsza przyjmuje dwa argumenty. Pierwszym z nich jest rodzaj zdarzenia. Mamy tutaj do wyboru:
`SENT` , `RECEIVED` , `FAILED` , `REPORT` lub `CALL` . Drugim zaś jest nazwa pliku z otrzymaną
wiadomością. Poniżej znajduje się log ze zdarzenia odebrania SMS'a:

    2016-05-06 14:50:16,5, GSM1: SMS received, From: 48600123456
    2016-05-06 14:50:16,6, GSM1: Wrote an incoming message file: /var/spool/sms/incoming/GSM1.H00yfm
    2016-05-06 14:50:16,7, GSM1: Running eventhandler: /opt/skrypty/lte-modem/lte RECEIVED /var/spool/sms/incoming/GSM1.H00yfm
    2016-05-06 14:50:16,7, GSM1: Done: eventhandler, execution time 0 sec., status: 0 (0)
    2016-05-06 14:50:16,6, GSM1: Deleting message 0

Widzimy wyżej, że wiadomość została zapisana w pliku `/var/spool/sms/incoming/GSM1.H00yfm` oraz, że
`smsd` wywołał powyższy skrypt i podał mu dwa argumenty. Niżej znajduje się format otrzymanej
wiadomości SMS:

    From: 48600123456
    From_TOA: 91 international, ISDN/telephone
    From_SMSC: 4860654321
    Sent: 16-05-06 14:50:00
    Received: 16-05-06 14:50:16
    Subject: GSM1
    Modem: GSM1
    IMSI: 123456789011121
    Report: no
    Alphabet: UTF-8
    Length: 19

    Wiadomosc dla Morfitronik

Mamy tutaj szereg informacji opisujących tego SMS. Mamy numer, z którego wiadomość nadeszła (
`From:` ) oraz na samym dole treść wiadomości. W oparciu o te dane można zrobić szereg warunków. I
to w zasadzie cała magia. To jak zaprogramujemy sobie `smsd` zależy głównie od naszej wyobraźni.
