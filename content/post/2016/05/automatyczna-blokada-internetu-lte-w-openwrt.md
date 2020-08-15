---
author: Morfik
categories:
- OpenWRT
date: "2016-05-05T15:20:14Z"
date_gmt: 2016-05-05 13:20:14 +0200
published: true
status: publish
tags:
- chaos-calmer
- lte
- rbm
- play
- router
title: Automatyczna blokada internetu LTE w OpenWRT
---

Jakiś czas temu opisywałem [darmowy internet LTE w
RBM/PLAY]({{< baseurl >}}/post/darmowy-internet-lte-od-rbmplay/). Jego niewątpliwą zaletą jest
fakt, że jest za free, o ile posiadamy odpowiedni modem. Niemniej jednak, ta usługa jest na 30 dni,
po upływie których trzeba ją aktywować na nowo. Jeśli z jakichś przyczyn się tego nie zrobi, to
wtedy korzystanie z internetu może nas słono kosztować. Niby można zaprzęgnąć do pomocy [gammu-smsd,
który będzie nas powiadamiał
SMS'em]({{< baseurl >}}/post/gammu-smsd-czyli-wysylanie-odbieranie-sms/), że usługa została
wyłączona lub włączona. Niemniej jednak, w dalszym ciągu pozostaje do ogarnięcia kwestia czasu,
przez który czekamy na włączenie usługi. Najlepszym wyjściem jest całkowita blokada internetu na
routerze, tak by przez ten moment nie nawiązać żadnego połączenia. Jeśli nie nawiążemy połączenia,
to dane z pakietu danych nie będą nam uciekać. W momencie, gdy usługa zostanie aktywowana, to
blokada będzie zdejmowana. Tego typu rozwiązanie można zaimplementować w OpenWRT za sprawą
[oprogramowania smstools](http://smstools3.kekekasvi.com/) dostępnego w pakiecie `smstools3` . W tym
wpisie postaramy się zaprojektować swojego rodzaju automatyczne blokowanie internet w zależności od
otrzymywanych komunikatów od operatora GSM.

<!--more-->
## Gammu vs. smstools3

W OpenWRT mamy kilka pakietów, które są w stanie realizować kwestię automatycznego odcinania
połączenia internetowego w oparciu o wiadomości SMS pochodzące z sieci operatora GSM. Te, które
leżą w obrębie naszego zainteresowania to `gammu` oraz `smstools3` . W tym przypadku nie korzystamy
z `gammu` z dwóch powodów. Po pierwsze jest on bardzo ciężki jeśli chodzi o zależności. Na dobrą
sprawę, by z niego korzystać pod OpenWRT, potrzebny będzie nam [mechanizm
extroot/whole\_root]({{< baseurl >}}/post/extroot-whole_root-fullroot-pod-openwrt/). Nawet mój
router [TP-LINK Archer C7 v2](http://www.tp-link.com.pl/products/details/Archer-C7.html) wyposażony
w 16 MiB flash'a nie był w stanie pomieścić wszystkich potrzebnych zależności. Dlatego też `gammu`
niezbyt się nadaje do stosowania w przypadku domowych routerów. Drugi powód, dla którego nie
będziemy korzystać z `gammu` jest taki, że najwyraźniej nie działa on poprawnie pod OpenWRT.
Próbowałem w tej sprawie zapytać na [forum
OpenWRT](https://forum.openwrt.org/viewtopic.php?id=64465) ale nie uzyskałem żadnej odpowiedzi póki
co. Być może z racji ogromnej ilości zależności nie wiele osób używa tego oprogramowania.

Dlatego właśnie będziemy korzystać z pakietu `smstools3` . W przeciwieństwie do poprzednika, nie
zaciąga całej masy pakietów i jego instalacja wraz z oprogramowaniem obsługującym modem LTE zmieści
się bez problemu na flash'u routera o pojemności 8 MiB. Zatem zainstalujmy ten pakiet:

    # opkt update
    # opkg install smstools3

## Konfiguracja pakietu smstools3

Po instalacji pakietu `smstools3` zostanie utworzony plik `/etc/smsd.conf` . Musimy go odpowiednio
uzupełnić. Poniżej jest skrócona wersja mojego pliku konfiguracyjnego. Bardziej obszerna wersja jest
na [moim github'ie](https://github.com/morfikov/files/blob/master/configs/etc/smsd.conf):

    devices = GSM1
    outgoing = /var/spool/sms/outgoing
    checked = /var/spool/sms/checked
    incoming = /var/spool/sms/incoming
    failed = /var/spool/sms/failed
    sent = /var/spool/sms/sent
    incoming_utf8 = yes
    logfile = /var/log/smsd.log
    loglevel = 5
    delaytime = 10
    errorsleeptime = 10
    eventhandler = /etc/skrypty/lte


    [GSM1]
    pre_init = yes
    device = /dev/ttyUSB1
    pin = ignore
    baudrate = 115200
    check_network = 2
    check_memory_method = 2
    primary_memory = SM
    secondary_memory = SM
    secondary_memory_max = 25
    memory_start = 1
    detect_unexpected_input = no
    status_include_counters = no
    status_signal_quality = no
    #report_device_details = no
    device_open_retries = -1
    keep_open = no
    mode = new
    rtscts = no
    incoming = yes

Wszystkie wykorzystane wyżej opcje zostały opisane przy okazji bawienia się [demonem smsd pod
debianem]({{< baseurl >}}/post/smstools-smsd-automat-wysylania-sms/). Jeśli ktoś jest ciekaw
znaczenia powyższych parametrów, to odsyłam do podlinkowanego artykułu.

## Skrypt blokujący połączenie LTE

Demon `smsd` , który będzie zajmował się obsługą SMS, [przekazuje dwa lub trzy argumenty do
mechanizmu obsługi zdarzeń](http://smstools3.kekekasvi.com/index.php?p=eventhandler). Pierwszym z
nich są zmienne i tu mamy do dyspozycji `SENT` , `RECEIVED` , `FAILED` , `REPORT` lub `CALL` .
Drugim parametrem jest nazwa pliku. Trzecim zaś ID wysłanej z powodzeniem wiadomości przy włączonym
statusie raportowania. Jako, że nas interesuje jedynie przypadek odbierania wiadomości od operatora
GSM, to będziemy rozpatrywać jedynie zmienną `RECEIVED` . Skrypt, który widnieje w `eventhandler`
można pobrać z [mojego
github'a](https://github.com/morfikov/files/blob/master/scripts/smstools-lte-script.sh).

## Zasada działania blokady internetu LTE

Zasada działania tego mechanizmu jest następująca. W pewnym momencie, przychodzi na modem SMS, który
ma w treści komunikat informujący nas o fakcie wyłączenia usługi LTE. Zawiera on słowa kluczowe
`LTE` oraz `zostala wylaczona` . W oparciu o nie zostanie uruchomionych szereg akcji. Przede
wszystkim, zostanie położony interfejs sieciowy, na którym operuje modem LTE. W ten sposób router
nie będzie w stanie wysłać i odebrać żadnych danych z internetu. O fakcie założenia blokady
zostaniemy powiadomieni wiadomością SMS, która zostanie przesłana na wskazany numer telefonu. System
wyśle również automatycznie kod USSD aktywujący usługę LTE i będzie oczekiwał na zwrotną wiadomość
SMS od operatora GSM. W tej wiadomości znajdą się słowa kluczowe `LTE` oraz `zostala wlaczona` i w
oparciu o nie zostaną podjęte następujące kroki. Najpierw zostanie podniesiony interfejs sieciowy
modemu, po czym zostanie wysłany SMS o zdarzeniu na określony numer telefonu. Wszystkie komunikaty z
przeprowadzanych operacji są logowane do pliku `/var/log/smsd.log` .
