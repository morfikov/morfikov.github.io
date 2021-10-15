---
author: Morfik
categories:
- Android
date:    2021-10-15 19:46:00 +0200
lastmod: 2021-10-15 19:46:00 +0200
published: true
status: publish
tags:
- smartfon
- mac
- wifi
- prywatność
- xiaomi
- redmi-9
GHissueID: 576
title: Randomizacja adresów MAC WiFi w smartfonie z Androidem
---

Posiadacze smartfonów z Androidem w wersji 10+ rzadko kiedy orientują się, że w tej wersji został
wprowadzony dość [ciekawy mechanizm mający na celu poprawnie prywatności][1] osób korzystających z
takich urządzeń. Chodzi o randomizację adresów MAC przy połączeniach z sieciami WiFi. Nie zawsze
ten mechanizm jest włączony domyślnie, przez co w kwestii prywatności użytkowników telefonów
niewiele się zmienia, np. tak jest w przypadku mojego Xiaomi Redmi 9. Samo włączenie randomizacji
MAC nie jest niczym skomplikowanym i nie trzeba do tego nawet praw administratora root (stosowna
opcja jest do załączenia w oknie konfiguracji połączenia WiFi). Zwykle jednak przy wykorzystaniu
standardowej randomizacji MAC mamy generowany unikalny adres MAC dla danej sieci WiFi (w oparciu o
jej ESSID). Można jednak pójść o krok dalej i sprawić, że przy każdym połączeniu, nawet do tej
samej sieci WiFi, będziemy mieli generowany inny adres MAC, co z kolei powinno nam zapewnić nieco
więcej prywatności w publicznych sieciach WiFi oraz też przeciwdziałać śledzeniu nas przy roamingu
między takimi sieciami.

<!--more-->
## Podstawowa randomizacja adresów MAC

Zapewne każdy z nas już nieraz konfigurował sieć WiFi w swoim smartfonie z Androidem. Ta czynność
nie jest jakoś bardziej skomplikowana i wystarczy w ustawieniach systemu przejść do WiFi i tam
wybrać jedną z dostępnych sieci. Pod zaawansowanymi opcjami będziemy mogli określić czy dla tej
sieci WiFi ma być wykorzystywany fizyczny adres MAC czy też bliżej nieokreślony losowy adres:

|   |   |   |
|---|---|---|
| ![](/img/2021/10/001.android-wifi-mac-address-randomization-settings.jpg#small) | ![](/img/2021/10/002.android-wifi-mac-address-randomization-settings.jpg#small) | ![](/img/2021/10/003.android-wifi-mac-address-randomization-settings.jpg#small) |

W taki oto prosty sposób możemy sprecyzować sobie by przy połączeniu do konkretnych sieci WiFi był
wykorzystywany fizyczny adres MAC urządzenia, a w przypadku pozostałych sieci losowy MAC. Takie
rozgraniczenie może nam się przydać, np. w przypadku gdy chcemy by w domowej sieci przydział
adresów IP za sprawą protokołu DHCP był konsekwentny ze względu na statyczne lease DHCP. Przy stałym
adresie MAC nasz telefon w naszej sieci domowej będzie miał zawsze ten sam adres IP. Natomiast w
każdej innej sieci ten adres IP będzie się różnił, bo adres MAC ulegnie zmianie.

Niemniej jednak, w przypadku tej podstawowej randomizacji MAC kluczowe znaczenie ma ESSID sieci
WiFi. Jeśli ten ESSID, do którego będziemy chcieli się podłączyć, będzie taki sam, np. do sieci
WiFi kumpla, to zostanie użyty dokładnie ten sam adres MAC, który wcześniej został wygenerowany
losowo dla tej sieci. Nawet jeśli usuniemy zapisaną sieć WiFi z telefonu i ponownie ją dodamy sobie
w ustawieniach Androida, to i tak ten sam adres MAC będzie wykorzystywany dla tej konkretnej sieci
WiFi z tym samym ESSID.

## Rozszerzona randomizacja adresów MAC

O ile ta podstawowa randomizacja adresów MAC znajdzie zastosowanie w przypadku większości
użytkowników Androida, o tyle w moim przypadku jest ona niezbyt wystarczająca. By faktycznie
zapewnić użytkownikom maksimum prywatności, ten adres MAC musi się zmieniać w przypadku każdego
nowego połączeniem z sieciami WiFi, które będziemy nawiązywać w przyszłości.

Tutaj do gry wchodzi właśnie rozszerzona wersja randomizacji adresów MAC (WiFi enchanced MAC
randomization). Niemniej jednak, to ustawienie jest ukryte przed oczami przeciętnego użytkownika
telefonu. By się do tej opcji dostać, trzeba włączyć w ustawieniach Androida opcje deweloperskie
(klikając parę razy w numerek kompilacji w informacji o telefonie). Gdy już mamy dostęp do opcji
deweloperskich, przewijamy ustawienia, aż zauważymy pozycję `WiFi enchanced MAC randomization` :

|   |   |
|---|---|
| ![](/img/2021/10/004.android-wifi-mac-address-randomization-developer-options.jpg#small) | ![](/img/2021/10/005.android-wifi-mac-address-randomization-developer-options.jpg#small) |

W przypadku mojego Xiaomi Redmi 9, opcja `WiFi enchanced MAC randomization` była wyłączona domyślnie
i trzeba ją było ręcznie przestawić.

Ponownie dodajemy sieć WiFi w ustawieniach systemu i również zaznaczamy opcję randomizacji adresu MAC (tak jak to było przy podstawowej randomizacji MAC).

![](/img/2021/10/006.android-wifi-mac-address-randomization-settings.jpg#small)

Po uzyskaniu połączenia, adres IP powinien być inny niż ten uzyskany wcześniej.

### Test randomizacji MAC

Poniżej znajduje się krótki test randomizacji adresów MAC przy podłączaniu telefonu do mojej
domowej sieci WiFi.

    RedViper hostapd: wifi5g: STA 7c:fd:6b:e4:54:5a IEEE 802.11: authenticated
    RedViper hostapd: wifi5g: STA 7c:fd:6b:e4:54:5a IEEE 802.11: associated (aid 1)
    RedViper hostapd: wifi5g: AP-STA-CONNECTED 7c:fd:6b:e4:54:5a
    RedViper hostapd: wifi5g: STA 7c:fd:6b:e4:54:5a RADIUS: starting accounting session 8F77263AF65AED9C
    RedViper hostapd: wifi5g: STA 7c:fd:6b:e4:54:5a WPA: pairwise key handshake completed (RSN)
    RedViper dnsmasq-dhcp[3394]: DHCPDISCOVER(br-lan) 7c:fd:6b:e4:54:5a
    RedViper dnsmasq-dhcp[3394]: DHCPOFFER(br-lan) 192.168.1.188 7c:fd:6b:e4:54:5a
    RedViper dnsmasq-dhcp[3394]: DHCPREQUEST(br-lan) 192.168.1.188 7c:fd:6b:e4:54:5a
    RedViper dnsmasq-dhcp[3394]: DHCPACK(br-lan) 192.168.1.188 7c:fd:6b:e4:54:5a

Mamy tutaj adres MAC `7c:fd:6b:e4:54:5a` oraz przydzielony mu adres IP `192.168.1.188` . Patrząc w
ustawienia telefonu można stwierdzić, że jest to fizyczny adres MAC dla karty WiFi.

Przy załączeniu podstawowej randomizacji adresów MAC mamy już nieco inny log:

    RedViper hostapd: wifi5g: STA f6:16:e7:49:bd:28 IEEE 802.11: authenticated
    RedViper hostapd: wifi5g: STA f6:16:e7:49:bd:28 IEEE 802.11: associated (aid 1)
    RedViper hostapd: wifi5g: AP-STA-CONNECTED f6:16:e7:49:bd:28
    RedViper hostapd: wifi5g: STA f6:16:e7:49:bd:28 RADIUS: starting accounting session 950DD7284630EA51
    RedViper hostapd: wifi5g: STA f6:16:e7:49:bd:28 WPA: pairwise key handshake completed (RSN)
    RedViper dnsmasq-dhcp[3394]: DHCPREQUEST(br-lan) 192.168.1.188 f6:16:e7:49:bd:28
    RedViper dnsmasq-dhcp[3394]: DHCPDISCOVER(br-lan) f6:16:e7:49:bd:28
    RedViper dnsmasq-dhcp[3394]: DHCPOFFER(br-lan) 192.168.1.225 f6:16:e7:49:bd:28
    RedViper dnsmasq-dhcp[3394]: DHCPREQUEST(br-lan) 192.168.1.225 f6:16:e7:49:bd:28
    RedViper dnsmasq-dhcp[3394]: DHCPACK(br-lan) 192.168.1.225 f6:16:e7:49:bd:28

Mamy tutaj już całkiem inny adres MAC, tj. `f6:16:e7:49:bd:28` i został mu też przydzielony inny
adres IP `192.168.1.225` . Próba usunięcia sieci z telefonu i dodanie jej jeszcze raz (wciąż z
załączoną podstawową randomizacją MAC) skutkuje przydziałem tego samego adresu IP, bo adres MAC nie
uległ w żaden sposób zmianie.

Jeśli zaś w ustawieniach deweloperskich załączymy rozszerzoną randomizację, to z każdym nowym
połączeniem do tej samej sieci WiFi powinniśmy mieć generowany inny MAC, co będzie skutkować
naturalnie innym adresem IP:

    RedViper hostapd: wifi5g: STA 3e:20:65:f0:15:a8 IEEE 802.11: authenticated
    RedViper hostapd: wifi5g: STA 3e:20:65:f0:15:a8 IEEE 802.11: associated (aid 1)
    RedViper hostapd: wifi5g: AP-STA-CONNECTED 3e:20:65:f0:15:a8
    RedViper hostapd: wifi5g: STA 3e:20:65:f0:15:a8 RADIUS: starting accounting session 447299657AA5104E
    RedViper hostapd: wifi5g: STA 3e:20:65:f0:15:a8 WPA: pairwise key handshake completed (RSN)
    RedViper dnsmasq-dhcp[3394]: DHCPDISCOVER(br-lan) 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPOFFER(br-lan) 192.168.1.144 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPDISCOVER(br-lan) 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPOFFER(br-lan) 192.168.1.144 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPDISCOVER(br-lan) 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPOFFER(br-lan) 192.168.1.144 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPREQUEST(br-lan) 192.168.1.144 3e:20:65:f0:15:a8
    RedViper dnsmasq-dhcp[3394]: DHCPACK(br-lan) 192.168.1.144 3e:20:65:f0:15:a8

#### Rozpoznanie czy MAC jest losowy

Jeśli się uważnie przyjrzymy, to dostrzeżemy, że losowe adresy MAC można dość prosto zidentyfikować.
Wszystko za sprawą uniwersalnego/lokalny bitu (universal/local bit), który [jest ustawiony w części
OUI][4] (Organizationally Unique Identifier) adresu MAC, by właśnie tę losowość adresu
zasygnalizować. Jeśli w tym najbardziej znaczącym oktecie adresu MAC [na drugiej pozycji][5]
będziemy mieli  `2` , `6`, `A` albo `E` , to mamy do czynienia z losowym adresem MAC. Czemu akurat
te konkretne wartości? Bo trzeba zanegować ten uniwersalny/lokalny bit oryginalnego adresu MAC.
Więcej na ten temat pisałem w [artykule o randomizacji adresów IPv6][6].

### Stock'owy ROM vs. ROM AOSP/LineageOS

Wygląda na to, że rozszerzona randomizacja adresów MAC dla sieci WiFi działa ździebko inaczej w
przypadku stock'owych ROM'ów producentów smartfonów (tutaj MIUI) i ROM'ów opartych na
AOSP/LineageOS. W przypadku tych pierwszych, Android zapisuje sobie losowy MAC dla danego ESSID i
przechowuje go do momentu usunięcia sieci WiFi z ustawień systemu. W przypadku AOSP, adres MAC
będzie losowany nawet bez zapominania/usuwania sieci WiFi. Jedno i drugie rozwiązanie ma swoje wady
i zalety, niemniej jednak ja preferuję model, który [figuruje w AOSP][7].

## A co z nazwą hosta (hostname)

Trzeba jednak pamiętać, że inny adres MAC nie zawsze może nam w pełni zagwarantować prywatność.
Często zapomina się o nazwie hosta, która zwykle jest przesyłana przez klienta DHCP do serwera DHCP
w celu uzyskania adresacji IP. W starszych wersjach Androida w zapytaniu DHCP był przesyłany w
hostname ciąg podobny do `android-4c52c33baae0b4fa` , gdzie pierwsza część nazwy tego hosta wskazuje
na system operacyjny, a drugi kawałek to unikalny numerek ID. W ten sposób można było bez większego
problemu zidentyfikować dane urządzenie w sieci, nawet jeśli zmieniło ono sobie adres MAC/IP.

A jak wygląda sprawa w przypadku nowszych Androidów? Teoretycznie w Androidach od wersji 8, [klient
DHCP powinien zwracać pusty hostname][3]. Sprawdźmy na przykładzie Androida 11 w moim Xiaomi Redmi
9 czy tak faktycznie się dzieje. Po zalogowaniu się na router, mamy coś takiego w aktualnie
przydzielonych lease DHCP:

![](/img/2021/10/007.android-wifi-mac-address-randomization-dhcp-lease.png#huge)

Wszystkie pozycje z pustym hostname to właśnie mój telefon. Zatem te nowsze Androidy zdają się nie
wysyłać hostname w zapytaniach DHCP ale czy aby na pewno? Postanowiłem sprawdzić przy pomocy
`tcpdump` jak wyglądają pakiety z zapytaniem DHCP i otrzymałem coś takiego:

    root@RedViper:~# tcpdump -i br-lan -pvn port 67 and port 68
    tcpdump: listening on br-lan, link-type EN10MB (Ethernet), capture size 262144 bytes

    02:08:08.571984 IP (tos 0x10, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 316)
        0.0.0.0.68 > 255.255.255.255.67: BOOTP/DHCP, Request from da:7f:8b:33:a9:f0, length 288, xid 0x57f78237, Flags [none]
              Client-Ethernet-Address da:7f:8b:33:a9:f0
              Vendor-rfc1048 Extensions
                Magic Cookie 0x63825363
                DHCP-Message Option 53, length 1: Discover
                Client-ID Option 61, length 7: ether da:7f:8b:33:a9:f0
                MSZ Option 57, length 2: 1500
                Vendor-Class Option 60, length 15: "android-dhcp-11"
                Parameter-Request Option 55, length 12:
                  Subnet-Mask, Default-Gateway, Domain-Name-Server, Domain-Name
                  MTU, BR, Lease-Time, RN
                  RB, Vendor-Option, URL, Option 108
    ...

No jak widać mamy tutaj w dalszym ciągu rozszerzenie DHCP z kodem `60` odpowiadającym za `Vendor
Class Identifier` wskazującym na `android-dhcp-11` . To rozszerzenie ma na celu pobrać stosowne
opcje z serwera DHCP dla tej konkretnej klasy urządzeń, jeśli oczywiście serwer DHCP takie opcje
oferuje. Niemniej jednak, za sprawą tego VCI można ustalić system operacyjny oraz jego wersję, w
tym przypadku Android 11. Jeśli takich urządzeń nie ma za wiele aktualnie podłączonych do danej
sieci WiFi, to potencjalnie dałoby radę nas zidentyfikować, nawet gdy adres MAC/IP uległby zmianie.

## Podsumowanie

Trzeba przyznać, że tym razem deweloperzy Androida postarali się, by nieco zadbać o prywatność
swoich użytkowników oferując randomizację adresów MAC. Trzeba tutaj jednak wyraźnie zaznaczyć, że
ten mechanizm dotyczy jedynie adresów MAC kart WiFi w naszych smartfonach. W przypadku korzystania
z internetu mobilnego, adresy MAC modemu GSM nie będą losowe, choć tutaj ma to raczej marginalne
znaczenie, bo przecie operator GSM jest nas w stanie śledzić na podstawie unikalnych numerów IMEI
przypisanych zwykle do imiennego konta abonenckiego.


[1]: https://source.android.com/devices/tech/connect/wifi-mac-randomization
[2]: /post/jak-zmienic-hostname-w-telefonie-z-androidem/
[3]: https://source.android.com/security/enhancements/enhancements80
[4]: https://datatracker.ietf.org/doc/html/rfc7042#section-2.1
[5]: https://groups.google.com/g/comp.mobile.android/c/BdVUmqKBB1E
[6]: /post/jak-wlaczyc-ipv6-privacy-extensions-w-debianie-slaac/
[7]: https://source.android.com/devices/tech/connect/wifi-mac-randomization-behavior#persistent
