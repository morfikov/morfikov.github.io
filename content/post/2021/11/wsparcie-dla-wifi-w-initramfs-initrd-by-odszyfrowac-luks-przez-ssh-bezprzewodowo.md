---
author: Morfik
categories:
- RaspberryPi
date:    2021-11-13 13:26:00 +0100
lastmod: 2021-11-13 13:26:00 +0100
published: true
status: publish
tags:
- luks
- raspberry-pi-4b
- raspios
- raspbian
- szyfrowanie
- ssh
- initramfs
- initrd
- wifi
- firmware
GHissueID: 582
title: Wsparcie dla WiFi w initramfs/initrd by odszyfrować LUKS przez SSH bezprzewodowo
---

W poprzednim artykule, który traktował o [odszyfrowaniu kontenera LUKS przez SSH z poziomu
initramfs/initrd na Raspberry Pi][1], została poruszona kwestia adresacji IP, która w opisanym tam
rozwiązaniu miała pewne ograniczenia. Chodziło o to, że połączenie SSH do RPI mogło być realizowane
jedynie przez przewodowy interfejs sieciowy `eth0` . Trzeba było zatem się zastanowić nad
rozwiązaniem, które umożliwiłoby korzystanie również z bezprzewodowego interfejsu WiFi, tj.
`wlan0` . Celem niniejszego wpisu jest pokazanie w jaki sposób można dorobić wsparcie dla
połączeń WiFi w naszej malinie, tak by szło odszyfrować kontener LUKS przez SSH, w sytuacji gdy z
jakiegoś powodu nie chcemy lub też nie możemy korzystać z przewodowego interfejsu sieciowego tego
minikomputera z zainstalowanym system RasPiOS/Raspbian.

<!--more-->
## Problematyczne wsparcie dla WiFi w initramfs/initrd

Te nieco bardziej spostrzegawcze osobniki zapewne zdążyły już zauważyć, że w przypadku przewodowego
interfejsu sieciowego nie trzeba było za bardzo zmieniać konfiguracji obrazu initramfs/initrd. No i
faktycznie połączenie przewodowe w tym stadium rozruchu systemu działa w zasadzie OOTB. W przypadku
interfejsów bezprzewodowych trzeba się będzie trochę wysilić, a to ze względu na fakt, że w obrazie
initramfs/initrd brakuje oprogramowania odpowiedzialnego za zestawienie połączenia WiFi.

W dystrybucjach bazujących na Debianie, w tym też RasPiOS/Raspbian, oprogramowaniem odpowiedzialnym
za konfigurację połączenia WiFi jest pakiet `wpasupplicant` , który jest domyślnie zainstalowany w
Raspberry Pi. Niemniej jednak, binarki pokroju `wpa_supplicant` czy `wpa_cli` to nie wszystko, by
komunikacja bezprzewodowa mogła zostać ustanowiona. Potrzebne są sterowniki i firmware od tego
modelu karty WiFi, który siedzi w naszym RPI. Dodatkowo trzeba też określić parametry dla
połączenia WiFi, takie jak ESSID czy hasło, które zwykle trzymane są w pliku
`/etc/wpa_supplicant.conf` .

O ile te wszystkie wyżej wymienione rzeczy są standardowo dostępne w działającym systemie
RasPiOS/Raspbian, o tyle żadna z nich nie jest dostępna w obrazie initramfs/initrd, przez co
połączenie WiFi na tym etapie startu systemu nam zwyczajnie działać nie może. Dobre wieści są
jednak takie, że skoro WiFi działa nam po uruchomieniu systemu, to możne też działa w fazie
initramfs/initrd. Trzeba tylko te wyżej wypisane komponenty zapakować do tego obrazu i odpowiednio
skonfigurować.

Chciałbym tutaj wyraźnie zaznaczyć, że informacje na temat tego [jak skonfigurować usługę SSH, tak
by działała ona w fazie initramfs/initrd na Raspberry Pi][1] znajdują się w osobnym artykule i
nie będę tutaj tego zagadnienia opisywał ponownie. Dalsza część artykułu zakłada, że jesteśmy w
stanie podłączyć się po SSH z wykorzystaniem przewodowego interfejsu sieciowego w naszej malinie.

## Konfiguracja obrazu initramfs/initrd na potrzeby WiFi

