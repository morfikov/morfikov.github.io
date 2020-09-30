---
author: Morfik
categories:
- Linux
date:    2016-04-02 18:47:47 +0200
lastmod: 2016-04-02 18:47:47 +0200
published: true
status: publish
tags:
- resolver
- sieć
- aero2
- lte
- modem
- huawei
- e3372
- dns
- dnsmasq
- dnscrypt
title: Aero2 w połączeniu z dnsmasq i dnscrypt-proxy
---

[Aero2][1] już od dość dawna oferuje darmowy dostęp do internetu w technologi LTE ale jakoś
wcześniej nie byłem tym tematem zainteresowany. Parę dni temu złożyłem jednak wniosek o kartę SIM,
tak by posiadać zapasowe łącze na wypadek, gdyby mój obecny ISP z jakiegoś powodu padł. Aero2
oferuje wersję komercyjną jak i tę za free i każda z nich ma swoje wady i zalety. Jako, że to łącze
ma robić jedynie za zapas, to korzystam z wersji FREE, a jest ono dość poważne ograniczenie, tj.
występuje tutaj [kod CAPTCHA][2], który trzeba wpisywać tak co godzinę, po czym należy resetować
modem. Ten kod może zostać zaserwowany jedynie w przypadku korzystania z DNS Aero2 i pozornie
odpada możliwość używania własnego systemu DNS opartego o [dnsmasq][3] i [dnscrypt-proxy][4]. Po
kilku dniach eksperymentów udało mi się wypracować przyzwoitą konfigurację, która potrafi obejść to
ograniczenie, poprawiając tym samym prywatność i bezpieczeństwo w internecie korzystając z
darmowego LTE za sprawą Aero2. W tym wpisie postaramy się zaimplementować ten mechanizm na debianie.

<!--more-->
## Resolver na potrzeby Aero2

Przede wszystkim, potrzebne nam będą odpowiednie pakiety i konfiguracja systemu. Nie będziemy
korzystać z automatów, bo one tylko będą przeszkadzać. [Tutaj znajduje się opis konfiguracji
narzędzia dnscrypt-proxy][5], a [tutaj zaś jest opisana konfiguracja dnsmasq][6]. Nie będę
poruszał tutaj tych tematów i zakładam, że mamy już odpowiedni setup zaprojektowany oraz, że on
działa.

By ułatwić nieco zrozumienie samej konfiguracji, trzeba ustalić kilka rzeczy. Przede wszystkim,
wpisy w pliku `/etc/resolv.conf` wskazują na adres `127.0.0.1` . Jest to adres, na którym nasłuchuje
`dnsmasq` . Zapytania DNS wędrują do `dnsmasq` i ten albo odpowiada adresami IP ze swojego cache,
albo forward'uje te zapytania do `dnscrypt-proxy` na adres `127.0.2.1:5353` . Później te zapytania
są szyfrowane i przesyłane do upstream'owego serwera DNS.

By darmowy internet LTE od Aero2 nam działał, musimy wpisywać kod CAPTCHA co 60 minut. Za każdym
razem, gdy upływa ten czas, po przejściu na dowolny adres w przeglądarce, zostaniemy przekierowani
automatycznie na stronę `bdi.free.aero2.net.pl` . Oczywiście, pod warunkiem, że korzystamy z DNS od
Aero2. I tu właśnie napotykamy problem pozornie nie do obejścia. Korzystając z zaszyfrowanego
systemu DNS, zapytania nigdy nie dotrą do serwerów DNS Aero2, a bez tego nie zostaniemy
przekierowani na stronę z kodem. Nie wpiszemy kodu, to nie będzie połączenia. Musimy zatem
zaprojektować taki resolver, który w odpowiedniej chwili przekieruje nas na stronę z kodem CAPTCHA,
a po wszystkim będzie wysyłał zapytania do naszego lokalnego resolver'a.

## Konfiguracja dnsmasq

W `dnsmasq` mamy możliwość zdefiniowania wielu upstream'owych serwerów DNS i rozdzielenia ruchu do
nich na podstawie domen. Nie możemy jednak operować na domenie `bdi.free.aero2.net.pl` , bo ta jest
rozwiązywana na adres lokalny `10.2.37.78` . Można to sprawdzić znając serwery DNS Aero2 (
`212.2.96.51` i `212.2.96.52` ) w poniższy sposób:

    $ nslookup bdi.free.aero2.net.pl 212.2.96.51
    Server:         212.2.96.51
    Address:        212.2.96.51#53

    Non-authoritative answer:
    Name:   bdi.free.aero2.net.pl
    Address: 10.2.37.78

Musimy zatem zrobić inny trik, który polega na zdefiniowaniu tego powyższego powiązania w pliku
`/etc/dnsmasq.conf` oraz wytypowaniu jednej domeny, która zostanie przesłana do serwerów DNS Aero2.
Poniżej jest przykład konfiguracji, którą trzeba dodać do powyższego pliku:

    server=127.0.2.1#5353
    server=/wp.pl/212.2.96.51
    address=/bdi.free.aero2.net.pl/10.2.37.78

