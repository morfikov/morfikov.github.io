---
author: Morfik
categories:
- Linux
date: "2016-12-11T15:59:06Z"
date_gmt: 2016-12-11 14:59:06 +0100
published: true
status: publish
tags:
- ssh
- prywatność
- vpn
- debian
title: Jak skonfigurować połączenie VPN przez SSH
---

Szukając informacji na [temat ukrycia ruchu generowanego przez OpenVPN][1], natrafiłem także na
sposób, który [wykorzystuje do tego celu połączenie SSH][2]. W efekcie jesteśmy w stanie upodobnić
ruch VPN do tego, który zwykle służy do zarządzania zdalnymi systemami linux. Jako, że temat
maskowania połączenia VPN jest kluczowy w walce z cenzurą internetu, to im więcej sposobów, by taki
zabieg przeprowadzić, tym lepiej. Dlatego też postanowiłem odświeżyć nieco podlinkowany wyżej
artykuł i sprawdzić czy jest on jeszcze aktualny. Wprawdzie nie dysponuję Ubuntu, a jedynie
dystrybucją Debian ale raczej nie powinno być problemów z odwzorowaniem konfiguracji na tym
systemie, choć artykuł jest dość leciwy już i pewnie trzeba będzie kilka rzeczy zaktualizować.

<!--more-->
## Konfiguracja SSH pod tunel VPN

Począwszy od wersji 4.3, OpenSSH zyskał ciekawą zdolność tworzenia VPN w locie wykorzystując do tego
celu interfejsy `tun` . Operowanie na interfejsach `tun` wymaga od nas posiadania praw
administratora systemu zarówno na kliencie jak i na serwerze. Musimy zatem tak dostosować
konfigurację demona `ssh` , by akceptował żądanie tunelu VPN. Niemniej jednak, ze względu na fakt
tworzenia interfejsów `tun` po stronie serwera, musimy także umożliwić zalogowanie się na
użytkownika root na serwerze. Nie jest to jednak bezpieczne rozwiązanie, chyba, że zaimplementujemy
na serwerze SSH obsługę [uwierzytelniających kluczy SSH][3]. Wtedy będziemy mogli zrezygnować z
wykorzystywania haseł, co znacząco podwyższy poziom zabezpieczeń usługi.

Zakładając, że dysponujemy już odpowiednimi kluczami SSH, logujemy się na serwer i w pliku
`/etc/ssh/sshd_config` dopisujemy te dwa parametry:

    PermitRootLogin without-password
    PermitTunnel yes

Następnie restartujemy usługę:

    # systemctl restart ssh.service

Teraz, by utworzyć interfejsy `tun` pod tunel VPN wystarczy wydać na kliencie poniższe polecenie:

    $ sudo ssh -w 9:9 root@11.22.33.44

Po pomyślnym zalogowaniu się na serwer z wykorzystaniem tego polecenia, na obu krańcach połączenia
powinniśmy mieć już dostępne interfejsy `tun9` . Możemy to sprawdzić za pomocą polecenia `ip` lub
`ifconfig` . Niemniej jednak, te interfejsy jeszcze nic nie robią, bo ani nie mamy ustawionej na
nich adresacji, ani nie skonfigurowaliśmy tablicy routingu, ani też nie ustawiliśmy forwarding'u i
nie dostosowaliśmy reguł zapory sieciowej.

## Konfiguracja interfejsów tun na potrzeby SSH-VPN

By interfejsy `tun` były w stanie przesyłać pakiety, musimy na nich skonfigurować adresację IP. W
przypadku standardowego OpenVPN, ta czynność jest przeprowadzana automatycznie bez udziału
użytkownika. W przypadku SSH-VPN musimy niestety ręcznie te interfejsy skonfigurować i to zarówno
po stronie serwera jak i po stronie klienta.

### Konfiguracja SSH-VPN po stronie klienta

Zacznijmy może od konfiguracji na kliencie. Edytujemy plik `/etc/network/interfaces` i dodajemy w
nim poniższy blok. Pamiętajmy tylko o odpowiednim dostosowaniu numerów interfejsu `tun` :

    iface tun9 inet static
        pre-up ssh -S /var/run/vpn-tunnel-control -M -f -w 9:9 root@11.22.33.44 -p 22 'ifdown tun9; ifup tun9'
        pre-up sleep 5
        address 10.100.0.2
        pointopoint 10.100.0.1
        netmask 255.255.255.255
        up ip route add 10.100.0.0/24 via 10.100.0.2
        up ip route add 11.22.33.44/32 via $(ip route show | awk '$1 =="default" {print $3}')
        up ip route add 0.0.0.0/1 via 10.100.0.1 dev $IFACE
        up ip route add 128.0.0.0/1 via 10.100.0.1 dev $IFACE
        down ip route del 10.100.0.0/24 via 10.100.0.2
        down ip route del 11.22.33.44/32 via $(ip route show | awk '$1 =="default" {print $3}')
        down ip route del 0.0.0.0/1 via 10.100.0.1 dev $IFACE
        down ip route del 128.0.0.0/1 via 10.100.0.1 dev $IFACE
        post-down ssh -S /var/run/vpn-tunnel-control -O exit root@11.22.33.44 -p 22