Szukając informacji na temat tego jak włączyć wsparcie dla WiFi w obrazie initramfs/initrd
natrafiłem na [ten oto dość przyzwoicie napisany post][2]. Problem z przedstawionym w tym poście
rozwiązaniem jest jednak taki, że po uruchomieniu się systemu (do momentu, w którym trzeba
odszyfrować kontener LUKS) wcina nam całkowicie interfejs WiFi. Jest to efektem braku dołączenia do
obrazu initramfs/initrd plików z firmware, których wymaga karta WiFi do pracy. Postanowiłem zatem w
oparciu o tego podlinkowanego wyżej posta naskrobać nieco bardziej aktualną wersję opisanego tam
rozwiązania, które będzie działać na Raspberry Pi 4B i być może też i innych malinach.

### Plik /etc/initramfs-tools/scripts/init-premount/a_enable_wireless

W katalogu `/etc/initramfs-tools/scripts/init-premount/` tworzymy plik `a_enable_wireless` .
Nadajemy mu również prawa wykonywania:

    root@raspberrypi:~# touch /etc/initramfs-tools/scripts/init-premount/a_enable_wireless
    root@raspberrypi:~# chmod +x /etc/initramfs-tools/scripts/init-premount/a_enable_wireless

Następnie wrzucamy do tego pliku poniższą zawartość, odpowiednio zmieniając wartości w zmiennych
`AUTH_LIMIT=` oraz `INTERFACE=` :

    #!/bin/sh
    # this goes into /etc/initramfs-tools/scripts/init-premount/a_enable_wireless
    PREREQ=""
    prereqs()
    {
        echo "$PREREQ"
    }

    case $1 in
    prereqs)
        prereqs
        exit 0
        ;;
    esac

    . /scripts/functions

    AUTH_LIMIT=30
    INTERFACE="wlan0"
    alias WPACLI="/sbin/wpa_cli -p/tmp/wpa_supplicant -i$INTERFACE "

	log_begin_msg "Setting WLAN regdomain"
	# Set this to your region
	/sbin/iw reg set PL

    log_begin_msg "Starting WLAN connection"
    sleep 1 && /sbin/wpa_supplicant  -i$INTERFACE -c/etc/wpa_supplicant.conf -P/run/initram-wpa_supplicant.pid -B -f /tmp/wpa_supplicant.log

    # Wait for AUTH_LIMIT seconds, then check the status
    limit=${AUTH_LIMIT}

    echo -n "Waiting for connection (max ${AUTH_LIMIT} seconds)"
    while [ $limit -ge 0 -a `WPACLI status | grep wpa_state` != "wpa_state=COMPLETED" ]
    do
        sleep 1
        echo -n "."
        limit=`expr $limit - 1`
    done
    echo ""

    if [ `WPACLI status | grep wpa_state` != "wpa_state=COMPLETED" ]; then
      ONLINE=0
      log_failure_msg "WLAN offline after timeout"
      panic
    else
      ONLINE=1
      log_success_msg "WLAN online"
    fi

    configure_networking

Celem tego skrypt jest podłączenia nas do określonej sieci WiFi. W moim przypadku przy
wykorzystaniu oryginalnego skryptu pojawił się błąd: `Failed to connect to non-global ctrl_ifname
error: No such file or directory` , co można zaobserwować na poniższej fotce:

![raspberry-pi-initramfs-initrd-ssh-wifi-luks-error](/img/2021/11/001.raspberry-pi-initramfs-initrd-ssh-wifi-luks-error.jpg#huge)

Efektem naturalnie był brak połączenia WiFi w Raspberry Pi, przez co nie szło się zalogować do
systemu po SSH. Natomiast rozwiązaniem tego problemu okazało się opóźnienie wywołania polecenia
`/sbin/wpa_supplicant` przy pomocy `sleep 1 &&` .

### Plik /etc/initramfs-tools/hooks/enable-wireless

W katalogu `/etc/initramfs-tools/hooks/` tworzymy plik `enable-wireless` i nadajemy mu prawa
wykonywania:

    root@raspberrypi:~# touch /etc/initramfs-tools/hooks/enable-wireless
    root@raspberrypi:~# chmod +x /etc/initramfs-tools/hooks/enable-wireless

Następnie wrzucamy do tego pliku poniższą zawartość, odpowiednio dostosowując moduły, których
wymaga karta WiFi (w tym przypadku `brcmfmac` ):

    # !/bin/sh
    # This goes into /etc/initramfs-tools/hooks/enable-wireless
    set -e
    PREREQ=""
    prereqs()
    {
        echo "${PREREQ}"
    }
    case "${1}" in
        prereqs)
            prereqs
            exit 0
            ;;
    esac

    . /usr/share/initramfs-tools/hook-functions

    # CHANGE HERE for your correct modules.
    manual_add_modules brcmfmac

    copy_exec /sbin/wpa_supplicant
    copy_exec /sbin/wpa_cli
    copy_exec /sbin/iw

    copy_file config /etc/initramfs-tools/wpa_supplicant.conf /etc/wpa_supplicant.conf

    #copy_file firmware /lib/firmware/brcm/brcmfmac43455-sdio.bin
    copy_file firmware /lib/firmware/brcm/brcmfmac43455-sdio.clm_blob
    copy_file firmware /lib/firmware/brcm/brcmfmac43455-sdio.txt

	copy_file config /lib/crda/regulatory.bin
	copy_file config /lib/firmware/regulatory.db
	copy_file config /lib/firmware/regulatory.db.p7s

