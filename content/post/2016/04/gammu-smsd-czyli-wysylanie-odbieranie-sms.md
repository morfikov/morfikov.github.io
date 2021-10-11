---
author: Morfik
categories:
- Linux
date: "2016-04-09T18:53:58Z"
date_gmt: 2016-04-09 16:53:58 +0200
published: true
status: publish
tags:
- debian
- sms
- modem
- huawei
- e3372
GHissueID: 423
title: Gammu-smsd, czyli wysyłanie i odbieranie SMS
---

Eksperymentując ostatnio z modemem Huawei E3372s-153 w wersji NON-HiLink, pomyślałem, że przydałoby
się zaprojektować jakiś prosty system do odbioru i wysyłania SMS. Nie chodzi tutaj o zaprzęgnięcie
do pracy oprogramowania takiego jak [modem-manager-gui](https://linuxonly.ru/cms/page.php?7) czy też
[wammu](https://wammu.eu/wammu/) ale bardziej o przekazywanie tych wiadomości SMS, które trafiają na
modem, na inny numer telefonu komórkowego. Czyli stworzenie takiego telefonicznego proxy, które te
SMS będzie przekazywał dalej. Tego typu funkcjonalność można zaimplementować praktycznie w każdym
linux'ie, tylko wymagane jest posiadanie odpowiednich narzędzi. W tym przypadku rozchodzi się o
`gammu-smsd` i to o nim będzie ten wpis.

<!--more-->
## Parę słów o gammu-smsd

[Gammu-smsd](https://wammu.eu/smsd/) to demon, który co jakiś czas skanuje modem i patrzy czy
dotarły do niego jakieś wiadomości SMS. Jeśli takowe wiadomości zostaną zarejestrowane, podejmowana
jest określona akcja zależna od widzimisię administratora systemu. Pakiet `gammu-smsd` jest
standardowo dostępny w debianie i nie powinno być problemów z jego instalacją. Niemniej jednak,
trzeba wziąć pod uwagę, że w tym pakiecie dostarczana jest usługa dla systemd. Ta usługa ma jedną
wadę. Chodzi o to, że by demon `gammu-smsd` mógł nasłuchiwać, musi być podpięty modem lub inne
urządzenie zdolne odbierać i wysyłać wiadomości SMS. W przypadku, gdy taki modem nie jest
podłączony lub też odłączymy go w trakcie pracy demona, w logu zostanie zarejestrowana cała masa
błędów. Przydałoby się coś w tej sprawie zrobić. Dlatego też przerobimy sobie nieco tę standardową
usługę. Konkretnie chodzi o dodanie warunku, by usługa odpaliła się jedynie w przypadku, gdy w
systemie został wykryty modem. Możemy to zrobić w następujący sposób. W terminalu wpisujemy to
poniższe polecenie:

    # systemctl edit --full gammu-smsd.service

I dopisujemy do pliku usługi poniższą linijkę:

    ConditionPathExists=/dev/huawei-E3372-1

Ścieżkę do urządzenia trzeba będzie prawdopodobnie sobie dostosować. Zwykle taki modem USB ma kilka
interfejsów, które są obecne w katalogu `/dev/` , np. `ttyUSB0` , `ttyUSB1` , etc. Niemniej jednak,
w przypadku podpięcia do komputera większej liczby urządzeń, takich jak modemy czy telefony
komórkowe, te nazwy mogą ulec zmianie. Nie możemy do takiej sytuacji dopuścić i musimy zatroszczyć
się o stałość tych nazw. [Możemy to osiągnąć, np. za pomocą reguł
udev'a](/post/zmiana-nazwy-interfejsu-modemu-ttyusb0/). W taki sposób można
stworzyć link do interfejsu modemu i to go wykorzystywać w konfiguracji. Po szczegóły odsyłam do
podlinkowanego wpisu. Tak czy inaczej, po edycji pliku usługi dla systemd, musimy przeładować
konfigurację w poniższy sposób:

    # systemctl daemon-reload

Usługa, którą przed chwilą zmieniliśmy, startuje przy uruchamianiu linux'a. W przypadku, gdy modem
nie będzie podpięty, to ona się nam nie uruchomi. Musimy jeszcze sobie zaprojektować mechanizm,
który będzie aktywował i dezaktywował demona `gammu-smsd` w zależności od tego czy ten modem
zostanie wpięty/wypięty z portu USB. Do tego celu musimy napisać regułę dla udev'a.

## Reguła udev'a aktywująca i dezaktywacja demona gammu-smsd

Oczywiście nic nie stoi na przeszkodzie, by ten krok pominąć. W tym przypadku modem jest jedynie
podłączany sporadycznie i usługa jaką oferuje demon `gammu-smsd` nie może działać non stop. Jeśli
potrzebujemy wyciągać modem z portu USB co jakiś czas, to stwórzmy sobie plik `80-modem.rules` w
katalogu `/etc/udev/rules.d/` i dodajmy do niego tę poniższą treść:

    ACTION=="add", KERNEL=="ttyUSB?", SUBSYSTEM=="tty", \
          ENV{ID_VENDOR_ID}=="12d1", \
          ENV{ID_MODEL_ID}=="15b6", \
          ENV{ID_USB_INTERFACE_NUM}=="00", \
          SYMLINK+="huawei-E3372-0", \
          ENV{REMOVE_CMD}="/bin/systemctl stop gammu-smsd.service" , \
          RUN+="/bin/systemctl start gammu-smsd.service"

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

## Konfiguracja gammu-smsd

Mając odpowiednio dostosowaną usługę, możemy teraz przejść do konfiguracji `gammu-smsd` . Interesuje
nas głównie plik `/etc/gammu-smsdrc` , gdzie musimy zdefiniować szereg parametrów. Poniżej przykład
konfiguracji:

    [gammu]
    port = /dev/huawei-E3372-1
    connection = at115200
    name=Huawei E3372
    model=at

    [smsd]
    service = files
    logfile = syslog
    debuglevel = 0
    pin = ""
    CheckSecurity = 0
    CheckBattery = 0
    #SendTimeout = 30
    #CommTimeout = 15
    ReceiveFrequency = 5

    inboxpath = /var/spool/gammu/inbox/
    outboxpath = /var/spool/gammu/outbox/
    sentsmspath = /var/spool/gammu/sent/
    errorsmspath = /var/spool/gammu/error/

    #RunOnFailure =
    #RunOnSent =
    RunOnReceive = /opt/skrypty/sms-rec-sh

Musimy odpowiednio dostosować sobie sekcję `[gammu]`. Wszystkie wykorzystane wyżej parametry nie
powinny sprawić problemu i są raczej zrozumiałe. Jeśli ktoś chce nieco więcej poczytać o tych
powyższych opcjach, jak i o szeregu innych, które można wykorzystać w tym pliku konfiguracyjnym, to
[zapraszam zapoznać się z tym linkiem](https://wammu.eu/docs/manual/smsd/config.html).

To co nas będzie w tym pliku interesować najbardziej, to ostatnia linijka, tj. `RunOnReceive` .
Odpowiada ona za wykonanie skryptu, do którego ścieżka została tam określona. Jako, że `gammu-smsd`
ma działać na zasadzie przekaźnika SMS, to niezbędny będzie nam poniższy kod, który zapisujemy w
ww. pliku:

    #!/bin/sh

    SMS_MESSAGES=1

    PROGRAM="/bin/echo -e"

    for i in `seq $SMS_MESSAGES`
    do
          eval "$PROGRAM \"Number: \${SMS_${i}_NUMBER} | Message: \${SMS_${i}_TEXT}\""
          eval "$PROGRAM \"Numer: \${SMS_${i}_NUMBER}\nWiadomosc: \${SMS_${i}_TEXT}\"" | gammu-smsd-inject TEXT +48600123456
    done

W skrypcie są przeprowadzane dwie akcje. Pierwsza z nich wyświetla numer i treść wiadomości SMS w
logu systemowym. Druga akcja wydobywa zarówno numer nadawcy jak i sam komunikat i przesyła całość we
wskazanej formie na zdefiniowany numer telefonu przy pomocy `gammu-smsd-inject` . Wszystkie odebrane
i wysłane wiadomości są katalogowane w plikach pod `/var/spool/gammu/` . [Więcej informacji na temat
gammu-smsd-inject można znaleźć tutaj](https://wammu.eu/docs/manual/smsd/inject.html).

## Testy demona gammu-smsd

Przydałoby się jeszcze ten cały mechanizm przetestować. Zatem wyślijmy wiadomość SMS na numer karty
SIM, która jest aktualnie w modemie. Jeśli wszystko pójdzie zgodnie z planem, to zarówno system
powinien zalogować odpowiedni komunikat, jak i odebraną wiadomość posłać pod wskazany numer. Wyszło
to mniej więcej tak:

![](/img/2016/04/1.gammu-smsd-modem-sms-linux.png#huge)

Jak widać, wiadomość została odebrana. Po chwili skrypt został wykonany i komunikat został posłany
na telefon.