Dzięki tej powyższej zwrotce, zestawienie tunelu będzie sprawdzać się jedynie do wydania polecenia
`ifup tun9` . Linijki z `pre-up` mają za zadanie nawiązać połączenie z serwerem SSH. Generalnie
rzecz biorąc, tutaj mamy polecenia shell'owe, które byśmy zwyczajnie wpisali w terminalu przy
ręcznej próbie skonfigurowania takiego tunelu SSH-VPN. Prawdopodobnie trzeba będzie dostosować
sobie wartość `sleep` , która ma na celu dać trochę czasu na nawiązanie tego połączenia z drugą
stroną. Dopiero po nawiązaniu połączenia, w systemie będziemy mieć interfejs `tun9` . Dalej mamy
standardową konfigurację interfejsu, tj. adres, maskę, no i w przypadku interfejsów `tun` również
adres drugiego końca połączenia. Wpisy z `up` mają na celu skonfigurowanie tras routingu, tak by
cały ruch z klienta przesłać właśnie na interfejs `tun9` , czyli wrzucić go w tunel SSH. Linijki z
`down` z kolei mają na celu te trasy usunąć przy rozłączaniu połączenia VPN. Z kolei ostatnia
linijka z `post-down` ma za zadanie rozłączyć połączenie SSH.

Polecenie, które zostało użyte do zestawienia połączenia SSH może wydać się ździebko skomplikowane
na pierwszy rzut oka. Opcje `-S` i `-M` włączają możliwość korzystania z soketu kontrolnego, przez
który jesteśmy w stanie sterować połączeniem. Bez nich, po położeniu interfejsu `tun9` , połączenie
SSH byłoby cały czas aktywne i nie moglibyśmy ponownie zestawić tunelu na potrzeby VPN bez
uprzedniego ubicia demona SSH. Opcja `-f` ma za zadanie zainicjować połączenie w tle, tak by nie
blokowało konfiguracji interfejsu. Dalej jest parametr `-w` , który definiuje użyty interfejs
`tun` . Cyferki `9:9` określają numer interfejsu zarówno po stronie klienta jak i serwera, czyli w
obu przypadkach zostanie użyty interfejs `tun9` . Następnie mamy użytkownika SSH wraz z adres IP
serwera, no i oczywiście jego port. Wszystko co zostało ujęte w `' '` , to polecenie, które ma
zostać wykonane na drugim końcu połączenia po zalogowaniu się via SSH. W tym przypadku są to
polecenia `ifdown` i `ifup` , które mają za zadanie skonfigurować interfejs `tun9` na serwerze.
Dlatego też będzie nam potrzebna odpowiednia konfiguracja tego interfejsu również i tam.

Jeśli zaś chodzi o same trasy routingu, to `10.100.0.0/24` ma za zadanie dostarczyć informacji o
sieci serwera VPN. Z kolei zaś wpisy `0.0.0.0/1` oraz `128.0.0.0/1` przekierowują cały ruch klienta
do interfejsu `tun9` . No i oczywiście mamy również
`11.22.33.44/32 via $(ip route show | awk '$1 =="default" {print $3}')` , który umożliwi opuszczenie
tunelu pakietom i posłanie ich dalej w świat przez serwer. Może i zapis jest trochę dziwny ale
generalnie są tutaj użyte dwa adresy. Pierwszy z nich to zewnętrzny adresem serwera, drugi zaś jest
dynamicznie dobranym adresem bramy domyślnej zewnętrznego interfejsu sieciowego na kliencie.

### Konfiguracja serwera

Jako, że polecenie zestawiające tunel SSH na kliencie miało za zadanie przesłać do serwera komendę
podnoszącą interfejs `tun9` , to na serwerze musimy również ten interfejs skonfigurować przez
umieszczenie odpowiedniej zwrotki w pliku `/etc/network/interfaces` :

    iface tun9 inet static
        pre-up sleep 5
        address 10.100.0.1
        pointopoint 10.100.0.2
        netmask 255.255.255.0

Nie ma może tutaj nic zaawansowanego ale to nie jest koniec naszej zabawy po stronie serwera. Musimy
jeszcze skonfigurować forwarding i reguły `iptables` .

#### Konfiguracja forwarding'u i iptables

Będąc zalogowanym po SSH na serwerze włączamy forwarding dopisując poniższą linijkę do pliku
`/etc/sysctl.conf` :

    net.ipv4.ip_forward = 1

Akceptujemy zmiany wydając poniższe polecenie:

    # sysctl -p

Następnie dodajemy poniższe reguły do skryptu firewall'a:

    iptables -t nat -A POSTROUTING -s 10.100.0.0/24 -o eth0 -j MASQUERADE
    iptables -A FORWARD -s 10.100.0.0/24 -i tun+ -o eth0 -j ACCEPT
    iptables -A FORWARD -d 10.100.0.0/24 -i eth0 -o tun+ -j ACCEPT

## Test połączenia SSH-VPN

Jeśli wszystkie powyższe kroki przeprowadziliśmy prawidłowo, to nasz VPN po SSH powinien działać bez
zarzutu. By się o tym przekonać, odpalamy terminal na kliencie i wpisujemy w nim poniższe polecenie:

    # ifup tun9

Nie powinno być żadnych błędów. By zakończyć połączenie, w terminalu na kliencie wpisujemy:

    # ifdown tun9


[1]: /post/jak-ukryc-ruch-openvpn-przy-pomocy-stunnel/
[2]: https://help.ubuntu.com/community/SSH_VPN
[3]: /post/uwierzytelniajace-klucze-ssh/
