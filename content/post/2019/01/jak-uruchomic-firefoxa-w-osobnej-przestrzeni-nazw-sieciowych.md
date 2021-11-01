---
author: Morfik
categories:
- Linux
date: "2019-01-20T21:10:32Z"
published: true
status: publish
tags:
- debian
- firefox
- namespaces
GHissueID: 315
title: Jak uruchomić Firefox'a w osobnej przestrzeni nazw sieciowych
---

Domyślnie każdy proces uruchomiony na linux (w tym przypadku Debian) dziedziczy swoją przestrzeń
nazw sieciowych (network namespaces) od procesu nadrzędnego, standardowo od procesu init (tego z
pid 1). W takim przypadku, wszystkie procesy współdzielą tę samą przestrzeń nazw sieciowych, przez
co mają dostęp do tych samych interfejsów sieciowych, tych samych tras routingu, a reguły
firewall'a czy ustawienia serwerów DNS w jednakowym stopniu dotyczą wszystkich procesów i
zmieniając sieciową konfigurację systemu robimy to globalnie dla wszystkich tych procesów
jednocześnie. Czasami tego typu mechanika działania sieci nie jest zbyt pożądana z punktu
widzenia bezpieczeństwa lub też prywatności użytkownika. Przykładem mogą być przeglądarki
internetowe, np. Firefox, Opera czy Google Chrome/Chromium, które mogą zdradzić nasz lokalny adres
IP (w przypadku stosowania NAT). Jako, że też zostawiamy wszędzie nasz namiar w postaci
zewnętrznego adresu IP, to oba te adresy mogą nas bez większego problemu zidentyfikować w
internecie. Można jednak postarać się, by ten adres lokalny, który zwróci przeglądarka
internetowa, różnił się od tego, który przydziela nam nasz operator ISP.

<!--more-->
## Konfiguracja interfejsów w osobnej przestrzeni nazw sieciowych

Przede wszystkim musimy zacząć od stworzenia nowej przestrzeni nazw sieciowych. Do tego celu służy
polecenie `ip netns add` . W argumencie podajemy nazwę, pod którą zostanie stworzona nowa
przestrzeń nazw, np. `firefox` :

    # ip netns add firefox
    # ip netns list
    firefox

Mając nową przestrzeń nazw sieciowych, możemy stworzyć parę wirtualnych interfejsów, z których
jeden będzie obecny w domyślnej przestrzeni nazw, a drugi wewnątrz tej nowo utworzonej:

    # ip link add veth90 type veth peer name veth91
    # ip link set veth91 netns firefox

Każdy z tych wirtualnych interfejsów sieciowych wymaga poprawnej adresacji IP. Ich adresy muszą być
z tej samej podsieci. Adresację lokalnej końcówki tunelu tworzymy standardowo, czyli tak samo jak
dla zwykłego fizycznego interfejsu sieciowego:

    # ip addr flush dev veth90
    # ip addr add 192.168.10.1/24 dev veth90
    # ip link set dev veth90 up

Wyżej została wykorzystana sieć `192.168.10.0/24` . Ta sieć musi być wolna, tj. nie może być
przypisana do żadnego innego interfejsu w systemie.

Jeśli zaś chodzi o konfigurację adresacji końcówki tunelu w nowo utworzonej przestrzeni nazw
sieciowych, to musimy do tego celu skorzystać z `ip netns exec` :

    # ip netns exec firefox ip addr flush dev veth91
    # ip netns exec firefox ip addr add 192.168.10.2/24 dev veth91
    # ip netns exec firefox ip link set dev veth91 up

Dodatkowo, wewnątrz naszego nowego network namespace'a musimy skonfigurować routing. W zasadzie to
musimy dodać tylko trasę domyślną, by pakiety wiedziały gdzie mają zostać przesłane. Adresem bramki
domyślnej będzie adres IP użyty na lokalnej końcówce tunelu:

    # ip netns exec firefox ip route add default via 192.168.10.1

Możemy też podejrzeć aktualną tablice routingu, by upewnić się, że wszystko się póki co zgadza:

    # ip netns exec firefox ip route show
    default via 192.168.10.1 dev veth91
    192.168.10.0/24 dev veth91 proto kernel scope link src 192.168.10.2

Wygląda w porządku.

Podnosimy także interfejs pętli zwrotnej `lo` wewnątrz naszej przestrzeni nazw sieciowych:

    # ip netns exec firefox ip link set dev lo up

## Konfiguracja forwarding'u

By procesy w nowej przestrzeni nazw sieciowych mogły łączyć się z internetem, musimy włączyć
forwarding w kernelu:

    # echo 1 > /proc/sys/net/ipv4/ip_forward

Następnie konfigurujemy netfilter. W tym przypadku został wykorzystany `nftables` ale bez problemu
można wykorzystać i `iptables` , tylko trzeba nieco składnie reguł dostosować. Poniżej znajduje się
plik z regułami, który trzeba podać do `nft` :

    #!/usr/sbin/nft -f

    add rule inet filter FORWARD meta iifname { bond0 } meta oifname { veth90 } counter jump check-state
    add rule inet filter FORWARD meta iifname { veth90 } meta oifname { bond0 } counter accept

    add rule inet filter check-state ct state { established, related } accept
    add rule inet filter check-state ct state { invalid } counter drop

    add rule ip nat POSTROUTING meta oifname != "veth-90" ip saddr { 192.168.10.0/24 } counter masquerade