Pierwsza linijka odpowiada za przesłanie wszystkich zapytań do określonego serwera DNS. Druga
linijka przesyła zapytania o domenę `wp.pl` do serwera DNS Aero2. W trzeciej zaś mamy statyczny wpis
rozwiązujący domenę `bdi.free.aero2.net.pl` na adres prywatny `10.2.37.78` .

By dodatkowo zwiększyć bezpieczeństwo przeglądania internetu za pośrednictwem Aero2, możemy jeszcze
dodać ten poniższy blok kodu do pliku `/etc/dnsmasq.conf` :

    stop-dns-rebind
    rebind-localhost-ok
    rebind-domain-ok=free.aero2.net.pl

Opcja `stop-dns-rebind` zapobiega przekierowaniu w oparciu od DNS, tj. wpisujemy jedną domenę, a
jest nam zwracana inna, tzw [DNS rebinding][7] i dokładnie ten mechanizm ma miejsce w przypadku
serwowania strony z kodem CAPTCHA przez Aero2. Dlatego też musimy jedną domenę wyłączyć spod
działania tego mechanizmu i za to odpowiada parametr `rebind-domain-ok` .

## Konfiguracja wvdial

Musimy jeszcze skonfigurować połączenie. Na debianie sid, ta procedura sprowadza się do
zainstalowania pakietów `ppp` , `usb-modeswitch` oraz `wvdial` . W zależności od posiadanego modemu,
być może trzeba będzie zaaplikować konfigurację dla `usb-modeswitch` w pliku
`/etc/usb_modeswitch.conf` . Ja korzystam z modemu Huawei E3372s-153 w wersji NON-HiLink i żadnych
dodatkowych czynności pod kątem zmiany jego trybu pracy nie musiałem przeprowadzać. Teraz w pliku
`/etc/wvdial.conf` dodajemy ten poniższy blok kodu:

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

    [Dialer set-new]
    Init4 = AT^SYSCFGEX="0302",3FFFFFFF,1,2,800C5,,

    [Dialer aero2]
    Init6 = AT+CGDCONT=1,"IP","darmowy"
    Phone = *99#
    Password = "aero"
    Username = "aero"

Sekcja `[Dialer Defaults]` może być inna w zależności od posiadanego modemu i dobrze jest ją sobie
wygenerować za pomocą polecenia `wvdialconf` . Większość parametrów w tej sekcji powinna być raczej
zrozumiała. Dobrze jest ustawić `Check DNS` na `1` oraz dodać dwie domeny, które będą odpytywane
przy zestawianiu połączenia. Jedna z nich niech wskazuję na domenę, która została przekierowana do
DNS Aero2. Dodatkowo, by zapobiec przepisywaniu pliku `/etc/resolv.conf` powinniśmy ustawić `Auto
DNS` na `0` oraz wykomentować `usepeerdns` w pliku `/etc/ppp/peers/wvdial` . Można też oczywiście
nadać atrybut odporności na plik `/etc/resolv.conf` w poniższy sposób:

    # chattr +i /etc/resolv.conf

Pozostałe sekcje zawierają polecenia AT, które mówią modemowi co ma robić. Omówienie ich wychodzi
poza zakres tego artykułu. Jeśli ktoś jest ciekaw co te komendy oznaczają, to zawsze może rzucić
okiem na [mój plik
/etc/wvdial.conf](https://github.com/morfikov/files/blob/master/configs/etc/wvdial.conf).

## Testy połączenia LTE od Aero2 z własnym DNS

Czas najwyższy przetestować cały mechanizm. Podłączamy zatem modem do portu USB i łączymy się za
pomocą tego poniższego polecenia:

    # wvdial modem-start set-new aero2
    ...
    --> Default route Ok.
    --> Nameserver (DNS) Ok.
    --> Connected... Press Ctrl-C to disconnect

Powyżej widzimy, że połączenie zostało nawiązane z powodzeniem. W tech wili powinniśmy być w stanie
przejść na dowolny adres wpisując w przeglądarce w pasku adresu jakąś domenę. Po jakimś czasie
strona nie będzie nam się chciała załadować. W takim przypadku, musimy przejść na adres `wp.pl` i
zostaniemy przekierowani na stronę z kodem CAPTCHA:

![](/img/2016/04/1.aero2-captcha-kod.png#huge)

Wpisujemy kod i po chwili powinien nam zostać przywrócony dostęp do internetu:

![](/img/2016/04/2.aero2-captcha-kod.png#huge)

Musimy jeszcze zresetować połączenie modemu. Przechodzimy do terminala i wciskamy Ctrl-C . Po chwili
ponowne wpisujemy poniższe polecenie:

    # wvdial modem-start set-new aero2

Od tego momentu przez następną godzinę możemy przeglądać internet swobodnie, a wszystkie domeny (no
może poza jedną i jej subdomenami) będą rozwiązywane przez lokalny resolver, ewentualnie przesyłane
do upstream'owego serwera DNS drogą szyfrowaną i nikt nie będzie w stanie zidentyfikować domen,
które odwiedzaliśmy korzystając z darmowego internetu LTE od Aero2.

[1]: https://aero2.pl/
[2]: https://pl.wikipedia.org/wiki/CAPTCHA
[3]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[4]: https://dnscrypt.org/
[5]: /post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/
[6]: /post/cache-dns-buforowania-zapytan/
[7]: https://en.wikipedia.org/wiki/DNS_rebinding
