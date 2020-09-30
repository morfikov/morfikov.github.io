---
author: Morfik
categories:
- Linux
date: "2017-01-22T18:48:17Z"
date_gmt: 2017-01-22 17:48:17 +0100
published: true
status: publish
tags:
- debian
- monitor
title: Redshift i dostosowanie temperatury kolorów ekranu
---

Pewnie spotkaliście się już wiele razy z informacją, że ekrany naszych smartfonów, tabletów czy
komputerów szkodzą naszym oczom. Chodzi generalnie o to, że wyświetlacze LCD emitują światło
niebieskie, które w nadmiarze nie wpływa dla nas (i naszego wzroku) korzystnie, a przecie każdy z
nas siedzi godzinami przed komputerem. W zasadzie te negatywne efekty ciągłego spoglądania na
wyświetlacz nasilają się zwłaszcza wieczorami i w nocy. Jeśli pracujemy na laptopie do późna i przy
okazji mamy na tej maszynie zainstalowaną jakąś dystrybucję linux'a, np. Debian, to możemy złagodzić
skutki zmęczenia oczu przez dobór nieco innej temperatury kolorów wyświetlanego obrazu. W tym
zadaniu może nam pomóc oprogramowanie [redshift][1] i to jemu będzie poświęcony poniższy wpis.

<!--more-->
## Redshift czy f.lux

Na `redshift` natknąłem się przypadkiem szukając informacji na temat nieco podobny ale związany
bardziej ze smartfonami niż z laptopami czy desktopami. Generalnie rzecz biorąc, to pierw wpadł mi w
oczy [f.lux][2]. Niemniej jednak, może i `f.lux` ma klienta dla linux'a ale nie jest on OpenSource.
Dlatego właśnie moje poszukiwania trwały trochę dłużej i w taki oto sposób natrafiłem właśnie na
[otwartoźródłową alternatywę][3] w postaci `redshift`.

Jak możemy się łatwo domyśleć, tylko jedna opcja z tych dwóch wymienionych powyżej trafiła do
repozytorium Debiana, no i dlatego też to o `redshift` postanowiłem napisać kilka słów.

## Pakiety redshift, redshift-gtk oraz geoclue-2.0

Przy instalacji pakietu `redshift` , w rekomendowanych zależnościach jest zwracany inny pakiet, tj.
`geoclue-2.0` . Nie jest on obowiązkowy z punktu widzenia działania samej aplikacji ale jak jego
nazwa może wskazywać, ma on coś wspólnego z geolokalizacją.

W zasadzie mamy do wyboru dwie opcje. Możemy manualnie podać dane lokalizacji, w której się
znajdujemy. Możemy też poprosić automat o to by ustalił naszą pozycję i dane przesłał do
`redshift` . Jeśli chcemy manualnie określać położenie, to możemy odpuścić sobie ten pakiet
`geoclue-2.0` , co nam zaoszczędzi instalacji kilku zależności.

Warto też wiedzieć, że w repozytorium Debiana mamy także dostępny pakiet `redshift-gtk` , który
dostarcza dość ubogiej nakładki na `redshift` . Moim zdaniem, to GUI jest zbędne i pociąga za sobą
kolejne zależności, bez których spokojnie można się obejść w minimalistycznych środowiskach
graficznych, czy też środowiskach opartych o menadżer okien typu OpenBOX. Ja generalnie tutaj skupię
się jedynie na skonfigurowaniu i obsłudze tekstowej wersji aplikacji.

## Konfiguracja redshift przez plik ~/.config/redshift.conf

W zasadzie mamy dwie możliwości podawania parametrów konfiguracyjnych programowi `redshift` .
Pierwsza opcja to wywołanie tej aplikacji standardowo z wiersza poleceń i podanie w komendzie
stosownych parametrów i ich wartości. Drugi sposób zakłada stworzenie pliku
`~/.config/redshift.conf` i umieszczenie w nim całej konfiguracji. My z tej drugiej opcji właśnie
skorzystamy. Tworzymy zatem ww. plik i wrzucamy do niego poniższą zawartość:

    [redshift]
    temp-day=5700
    temp-night=3600
    ;brightness=0.9
    ;gamma=0.8
    location-provider=manual
    ;location-provider=geoclue2

    [manual]
    lat=52.22
    lon=21.01

W pliku `~/.config/redshift.conf` mamy dwie sekcje: `[redshift]` zawierającą konfigurację aplikacji,
oraz `[manual]` zawierającą dane lokalizacji. Komentarze w tym pliku są oznaczane przez `;` .

### Sekcja [redshift]

