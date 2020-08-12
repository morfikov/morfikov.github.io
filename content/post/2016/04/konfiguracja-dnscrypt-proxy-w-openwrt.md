---
author: Morfik
categories:
- OpenWRT
date: "2016-04-24T20:10:01Z"
date_gmt: 2016-04-24 18:10:01 +0200
published: true
status: publish
tags:
- dns
- resolver
- chaos-calmer
- router
title: Konfiguracja dnscrypt-proxy w OpenWRT
---

Realizowanie zapytań DNS jest kluczowe do poprawnego działania min. stron internetowych. Router z
OpenWRT na pokładzie bez problemu potrafi rozwiązywać domeny na adresy IP. Jest to realizowane przez
oprogramowanie `dnsmasq` . Problem w tym, że zwykle resolver, który będzie uwzględniany w
konfiguracji routera, wskazuje na serwery DNS naszego ISP, czy też jakiejś większej korporacji. W
ten sposób, wszystkie dane z przeglądania stron internetowych podajemy tym organizacjom za free.
Przy pomocy [narzędzia dnscrypt-proxy](https://dnscrypt.org/) jesteśmy w stanie zabezpieczyć naszą
sieć przed tego typu zabiegami zbierania danych. Po części też możemy uchronić się przed cenzurą,
którą może nam zafundować lokalny provider internetowy. W tym artykule zaimplementujemy obsługę
szyfrowanego resolver'a DNS na naszym domowym routerze.

<!--more-->
## Instalacja dnscrypt-proxy w OpenWRT

We wcześniejszych wersjach OpenWRT, pakiet `dnscrypt-proxy` nie był dostępny w oficjalnych
repozytoriach. W przypadku Chaos Calmer, ten pakiet jest już obecny i jego instalacja sprowadza się
do wydania tych poniższych poleceń:

    # opkg update
    # opkg install dnscrypt-proxy

Usługa standardowo nasłuchuje zapytań na adresie `127.0.0.1` i porcie `5353` . Jeśli komuś nie
odpowiadają te parametry, to może je zwyczajnie zmienić w pliku `/etc/config/dnscrypt-proxy` :

    config dnscrypt-proxy
            option address '127.0.0.1'
            option port '5353'
            # option resolver 'opendns'
            # option resolvers_list '/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv'

Wyżej mamy także wykomentowane dwie inne opcje: `resolvers_list` wskazuje plik z serwerami, które
obsługują szyfrowanie zapytań, oraz `resolver` określający wybór jednego z tych resolver'ów. W pliku
`/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv` mamy sporo informacji na temat konfiguracji
danego resolver'a, min. dane dotyczące tego, czy są przechowywane logi. Warto zaznajomić się z tym
listingiem i wybrać dla siebie odpowiedni serwer DNS.

## Przekierowanie zapytań DNS

Teraz musimy przekierować zapytania DNS. W tym celu musimy poinstruować `dnsmasq` by korzystał z
`dnscrypt-proxy` , a nie z domyślnych serwerów DNS pobranych z serwera DHCP od naszego ISP. Na
początek, w pliku `/etc/config/dhcp` komentujemy poniższy wpis:

    #      option resolvfile '/tmp/resolv.conf.auto'

Następnie dodajemy ten poniższy blok kodu:

``` 
      option noresolv '1'
      list server '127.0.0.1#5353'
      list server '/pool.ntp.org/208.67.222.222'
```

Opcja `noresolv` sprawi, że `dnsmasq` nie będzie używał DNS'ów pobranych z lease DHCP. Druga linijka
określa, gdzie przesyłać zapytania DNS. Ostatni wpis sprawi, że wszelkie zapytania do serwerów czasu
będą rozwiązywane przez OpenDNS w formie niezaszyfrowanej. Chodzi generalnie o precyzję pobranego
czasu tak, by był możliwie dokładny. Szyfrowanie, jakby nie patrzeć, spowalnia nieco synchronizację
czasu. Można tutaj wpisać dowolne DNS'y.

Resetujemy teraz demona `dnsmasq` za pomocą poniższego polecenia:

    # /etc/init.d/dnsmasq restart

W logu, który możemy odczytać za pomocą `logread` powinniśmy ujrzeć min. te poniższe komunikaty:

    dnsmasq[2118]: using nameserver 208.67.222.222#53 for domain pool.ntp.org
    dnsmasq[2118]: using nameserver 127.0.0.1#5353

## Testy szyfrowania zapytań DNS

Sprawdźmy zatem czy faktycznie te zapytania są szyfrowane. Z dowolnego hosta w sieci, który
otrzymuje konfigurację za sprawą protokołu DHCP, wydajemy poniższe polecenie:

    $ dig debug.opendns.com txt
    ...
    debug.opendns.com.      0       IN      TXT     "dnscrypt enabled (717744506545635A)"
    ...

Powyższa informacja jednoznacznie stwierdza, że zapytania DNS są szyfrowane.

## OpenWRT nie rozwiązuje domen

Może się zdarzyć tak, że po zaimplementowaniu szyfrowanych zapytań utracimy połączenie ze światem.
Niekoniecznie musi się to nam przytrafić od razu po instalacji i konfiguracji `dnscrypt-proxy` .
Niemniej jednak, warto pamiętać, by przeprowadzić test, który może nam wskazać czy winny jest
resolver DNS. Sam test sprowadza się do odpytania przy pomocy `ping` dwóch hostów. Jednego po
adresie IP, drugiego po domenie. Poniżej przykład:

    # ping wp.pl
    ping: bad address 'wp.pl'
    
    # ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8): 56 data bytes
    64 bytes from 8.8.8.8: seq=0 ttl=44 time=58.543 ms
    64 bytes from 8.8.8.8: seq=1 ttl=44 time=59.077 ms
    64 bytes from 8.8.8.8: seq=2 ttl=44 time=59.445 ms
    ^C
    --- 8.8.8.8 ping statistics ---
    3 packets transmitted, 3 packets received, 0% packet loss
    round-trip min/avg/max = 58.543/59.021/59.445 ms

Tutaj mamy ewidentnie problemy z resolver'em DNS. Trzeba zatem przejrzeć konfigurację, a jeśli
wszystko się zgadza, to zresetować usługę. Być może nie została automatycznie dodana do autostartu i
trzeba ją dodać ręcznie w poniższy sposób:

    # /etc/init.d/dnscrypt-proxy enable
    # /etc/init.d/dnscrypt-proxy start

Może się także zdarzyć tak, że po restarcie routera domeny nie będą rozwiązywane. W takim przypadku
możemy edytować skrypt startowy, który znajduje się w pliku `/etc/init.d/dnscrypt-proxy` i przepisać
w nim numerek odpowiedzialny za sekwencję startową. Robimy to w poniższy sposób:

    START=55

Można także dopisać do pliku `/etc/rc.local` linijkę restartującą usługę resolver'a chwilę po
zakończeniu procedury startowej routera:

    /etc/init.d/dnscrypt-proxy restart

## Komunikaty w logu

W przypadku, gdy drażnią nas nieco zasyfione logi, możemy zmniejszyć poziom logowania przez
dopisanie w `/etc/init.d/dnscrypt-proxy` parametru `--loglevel` :

    ...
          service_start /usr/sbin/dnscrypt-proxy -d \
                -a ${address}:${port} \
                -u nobody \
                -L ${resolvers_list:-'/usr/share/dnscrypt-proxy/dnscrypt-resolvers.csv'} \
                -R ${resolver:-'opendns'}
                --loglevel=5
    ...
