---
author: Morfik
categories:
- OpenWRT
date: "2016-05-05T18:04:55Z"
date_gmt: 2016-05-05 16:04:55 +0200
published: true
status: publish
tags:
- wps
- wifi
- chaos-calmer
- router
GHissueID: 494
title: WPS, czyli WiFi Protected Setup w OpenWRT
---

[Wi-Fi Protected Setup (WPS)](https://en.wikipedia.org/wiki/Wi-Fi_Protected_Setup) powstał w celu
ułatwienia konfiguracji urządzeń w sieci WiFi. Przy WPS nie musimy ustawiać wszystkich parametrów
połączenia ręcznie. Nie musimy także pamiętać nazwy sieci czy samego hasła, które może mieć nawet 64
znaki. Zamiast tego, cała konfiguracja sprowadza się to wciśnięcia dwóch przycisków: jednego na
karcie WiFi, drugiego na obudowie routera. Niemniej jednak, wszędzie gdzie nie spojrzeć, ludzie
rozpisują się na temat tego jakim to niebezpieczeństwem jest włączenie w routerach WiFi opcji WPS. O
tych zagrożeniach, jeśli kogoś interesują, można poczytać, np.
[tutaj](https://dankaminsky.com/2012/01/26/wps2/). W tym wpisie rozprawimy się raz na zawsze z
mitami dotyczącymi WPS, który jest implementowany w routerach WiFi. Postaramy się skonfigurować ten
mechanizm pod OpenWRT i zobaczymy czy cokolwiek z tego co ludzie piszą na necie ma zastosowanie w
praktyce.

<!--more-->
## WPS PIN, a może PBC

Na wiki OpenWRT możemy wyczytać, że jest do wyboru [spory wachlarz metod konfiguracyjnych
WPS](https://wiki.openwrt.org/doc/uci/wireless#wps_options). Mamy tam: `usba` , `ethernet` ,
`label` , `display` , `ext_nfc_token` , `int_nfc_token` , `nfc_interface` , `push_button` ,
`keypad` , `virtual_display` , `physical_display` , `virtual_push_button` oraz
`physical_push_button` . Natomiast wspierana jest, póki co, jedna z nich: `push_button` . Dodatkowo
mamy również w standardzie zaimplementowaną obsługę metody z PIN'em. Swego czasu ten PIN był
zaimplementowany tak, że domyślnie przybierał postać `12345670` i jakby tego było mało, można było
użyć tego PIN'u i zostać podłączonym do sieci nawet przy wyraźnym określeniu, że chce się korzystać
z przycisków. Wszystko to za sprawą błędnej konfiguracji w pliku `/lib/netifd/hostapd.sh` . Ta
dziura została załatana w nowszych wersjach OpenWRT, dokładniej w
[r42553](https://dev.openwrt.org/changeset/42553) i w przypadku, gdy używamy starszej wersji
firmware, przydałoby się ją zaktualizować.

Przede wszystkim, na necie zarzuty pod adresem WPS dotyczą kodu PIN. W skrócie, chodzi oto, że może
i ten kod ma 8 cyfr ale z racji swojego zaimplementowania potrzeba maksymalnie 11000 prób, by go
złamać. To nie jest dużo, bo można to zrobić w parę godzin, najwyżej kilka dni. W zasadzie to WPS w
oparciu o kod PIN jest bezpieczny, tylko producent routera musi w nim zastosować WPS LOCK. Jest to
blokada, która po kilku nieudanych próbach wpisania kodu PIN blokuje cały mechanizm WPS. W efekcie
nie możemy wpisać żadnego kodu PIN, bez znaczenia przy tym czy trafimy na prawidłowy. Oczywiście są
różne implementacje tej blokady. Różna jest ilość prób wyzwalająca LOCK, jak i czas, przez który
jest on aktywny. Inną formą WPS LOCK jest ręczne konfigurowanie blokady przed dodaniem oraz po
dodaniu urządzeń. W ten sposób, WPS pozostaje aktywny tylko wtedy, gdy my chcemy dodać jakieś
urządzenie do sieci. Tego typu rozwiązanie praktycznie niweluje możliwość ataku pod kątem złamania
kodu PIN w mechanizmie WPS. Jak widzimy z powyższego opisu, kod PIN może być bezpieczny, tylko
trzeba się upewnić, że producent routera zastosował blokadę, poniżej przykład:

![wps-lock-openwrt-test](/img/2015/06/1.wps-lock-openwrt-test.png#huge)

W przypadku OpenWRT ta blokada wykonaniu `hostapd` jest średnia. Z moich ustaleń wynika, że jeden
nieprawidłowy kod PIN powoduje założenie blokady na jakieś 30 sekund. Czasami mniej czasami więcej.
Nie wiem od czego to zależy. Podczas blokady nie wchodzi prawidłowy PIN. Także mechanizm WPS nie
jest wyłączany całkowicie i wykorzystanie każdego PIN'u zajmie jedynie 30 sekund, najwyżej 1 minutę.
Przy okazji czytania szeregu artykułów natrafiłem na informacje o korzystaniu z losowego PIN'u,
który jest zmieniany chyba z każdą próbą połączenia. Dokładnie nie wiem, bo coś brak dokumentacji
na ten temat.

Jeśli jednak nie przekonuje nas kod PIN, to zawsze możemy pokusić się o naciskanie przycisków (PBC,
Push Button Connect). Zwykle karta WiFi posiada taki na swojej obudowie. Router również ma przycisk
WPS na swoim wyposażeniu. W takim przypadku, by dodać urządzenie do sieci potrzebny jest fizyczny
dostęp do routera i wszelkie ataki zdalne praktycznie zostają ograniczone do zera. Nawet jeśli żadne
z tych urządzeń nie posiada fizycznego przycisku, to nie ma to większego znaczenia. Pod linux'ami, w
tym też OpenWRT, mamy możliwość generowania event'ów, które symulują wciśnięcie przycisku. Zatem
możemy programowo te przyciski wcisnąć i podłączyć się do sieci.

## Implementacja WPS na routerze

Domyślnie w OpenWRT jest zainstalowany pakiet `wpad-mini` , który nie posiada obsługi WPS. Trzeba go
odinstalować i doinstalować zamiast niego poniższe pakiety:

    # opkg remove wpad-mini
    # opkg install wpad hostapd-utils

WPS jest dostępny tylko w konfiguracji WPA2 Personal (PSK). Jeśli chcemy zaimplementować sobie
obsługę WPS (PBC/PIN), to musimy mieć odpowiednio [skonfigurowane połączenie
WiFi](/post/siec-bezprzewodowa-wifi-w-openwrt-wlan/) na routerze. Wymagane jest, by
w pliku `/etc/config/wireless` w sekcji `config wifi-iface` były obecne minimum trzy opcje:
`encryption` , `ssid` oraz `key` :

    config wifi-iface
        ...
        option ssid 'Valar_Morghulis'
        option encryption 'psk2+aes'
        option key 'pass1234'
        ...

Teraz dopisujemy jeden z tych dwóch poniższych bloków. W zależności czy chcemy mieć PBC:

    config wifi-iface
        ...
        option wps_pushbutton '0'
        option wps_config 'push_button'
        option wps_device_type '6-0050F204-1'
        option wps_device_name 'the-mountain-2'
        ...

czy PIN:

    config wifi-iface
        ...
        option wps_label '0'
        option wps_pin '12345670'
        option wps_device_type '6-0050F204-1'
        option wps_device_name 'the-mountain-2
        ...

Opcje `wps_pushbutton` oraz `wps_label` aktywują WPS na routerze. Jeśli sparowaliśmy wszystkie
urządzenia, możemy tę funkcjonalność po prostu wyłączyć przestawiając wartość z `1` na `0` . Dalej
mamy opcję `wps_device_type` , która określa typ urządzenia i tu mamy do wyboru `1-0050F204-1`
(Computer / PC), `1-0050F204-2` (Computer / Server), `5-0050F204-1` (Storage / NAS), `6-0050F204-1`
(Network Infrastructure / AP). Ostatnia opcja, tj. `wps_device_name` , to opis urządzenia.
Zapisujemy konfigurację i resetujemy połączenie bezprzewodowe wpisując w konsolę polecenie ` wifi` .
Jeśli nie korzystamy z opcji PIN, to zweryfikujmy czy w pliku `/var/run/hostapd-phy0.conf` jest
jakaś wzmianka dotycząca kodu PIN. Nie powinno jej być.

## Przycisk WPS/reset

W przypadku routera [TP-LINK Archer C7
v2](http://www.tp-link.com.pl/products/details/Archer-C7.html) mamy do dyspozycji dwa przyciski.
Jeden z nich jest od resetowania routera. Drugi zaś to przełącznik WiFi. Pod OpenWRT nie ma
domyślnie zaimplementowanej obsługi WPS w przyciskach. Nic jednak nie stoi na przeszkodzie, by taką
opcję sobie dodać. Musimy tylko nieco [zmienić konfigurację przycisków
routera](/post/konfiguracja-przyciskow-w-openwrt/). Przechodzimy zatem do katalogu
`/etc/rc.button/` i edytujemy odpowiedni plik. W przypadku tego routera jest to `wps` . Musimy
przepisać jego zawartość, tak by wyglądała mniej więcej jak ta poniżej:

    #!/bin/sh

    [ "${ACTION}" = "released" ] || exit 0

    . /lib/functions.sh

    logger "$BUTTON pressed for $SEEN seconds"

    if [ "$SEEN" -lt 2 ]; then
        # Interface for 2GHz WiFi
        iface="wlan1"

        is_wps_led=`ls -al /sys/class/leds/ | grep wps | cut -d " " -f 30`
        is_qss_led=`ls -al /sys/class/leds/ | grep qss | cut -d " " -f 30`

        if [ ! -z "$is_wps_led" ]; then
            logger "WPS at $is_wps_led"
            led="$is_wps_led"
        elif [ ! -z "$is_qss_led" ]; then
            logger "WPS at $is_qss_led"
            led="$is_qss_led"
        fi

        wps_state_iface=`hostapd_cli -i $iface get_config  2>/dev/null | grep wps_state | cut -d= -f2`
        logger $wps_state_iface

        if [ "$wps_state_iface" = "disabled" ]; then
            logger "The WPS feature is disabled, please enable it in the /etc/config/wireless file."
            exit 1
        elif [ "$wps_state_iface" = "configured" ]; then

            logger "Enabling WPS for interface: $iface"
            hostapd_cli -i $iface -p /var/run/hostapd/ wps_pbc

            echo "255" > /sys/class/leds/$led/brightness
            logger "WPS activated for 15s or until it gets a request"

            for i in 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
            do
                is_wps_active=`hostapd_cli -i $iface wps_get_status | grep "PBC Status" | cut -d " " -f 3`

                if [ "$is_wps_active" != "Active" ]; then
                        echo "0" > /sys/class/leds/$led/brightness
                fi

                sleep 1
            done

            hostapd_cli -i $iface wps_cancel
            logger "WPS has been deactivated for interface: $iface"
            echo "0" > /sys/class/leds/$led/brightness
            sleep 3
        else
            logger "WPS hasn't been activated because of some error."
            exit 1
        fi
    elif [ "$SEEN" -gt 2 ] && [ "$SEEN" -lt 5 ]; then
        # Interface for 5GHz WiFi
        iface="wlan0"

        is_wps_led=`ls -al /sys/class/leds/ | grep wps | cut -d " " -f 30`
        is_qss_led=`ls -al /sys/class/leds/ | grep qss | cut -d " " -f 30`

        if [ ! -z "$is_wps_led" ]; then
            logger "WPS at $is_wps_led"
            led="$is_wps_led"
        elif [ ! -z "$is_qss_led" ]; then
            logger "WPS at $is_qss_led"
            led="$is_qss_led"
        fi

        wps_state_iface=`hostapd_cli -i $iface get_config  2>/dev/null | grep wps_state | cut -d= -f2`

        if [ "$wps_state_iface" = "disabled" ]; then
            logger "The WPS feature is disabled, please enable it in the /etc/config/wireless file."
                exit 1
        elif [ "$wps_state_iface" = "configured" ]; then

            logger "Enabling WPS for interface: $iface"
            hostapd_cli -i $iface -p /var/run/hostapd/ wps_pbc

            echo "255" > /sys/class/leds/$led/brightness
            logger "WPS activated for 15s or until it gets a request"

            for i in 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
            do
                is_wps_active=`hostapd_cli -i $iface wps_get_status | grep "PBC Status" | cut -d " " -f 3`

                if [ "$is_wps_active" != "Active" ]; then
                        echo "0" > /sys/class/leds/$led/brightness
                fi

                sleep 1
            done

            hostapd_cli -i $iface wps_cancel
            logger "WPS has been deactivated for interface: $iface"
            echo "0" > /sys/class/leds/$led/brightness
            sleep 3
        else
            exit 1
        fi
    elif [ "$SEEN" -gt 5 -a "$SEEN" -lt 10 ]
    then
        echo "REBOOT" > /dev/console
        /etc/init.d/statistics stop
        sync
        reboot

    elif [ "$SEEN" -gt 10 ]
    then
        echo "FACTORY RESET" > /dev/console
        jffs2reset -y && reboot &
    fi

    return 0

Skrypt obszerny bo i sporo akcji jest przypisanych do tego przycisku `wps` . Mamy tam w sumie
cztery, z czego dwie (restart i
[firstboot](/post/reset-ustawien-w-openwrt-firstboot/)) są standardowe. My
dodaliśmy tam te dwa pierwsze bloki. Kluczowe w nich jest ustawienie zmiennej `iface` , która
odpowiada za interfejs bezprzewodowy. Oczywiście w powyższym przypadku mamy dwa radia: 2,4 GHz oraz
5 GHz. Sporo routerów nie obsługuje dwóch pasm, zatem jeden z tych dwóch pierwszych bloków można
usunąć.

Po wciśnięciu przycisku `wps` sprawdzane są kolejno warunki. W przypadku, gdy WPS jest włączony w
konfiguracji WiFi w pliku `/etc/config/wireless` , zostaje on aktywowany i zapalana jest dioda WPS
na routerze. Następnie przez 15 sekund (co 1 sekundę) sprawdzana jest aktywność WPS. Można sobie
dostosować zarówno interwał jak i długość okresu ale nie więcej niż 120 sekund. Po tym jak WPS
został aktywowany, mogą wystąpić dwa zdarzenia. WPS może przejść w stan spoczynku z powodu wybicia
zegara. Zwykle ma to miejsce, gdy urządzenia nie zdążyły się sparować w czasie przeznaczonym dla
tego procesu. Może także się zdarzyć tak, że parowanie urządzeń przebiegnie w krótszym czasie niż
ustawione wyżej 15 sekund. W obu przypadkach nie potrzebujemy, by WPS był dalej aktywny i może on
zostać wyłączony. Następnie gaszona jest również dioda sygnalizując tym samym, że WPS nie jest już
dłużej aktywny.

## Konfiguracja WPS na kliencie

Praktycznie wszystkie systemy linux'owe wykorzystują do konfiguracji połączenia bezprzewodowego
narzędzie `wpasupplicant` dostępnego w pakiecie pod tą samą nazwą. Z tym, że sprawa dotyczy tych
maszyn, które nie używają automatów w stylu
[network-manager](https://wiki.gnome.org/Projects/NetworkManager) czy
[wicd](https://launchpad.net/wicd). Tam raczej nie powinno być problemów ze skonfigurowaniem sieci
WiFi na stacjach klienckich. Jeśli jednak nie korzystamy z tych powyższych automatów, to musimy
utworzyć plik `/etc/wpa_supplicant/wpa_supplicant.conf` i wrzucić do niego poniższą linijkę:

    update_config=1

Powyższy parametr nakazuje narzędziu `wpasupplicant` , by zaktualizował ten plik o wartości, które
otrzyma za sprawą protokołu WPS. Po uzupełnieniu tego pliku, połączenie zostanie ustanowione w
oparciu o te dane. Jedyny problem jaki jest z powyższą opcją, to taki, że czyści ona wszelkie
komentarze jakie mamy w tym pliku.

Można też zrezygnować z pliku `/etc/wpa_supplicant/wpa_supplicant.conf` całkowicie, tylko wtedy za
każdym razem, gdy będziemy się podłączać do sieci, trzeba będzie wciskać przyciski, by na nowo
powiązać urządzenia.

Teraz jeszcze potrzebujemy konfiguracji dla interfejsu sieciowego w pliku
`/etc/network/interfaces` :

    auto wlan1
    allow-hotplug wlan1
    iface wlan1 inet dhcp
          wpa-driver nl80211
          wpa-debug-level -1
          wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

## Parowanie urządzeń

Podłączamy kartę do PC i skanujemy eter w poszukiwaniu sieci:

    # wpa_cli scan_results
    Selected interface 'wlan1'
    bssid / frequency / signal level / flags / ssid
    ...
    e8:94:f6:68:79:f0       2447    -33     [WPA2-PSK-CCMP][WPS-PBC][ESS]       Winter Is Coming
    ...

Jak widzimy wyżej, `[WPS-PBC]`sugeruje, że opcja WPS jest aktywowana oraz, że korzystamy z
przycisków. W przypadku gdyby tam widniał sam `[WPS]` , oznaczałoby to, że wykorzystywany jest PIN.
Dlatego też dobrze jest zwrócić na ten szczegół uwagę po naciśnięciu przycisku WPS.

Ze względów bezpieczeństwa dobrze jest najpierw wciskać przycisk na kliencie, a dopiero na routerze.
Jeśli router wykryje co najmniej dwa klienty oczekujące na konfigurację przez protokół WPS, żaden z
nich jej nie otrzyma po wciśnięciu przycisku na routerze. Tego typu zachowanie ma zapobiec wyciekowi
poufnych danych na wypadek, gdyby ktoś czekał na naciśnięcie przez nas przycisku.

### WPS PBC bez przycisków

Naturalnie nie musimy przyciskać przycisków na obudowach, by sparować urządzenia. Tę akcję możemy
zwyczajnie zasymulować. W tym celu, na maszynie klienckiej wydajemy poniższe polecenie:

    # wpa_cli wps_pbc
    Selected interface 'wlan1'
    OK

Na routerze również możemy zasymulować wciśnięcie przycisku:

    # hostapd_cli wps_get_status
    Selected interface 'wlan0'
    PBC Status: Disabled
    Last WPS result: None

    # hostapd_cli wps_pbc
    Selected interface 'wlan0'
    OK

    # hostapd_cli wps_get_status
    Selected interface 'wlan0'
    PBC Status: Active
    Last WPS result: None

Czas przewidziany na parowanie urządzeń to równo dwie minuty. Podczas tego okresu musimy wcisnąć
przyciski na drugim urządzeniu. Jeśli nie zmieścimy się w tym przedziale, na routerze będziemy mogli
odczytać poniższą wiadomość:

    # hostapd_cli wps_get_status
    Selected interface 'wlan0'
    PBC Status: Timed-out
    Last WPS result: None

Po rozpoczęciu procesu parowania, diody na karcie oraz routerze powinny zacząć migać. Po chwili
powinien zostać uzupełniony plik `/etc/wpa_supplicant/wpa_supplicant.conf` . Sprawdźmy czy tak się
stało:

    ctrl_interface=/var/run/wpa_supplicant
    update_config=1

    network={
          ssid="Winter Is Coming"
          psk=krotszego-hasla-to-juz-nie-bylo?
          proto=RSN
          key_mgmt=WPA-PSK
          pairwise=CCMP
          auth_alg=OPEN
    }

Mamy zatem skonfigurowane parametry dla sieci WiFi. Teraz tylko zresetujmy interfejs sieciowy by
dostać adres IP:

    # ifdown wlan1

    # ifup wlan1
    ...
    DHCPACK from 192.168.1.1
    bound to 192.168.1.207 -- renewal in 36711 seconds.

### Stan spoczynku WPS

Po dokonaniu konfiguracji, WPS automatycznie przechodzi w stan spoczynku i drugie urządzenie, które
by chciało skorzystać i potajemnie wtargnąć do naszej sieci, nie będzie miało takiej możliwości.
Jeśli wydamy poniższe polecenie na routerze:

    # hostapd_cli wps_get_status
    Selected interface 'wlan0'
    PBC Status: Disabled
    Last WPS result: Success
    Peer Address: e8:94:f6:1e:15:e9

Możemy dostrzec `Peer Address`. Dobrze jest zweryfikować tego MACa ale nawet bez tego będzie wiadomo
czy komuś przez przypadek udało się uzyskać dostęp do naszej sieci via WPS, bo wtedy my go
zwyczajnie nie uzyskamy.

### Przerwanie procesu parowania urządzeń

W przypadku gdybyśmy przez przypadek aktywowali funkcję WPS, istnieje możliwość ręcznego przerwania
procesu parowania urządzeń i nie trzeba czekać 2 minuty aż ten proces się zakończy sam z siebie.
Wystarczy na routerze wpisać poniższe polecenie:

    # hostapd_cli wps_cancel
    Selected interface 'wlan0'
    OK

## A co z PIN'em?

Jeśli ktoś jednak chciałby korzystać z PIN'u, to może naturalnie z niego korzystać, tylko przydałoby
się pierw ustalić czy nasz router posiada mechanizm zabezpieczający przed wprowadzaniem zbyt wielu
błędnych kodów PIN pod rząd. Jeśli tak, to na jaki czas zostaje zablokowana funkcja WPS. Można to
ustalić posługując się narzędziem `reaver` . Nie będę tutaj opisywał całego procesu, jedynie to w
jaki sposób ustalić czas WPS LOCK:

    # reaver -i mon0 -b E8:94:F6:68:79:F0 -vv -c 8 --lock-delay 10
    # reaver -i mon0 -b E8:94:F6:68:79:F0 -vv -c 8 --lock-delay 30
    # reaver -i mon0 -b E8:94:F6:68:79:F0 -vv -c 8 --lock-delay 50
    # reaver -i mon0 -b E8:94:F6:68:79:F0 -vv -c 8 --lock-delay 60
    # reaver -i mon0 -b E8:94:F6:68:79:F0 -vv -c 8 --lock-delay 50

Dostrajamy odpowiednio parametr `--lock-delay` zwiększając czas, a ten jest w sekundach. Jeśli będą
wyrzucane komunikaty o zablokowaniu WPS, trzeba zwiększyć odstępy czasu, aż do chwili, gdy nie
będziemy łapać się na WPS LOCK. Trzeba jednak pamiętać, że nawet w przypadku WPS LOCK ustawionego
na 60 sekund na przeskanowanie 11000 możliwości, potrzeba 660000 sekund, czyli nieco ponad 7 dni.
Biorąc pod uwagę średnią, złamanie PIN'u potrwa maksymalnie 4 dni.