Ten plik różni się trochę od oryginału. Wszystko dlatego, że gdy próbowałem uruchomić system na
oryginalnej wersji tego pliku, to w systemie nie został wykryty interfejs `wlan0` , a w logu można
było zaobserwować poniższy komunikat:

	raspberrypi kernel: brcmfmac: F1 signature read @0x18000000=0x15264345
	raspberrypi kernel: brcmfmac: brcmf_fw_alloc_request: using brcm/brcmfmac43455-sdio for chip BCM4345/6
	raspberrypi kernel: brcmfmac mmc1:0001:1: Direct firmware load for brcm/brcmfmac43455-sdio.txt failed with error -2
	raspberrypi kernel: Console: switching to colour frame buffer device 160x64
	raspberrypi kernel: vc4-drm gpu: [drm] fb0: vc4drmfb frame buffer device
	raspberrypi kernel: brcmfmac: brcmf_sdio_htclk: HT Avail timeout (1000000): clkctl 0x50
	raspberrypi kernel: usbcore: registered new interface driver brcmfmac

W przypadku mojego Raspberry Pi 4B, mamy do czynienia z kartą Broadcom, która wymaga do pracy
modułu kernela `brcmfmac` . Gdy się zajrzy w informacje o tym module, to zobaczymy listę plików
firmware, które ten moduł może chcieć zażądać. Na tej liście są uwzględnione firmware do różnych
kart WiFi ale nas w tym momencie interesuje jedynie brakujący plik `brcm/brcmfmac43455-sdio.txt` :

    root@raspberrypi:/tmp# modinfo -F firmware /lib/modules/5.10.63-v7l+/kernel/drivers/net/wireless/broadcom/brcm80211/brcmfmac/brcmfmac.ko
    ...
    brcm/brcmfmac43455-sdio.bin
    ...

Jak widać, jedyny plik mający w nazwie `brcm/brcmfmac43455-sdio` , to
`brcm/brcmfmac43455-sdio.bin` , a potrzebnego nam pliku `brcm/brcmfmac43455-sdio.txt` nie ma już na
tej liście. Dlatego też polecenie `manual_add_modules` , z którego korzystaliśmy w powyższym
skrypcie, nie było w stanie skopiować wszystkich potrzebnych plików do obrazu initramfs/initrd.

Jeśli zajrzymy do katalogu `/lib/firmware/` , to zastaniemy tam te trzy poniższe pliki zamiast
jednego:

    root@raspberrypi:~# ls -al /lib/firmware/brcm/brcmfmac43455*
    -rw-r--r-- 1 root root 631467 Nov  1 23:07 /lib/firmware/brcm/brcmfmac43455-sdio.bin
    -rw-r--r-- 1 root root   7163 Nov  1 23:07 /lib/firmware/brcm/brcmfmac43455-sdio.clm_blob
    -rw-r--r-- 1 root root   2074 Nov  1 23:07 /lib/firmware/brcm/brcmfmac43455-sdio.txt

Wszystkie te pliki firmware trzeba skopiować do obrazu initramfs/initrd, by karta WiFi w Raspberry
Pi działała prawidłowo. Możemy naturalnie te wszystkie pliki skopiować do obrazu via `copy_file
firmware` ale lepszym rozwiązaniem jest pozostawić `manual_add_modules` i skopiować jedynie
brakujące pliki. Chodzi generalnie o to, że `manual_add_modules` [bierze pod uwagę zależności
modułów][4], czego efektem będzie skopiowanie wszystkich niezbędnych modułów do obrazu
initramfs/initrd. Jeśli jednak chcielibyśmy ręcznie kopiować wszystkie rzeczy, to zależności
modułów możemy wyciągnąć w poniższy sposób:

    root@raspberrypi:~# modprobe --show-depends --ignore-install brcmfmac
    insmod /lib/modules/5.10.63-v7l+/kernel/net/rfkill/rfkill.ko
    insmod /lib/modules/5.10.63-v7l+/kernel/net/wireless/cfg80211.ko
    insmod /lib/modules/5.10.63-v7l+/kernel/drivers/net/wireless/broadcom/brcm80211/brcmutil/brcmutil.ko
    insmod /lib/modules/5.10.63-v7l+/kernel/drivers/net/wireless/broadcom/brcm80211/brcmfmac/brcmfmac.ko

