---
author: Morfik
categories:
- Linux
date: "2016-04-25T14:37:51Z"
date_gmt: 2016-04-25 12:37:51 +0200
published: true
status: publish
tags:
- debian
- vpn
- prywatność
- sieć
- dns
- resolver
title: Przeciek DNS (DNS leak) w VPN (resolvconf)
---

[Przeciek DNS (dns leak)](https://dnsleaktest.com/what-is-a-dns-leak.html) to nic innego jak wyciek
poufnej informacji, za sprawą nieprawidłowej konfiguracji resolver'a DNS. Może niekoniecznie jest
winne tutaj samo oprogramowanie, które realizuje zapytania DNS, czy też serwer domen jakiejś
organizacji. Chodzi głównie o tematykę [VPN](https://pl.wikipedia.org/wiki/Virtual_Private_Network),
gdzie cały ruch sieciowy powinien być wrzucany do tunelu SSL/TLS i szyfrowany. W pewnych sytuacjach,
zapytania DNS mogą zostać wysyłane pod zewnętrzny resolver, często w formie niezaszyfrowanej i do
tego poza połączeniem VPN. Ten ruch można podsłuchać, przechwycić i poddać analizie. Celem tego
artykułu jest tak skonfigurowanie linux'a (w tym przypadku dystrybucja Debian), by te przecieki
wyeliminować. Jest to możliwe za sprawą narzędzia `resolvconf`.

<!--more-->
## Realizowanie zapytań DNS w linux'ie

Linux'y standardowo nie posiadają żadnego oprogramowania, które cache'uje zapytania DNS w celu
szybszego zwracania wyników, tak jak to robi, np. windows. Tutaj trzeba świadomie sobie taką usługę
zainstalować i skonfigurować. W przeciwnym wypadku zapytania będą przesyłane bezpośrednio do
upstream'owego serwera DNS. Adresy takiego serwera domen są pobierane zwykle z lease DHCP, którą
nasz system otrzymuje od serwera DHCP providera internetowego. Następnie ten wpis jest przetwarzany
i adres serwera DNS wędruje do pliku `/etc/resolv.conf` . Większość aplikacji internetowych przy
rozwiązywaniu domen odpytuje adres IP, który w tym pliku się znajduje. Oczywiście w pliku
`/etc/resolv.conf` może się znajdować kilka adresów (maksymalnie 3). W takim przypadku, jeśli system
nie będzie w stanie się skontaktować z pierwszym adresem, automatycznie odpyta drugi i trzeci w
kolejności, w której występują w tym pliku. Domyślny przedział czasu między przejściem do kolejnego
serwera nazw w przypadku braku odpowiedzi to 30 sekund.

Załóżmy teraz na moment, że skonfigurowaliśmy sobie połączenie VPN. Niemniej jednak, zapomnieliśmy
przepisać plik `/etc/resolv.conf` . W lease otrzymanej z serwera DHCP był zawarty wpis z serwerem
DNS o adresie 192.168.3.1 . W taki sposób, wszystkie zapytania o domeny zostaną przesłane do tego
lokalnego serwera DNS bezpośrednio. Dzieje się tak dlatego, że ten adres pochodzi z sieci, do której
jesteśmy fizycznie podłączeni. W oparciu o tablicę routingu, system wie gdzie przesyłać zapytania w
zależności od docelowych adresów IP. Wszystkie nielokalne zapytania są kierowana na bramę domyślną.
Zatem jeśli chcemy się połączyć ze zdalnym serwerem, który nie należy do naszej sieci lokalnej, to
zapytania powędrują przez VPN. Natomiast ruch DNS zostanie przesłany bezpośrednio do lokalnego
serwera DNS i tu właśnie doświadczymy DNS leak. Poniżej przykład z wireshark'a. Z lewej strony mamy
interfejs `tun0` . Po prawej zaś jest fizyczny interfejs `eth0` :

![]({{< baseurl >}}/img/2016/04/1.dns-leak-lokalny-serwer-isp.png#huge)

Trzeba wyraźnie zaznaczyć, że ten problem dotyczy jedynie lokalnych serwerów DNS, które zwykle są
implementowane w domowych routerach WiFi. Po części też taki DNS leak może wystąpić w przypadku
serwerów DNS od ISP ale to tylko wtedy, gdy ISP korzysta z prywatnych bloków adresowych, tj
10.0.0.0/8 czy 192.168.0.0/16 . W przypadku globalnych providerów DNS, takich jak google, czy
OpenDNS, DNS leak nie występuje. Poniżej przykład:

![]({{< baseurl >}}/img/2016/04/2.brak-dns-leak-google-serwer.png#huge)

Dzieje się tak dlatego, że adres IP 8.8.8.8 nie jest lokalny i zapytania do tego serwera muszą
zostać przesłane przez bramę domyślną, tj. wrzucone w szyfrowany tunel, którym powędrują do VPN.
VPN otrzyma takie żądanie DNS i prześle je w naszym imieniu do serwera DNS google. Następnie otrzyma
od niego odpowiedź, którą prześle nam z powrotem. W efekcie cały ruch między nami i VPN jest
szyfrowany. W ten sposób nikt nie jest w stanie podsłuchać zapytań DNS, które zostały przesłane do
VPN i to mimo faktu, że korzystamy z zewnętrznego serwera DNS.

Mając na uwadze powyższe informacje, możemy zrobić dwie rzeczy. Możemy wymusić korzystanie z
lokalnego serwera DNS, np. w oparciu o oprogramowanie
[dnsmasq]({{< baseurl >}}/post/cache-dns-buforowania-zapytan/) +
[dnscrypt-proxy]({{< baseurl >}}/post/dnscrypt-proxy-czyli-szyfrowanie-zapytan-dns/). Zaowocuje to
tym, że wszystkie zapytania DNS będą przesyłane na adres 127.0.0.1, następnie szyfrowane, po czym
przesyłane, np. do OpenDNS. Ten setup jest niezależny od providera internetowego i konfiguracji
sieciowej.

Z kolei, jeśli konfiguracja hosta w sieci lokalnej odbywa się za sprawą protokołu DHCP i nie chce
nam się korzystać z tego powyższego rozwiązania, to musimy w pliku `/etc/resolv.conf` zawrzeć adres
DNS dostarczany przez VPN. Zwykle taki adres może zostać odczytany z logu podczas nawiązywania
połączenia. Poniżej przykład:

    ovpn-riseup[38262]: [vpn.riseup.net] Peer Connection Initiated with [AF_INET]198.252.153.226:1194
    ovpn-riseup[38262]: SENT CONTROL [vpn.riseup.net]: 'PUSH_REQUEST' (status=1)
    ovpn-riseup[38262]: PUSH: Received control message: 'PUSH_REPLY,redirect-gateway def1,dhcp-option DNS 172.27.100.1,
    route-gateway 172.27.100.1,topology subnet,ping 7,ping-restart 35,socket-flags TCP_NODELAY,ifconfig 172.27.100.7 255.255.252.0'

Adres serwera DNS dla tego VPN, to `172.27.100.1` . Moglibyśmy ręcznie uzupełniać plik
`/etc/resolv.conf` tak, by wskazywał na ten adres ilekroć tylko łączymy się via VPN ale takie
rozwiązanie ma dwie wady. Po pierwsze, wymagana jest nasza ingerencja za każdym razem, gdy będziemy
nawiązywać połączenie, a po drugie, zawsze możemy zapomnieć to zrobić. W takim przypadku dużo lepiej
jest zaprzęgnąć do pracy narzędzie `resolvconf` , które cały ten proces przepisywania pliku
`/etc/resolv.conf` ogarnie za nas.

## Eliminowanie DNS leak za sprawą resolvconf

W Debianie standardowo mamy dostępny pakiet o nazwie `resolvconf` . Z jego instalacją nie powinno
być problemu ale trzeba powiedzieć kilka słów na temat konfiguracji tego narzędzia. Przede
wszystkim, z tym pakietem jest dostarczany szereg plików. Nas interesuje głównie katalog
`/etc/resolvconf/` , a w szczególności plik `interface-order` . W oparciu o ten plik, system wie w
jakiej kolejności wpisać adresy serwerów DNS do pliku `/etc/resolv.conf` . Interfejsy wirtualne,
takie jak `tun*` czy `tap*` , są w tym pliku wyżej niż standardowe interfejsy fizyczne. Zatem jeśli
zostanie zarejestrowany serwer DNS dla tych interfejsów, to zostaną one umieszczone wyżej w pliku
`/etc/resolv.conf` . Pamiętajmy, że standardowo wszystkie zapytania DNS zostaną przesłane pod
pierwszy adres IP w tym pliku. W przypadku odpalenia VPN, będzie to adres serwera DNS dostarczany
przez VPN, a w każdym innym wypadku, będzie to standardowa konfiguracja resolver'a dla naszego
systemu.

By `resolvconf` spełniał swoje zadanie w przypadku VPN, musimy dodatkowo stworzyć [skrypt
update-resolv-conf.sh](https://github.com/masterkorp/openvpn-update-resolv-conf) , który będzie
wykonywany przy podnoszeniu i ubijaniu połączenia. Niestety zawiera on błędną opcję `-x` przy
wywołaniu `resolvconf` . Dlatego, by ten skrypt działał prawidłowo trzeba tę opcję usunąć z
poniższej linijki:

    echo -n "$R" | $RESOLVCONF -a "${dev}.inet"

Skrypt zapisujemy w katalogu `/etc/openvpn/` i nadajemy mu prawa wykonywania. Następnie edytujemy
konfigurację połączenia VPN i dodajemy do niej te poniższe linijki:

    script-security 2
    up /etc/openvpn/update-resolv-conf.sh
    down /etc/openvpn/update-resolv-conf.sh

Możemy teraz przetestować połączenie. Jeśli plik `/etc/resolv.conf` zostanie przepisany do mniej
więcej poniższej postaci:

    $ cat /etc/resolv.conf
    # Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
    #     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
    nameserver 172.27.100.1
    nameserver 192.168.3.1

To oznacza, że mechanizm działa prawidłowo i zapytania DNS są kierowane do serwera DNS otrzymanego
od VPN. Możemy także zweryfikować połączenie pod kątem DNS leak zapuszczając ponownie wireshark'a na
interfejsach `eth0` i `tun0` . Poniżej przykład:

![]({{< baseurl >}}/img/2016/04/3.brak-dns-leak-serwer-vpn.png#huge)

## Dodatkowe zabezpieczenia przed DNS leak

Dodatkowym zabezpieczeniem, które ochroni nas przed DNS leak jest zapora sieciowa. Przy pomocy
`iptables` jesteśmy w stanie zablokować pakiety, które są przeznaczone na port 53. Chcielibyśmy
raczej też, by taka blokada dotyczyła serwerów DNS innych niż te od VPN. No i też raczej nie będzie
nam się chciało pisać odpowiednich regułek ręcznie za każdym razem, gdy nawiązujemy połączenie przez
VPN. Jak zatem zrealizować takie zadanie? Możemy posłużyć się skryptem
`/etc/openvpn/update-resolv-conf.sh` . Trzeba go tylko uzupełnić o odpowiedni kod.

Skrypt jest wywoływany przy zestawianiu połączenia i przy jego niszczeniu. Są tam dwa przypadki:
`up` i `down` . W `up` na końcu dopisujemy ten poniższy blok kodu:

      $IPT -N openvpn
      $IPT -I OUTPUT 1 -p tcp --dport 53 -j openvpn
      $IPT -I OUTPUT 1 -p udp --dport 53 -j openvpn
      $IPT -A openvpn -p tcp -o lo -j ACCEPT
      $IPT -A openvpn -p udp -o lo -j ACCEPT
      for NS in $IF_DNS_NAMESERVERS ; do
        $IPT -A openvpn -p tcp -o ${dev} -d $NS -j ACCEPT
        $IPT -A openvpn -p udp -o ${dev} -d $NS -j ACCEPT
      done
      $IPT -A openvpn -p udp -j DROP
      $IPT -A openvpn -p tcp -j DROP

Ma on za zadanie stworzyć nowy łańcuch `openvpn` , przekierować do niego ruch kierowany na port `53`
oraz umieścić w nim szereg reguł. Zezwalamy w ten sposób na rozwiązywanie domen z interfejsu `lo`
oraz interfejsu VPN, w tym przypadku `tun0` ale tylko, gdy te zapytania są kierowane do serwera DNS
od VPN. Każde inne zapytanie DNS będzie blokowane przez dwie ostatnie reguły.

W przypadku `down` musimy dopisać na końcu ten poniższy kod:

	  if [ ! -z "$foreign_option_1" ] || [ ! -z "$foreign_option_2" ] ; then
		$IPT -D OUTPUT -p tcp --dport 53 -j openvpn
		$IPT -D OUTPUT -p udp --dport 53 -j openvpn
		$IPT -F openvpn
		$IPT -X openvpn
	  fi

Sprawdza on czy zmienne, w których są przechowywane adresy serwerów DNS, są puste. Jeśli któraś z
nich nie jest, oznacza to, że VPN dysponuje serwerami DNS. W takim przypadku należy usunąć
przekierowanie do łańcucha `openvpn` , wyczyścić go z reguł, a następnie usunąć sam łańcuch.
