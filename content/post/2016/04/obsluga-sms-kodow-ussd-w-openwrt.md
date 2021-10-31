---
author: Morfik
categories:
- OpenWRT
date: "2016-04-23T15:16:00Z"
date_gmt: 2016-04-23 13:16:00 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- sms
- ussd
- modem
- huawei
- e3372
GHissueID: 420
title: Obsługa SMS i kodów USSD w OpenWRT
---

[W artykule poświęconym 3ginfo](/post/monitor-polaczenia-3glte-w-openwrt-3ginfo/)
omawialiśmy monitorowanie połączenia 3G/LTE pod OpenWRT. Zabrakło tam jednak pewnej funkcjonalności,
która ucieszyłaby chyba każdego użytkownika tego firmware. Chodzi oczywiście o wysyłanie i
odbieranie SMS oraz przesyłanie kodów USSD. Okazuje się, że `3ginfo` potrafi nam tę funkcjonalność
zapewnić, tylko wymagane jest doinstalowanie kilku pakietów i odpowiednie skonfigurowanie systemu. W
tym wpisie spróbujemy sobie ten mechanizm wysyłania SMS i kodów USSD skonfigurować.

<!--more-->
## Instalacja potrzebnego oprogramowania

By zaimplementować obsługę SMS w `3ginfo` i mieć możliwość łatwego wysyłania i odbierania tych
wiadomości, musimy doinstalować pakiet `gnokii` . Z kolei, jeśli chodzi o kody USSD, to potrzebny
nam jest pakiet `ussd159` . Problem w tym, że w wydaniu Chaos Calmer nie ma już tego drugiego
pakietu. W ostateczności można go [spróbować dociągnąć ze wcześniejszego wydania Barrier
Breaker](http://eko.one.pl/forum/viewtopic.php?id=13189) i zainstalować manualnie. Logujemy się
zatem przez SSH na router i wydajemy w terminalu te poniższe polecenia:

    # opkg update
    # opkg install gnokii
    # opkg install http://dl.eko.one.pl/barrier_breaker/ar71xx/packages/ussd159_20130115_ar71xx.ipk

## Konfiguracja gnokii

[Konfiguracja narzędzia gnokii](http://wiki.gnokii.org/index.php/Main_Page) sprowadza się do
utworzenia pliku `/etc/gnokiirc` i dodania w nim tej poniższego zawartości:

    [global]
    model = AT
    port = /dev/ttyUSB1
    connection = serial

Zapisujemy plik i sprawdzamy, czy modem LTE został przez `gnokii` poprawnie rozpoznany:

    # gnokii --identify
    GNOKII Version 0.6.21
    Couldn't read /root/.gnokiirc config file.
    IMEI         : 123456789101112
    Manufacturer : huawei
    Model        : E3372
    Product name : E3372
    Revision     : 21.286.03.01.159

Wygląda dobrze. Ten powyższy komunikat `Couldn't read /root/.gnokiirc config file` informuje nas
tylko, że nie odnaleziono konfiguracji użytkownika, którym w tym przypadku jest root. Niemniej
jednak, plik `/etc/gnokiirc` , który stworzyliśmy wyżej, jest brany pod uwagę, jako,  że jest to
konfiguracja systemowa. Można zatem to ostrzeżenie bez problemu zignorować.

Odpalamy przeglądarkę i przechodzimy na adres `http://192.168.1.1:81/` . Tutaj nasłuchuje demon
`uhttpd` , który obsługuje skrypt `3ginfo` . Po wgraniu powyższych pakietów, na stronie powinny
pojawić się nam dwa dodatkowe przyciski. Wygląda to mniej więcej tak:

![3ginfo-sms-ussd](/img/2016/04/1.3ginfo-sms-ussd.png#big)

### Wysyłanie SMS

Poziomu tego powyższego panelu www możemy jedynie wysyłać wiadomości. Nie mamy możliwości odbierania
SMS. Niemniej jednak, wysyłanie ich działa przyzwoicie. Wystarczy podać numer i wpisać treść
wiadomości. Unikajmy tylko stosowania polskich znaków:

![3ginfo-gnokii-wysylanie-sms](/img/2016/04/2.3ginfo-gnokii-wysylanie-sms.png#medium)

Jeśli nie chce nam się instalować pakietu `3ginfo` , to nic nie stoi na przeszkodzie, aby korzystać
z `gnokii` z poziomu terminala. Jeśli życzylibyśmy sobie wysłać wiadomość SMS w taki sposób, to po
zalogowaniu się na router musimy wpisać polecenie podobne do tego poniżej:

    # echo "Testowy SMS" | gnokii --sendsms +48600123456 -C 1
    GNOKII Version 0.6.21
    Send succeeded!

Przy pomocy `echo` komponujemy wiadomość, którą następnie podajemy `gnokii` . Ten zaś przyjmuje
argument `--sendsms` , w którym to podajemy numer, na który wysłać wiadomość SMS. Parametr `-C`
określa klasę SMS, [do
wyboru 0-3](http://devlib.symbian.slions.net/s3/GUID-CBFDD753-BAE3-5C40-B947-EB8CDA11CD23.html).

### Odbieranie SMS

Może i nie mamy możliwości odebrać SMS z poziomu www ale możemy to zrobić z konsoli. Logujemy się na
router i w terminalu wydajemy poniższe polecenie:

    # gnokii --getsms SM 0 end

Mamy tutaj parametr `--getsms` , który przyjmuje trzy argumenty. Pierwszy z nich, to typ pamięci, z
której chcemy odczytać SMS. W tym przypadku `SM` wskazuje na kartę SIM. Karta SIM może pomieścić
kilka wiadomości, które są numerowane od `0` do `24` . Łącznie zatem 25 wiadomości może być
przechowywanych. Powyższy zapis, tj. `0 end` oznacza, że chcemy odczytać wszystkie wiadomości z
karty. Możemy naturalnie zawęzić przedział precyzując, np. `3 6` . Można także zwrócić konkretną
wiadomość nie podając ostatniego parametru. Poniżej przykład odczytania wszystkich wiadomości SMS:

    GNOKII Version 0.6.21
    0. Inbox Message (unread)
    Date/time: 23/04/2016 14:03:50 +0200
    Sender: +48600123456 Msg Center: +48777332211
    Text:
    Wiadomosc dla bloga Morfitronik do oczytania z terminala

    GetSMS SM 1 failed! (The given location is empty.)
    GetSMS SM 2 failed! (The given location is empty.)
    ...
    GetSMS SM 25 failed! (Unknown error - well better than nothing!!)

Wyżej widzimy, że jedna wiadomość została odebrana przez system. Mamy dokładną datę i czas, numer, z
którego nadesłano SMS oraz treść wiadomości. Pozostałe pozycje są puste. Natomiast na ostatniej
został zwrócony błąd informujący nas o tym, że na liście jest 25 pozycji (od 0 do 24 włącznie).

### Kasowanie SMS

Wszystkie odebrane wiadomości są przechowywane na karcie. Z powodu ograniczonej ilości miejsca,
dobrze jest regularnie czyścić wiadomości przez ich kasowanie. Jeśli chcemy skasować wszystkie
wiadomości, to wpisujemy to poniższe polecenie:

    # gnokii --deletesms SM 0 25

Jeśli chcemy się pozbyć tylko jednego konkretnego SMS'a, to podajemy w miejscu `0-25` pozycję na
liście, np. `5` .

## Wysyłanie kodów USSD

W przypadku kodów USSD sprawa ma się nieco gorzej. Nie wiem czy to przez fakt przestarzałego pakietu
`ussd159`, który już nie jest obecny w wydaniu Chaos Calmer, czy też w grę wchodzą jakieś bliżej
nieokreślone czynniki. Problem w tym, że może i mamy możliwość wysłania kodu USSD z poziomu www ale
nie udało mi się za pomocą tego formularza uzyskać odpowiedzi zwrotnej. Poniżej próba wysłania kodu
USSD `*101#` :

![3ginfo-wysylanie-ussd](/img/2016/04/3.3ginfo-wysylanie-ussd.png#medium)

Niżej zaś brak odpowiedzi na wysłane żądanie:

![3ginfo-odpowiedz-ussd](/img/2016/04/4.3ginfo-odpowiedz-ussd.png#medium)

Trzeba zatem poszukać innej opcji, która umożliwi nam operowanie na kodach USSD.

### Polecenia AT

Alternatywą dla pakietu `ussd159` są polecenia AT. Nie są one zbytnio wygodne ale działają zwykle
bez zastrzeżeń. Możemy korzystać z takich narzędzi jak `picocom` czy `minicom` do komunikacji z
modemem i wprowadzania poleceń AT. Możemy także pójść na skróty i odpalić dwa okna terminala, na obu
zalogować się po SSH i rozmawiać z modemem bez problemów. W jednym oknie będą rejestrowane zwracane
komunikaty, w drugim zaś będziemy wysyłać polecenia AT. Wygląda to mniej więcej tak:

![openwrt-ussd-terminal](/img/2016/04/5.openwrt-ussd-terminal.png#huge)

Po lewej stronie są odpowiedzi zwracane przez `cat /dev/ttyUSB0` na zapytania wysłane po prawej
stronie . Mamy tam szereg przesłanych poleceń AT. Są to te poniższe:

    # echo -e "AT+CUSD=1,\"*111#\",15\r" >/dev/ttyUSB0
    # echo -e "AT+CUSD=1,\"1\",15\r" >/dev/ttyUSB0
    # echo -e "AT+CUSD=1,\"1\",15\r" >/dev/ttyUSB0

Mamy tutaj jeden kod USSD `*111#` oraz dwie odpowiedzi, kolejno `1` i `1` . Z lewej strony widzimy
log, który przewija się w zależności od tego jakie polecenie AT prześlemy po prawej stronie. W tym
przypadku sprawdziliśmy stan konta karty SIM, która jest w tym modemie LTE.

Dlaczego wolę ten powyższy sposób od `picocom` czy `minicom` ? W tym przypadku mamy wsparcie shell'a
i jego historii poleceń. Bardzo szybko można takie polecenie AT przesłać przez wybieranie
odpowiedniej pozycji w historii (wciskając klawisz Up ). Poza tym, nie musimy instalować dodatkowych
pakietów na routerze, co zaoszczędza trochę miejsca.
