---
author: Morfik
categories:
- Android
date:    2020-01-29 19:00:00 +0100
lastmod: 2020-01-29 19:00:00 +0100
published: true
status: publish
tags:
- prywatność
- sieć
- wifi
- hostname
- smartfon
GHissueID: 29
title: Jak zmienić hostname w telefonie z Androidem
---

Przeglądając ostatnio listę sprzętów podłączonych do mojego routera WiFi, zauważyłem, że niektóre
pozycje na niej w polu z hostname mają coś na wzór `android-4c52c33baae0b4fa` . Pierwsza część
nazwy tego hosta wskazuje na system operacyjny, a drugi kawałek to unikalny numerek ID.  Nie jestem
zbytnio fanem rozgłaszania takich informacji publicznie, bo mogą one ułatwić ewentualne ataki, oraz
też identyfikują jednoznacznie dane urządzenie ([osobną kwestią jest adres MAC karty sieciowej][4]).
Ponadto, mając w sieci wiele mobilnych urządzeń, ciężko jest czasem połapać się który telefon ma
przypisany konkretny adres IP (bez patrzenia w ustawienia telefonu). Z reguły na linux'owym
desktopie czy laptopie zmiana hostname jest stosunkowo łatwym zadaniem ale w przypadku smartfona z
Androidem ten zabieg okazał się niezmiernie trudnym procesem. Jak zatem zmienić hostname telefonu,
by można było mu przypisać jakaś w miarę ludzką nazwę?

<!--more-->
## Hostname w zapytaniu DHCP

Jako, że dysponuję routerem, który ma wgrane OpenWRT, to postanowiłem dograć w nim pakiet `tcpdump`
i zobaczyć co tak naprawdę jest przesyłane w zapytaniu DHCP. Poniżej jest pierwszy pakiet, który
Android przesyła do routera w procesie uzyskiwania adresu IP:

    # tcpdump -i br-lan -pvn port 67 and port 68
    tcpdump: listening on br-lan, link-type EN10MB (Ethernet), capture size 262144 bytes

    22:19:39.643449 IP (tos 0x10, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 344)
        0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from a0:39:f7:5f:c0:45, length 316, xid 0x12f8e8e0, Flags [none]
              Client-Ethernet-Address a0:39:f7:5f:c0:45
              Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message Option 53, length 1: Discover
                Client-ID Option 61, length 7: ether a0:39:f7:5f:c0:45
                MSZ Option 57, length 2: 1500
                Vendor-Class Option 60, length 18: "android-dhcp-7.1.2"
                Hostname Option 12, length 24: "android-4c52c33baae0b4fa"
                Parameter-Request Option 55, length 10:
                  Subnet-Mask, Default-Gateway, Domain-Name-Server, Domain-Name
                  MTU, BR, Lease-Time, RN
                  RB, Vendor-Option

Jak widać, mamy adres MAC karty WiFi smartfona `a0:39:f7:5f:c0:45` . Jest też hostname
`android-4c52c33baae0b4fa` . Co ciekawe, mamy też wersję Androida `android-dhcp-7.1.2` i wygląda na
to, że [bez poprawy źródeł][2] nie da się tej informacji wymazać. Tak czy inaczej, jeśli nasz
smartfon łączy się z routerem i chce uzyskać adres IP za pośrednictwem protokołu DHCP, to prześle w
zapytaniu DHCP tę unikalną dość nazwę hosta i będzie ona widoczna m.in. na routerze:

