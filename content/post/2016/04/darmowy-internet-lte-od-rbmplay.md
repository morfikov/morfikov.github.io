---
author: Morfik
categories:
- Linux
date: "2016-04-03T14:57:40Z"
date_gmt: 2016-04-03 12:57:40 +0200
published: true
status: publish
tags:
- sieć
- lte
- rbm
- play
- debian
- modem
- huawei
- e3372
title: Darmowy internet LTE od RBM (Play)
---

We wpisie dotyczącym [konfiguracji serwerów DNS na potrzeby
Aero2]({{< baseurl >}}/post/aero2-w-polaczeniu-z-dnsmasq-dnscrypt-proxy/) wspomniałem, że ten
operator daje możliwość korzystania z internetu LTE praktycznie za darmo. Trzeba tam, co prawda,
złożyć wniosek i zapłacić jakieś grosze za przysłanie karty SIM ale opłat jako takich za
połączenie internetowe nie ma żadnych. Uporczywy może być jedynie kod CAPTCHA, który trzeba
wpisywać co 60 minut. Szukając na necie informacji na temat darmowego internetu LTE [doszukałem się
tego oto wpisu](http://jdtech.pl/2015/09/darmowy-internet-lte-w-redbullmobile-porady-2015.html).
Jest tam przedstawiony sposób na włączenie bezpłatnej usługi internetu LTE u operatora RBM (Play). Z
początku wydawało mi się to niezbyt wiarygodne, by tego typu oferta była w ogóle dostępna ale
okazało się jednak, że nie ma tutaj żadnego haczyka i ten internet LTE faktycznie można włączyć i
korzystać z niego za free. W tym wpisie postaramy się skonfigurować Debiana właśnie na potrzeby tej
usługi.

<!--more-->
## Aktywacja usługi darmowego internetu LTE w RBM (Play)

Jako, że w tym przytoczonym we wstępie linku jest opisana dokładna procedura zmiany taryfy na RBM5,
to tutaj tylko wymienię najważniejsze punkty. Po więcej informacji odsyłam do tamtego artykułu. Po
pierwsze musimy załatwić sobie starter RBM lub Play. Można zamówić darmowy starter za free bez
wychodzenia z domu ([szczegóły
tutaj](http://jdtech.pl/2015/07/darmowe-startery-operatorow-komorkowych.html)). Starter aktywujemy i
dzwonimy na infolinię [Fakt Mobile](http://www.faktmobile.pl/) pod numer `799 599 999` z prośbą o
przeniesienie numeru, z którego do nich dzwonimy. Trzeba będzie podać kod PUK, także przygotujmy go
sobie. Procedura przejścia zajmie kilka godzin, nie więcej jednak niż 3-4. Po za kończeniu tego
procesu, wracamy do RBM przy pomocy kodu USSD `*163#` , gdzie wybieramy `RBM na Kartę 5` . Ten krok
zajmie kilka czy nawet kilkanaście godzin, dlatego też lepiej uzbroić się w cierpliwość. Być może po
odczekaniu tego przedziału czasu będziemy musieli ponownie skorzystać z kodu `*163#` i jeszcze raz
ponowić żądanie zmiany taryfy. Tak przynajmniej to wyglądało w moim przypadku. Po chwili taryfa
powinna zostać aktywowana i powinniśmy wrócić z Fakt Mobile do RBM.

Odpalamy teraz przeglądarkę internetową i przechodzimy na adres `https://logowanie.play.pl/` , gdzie
logujemy się do serwisu online Play'a w celu włączenia darmowej usługi LTE. Wchodzimy kolejno w
`Pakiety i usługi` i wybieramy zakładkę `Pakiety` :

![]({{< baseurl >}}/img/2016/04/1.rbm-play-darmowy-internet-lte.png#huge)

Niżej mamy opcję włączenia usługi LTE:

![]({{< baseurl >}}/img/2016/04/2.rbm-play-darmowy-internet-lte.png#huge)

Aktywujemy i czekamy na potwierdzenie SMS. Tę usługę niekoniecznie trzeba aktywować via panel www i
można także to zrobić za pomocą kodu USSD `*111*480*1#` . Na wszelki wypadek sprawdzamy czy ta
usługa jest już aktywna kodem `*111*480*3#` . Jeśli usługa została włączona, to od tego momentu
przez 30 następnych dni jesteśmy w stanie korzystać z darmowego internetu LTE, o ile jesteśmy w
zasięgu nadajników Play i posiadamy urządzenie wpierające tę technologię. Po tym czasie zostanie
ona wyłączona i trzeba będzie ją ponownie aktywować w opisany wyżej sposób.

## Konfiguracja Debiana pod usługę darmowego internetu LTE

Mamy zatem aktywną usługę i teraz przyszła kolej na takie skonfigurowanie Debiana, by czasem nie
nastąpiło przełączenie na inną technologię, co zwykle będzie wiązać się z dodatkowymi kosztami lub
też uszczupleniem pakietu danych. Poniżej zostanie opisany sposób konfiguracji modemu Huawei
E3372s-153 w wersji NON-HiLink.

My nie będziemy tutaj korzystać z automatów typu `network-manager` i zwyczajnie skonfigurujemy sobie
to połączenie ręcznie. Przede wszystkim, potrzebne nam będą pakiety `ppp` , `usb-modeswitch` oraz
`wvdial` , które musimy zainstalować w swoim systemie. Podłączamy teraz modem do portu USB i
wydajemy polecenie `wvdialconf` w celu wygenerowania odpowiedniej sekcji `[Dialer Defaults]` w pliku
`/etc/wvdial.conf` . Edytujemy teraz ten plik i dodajemy do niego kilka rzeczy:

    [Dialer Defaults]
    Modem Type = Analog Modem
    Modem = /dev/ttyUSB1
    ISDN = 0
    Baud = 9600
    Dial Command = ATDT
    Dial Attempts = 3
    Dial Timeout = 30
    Auto Reconnect = 1
    Stupid mode = 1
    Auto DNS = 0
    Check DNS = 1
    DNS Test1 = wp.pl
    DNS Test2 = google.com
    Check Def Route = 1
    Idle Seconds = 0
    Init1 = ATZ
    Init2 = ATQ0 V1 E0 H0 S0=0

    [Dialer modem-start]
    Init1 = AT+CFUN=1

    [Dialer modem-stop]
    Modem = /dev/ttyUSB0
    Init1 = AT+CFUN=0

    [Dialer red_bull_mobile-auto]
    Init4 = AT^SYSCFGEX="030201",3FFFFFFF,1,2,800C5,,
    Init6 = AT+CGDCONT=1,"IP","internet"
    Phone = *99#
    Username = "blank"
    Password = "blank"

    [Dialer red_bull_mobile-lte]
    Init4 = AT^SYSCFGEX="03",3FFFFFFF,1,2,800C5,,
    Init6 = AT+CGDCONT=1,"IP","internet"
    Phone = *99#
    Username = "blank"
    Password = "blank"

Kluczowe jest ustawienie odpowiedniego trybu pracy modemu w linijce z `Init4` . Mamy tam polecenie
AT, które w pierwszym argumencie może przyjąć kilka wartości: `00` (auto) , `01` (GSM) , `02` (3G)
`03` (LTE) . Jeśli ustawimy tutaj, np. `030201` , to modem będzie preferował połączenie LTE ale w
przypadku jego braku połączy się po 3G, itd. Musimy tutaj wymusić tryb LTE. Wyżej mamy dwie sekcje
dla połączenia: jedną preferująca LTE, drugą wymuszającą LTE. Jeśli chcemy korzystać z darmowego
internetu LTE w RBM (Play), to łączymy się przy pomocy tego poniższego polecenia:

    # wvdial modem-start red_bull_mobile-lte

W przypadku, gdy nie będziemy w zasięgu, to modem nam się zwyczajnie nie połączy. Wtedy możemy
świadomie wskazać, by łączył się w pozostałych trybach korzystając z `red_bull_mobile-auto` .
