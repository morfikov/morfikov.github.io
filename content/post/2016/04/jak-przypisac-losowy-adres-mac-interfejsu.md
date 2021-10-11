---
author: Morfik
categories:
- Linux
date: "2016-04-29T16:42:55Z"
date_gmt: 2016-04-29 14:42:55 +0200
published: true
status: publish
tags:
- prywatność
- wifi
- debian
- sieć
- mac
- modem
- huawei
- e3372
GHissueID: 411
title: Jak przypisać losowy adres MAC do interfejsu
---

Interfejsy kart sieciowych, które są instalowane w komputerach, posiadają adres MAC ([Media Access
Control](https://en.wikipedia.org/wiki/MAC_address)). Jest to unikalny identyfikator, który wyróżnia
nasz komputer spośród tłumu. Na podstawie tego adresu można nie tylko określić markę sprzętu, którą
się posługujemy ale także można sklasyfikować cały nasz ruch sieciowy. W ten sposób bardzo prosto
możemy zostać zidentyfikowani wymieniając dane przez darmowe hotspoty sieci bezprzewodowych WiFi.
Niemniej jednak, jesteśmy się w stanie obronić przed tego typu inwigilacją zmieniając adres MAC
naszego komputera. Nie jest to zbytnio trudne ale trzeba uważać, by znowu nie przesadzić w drugą
stronę i czasem nie zostać zidentyfikowanym przez naszą "odmienność". W tym wpisie postaramy się
wypracować taki mechanizm, który zmieni nam adres MAC przy każdym podłączeniu do sieci i przy
zachowaniu zdroworozsądkowych zasad.

<!--more-->
## Jak wygląda adres MAC i gdzie go szukać

Zanim zmienimy adres MAC naszego komputera, musimy pierw poznać go nieco dokładniej. Przede
wszystkim, musimy ustalić jakim adresem posługuje się maszyna, którą podłączamy do sieci. Na
linux'ie służą do tego zwykle dwa narzędzia `ifconfig` oraz `ip` . Oczywiście można ten MAC odczytać
również z narzędzi graficznych, jak np. network-manager. My tutaj ograniczymy się jednak tylko do
narzędzi konsolowych wymienionych wyżej. Poniżej jest fotka przedstawiająca szereg interfejsów
sieciowych. Widnieją tam także adresy MAC:

![](/img/2016/04/1.statystyki-interfejsow-sieciowych-adres-mac.png#huge)

Adresy MAC są zaraz obok `link/ether` . I tak dla przykładu, interfejs `eth0` dysponuje adresem MAC
`f6:3a:37:fa:20:40` , natomiast interfejs `wwan0` ma przypisany adres `f2:f5:ed:74:0b:18` . Wyżej
widzimy, że trzy interfejsy mają ten sam MAC. Nie jest to błąd, a jedynie specyfikacja konfiguracji,
która jest używana na tej maszynie, konkretnie
[bonding](https://en.wikipedia.org/wiki/Link_aggregation).

Adres MAC ma 6 bajtów, czyli 48 bitów. Pierwsze trzy oktety określają producenta sprzętu. Natomiast
ostatnie trzy są losowo przypisywane do konkretnego urządzenia. Obecnie powoli odchodzi się od tych
48-bitowych adresów, na rzecz
[64-bitowych](https://en.wikipedia.org/wiki/Organizationally_unique_identifier#64-bit_Extended_Unique_Identifier_.28EUI-64.29),
gdzie w dalszym ciągu identyfikator producenta ma taką samą długość (24 bity) ale identyfikator
urządzenia ma już tych bitów 40.

Na necie można się spotkać z licznymi sposobami generowania losowych adresów MAC. Problem w tym, że
te rozwiązania często generują również całą masę niewłaściwych adresów, których system nie chce
zaakceptować. Jeśli spróbowalibyśmy taki adres wskazać systemowi, ten zwróci nam komunikat:
`RTNETLINK answers: Cannot assign requested address` . Poniżej znajduje się fotka, która rozjaśni
nam nieco sytuację i wyjaśni dlaczego nie wszystkie adresy się nadają do wykorzystania
([źródło](https://en.wikipedia.org/wiki/MAC_address)):

![](/img/2016/04/2.specyfikacja-adres-mac.png#big)

[Pierwsze dwa bity
decydują](https://superuser.com/questions/725467/set-mac-address-fails-rtnetlink-answers-cannot-assign-requested-address/725472#725472)
o tym, czy dany adres MAC jest do wykorzystania, czy też nie. Bit 1 odpowiada za określenie czy ten
adres jest typu rozgłoszeniowego (broadcast). Wszystkie adresy MAC stosowane w komputerach powinny
być unicast'owe. Dlatego też pierwszy bit musi być ustawiony na `0` . Z kolei drugi bit informuje
czy adres MAC jest zarządzany lokalnie, tj. czy administrator systemu/sieci go przypisał. Jeśli tak,
to może on nie być unikatowy. Ten bit możemy ustawić zarówno na `0` jak i `1` . Niemniej jednak,
jeśli chcemy ustawić losowy adres, to dobrze jest przestawić ten bit na `1` .

## Wady i zalety losowego MAC

Jeśli chcemy korzystać z losowego adresu MAC, to musimy zdawać sobie sprawę, że niesie to ze sobą
pewne następstwa. Przede wszystkim, jeśli w danej sieci lokalnej, do której jesteśmy podłączeni,
znajdzie się druga maszyna o takim samym MAC, to wystąpią problemy z komunikacją. Na szczęście, taki
schemat jest bardzo mało prawdopodobny. Idąc dalej, trzeba wziąć pod uwagę serwer DHCP. Te z reguły
przypisują statyczne lease w zależności od przesłanego adresu MAC. Może się zdarzyć taka sytuacja,
że nie będziemy mogli zwyczajnie otrzymać adresacji, bo ten adres MAC nie został uwzględniony na
serwerze DHCP. Ta sytuacja też raczej nam nie grozi, no chyba, że logujemy się do jakiejś sieci
prywatnej. Niemniej jednak, przeznaczeniem losowych adresów MAC są zwykle darmowe sieci WiFi, do
których może się zalogować każdy i tam nie powinniśmy takich problemów w ogóle napotkać. Trzeba
pamiętać też, że jeśli będziemy się łączyć w swojej sieci domowej, to również tutaj zostanie
przydzielony inny adres MAC, co nie zawsze jest pożądane. Dobrze jest zatem zmienić adres MAC tylko
w stosunku do określonych interfejsów, np. `wwan0` . Przykładowo, modemy LTE zwykle udostępniają
połączenie po interfejsie `ppp0` lub wspomnianym `wwan0` . Przy pomocy takiego modemu LTE zawsze
będziemy logowani do BTS'a operatora GSM i w takim przypadku, bez problemu możemy posługiwać się
losowym adresem MAC.

## Jak wygenerować losowy adres MAC

Ten adres MAC, który za moment wygenerujemy, będzie czysto losowy ale zostanie też na niego nałożony
filtr uwzględniający te powyższe założenia. W efekcie wynikowy MAC będzie nieco inny od tego
wygenerowanego losowo. Niemniej jednak, można powiedzieć, że ten adres będzie w znacznym stopniu
losowy. Przede wszystkim, musimy utworzyć sobie skrypt i wrzucić do niego tę poniższą zawartość:

    #!/bin/sh

    [ "$IFACE" != "lo" ] || exit 0

    macaddr=$(dd if=/dev/urandom bs=1024 count=1 2>/dev/null|md5sum|sed 's/^\(..\)\(..\)\(..\)\(..\)\(..\)\(..\).*$/:::::/')

    lastfive=$( echo "$macaddr" | cut -d: -f 2-6 )
    firstbyte=$( echo "$macaddr" | cut -d: -f 1 )

    firstbyte=$( printf '%02x' $(( 0x$firstbyte & 254 | 2)) )

    newmac="$firstbyte:$lastfive"

    echo "Setting a new MAC address $newmac ..."

    ip link set dev $IFACE address $newmac

    exit 0

Na samym początku mamy warunek, który omija tworzenie losowego adresu MAC dla pętli zwrotnej, tj.
interfejsu `lo` . Następnie generujemy losowy adres i umieszczamy go w zmiennej `$macaddr` . Kolejne
dwie linijki kroją ten adres, w taki sposób, by wyselekcjonować pierwszy bajt. Dalej odbywa się całą
magia. Mamy dwie operacje na bitach. Pierwsza z nich to logiczny AND z wartością 254. Natomiast
druga to logiczny OR z wartością 2. Dla przykładu załóżmy, że pierwszy bajt adresu MAC to `07` .
Wartość binarna, to `00000111` z kolei `254` binarnie, to `11111110` . Przeprowadzamy operację AND,
czyli jeśli mamy dwa bity o wartości `1` , to w wyniku również będzie `1` . W każdym innym przypadku
będzie `0`:

    11111110
    00000111
    -------- AND
    00000110

Tak otrzymaną wartość poddajemy operacji OR z wartością `2`, binarnie `00000010` . Tutaj z kolei
tylko wartości dwóch bitów wskazujących na `0` dadzą w wyniku `0` . W każdym innym przypadku mamy
mamy `1` :

    00000110
    00000010
    -------- OR
    00000110

W rezultacie otrzymujemy wartość `06` i to jest pierwszy bajt naszego losowego MAC. Więcej
informacji o operacjach logicznych [można znaleźć
tutaj](https://pl.wikipedia.org/wiki/Bramka_logiczna).

## Aplikowanie nowego adresu MAC przy podłączeniu do sieci

Wyżej zaprojektowaliśmy sobie milusi skrypt, który możemy wykorzystać w procesie nawiązywania
połączenia. W zależności od wykorzystywanego oprogramowania, ten powyższy plik będzie trzeba
umieścić w innym miejscu. Na debianie przy korzystaniu z pakietu `ifupdown` , skrypt wrzucamy do
katalogu `/etc/network/if-pre-up.d/` . Jak nazwa wskazuje, skrypty w tym katalogu wykonają się tuż
przed podniesieniem interfejsu sieciowego i pobraniem adresacji. Tylko w takim stanie interfejsu
można zmienić jego adres MAC.

Należy pamiętać, by skrypt nie posiadał żadnego rozszerzenia, typu `.sh` , bo wtedy nie zostanie
wzięty pod uwagę. Podobnie trzeba pilnować odpowiednich praw, zwłaszcza upewnić się, że skrypt ma
atrybut wykonywalności oraz, że jego właścicielem jest root. Dobrze jest sprawdzić przy pomocy
poniższego polecenia, czy skrypt zostanie wykonany. Wszystkie skrypty, które zostaną
wyszczególnione po wykonaniu tego polecenia, zostaną także uruchomione przy podnoszeniu interfejsów
sieciowych:

    # run-parts --test /etc/network/if-pre-up.d/

W zależności od konfiguracji, taki interfejs może się uaktywnić automatycznie po podłączeniu
przewodu sieciowego. Można także podnieść go ręcznie przy pomocy `ifup eth0` . Za każdym razem, gdy
się rozłączymy i połączymy ponownie, adres MAC ulegnie zmianie.

## Generowanie losowego MAC za pomocą macchanger

Istnieje także możliwość wykorzystania puli adresów MAC, która jest przypisana do poszczególnych
znanych producentów sprzętu. Niemniej jednak, to przedsięwzięcie wymaga dodatkowego oprogramowania w
postaci `macchanger` . Debian dysponuje tym pakietem, zatem nie powinno być problemów z jego
instalacją jeśli chcielibyśmy tego typu adresy uzyskać.

Adres MAC, który jest przypisany do urządzenia zawsze będzie widoczny i nigdy nie uda nam się
wymazać tej informacji. Możemy jedynie zmylić system i sprawić, by taką błędną informację przesyłał
w komunikatach sieciowych. Oryginalny adres można podejrzeć w poniższy sposób:

    # macchanger wwan0
    Current MAC:   00:1e:10:1f:00:00 (ShenZhen Huawei Communication Technologies Co.,Ltd.)
    Permanent MAC: 00:1e:10:1f:00:00 (ShenZhen Huawei Communication Technologies Co.,Ltd.)

Narzędzie `macchanger` dysponuje kilkoma ciekawymi przełącznikami:

  - `-r` -- ustawia w pełni losowy adres MAC.
  - `-e` -- przy ustawianiu MAC, część odpowiedzialna za identyfikację producenta nie zmienia się.
  - `-a` -- ustawia adres tego samego rodzaju, np. przydzielane będą jedynie adresy MAC kart WiFi.
  - `-A` -- ustawia adres dowolnego rodzaju.
  - `-b` -- ma znaczenie tylko przy generowaniu losowego adresu i gdy ta flaga nie zostanie
    określona, adres MAC będzie miał ustawiony bit lokalnego administrowania.
  - `-p` -- przywraca fabryczny MAC interfejsu.

Widzimy zatem, że `macchanger` potrafi nieco lepiej dobrać losowy adres MAC i w sporej części opcja
`-a` lub `-A` powinna nam zapewnić dostateczne ukrycie. W przypadku korzystania z `macchanger`
naturalnie nie potrzebujemy wyżej omawianego skryptu. Zamiast niego musimy edytować plik
`/etc/network/interfaces` i przy interfejsie sieciowym dopisać dwie dyrektywy: `pre-up` oraz
`post-down` . Pierwsza z nich zostanie wykonana przed podniesieniem interfejsu, druga zaś po jego
położeniu. Poniżej przykład konfiguracji interfejsu `wwan0` :

    allow-hotplug wwan0
    iface wwan0 inet dhcp
        pre-up macchanger -a $IFACE
        ...
        post-down macchanger -p $IFACE

Poniżej fotka obrazująca działający mechanizm zmiany MAC w oparciu o oprogramowanie `macchanger` :

![](/img/2016/04/3.automatyczna-zmiana-adres-mac-polaczenie.png#huge)

Więcej informacji na temat sposóbu zmiany adresów MAC przy pomocy różnych narzędzi można znaleźć na
[wiki ArchLinux'a](https://wiki.archlinux.org/index.php/MAC_address_spoofing), jak i również [na
stronie riseup](https://riseup.net/pl/security/network-security/mac-address).
