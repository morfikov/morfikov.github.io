---
author: Morfik
categories:
- Linux
date: "2015-06-21T17:05:45Z"
date_gmt: 2015-06-21 15:05:45 +0200
published: true
status: publish
tags:
- debian
- email
- szyfrowanie
- vpn
- sieć
- prywatność
title: Riseup VPN na straży prywatności
---

W dobie NSA, podsłuchów rządowych i wszelkiego rodzaju pogwałcenia prawa do prywatności, ludzie
wymyślili [VPN](https://pl.wikipedia.org/wiki/Virtual_Private_Network). VPN to nic innego jak tunel
łączący dwie odległe od siebie maszyny, które chcą wymieniać ze sobą dane w sposób uniemożliwiający
osobom trzecim podejrzenie całej tej komunikacji. Zalety korzystania z takiego dobrodziejstwa natury
są oczywiste. Można obejść cenzorów, gdyż cały ruch z naszej maszyny jest przesyłany do serwera w
postaci szyfrowanej, a dopiero z tego serwera sygnał idzie dalej w świat. Jest wiele komercyjnych
usług, które za drobną opłatą mogą zapewnić nam tego tupu usługi ale ja nie będę tutaj się o nich
rozpisywał. Bez problemu można je znaleźć w góglu. My tutaj za to zajmiemy się VPN jaki zapewnia
[riseup](https://riseup.net/). By móc korzystać z ich usług, trzeba założyć na ich stronie konto, a
to z kolei jest przyznawane tym, którzy w jakiś sposób działają na rzecz szeroko pojętej wolności.
Ja wysłałem zgłoszenie i konto zostało mi przyznane. Dlatego też opiszę jak wygląda konfiguracja VPN
na ich przykładzie.

<!--more-->
## Oprogramowanie

By móc korzystać z VPN potrzebujemy odpowiedniego oprogramowania. Mowa oczywiście o
[OpenVPN](https://openvpn.net/). W debianie jest ono dostępne w pakiecie `openvpn` , zatem
instalacja nie powinna sprawić nam problemów.

Jeśli chodzi zaś o konfigurację, to mamy do wyboru dwie opcje. Możemy ręcznie wpisywać wszelkie dane
potrzebne do połączenia albo też skorzystać z pliku konfiguracyjnego i w nim podać potrzebne
parametry. Skorzystamy z tego drugiego sposobu, bo prościej.

Tworzymy zatem plik `/etc/openvpn/riseup.conf` i wklejamy do niego poniższą zawartość:

    client
    dev tun
    proto tcp
    auth-user-pass /etc/openvpn/auth/riseup.auth
    ca /etc/openvpn/certs/riseup_ca.crt
    nobind
    persist-key
    persist-tun
    #log-append  /var/log/openvpn/riseup.log
    remote-cert-tls server
    #ns-cert-type server
    verb 4
    auth-nocache
    #script-security 1
    #user nobody
    #group nogroup
    script-security 2
    up /etc/openvpn/update-resolv-conf.sh
    down /etc/openvpn/update-resolv-conf.sh
    #remote nyc.vpn.riseup.net
    remote seattle.vpn.riseup.net
    port 1194
    #port 443
    #port 80

Serwery do wyboru mamy dwa: `seattle.vpn.riseup.net` (na zachodzie US) oraz `nyc.vpn.riseup.net `(na
wschodzie US) . Wybieramy ten, do którego jest nam bliżej. Sporo opcji w powyższym pliku trzeba
dostosować manualnie, bo nie zawsze mogą one działać w określonych sieciach, gdzie admini mogli nam
coś przyblokować. Prawdopodobnie trzeba będzie pobawić się protokołem (tcp/udp) lub/i portem
(1194/443/80).

Musimy też pozyskać certyfikat riseup. Można go pobrać
[stąd](https://riseup.net/security/network-security/riseup-ca/RiseupCA.pem). Zapisujemy go na
dysku, a ścieżkę do niego podajemy w pliku konfiguracyjnym. Pamiętajmy o tym, że ten certyfikat
[trzeba
zweryfikować](https://riseup.net/en/security/network-security/riseup-ca#verify-the-riseup-ca-certificate-optional).
Musimy także stworzyć plik zawierający dane uwierzytelniające -- login oraz pseudo hasło. To nie
będzie hasło do konta, które umożliwia nam logowanie się do systemu riseup. Będzie to VPN Secret,
czyli ciąg znaków wygenerowany przez system. W przypadku jego utraty na rzecz osób trzecich, nie
uzyskają oni dostępu do naszego konta na riseup.net . Tworzymy zatem plik
`/etc/openvpn/auth/riseup.auth` :

    kotbezbutow
    lMls2Mk4IasOs0R2psrGtAkcllK

Wszystkie parametry połączenia mamy już skonfigurowane, wydajemy zatem poniższą komendę:

    # openvpn /etc/openvpn/riseup.conf

Tunel powinien zostać utworzony. Powinniśmy mieć także do dyspozycji nowy interfejs sieciowy:

    tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
    inet addr:172.27.100.6  P-t-P:172.27.100.6  Mask:255.255.252.0
    UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
    RX packets:26 errors:0 dropped:0 overruns:0 frame:0
    TX packets:26 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:100
    RX bytes:1944 (1.8 KiB)  TX bytes:1904 (1.8 KiB)

Na stronie riseup piszą, by nie podłączać się bez firewalla, gdyż taki VPN obchodzi wszystkie
lokalne mechanizmy ochrony i włącza nas bezpośrednio do internetu, tak jak byśmy mieli stały
zewnętrzny adres i całe robactwo może się do nas wpakować. Jeśli mamy jakieś regułki w iptables,
trzeba pamiętać by dodać odpowiednie wpisy dla nowego interfejsu.

## Konfiguracja resolver'a

Oczywiście samo szyfrowanie komunikacji na nic się nie zda jeśli korzystamy z DNS od naszego ISP.
Riseup zapewnia własny szyfrowany DNS (172.27.0.1) i, jak podają, żadne zapytania DNS nie są
logowane. W każdym razie można albo się zdecydować na resolver riseup, albo skorzystać z innej
alternatywy, np.
[dnscrypt-proxy]({{< baseurl >}}/post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/) . W każdym
razie trzeba pamiętać by zaszyfrować również ruch DNS.

Jeśli chcemy skorzystać z DNS riseup, możemy również doinstalować w tym celu `resolvconf` , który to
zmieni nam automatycznie (przy pomocy skryptu `/etc/openvpn/update-resolv-conf.sh` ) resolver na
szyfrowany przy każdym łączeniu się do VPN. Ja jednak wolę polegać na manualnej konfiguracji niż na
automatach i ręcznie dopisuję odpowiedni adres w pliku `/etc/resolv.conf` . Trzeba tylko pamiętać by
skonfigurować w naszym systemie programy, które manipulują tym plikiem.

## Test połączenia VPN

By przekonać się i sprawdzić czy faktycznie ruch przepuszczany jest za pośrednictwem VPN riseup,
wchodzimy na pierwszą z brzegu stronę sprawdzającą adres IP. Ja korzystałem z
<http://whatismyipaddress.com/> . Poniżej wynik:

![]({{< baseurl >}}/img/2015/06/1.riseup-vpn-adres-ip-test.png#medium)

Właśnie przeprowadziłem się do US. Ale jest mały problem. Połączenia VPN, zwłaszcza te darmowe jak
riseup, czasem nie są zbyt szybkie. W prawdzie nie ma co porównywać do TORa ale pingi mówią same za
siebie:

    $ ping wp.pl -c 3
    PING wp.pl (212.77.100.101) 56(84) bytes of data.
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=1 ttl=235 time=504 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=2 ttl=235 time=446 ms
    64 bytes from www.wp.pl (212.77.100.101): icmp_seq=3 ttl=235 time=473 ms

    --- wp.pl ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 1999ms
    rtt min/avg/max/mdev = 446.842/474.946/504.414/23.523 ms

## Inne usługi riseup

Dobrze wiedzieć, że riseup dostarcza nie tylko usługę VPN ale także pocztę, w której mamy obsługę
kluczy szyfrujących gpg/pgp. Jeśli nie posiadamy swoich kluczy, riseup może nam je wygenerować.
Miejsca w skrzynce, co prawda, nie ma jakoś stosunkowo dużo -- w moim przypadku to 100 MiB ale mi to
spokojnie wystarczy. Poza tym, mam skonfigurowanego Thunderbirda i nie trzymam ważnych wiadomości na
serwerach mailowych, bez względu na to czyje one by to nie były. Załączniki mogą być wielkości max.
2 MiB. Także, pod względem przesyłania plików nie może się równać z gmailem. Ale za to gmail nie
może się pochwalić szyfrowaniem wiadomości -- coś za coś. No i oczywiście mamy możliwość
wykorzystania tego maila w serwisach, gdzie gmail i inne podobne korporacyjne usługi nie mają
wstępu.