![router-android-hostname-wifi](/img/2020/01/001-router-android-hostname-wifi.png#huge)

## Zmiana nazwy hosta w smartfonie

W niektórych wersjach Androida, lub też tych ich specyficznych modyfikacjach pochodzących od
konkretnego producenta telefonu, zmiany hostname można dokonać w ustawieniach WiFi albo też
niekiedy w zakładce informacji o telefonie. W przypadku mojego smartfona LG G4C, który ma wgrany
czystego Androida w wersji 7.1.2 (na bazie AOSP), nie ma tego typu opcji. Wygląda zatem, że w tym
telefonie nie idzie zmienić nazwy hosta.

### Hostname i /proc/sys/kernel/hostname

Standardowo w linux'ach, nazwę hosta ustawia się za pomocą polecenia `hostname` lub zapisując plik
`/proc/sys/kernel/hostname` . Do zmiany nazwy hosta są jednak potrzebne uprawnienia root, zatem
jeśli nasz telefon nie jest ukorzeniony, to możemy zwykle zapomnieć o całej zabawie. Tak czy
inaczej, to poniższe polecenie zwykle wystarcza na pełnowymiarowych linux'ach:

    c90:/ # hostname localhost

No i w przypadku Androida również zmianę nazwy hosta można w ten sposób przeprowadzić, przynajmniej
dla części systemu, bo niby hostname jest ustawiany poprawnie:

    c90:/ # hostname
    localhost

    c90:/ # cat /proc/sys/kernel/hostname
    localhost

ale Android dalej przesyła w pakietach DHCP tę nazwę, która przesyłał do tej pory,

### Klient dhcpcd

Jako, że sprawa dotyczy komunikacji z wykorzystaniem protokołu DHCP, to jakiś moduł/aplikacja w
Androidzie musi przez sieć przesłać [pakiet DHCPREQUEST][3]. Niby w Androidzie jest dostępny plik
`/etc/dhcpcd/dhcpcd.conf` , którego nazwa wskazuje na oprogramowanie `dhcpcd` , które jest
wykorzystywane w roli klienta DHCP. W tym pliku można wymusić przesyłanie konkretnej nazwy hosta
przez dopisanie w nim poniższej linijki:

    hostname localhost

Niemniej jednak, zmiana konfiguracji `dhcpcd` nie wpłynęła w żadnym stopniu na przesyłaną nazwę
hosta w zapytaniach DHCP i ciągle stary hostname jest obecny w pakiecie DHCPREQUEST.

Okazuje się jednak, że począwszy od wersji Androida 6.0, zmianie uległ klient DHCP, który był
używany w 5.1 i wcześniejszych wersjach tego systemu. W taki oto sposób `dhcpcd` został zastąpiony
przez `Java DHCP client` , i, jak można się domyśleć, tego klienta DHCP nie konfiguruje się przez
plik `/etc/dhcpcd/dhcpcd.conf` . Co ciekawe w Androidzie 6.0 w opcjach deweloperskich jest
możliwość przełączenia między tymi dwoma klientami w zależności czy zaznaczymy opcję `Użyj
starszego klienta DHCP` (Use legacy DHCP client):

![legacy-dhcp-client-android-dev-options](/img/2020/01/002-legacy-dhcp-client-android-dev-options.png#small)

W przypadku zaznaczenia tej opcji będzie wykorzystywany klient `dhcpcd`, natomiast w przeciwnym
przypadku będzie w użyciu ten nowy klient, tj. `Java DHCP client`. W nowszych wersjach Androida, ta
opcja może już nie być dostępna, przynajmniej u mnie jej już nie ma.

### Java DHCP client i net.hostname

Szukając informacji na temat tego nowszego klienta DHCP wykorzystywanego w Androidzie 6.0 i
następnych, natrafiłem na parametr `net.hostname` , który, jak nazwa może sugerować, odpowiada za
nazwę hosta w sieci. Ten parametr można odczytać przy pomocy `getprop` i w przypadku mojego
telefonu zwracana jest poniższa wartość:

    c90:/ # getprop net.hostname
    android-4c52c33baae0b4fa

No jak widać, ta nazwa hosta jest nieco inna niż ta zwracana przez polecenie `hostname` ale za to
pasuje do tej, która była obecna w zapytaniu DHCP. Na szczęście jest też do dyspozycji polecenie
`setprop` , które jest w stanie przepisać wartość parametru `net.hostname` :

    c90:/ # setprop net.hostname localhost
    c90:/ # getprop net.hostname
    localhost

Wydanie tego powyższego polecenia faktycznie przepisuje nazwę hosta, która jest przesyłana w
pakiecie DHCPREQUEST, co można potwierdzić w `tcpdump` :

    22:50:53.743535 IP (tos 0x10, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 328)
        0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from a0:39:f7:5f:c0:45, length 300, xid 0xddd7abec, Flags [none]
              Client-Ethernet-Address a0:39:f7:5f:c0:45
              Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message Option 53, length 1: Discover
                Client-ID Option 61, length 7: ether a0:39:f7:5f:c0:45
                MSZ Option 57, length 2: 1500
                Vendor-Class Option 60, length 18: "android-dhcp-7.1.2"
                Hostname Option 12, length 9: "localhost"
                Parameter-Request Option 55, length 10:
                  Subnet-Mask, Default-Gateway, Domain-Name-Server, Domain-Name
                  MTU, BR, Lease-Time, RN
                  RB, Vendor-Option

Niemniej jednak, posłużenie się `setprop` daje efekt tylko tymczasowy i po restarcie urządzenia,
system powraca do domyślnej wartości hostname. Niektórzy radzili też by edytować plik
`/system/build.prop` i dopisać w nim tę poniższą linijkę (wymagane zamontowanie partycji `/system/`
w trybie do zapisu):

    net.hostname=localhost

Ale dla mnie to rozwiązanie nie przyniosło żadnego efektu i system startuje z domyślną nazwą hosta
ilekroć się go uruchomi ponownie.

### Opcje deweloperskie

Szukając jeszcze głębiej, doszukałem się w opcjach deweloperskich pozycji `Device Hostname` :

|     |    |
| --- | ---|
| ![android-dev-options-device-hostname](/img/2020/01/003-android-dev-options-device-hostname.png#small) | ![android-dev-options-device-hostname](/img/2020/01/004-android-dev-options-device-hostname.png#small) |

Dopiero zmiana nazwy hosta w opcjach deweloperskich przyniosła pożądany skutek, tj. po restarcie
telefonu, w zapytaniu DHCP jest przesyłany ten hostname, który sobie tutaj ustawimy.

## Nowsze Androidy (>8.0)

[W Androidach począwszy od wersji 8.0][1] parametr `net.hostname` jest pusty i Androidowy klient
DHCP nie przesyła już do serwera DHCP nazwy hosta, co poprawia ździebko prywatność. Niemniej
jednak, jeśli nasz telefon ma starszą wersję Androida i chcielibyśmy zreprodukować to zachowanie,
to raczej nam się to nie uda. Ustawienie pustej nazwy hosta przywraca domyślną nazwę, także jakąś
nazwę hosta w tych starszych wersjach systemu trzeba ustawić. Gdy już to zrobimy, to bez problemu
na routerze odnajdziemy szukany telefon:

![router-android-new-hostname-wifi](/img/2020/01/005-router-android-new-hostname-wifi.png#huge)


[1]: https://source.android.com/security/enhancements/enhancements80
[2]: https://android.googlesource.com/platform/frameworks/base/+/refs/tags/android-7.1.2_r1/services/net/java/android/net/dhcp/DhcpPacket.java#617
[3]: https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#Request
[4]: https://source.android.com/devices/tech/connect/wifi-mac-randomization