Parametrów, które możemy umieścić w tej sekcji, jest naturalnie więcej i wszystkie z nich są
dostępne w [man redshift][4]. Te powyżej określone rzeczy są niezbędnym minimum, które powinniśmy
dostosować według swoich upodobań podczas eksperymentowania z wpisywanymi wartościami.

Generalnie rzecz biorąc, `redshift` potrafi dostosować temperaturę kolorów wyświetlanego obrazu w
zależności od tego czy za oknem panuje dzień czy noc. Dlatego też możemy podać różne wartości w
`temp-day` oraz `temp-night` . Wartości, które trzeba tutaj uwzględnić powinny pasować do poziomu
światła w naszym pokoju. Żarówki mają na opakowaniu określoną temperaturę barw i generalnie
ustawiamy taką, jaką odczytaliśmy z opakowania. Zwykle jest to około 3000K-4000K. W dzień zaś
temperatura kolorów powinna pasować do tej, którą mamy za oknem, zwykle 5500K-6500K.

Do ustalenia czy panuje akurat dzień czy noc, `redshift` wykorzystuje dane lokalizacyjne dla
konkretnego punktu na mapie w danym dniu roku. Innymi słowy potrzebuje on wiedzieć kiedy w naszej
lokalizacji wschodzi i zachodzi słońce. Te dane są dostarczane przez system ale lokalizację trzeba
jakoś określić i do tego celu używamy `location-provider` . W tym przypadku jest on ustawiony na
`manual` i trzeba będzie określić te dane ręcznie w sekcji `[manual]` , która jest opisana poniżej.

Parametry `gamma` i `brightness` mają na celu dostosowanie jasności obrazu. Zwykle wyświetlacze w
monitorach oferują zmianę jasności i w zasadzie można te parametry pominąć. Niemniej jednak, wartość
minimalna jasności jaka jest do określenia czasem może być dla nas ciągle za wysoka. W takim
przypadku, możemy jeszcze bardziej przyciemnić ekran właśnie korzystając z tych parametrów. Można
również osobno określić `gamma` i `brightness` dla koloru czerwonego, zielonego i niebieskiego
(RGB). Wystarczy podać trzy wartości w formie: `0.8:0.8:0.8` .

### Koordynaty w sekcji [manual]

W parametrze `location-provider` określiliśmy sobie, że mamy zamiar ręcznie sprecyzować koordynaty,
które wskażą nasze położenie. Musimy podać dwie współrzędne: szerokość geograficzną (latitude) i
długość geograficzną (longitude). Te dane można spisać z GPS lub też odszukać na necie położenie
miasta, w którym przebywamy. Wartości `lat=52.22` oraz `lon=21.01` są dla Warszawy.

## Autostart

W pakiecie `redshift` jest zawarta usługa dla systemd. Ta usługa może zostać włączona globalnie dla
wszystkich użytkowników lub też dla każdego użytkownika z osobna. Jeśli zamierzamy włączyć ją dla
wszystkich użytkowników w systemie, to wydaje poniższe polecenie:

    # systemctl --global enable redshift.service

Jeśli zaś zamierzamy włączyć tę usługę jedynie dla konkretnego użytkownika, to wpisujemy poniższe
polecenie:

    $ systemctl --user enable redshift.service

Po zresetowaniu systemu czy też samego środowiska graficznego, nasz ekran powinien przybrać nieco
cieplejsze barwy. Trochę to wygląda jakbyśmy używali komputera na Marsie (fotka ze strony projektu):

![](/img/2017/01/001.redshift-wyswietlacz-laptop-komputer-efekt.png#huge)

## Notyfikacje

Standardowo `redshift` nie dostarcza żadnych notyfikacji przy zdarzeniach przełączania temperatury
barw kolorów. Jeśli interesują nas jednak tego typu informacje, to naturalnie możemy sobie mechanizm
powiadamiania dorobić. Wymagane jest tylko posiadanie w systemie pakietu `libnotify-bin` , który
zwykle jest dostępny w minimalnych środowiskach. Trzeba również dodać skrypt, który wywoła
odpowiedni monit. Skrypt zaś trzeba wgrać do katalogu `~/.config/redshift/hooks/` . Poniżej jest
przykład takiego skryptu:

    #!/bin/sh
        case $1 in
            period-changed)
                exec notify-send "Redshift" "Period changed to $3"
        esac

Pamiętajmy, by temu skryptowi nadać prawa wykonywania.


[1]: http://jonls.dk/redshift/
[2]: https://justgetflux.com/
[3]: https://github.com/jonls/redshift/
[4]: http://manpages.ubuntu.com/manpages/zesty/en/man1/redshift.1.html
