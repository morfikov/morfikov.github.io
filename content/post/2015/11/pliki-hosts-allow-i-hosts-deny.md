---
author: Morfik
categories:
- Linux
date: "2015-11-04T22:36:05Z"
date_gmt: 2015-11-04 20:36:05 +0100
published: true
status: publish
tags:
- bezpieczeństwo
title: Pliki hosts.allow i hosts.deny
---

Obecnie najpopularniejszym rozwiązaniem pod kątem ograniczania dostępu do usług systemowych jest
zapora sieciowa. Reguły iptables są w tym przypadku wręcz niezastąpione. Jednak poleganie na samych
regułach iptables nie jest zbyt dobrym pomysłem. A to z tego względu, że jeśli [skrypt
firewall'a]({{< baseurl >}}/post/firewall-na-linuxowe-maszyny-klienckie/) z jakiegoś powodu nie
zostanie wywołany przy starcie systemu, to nasza maszyna pozostaje praktycznie bezbronna i będzie
akceptować wszelkie próby połączeń do wszystkich nasłuchujących w takim systemie usług. Na szczęście
nie jest znowu aż tak źle jak mogłoby się wydawać. Albowiem linux posiada dwa pliki
`/etc/hosts.allow` i `/etc/hosts.deny` , które są w stanie zarządzać dostępem do usług systemowych.
Poniższy wpis będzie poświęcony właśnie tym dwóm plikom.

<!--more-->
## Kolejność przetwarzania plików

Pliki `hosts.allow` i `hosts.deny` są przetwarzane jeden po drugi w celu odnalezienia dopasowań.
Pierw jest sprawdzany `hosts.allow` i jeśli tam nie zostanie znaleziona para usługa:host , to
zacznie się przeszukiwanie pliku `hosts.deny` . Jeśli w żadnym z tych plików nie zostanie znalezione
dopasowanie, to host próbujący się połączyć zostanie podłączony bez żadnego problemu.

## Plik hosts.deny

Jeśli już mówimy o polityce bezpieczeństwa, to na sam początek zajmijmy się plikiem `hosts.deny` .
Domyślnie jest on pusty, zatem każde połączenie z lokalną usługą będzie akceptowane. W przypadku
maszyn klienckich, czy też i serwerów, nie powinniśmy być aż tak rozwięźli. Domyśla polityka zawsze
powinna blokować cały dostęp, a gdy już to robi, to zezwalać na pewne określone wyjątki. Musimy
zatem tak skonfigurować system by dostęp do wszystkich usług był z automatu blokowany.

Edytujemy zatem plik `hosts.deny` i dopisujemy do niego tę poniższą linijkę:

    ALL:ALL EXCEPT localhost:DENY

Zgodnie z tym co można wyczytać w
[manualu](http://manpages.ubuntu.com/manpages/wily/en/man5/hosts.deny.5.html), ten powyższy wpis
uniemożliwi uzyskać dostęp do jakichkolwiek usług ( `ALL` ) znajdujących się na tej maszynie
wszystkim hostom ( `ALL` ). Problem w tym, że system operacyjny komunikuje się z pewnymi usługami
cały czas. Dlatego też musimy dodać tutaj wyjątek dla `localhost` . Zatem jedynie zdalne połączenia
będą blokowane i o takie coś nam chodzi.

## Plik hosts.allow

Po określeniu domyślnej polityki blokowania zdalnego dostępu do usług systemowych, możemy zająć się
plikiem `hosts.allow` . Podobnie jak w przypadku pliku `hosts.deny` , również określamy pary
usługa:host . Zatem by zabezpieczyć usługę SSH na wypadek problemów z firewall'em, możemy dodać ten
poniższy wpis:

    sshd: 192.168.100.0/24

Zmiany w tych plikach są natychmiastowe i nie musimy przeprowadzać jakichkolwiek innych czynności. W
powyższym przypadku, jedynie hosty z sieci 192.168.100.0/24 będą mieć dostęp do usługi SSH. Warto
również zainstalować sobie pakiet `tcpd` , w którym to znajduje się min. narzędzie `tcpdchk` . Przy
jego pomocy jesteśmy w stanie sprawdzić poprawność reguł zdefiniowanych w w/w plikach, przykładowo:

    # tcpdchk -v
    Using network configuration file: (null)

    >>> Rule /etc/hosts.allow line 12:
    daemons:  sshd
    clients:  192.168.10.0/24
    access:   granted

    >>> Rule /etc/hosts.allow line 13:
    daemons:  pulseaudio-native
    clients:  192.168.10.0/24
    access:   granted

    >>> Rule /etc/hosts.deny line 18:
    daemons:  ALL
    clients:  ALL EXCEPT localhost
    access:   denied

Więcej informacji i przykładów można znaleźć
[tutaj](http://static.closedsrc.org/articles/dn-articles/hosts_allow.html).