W tym przypadku interfejsem, przez który pakiety wychodzą w świat jest interfejs `bond0` , zaś
interfejs `veth90` to lokalna końcówka tunelu. Powyższy plik można dołączyć do obecnej konfiguracji
filtra pakietów lub też uruchomić go bezpośrednio z terminala wpisując:

    # nft -f firefox-ns

## Konfiguracja DNS dla network namespace

Jednym z problemów, z którym człowiek może się spotkać przy konfiguracji osobnych przestrzeni nazw
sieciowych, to ustawienia serwerów DNS. Standardowo do tego celu na linux (w tym Debian)
wykorzystywany jest plik `/etc/resolv.conf` . Pewna niestandardowa konfiguracja serwerów DNS, np.
wykorzystanie `dnscrypt-proxy` , sprawia, że wewnątrz nowego network namespace'a rozwiązywanie nazw
nam nie będzie działać. Można temu zaradzić edytując plik `/etc/resolv.conf` i ustawiając w nim
globalne adresy DNS, np. `8.8.8.8` czy `1.1.1.1` . Tylko, że przepisanie pliku `/etc/resolv.conf`
wpłynie także na domyślną przestrzeń nazw, a tego byśmy nie chcieli. Zamiast zmieniać plik
`/etc/resolv.conf` możemy stworzyć osobną konfigurację DNS na naszej nowej przestrzeni nazw
sieciowych.

Tworzymy zatem folder odpowiadający nazwie network namespace'a w katalogu`/etc/netns/` . Jeśli
katalog `/etc/netns/` nie istnieje, to trzeba go stworzyć:

    # mkdir -p /etc/netns/firefox/

Następnie tworzymy plik, którego ustawienia chcemy nadpisać (nazwa musi pasować). W tym przypadku
chcemy przepisać konfigurację DNS:

    # touch /etc/netns/firefox/resolv.conf
    # echo "nameserver 8.8.8.8" > /etc/netns/firefox/resolv.conf

## Testowanie nowej przestrzeni nazw sieciowych

Wypadałoby przetestować w tym momencie, czy aby nasz nowy network namespace działa prawidłowo:

    # ip netns exec firefox su -c 'ping 8.8.8.8 -c 4' morfik
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=120 time=66.9 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=120 time=27.6 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=120 time=32.5 ms
    64 bytes from 8.8.8.8: icmp_seq=4 ttl=120 time=21.8 ms

    --- 8.8.8.8 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 7ms
    rtt min/avg/max/mdev = 21.841/37.219/66.933/17.563 ms

To powyższe polecenie, mimo, że uruchomione jako `root`, jest wykonane jako użytkownik `morfik` .
Podobnie będzie odpalany każdy inny program, który ma działać w tym nowym network namespace. Widać
wyżej, że ping lata w obie strony. Zatem wszystko jest w porządku. Dla pewności możemy także
podejrzeć reguły `nftables` :

Tablica `nat` :

    # nft list table ip nat
    table ip nat {
            ...
            chain POSTROUTING {
                    type nat hook postrouting priority srcnat; policy accept;
                    oifname != "veth-90" ip saddr { 192.168.10.0/24 } counter packets 1 bytes 84 masquerade
            ...
    }

Tablica `filter` :

    # nft list table inet filter
    table inet filter {
            ...
            chain FORWARD {
                    type filter hook forward priority filter; policy drop;
                    ...
                    iifname { "bond0" } oifname { "veth90" } counter packets 4 bytes 336 jump check-state
                    iifname { "veth90" } oifname { "bond0" } counter packets 4 bytes 336 accept
                    ...
            }
            ...

            chain check-state {
                    ct state { established, related } accept
                    ct state { invalid } counter packets 0 bytes 0 drop
            }
            ...
    }

Widać, że na zaporze również wszystkie reguły spełniają swoje zadanie. Możemy zatem odpalić
Firefox'a w poniższy sposób:

    # ip netns exec firefox su -c 'firefox' morfik

Teraz możemy przejść na [testową stronę](https://browserleaks.com/webrtc), by sprawdzić jaki adres
IP zostanie wykryty za sprawą protokołu WebRTC:

![webrtc-firefox-ip-addrs-leak-linux-network-namespace](/img/2019/01/001-webrtc-firefox-ip-addrs-leak-linux-network-namespace.png#huge).

Wyżej mamy otwarte dwie przeglądarki. Firefox (po lewej) jest uruchomiony w skonfigurowanej
uprzednio przestrzeni nazw sieciowych, podczas gdy Opera (po prawej) korzysta z domyślnego netwrok
namespace'a. Widać, że adresy lokalne wykryte za pomocą protokołu WebRTC różnią się. Ten w Operze
wskazuje na faktyczny lokalny adres IP, natomiast Firefox zwrócił IP, który został użyty na
interfejsie wewnątrz naszej nowej przestrzeni nazw sieciowych.

Oczywiście WebRTC można wyłączyć na poziomie przeglądarki i żaden lokalny adres IP nie zostanie
ujawniony. Niemniej jednak, w tym całym eksperymencie chodziło nie tyle o fakt ukrycia lokalnego
adresu IP, co może budzić podejrzenia ale o fakt zwrócenia fałszywego adresu IP, którego trop nigdy
nie doprowadzi do nas. A poza tym, to zawsze można zapomnieć przestawić wszystkie te parametry
niezbędne w procesie anonimizacji ruchu sieciowego. Pamiętajmy też, że praktycznie każda aplikacja
odpalona w tej przestrzeni nazw sieciowych będzie dziedziczyć te ustawienia sieciowe i nie jest to
ustawienie tylko dla jednej instancji Firefox'a. Takich network namespace'ów można stworzyć sobie
ile dusza zapragnie.
