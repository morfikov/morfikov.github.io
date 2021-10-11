---
author: Morfik
categories:
- Linux
date: "2015-10-29T23:27:24Z"
date_gmt: 2015-10-29 21:27:24 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- lxc
GHissueID: 183
title: Konfiguracja kontenerów LXC
---

[Kontenery LXC](https://wiki.debian.org/LXC) mają za zadanie odizolować poszczególne usługi od
pozostałej części systemu. LXC jest podobny nieco do maszyn wirtualnych, np. tych tworzonych przez
VirtualBox. Niemniej jednak, oba mechanizmy różnią się trochę. Zasadnicza różnica między nimi polega
na tym, że LXC wykorzystuje [środowisko
chroot](/post/przygotowanie-srodowiska-chroot-do-pracy/) , w którym współdzielone
jest jądro operacyjne. Nie trzeba także z góry określać zasobów pod działanie takiego kontenera, tak
jak to ma w przypadku maszyn wirtualnych. Rzućmy zatem okiem jak wygląda konfiguracja takich
kontenerów na linux'ie.

<!--more-->
## Konfiguracja systemu pod LXC

Kontener LXC działa na zasadzie zwykłego środowiska `chroot` ale posiada kilka znaczących różnic. W
środowisku `chroot` nie ma możliwości określenia z jakich urządzeń będziemy korzystać, wobec czego
dostępny jest cały katalog `/dev/` . W przypadku LXC możemy określić, którym urządzeniom zezwalamy
na działanie. Kontener LXC niekoniecznie musi korzystać z ustawień sieci hosta. Można na nim ustawić
NAT i za pomocą `iptables` przepuszczać lub blokować połączenia. Jest też spora różnica w procesach,
bo w środowisku `chroot` , po zamontowaniu katalogu `/proc/` , mamy wgląd we wszystkie procesy
dostępne w całym systemie. LXC z kolei pozwala zobaczyć jedynie tylko te procesy, które działają w
jego obrębie. Naturalnie, host dalej ma możliwość wglądu w procesy wewnątrz kontenera LXC i może
dowolnie nimi zarządzać. Dodatkowo [LXC jest kontrolowany przez
cgroups](https://en.wikipedia.org/wiki/Cgroups) i można w ten sposób przydzielać zasoby pod
kontenery. Można nawet zawiesić (zamrozić) taki kontener, by nie pobierał żadnych zasobów, aż do
czasu odmrożenia.

By korzystać z kontenerów LXC, musimy doinstalować w systemie odpowiednie oprogramowanie, tj. pakiet
`lxc` . We wcześniejszych wersjach debiana była cała masa problemów z kontenerami LXC oraz
mechanizmem cgroups. Po tym jak systemd stał się domyślnym init'em, to praktycznie wszystkie
problemy znikły i nie trzeba przeprowadzać żadnych dodatkowych akcji konfiguracyjnych, by wszystko
działało jak należy. W każdym razie, jeśli mamy poprawnie skonfigurowany system pod kontener LXC, to
po wpisaniu w terminalu `lxc-checkconfig` wszystkie kontrolki powinny być zapalone na zielono.

## Przygotowanie kontenera LXC

Przy pomocy `lxc-create` budujemy kontener, w którym to zostanie zainstalowany minimalny system
operacyjny. Możemy określić czy będzie to debian oraz ewentualnie jego wydanie, np. jessie. Odpalamy
zatem terminal i wpisujemy w nim to poniższe polecenie:

    # lxc-create -n kontener -t debian -P /media/Kabi/lxc_machines/ -- -r jessie -a amd64

Parametry w powyższej linijce oznaczają:

  - `-n` -- nazwa kontenera
  - `-t` -- szablon użyty do zbudowania systemu
  - `-P` -- ścieżka do zapisu plików kontenera
  - `-r` -- release tego co zostało użyte w opcji `-t`
  - `-a` -- architektura budowanego systemu

Gdy budujemy kontener po raz pierwszy, zostanie zainicjowany `debootstrap` , który pobierze,
zainstaluje i skonfiguruje minimalny system. W przypadku budowania kolejnego kontenera (ta sama
architektura, ta sama gałąź), nie będzie trzeba pobierać minimalnego systemu przez sieć jeszcze raz.

Przechodzimy teraz do edycji pliku konfiguracyjnego kontenera. Znajduje się on w
`/media/Kabi/lxc_machines/kontener/config` . To w nim określamy urządzenia, punkty montowania oraz
konfigurację sieci dla kontenera.

### Urządzenia

W zależności od przeznaczenia kontenera LXC, będziemy musieli zdecydować, do których urządzeń taki
kontener ma mieć dostęp. Praktycznie każde urządzenie z katalogu `/dev/` możemy udostępnić z osobna
w pliku konfiguracyjnym przez dopisanie w nim pozycji `lxc.cgroup.devices.allow` , przykładowo:

    lxc.cgroup.devices.allow = c 195:0 rwm

Wyżej został użyty dziwny zapis: `c 195:0 rwm` . Co on oznacza? [Dostęp do urządzeń w kontenerach
LXC](https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt) jest przyznawany za pomocą
mechanizmu cgroups. Pierwszy znak ( `c` ) to urządzenie znakowe (char), jest także do wyboru `b`
odpowiadające za urządzenie blokowe. Dalej mamy dwa numerki `195:0` i są to liczby zwracane przy
listowaniu urządzeń w katalogu `/dev/` . Ostatnia pozycja odpowiada za uprawnienia jakie zostaną
przyznane kontenerowi w przypadku tego urządzenia: `r` , to odczyt, `w`, to zapis, a `m` odpowiada
za tworzenie plików urządzeń przy pomocy `mknod` .

#### Problemy z udev'em

Udev nie działa z kontenerami LXC i wszystkie pliki urządzeń musimy sobie stworzyć ręcznie. Nie ma
się co jednak przejmować, bo stworzenie takich urządzeń nie jest specjalnie trudne. Przede
wszystkim, musimy określić jakie urządzenia z katalogu `/dev/` nas interesują. Załóżmy, że chcemy by
kontener miał dostęp do urządzenia `/dev/nvidia0` , czyli do karty graficznej. Zaglądamy zatem do
katalogu `/dev/` i dla przykładu, to urządzenie `nvidia0` wygląda tak:

    $ ls -al /dev/nvidia0
    crw-rw-rw-+ 1 root root 195, 0 Mar 26 12:01 /dev/nvidia0

Kluczowa sprawa, to te dwie liczby: `195` oraz `0` . Urządzenia tworzymy przy pomocy `mknod` i by
stworzyć to urządzenie w kontenerze, wpisujemy w nim tę poniższą linijkę:

    root@kontener:/dev# mknod -m 666 /dev/nvidia0 c 195 0

Podobnie postępujemy dla wszystkich urządzeń, do których kontener ma mieć dostęp.

### Punkty montowania

W pliku konfiguracyjnym kontenera LXC możemy także określać punkty montowania zasobów, które mogą
być udostępniane wewnątrz kontenera. Służy do tego parametr `lxc.mount.entry`
    :

    lxc.mount.entry=/media/lxc-test/ /media/Kabi/lxc_machines/kontener/rootfs/lxc-test/ none bind 0 0

Pierwsza ścieżka odpowiada za zasób w systemie hosta. Druga za to gdzie go zamontować w kontenerze.
Dalej już mamy opcje co programu `mount` . Katalog docelowy musi istnieć wewnątrz kontenera, w
przeciwnym razie kontener się nam nie odpali i zostanie zwrócony poniższy
    błąd:

    lxc-start: No such file or directory - failed to mount '/media/lxc-test/' on '/media/Kabi/lxc_machines/kontener/rootfs/lxc-test/'
    lxc-start: failed to setup the mount entries for 'kontener'
    lxc-start: failed to setup the container
    lxc-start: invalid sequence number 1. expected 2
    lxc-start: failed to spawn 'kontener'

### Sieć

W pliku konfiguracyjnym umieszczamy też opcje dotyczące połączenia sieciowego kontenera LXC. W
poniższym przykładzie został skonfigurowany NAT, który wykorzystuje wirtualny interfejs mostka
`br-lxc` :

    #lxc.network.type = empty
    lxc.network.type = veth
    lxc.network.name = veth0
    lxc.network.flags = up
    lxc.network.link = br-lxc
    lxc.network.veth.pair = veth0-testing
    lxc.network.ipv4 = 192.168.10.10/24
    lxc.network.ipv4.gateway = 192.168.10.100

#### Mostek (bridge)

W tym przypadku wymagane jest także skonfigurować mostka. W przeciwnym razie kontener nie będzie
miał dostępu do internetu. By móc stworzyć wirtualny interfejs mostka, potrzebne nam są narzędzia,
a te są dostarczane przez pakiet `bridge-utils` . Instalujemy go i tworzymy trzy pliki w katalogu
`/etc/systemd/network/` :

Plik `20-br-lxc.netdev` :

    [NetDev]
    Description=Bridge for LXC containers
    Name=br-lxc
    Kind=bridge

Plik `30-br-lxc-static.network` :

    [Match]
    Name=br-lxc

    [Network]
    Description=LXC bridge configuration
    DHCP=no
    LinkLocalAddressing=no
    Address=192.168.10.100/24
    DNS=
    IPForward=true

Plik `35-veth.network` :

    [Match]
    Driver=veth
    Name=veth0

    [Network]
    Description=Virtual interfaces Configuration
    Bridge=br-lxc

Restartujemy teraz połączenie sieciowe (ewentualnie uruchamiamy komputer ponownie), by mieć pewność,
że wszystko zostało skonfigurowane jak należy:

    # systemctl daemon-reload
    # systemctl restart systemd-networkd

#### Forwarding pakietów i reguły dla iptables

Musimy dodać jeszcze kilka reguł do naszego [skryptu
firewall'a](/post/firewall-na-linuxowe-maszyny-klienckie/):

    iptables -t nat -A POSTROUTING -o bond0 -s 192.168.10.0/24 -j MASQUERADE

    iptables -t filter -A icmp_in -p icmp -i br-lxc -s 192.168.10.0/24 -j ACCEPT
    iptables -t filter -A udp -p udp -i br-lxc -s 192.168.10.0/24 -j ACCEPT
    iptables -t filter -A tcp -p tcp -i br-lxc -s 192.168.10.0/24 -j ACCEPT

    iptables -t filter -A fw-interfaces -i br-lxc -o br-lxc -s 192.168.10.0/24 -d 192.168.10.0/24 -j ACCEPT
    iptables -t filter -A fw-interfaces -i br-lxc -o bond0 -s 192.168.10.0/24 -j ACCEPT

Włączamy także forwarding dla pakietów sieciowych w kernelu przez dopisanie do pliku
`/etc/sysctl.conf` poniższych linijek:

    net.ipv4.ip_forward=1
    net.ipv6.conf.all.forwarding=1

#### Resolver

Połączenie kontenera ze światem jak i maszyną hosta powinno już zostać ustanowione. Mogą jednak nie
działać domeny. W takim przypadku dobrze jest dopisać te poniższe linijki do pliku
`/etc/resolv.conf` w kontenerze:

    nameserver 208.67.222.222
    nameserver 208.67.220.220

#### Plik hosts

Dobrze jest także skonfigurować sobie plik `/etc/hosts` zarówno na hoście jak i w kontenerze,
zwłaszcza jeśli chcemy używać jakichś usług, np. `apache2` .

Plik `/etc/hosts` :

    192.168.10.10 lxc.mhouse lxc

Plik `/media/Kabi/lxc_machines/kontener/rootfs/etc/hosts` :

    192.168.1.150 morfikownia.mhouse morfikownia

#### Plik sources.list

Jeśli zamierzamy instalować sporo rzeczy, powinniśmy rozważyć edycję pliku `sources.list` w
kontenerze. Domyślne wpisy wskazują na mirror, który jest dość wolny. Zmieńmy je zatem:

    deb     http://ftp.pl.debian.org/debian/ sid main non-free contrib
    #     deb-src http://ftp.pl.debian.org/debian/ sid main non-free contrib

### Chroot

W przypadku napotakania większych problemów z konfiguracją kontenera LXC, pamiętajmy, że zawsze
możemy skorzystać ze zwykłego środowiska chroot, w którym to możemy doinstalować pakiet
`iputils-ping` zawierający min. narzędzie `ping` .

### Podgląd konfiguracji

Jeśli teraz podejrzymy interfejsy sieciowe (przez `ifconfig` albo `ip addr show` ), powinniśmy
ujrzeć `br-lxc` :

    br-lxc: flags=4163  mtu 1500
            inet 192.168.10.100  netmask 255.255.255.0  broadcast 192.168.10.255
            ether 5e:c5:78:f0:4a:b0  txqueuelen 0  (Ethernet)
            RX packets 9789  bytes 16299493 (15.5 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 10513  bytes 4859846 (4.6 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Przy włączonym kontenerze, powinien być widoczny także jego interfejs sieciowy:

    veth0-testing: flags=4163  mtu 1500
            inet6 fe80::fcbd:d0ff:fe6b:219  prefixlen 64  scopeid 0x20
            ether fe:bd:d0:6b:02:19  txqueuelen 1000  (Ethernet)
            RX packets 7  bytes 578 (578.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 17  bytes 1372 (1.3 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

Powinna być także widoczna dodatkowa trasa w tablicy routingu:

    # ip route show
    ...
    192.168.10.0/24 dev br-lxc  proto kernel  scope link  src 192.168.10.100

Jeśli zaś chodzi o samą konfigurację mostka, to wygląda ona tak:

    # brctl show
    bridge name     bridge id               STP enabled     interfaces
    br-lxc          8000.5ec578f04ab0       no              veth0-testing

## Operowanie kontenerem LXC

By wystartować kontener, wklepujemy do terminala taką oto linijkę:

    # lxc-start -n kontener -P /media/Kabi/lxc_machines/

Logujemy się za pomocą użytkownika `root` oraz hasła, które zostało zwrócone po zakończeniu procesu
instalacyjnego i sprawdzamy czy działa sieć przy pomocy `apt-get update` . Jeśli lista repozytoriów
zostanie pobrana, znaczy, żę wszystko działa jak należy.

Zatrzymać kontener możemy albo przez zalogowanie się do niego i wydanie zwykłego polecenia
zamykającego system, np. `shutdown` z odpowiednimi opcjami, czy też `poweroff` . Możemy także
wyłączyć kontener z maszyny hosta przy pomocy:

    # lxc-stop -n kontener -P /media/Kabi/lxc_machines/