Ewentualne pliki z firmware wymagane przez te moduły również trzeba uwzględnić w obrazie
initramfs/initrd.

Ja jeszcze dodatkowo skopiowałem sobie [bazę danych regulacji WiFi][3], tj. pliki `regulatory.db*` ,
które kernel zwykle żąda przy konfiguracji połączeń bezprzewodowych.

### Plik /etc/initramfs-tools/scripts/local-bottom/kill_wireless

W katalogu `/etc/initramfs-tools/scripts/local-bottom/` tworzymy skrypt `kill_wireless` i nadajemy
mu prawa wykonywania:

    root@raspberrypi:~# touch /etc/initramfs-tools/scripts/local-bottom/kill_wireless
    root@raspberrypi:~# chmod +x /etc/initramfs-tools/scripts/local-bottom/kill_wireless

Następnie wrzucamy do tego pliku poniższą zawartość:

    #!/bin/sh
    # this goes into /etc/initramfs-tools/scripts/local-bottom/kill_wireless
    PREREQ=""
    prereqs()
    {
        echo "$PREREQ"
    }

    case $1 in
    prereqs)
        prereqs
        exit 0
        ;;
    esac

    echo "Killing wpa_supplicant so the system takes over later."
    kill `cat /run/initram-wpa_supplicant.pid`

Zadaniem tego skryptu jest zerwanie połączenia WiFi po pomyślnym odszyfrowaniu kontenera LUKS. W
późniejszym czasie startu systemu, Raspberry Pi ponownie się podłączy do WiFi, gdy już system się w
pełni zainicjuje.

### Plik /etc/initramfs-tools/wpa_supplicant.conf

W katalogu `/etc/initramfs-tools/` tworzymy plik `wpa_supplicant.conf` , w którym będzie
przechowywana konfiguracja sieci WiFi jedynie na potrzeby fazy initramfs/initrd:

    root@raspberrypi:~# touch /etc/initramfs-tools/wpa_supplicant.conf

Następnie wrzucamy do tego pliku poniższą zawartość:

    # sample /etc/initramfs-tools/wpa_supplicant.conf
    # note that this is independent of the system /etc/wpa_supplicant.conf (if any)
    # only add the network you need at boot time. **And keep the ctrl_interface** !!
    ctrl_interface=/tmp/wpa_supplicant
    country=PL

    network={
        ssid="Ever_Vigilant"
        scan_ssid=1
        psk="DontLookAtMe"
        key_mgmt=WPA-PSK
    }

### Plik /etc/initramfs-tools/initramfs.conf

W pliku `/etc/initramfs-tools/initramfs.conf` musimy jedynie zmienić interfejs z przewodowego na
bezprzewodowy, tak by system skonfigurował na nim adresację IP:

Dla statycznej adresacji IP:

    IP="192.168.1.239::192.168.1.1:255.255.255.0:rpi:wlan0:none"

Dla dynamicznej adresacji IP via DHCP:

    IP="::::rpi:wlan0:dhcp"

## Regeneracja obrazu initramfs/initrd

Po dostosowaniu tych wszystkich powyższych plików musimy wygenerować nowy obraz initramfs/initrd
przy pomocy polecenia `update-initramfs` :

    root@raspberrypi:~# update-initramfs -u -k $(uname -r)
    ln: failed to create hard link '/boot/initrd.img-5.10.63-v7l+.dpkg-bak' => '/boot/initrd.img-5.10.63-v7l+': Operation not permitted
    update-initramfs: Generating /boot/initrd.img-5.10.63-v7l+
    Building v7l+ image, updating top initrd

## Test połączenia SSH przez WiFi z poziomu initramfs/initrd

Pozostało nam już tylko uruchomić Raspberry Pi ponownie i sprawdzić czy w fazie initramfs/initrd
adresacja IP na interfejsie `wlan0` jest przydzielana oraz czy jesteśmy w stanie bez problemu
nawiązać połączenie SSH z RPI:

    $ ssh 192.168.1.239
    Please unlock disk rpi_crypt:

Połączenie zostało ustanowione i pojawił się prompt z prośbą o hasło by odszyfrować kontener LUKS.

Patrząc na monitor podłączony do Raspberry Pi, mamy taki oto obrazek chwilę przed wpisaniem hasła
do kontenera:

![raspberry-pi-initramfs-initrd-ssh-wifi-luks-ip-config](/img/2021/11/002.raspberry-pi-initramfs-initrd-ssh-wifi-luks-ip-config.jpg#huge)

Komunikat `Success: "WLAN online"` świadczy o fakcie podłączenia do zdefiniowanej sieci WiFi.
Adresacja na interfejsie `wlan0` również została skonfigurowana. Po wpisaniu prawidłowego hasła
przez SSH, kontener LUKS powinien zostać odszyfrowany, a system powinien kontynuować procedurę
startu w tradycyjny sposób:

![raspberry-pi-initramfs-initrd-ssh-wifi-luks-system-boot](/img/2021/11/003.raspberry-pi-initramfs-initrd-ssh-wifi-luks-system-boot.jpg#huge)

Jako, że mam na swoim domowym routerze WiFi wgrany firmware OpenWRT, to mogę podejrzeć o czym te
dwa urządzenia ze sobą rozmawiają. Chwilę po uruchomieniu Raspberry Pi ale przed otworzeniem
kontenera w logu routera można zarejestrować poniższe komunikaty:

    hostapd: wifi2g: STA dc:a6:32:af:5b:03 IEEE 802.11: authenticated
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 IEEE 802.11: associated (aid 2)
    hostapd: wifi2g: AP-STA-CONNECTED dc:a6:32:af:5b:03
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 RADIUS: starting accounting session 49BEFF3343C481BD
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 WPA: pairwise key handshake completed (RSN)

Zatem Raspberry Pi uwierzytelnia się i podłącza bez problemu do routera po WiFi.

Po wpisaniu hasła do kontenera LUKS via SSH, po chwili w logu na routerze pojawiają się takie oto
komunikaty:

    hostapd: wifi2g: AP-STA-DISCONNECTED dc:a6:32:af:5b:03
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 IEEE 802.11: disassociated
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 IEEE 802.11: deauthenticated due to inactivity (timer DEAUTH/REMOVE)

Mamy tutaj informację, żę Raspberry Pi rozłącza połączenie WiFi po odszyfrowaniu kontenera LUKS,
czyli dokładnie takie zachowanie, o jakie nam chodziło.

Po chwili w logu pojawiają się też te poniższe komunikaty:

    hostapd: wifi2g: STA dc:a6:32:af:5b:03 IEEE 802.11: authenticated
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 IEEE 802.11: associated (aid 2)
    hostapd: wifi2g: AP-STA-CONNECTED dc:a6:32:af:5b:03
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 RADIUS: starting accounting session FA7A4354549318AB
    hostapd: wifi2g: STA dc:a6:32:af:5b:03 WPA: pairwise key handshake completed (RSN)
    dnsmasq-dhcp[3510]: DHCPREQUEST(br-lan) 192.168.1.239 dc:a6:32:af:5b:03
    dnsmasq-dhcp[3510]: DHCPACK(br-lan) 192.168.1.239 dc:a6:32:af:5b:03 RaspberryPi

Czego efektem naturalnie jest ponowne uzyskanie przez Raspberry Pi połączenia drogą bezprzewodową
z domowym routerem WiFi i tym razem to router przydziela RPI adresację IP przy wykorzystaniu
protokołu DHCP.

## Podsumowanie

Implementując wsparcie dla połączeń WiFi w fazie initramfs/initrd usunęliśmy kluczowy problem, z
jakim mogą zmagać się osoby próbujące otworzyć kontener LUKS w tej fazie korzystając z połączenia
SSH. Możemy teraz sobie wybrać czy połączenia do Raspberry Pi mają być realizowane przewodowo, czy
też bezprzewodowo. Opisane tutaj rozwiązanie nie jest specyficzne dla Raspberry Pi i można je
wykorzystać również na innych dystrybucjach linux'a.


[1]: /post/odszyfrowanie-luks-przez-ssh-z-poziomu-initramfs-initrd-na-raspberry-pi/
[2]: https://gist.github.com/telenieko/d17544fc7e4b347beffa87252393384c
[3]: https://www.kernel.org/doc/html/latest/networking/regulatory.html
[4]: https://manpages.debian.org/stretch/initramfs-tools-core/initramfs-tools.8.en.html
