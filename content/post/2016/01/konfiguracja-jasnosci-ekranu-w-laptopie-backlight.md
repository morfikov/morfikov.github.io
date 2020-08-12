---
author: Morfik
categories:
- Linux
date: "2016-01-05T21:52:41Z"
date_gmt: 2016-01-05 20:52:41 +0100
published: true
status: publish
tags:
- xserver
- systemd
- monitor
title: Konfiguracja jasności ekranu w laptopie (backlight)
---

Linux nigdy nie był popularnym systemem operacyjnym na stacjach klienckich. Może i cieszy się sporym
wzięciem wśród środowisk serwerowych ale jego niezbyt prosta obsługa nie przemawia do przeciętnego
użytkownika komputera. Wraz z rozwojem technologi, zmienił się też rodzaj urządzeń na jakich
obecnie pracujemy. W dzisiejszych czasach liczy się przede wszystkim mobilność i praktycznie każdy z
nas miał już prawdopodobnie do czynienia z laptopami. Producenci tych urządzeń lubią nam wciskać
windowsa z dopiskiem, że ten laptop bez niego nie będzie w ogóle nam działał. Już od dość dawna
szereg laptopów jest w stanie pracować pod linux'em ale, jakby nie patrzeć, sporo rzeczy w nich nie
działa OOTB i trzeba poświęcić trochę czasu, by tę maszynę doprowadzić do porządku. W tym wpisie
zajmiemy się zagadnieniem podświetlania matrycy (backlight), tak by system był w stanie bez problemu
zapisać ustawienia jasności ekranu przy wyłączaniu komputera oraz, by potrafił je także wczytać
podczas fazy rozruchu.

<!--more-->
## Sterowanie backlight'em

Jeśli już odważyliśmy się na zainstalowanie linux'a na laptopie, lub też jedynie korzystamy z
systemu live, to przydałoby się nieco zapoznać z informacjami jakie system jest nam w stanie podać
jeśli chodzi w kwestii backlight. Przede wszystkim, w systemd do zapisywania jasności ekranu jest
wyznaczona odpowiednia usługa,
[systemd-backlight@.service](https://www.freedesktop.org/software/systemd/man/systemd-backlight@.service.html).
Musimy się zatem upewnić, że ta usługa jest włączona oraz, że działa poprawnie. Jako, że w nazwie
usługi mamy `@` , oznacza to, że faktyczna nazwa usługi wygląda nieco inaczej. By ją ustalić musimy
zajrzeć do katalogu `/sys/class/backlight/` , przykładowo:

    $ ls /sys/class/backlight/
    ./  ../  acpi_video0@  intel_backlight@

Generalnie rzecz biorąc, sterowanie jasnością matrycy może leżeć w kwestii sterowników do grafiki
lub też może odbywać się za sprawą zdarzeń [ACPI](https://pl.wikipedia.org/wiki/ACPI). W powyższym
przypadku wykorzystywane są obie te metody i to zwykle powoduje problemy. Może się okazać, że
jasność jest zarządzana nie przez ten moduł, który próbujemy skonfigurować lub też następuje
przepisanie ustawień w zależności od tego, który z powyższych zainicjuje się jako ostatni. By
wyeliminować większość problemów z backlight'em, musimy zdecydować się na tylko jedną opcję i
trzymać się jej cały czas.

Wartości aktualnego podświetlenia są przez systemd przechowywane w katalogu
`/var/lib/systemd/backlight/` . Są tam osobne pliki dla każdego z modułów, przykładowo:

    $ cat /var/lib/systemd/backlight/pci-0000:00:02.0:backlight:acpi_video0
    4

Te pliki są przepisywane za każdym razem, gdy tylko przyciśniemy na klawiaturze przycisk od zmiany
jasności ekranu. Przy starcie systemu, ta wartość jest podawana sterownikowi i ten ustawia
odpowiedni poziom podświetlenia. Dlatego właśnie nie możemy korzystać z obu modułów jednocześnie.

## Usługa systemd-backlight@.service

Standardowo systemd będzie próbował uruchamiać usługę `systemd-backlight@.service` dla każdego linku
wykrytego w `/sys/class/backlight/` . Musimy go zatem tego oduczyć. W przypadku tego laptopa,
`intel_backlight` nie działa za dobrze, zatem będziemy korzystać z `acpi_video0` . Musimy więc
zamaskować tę drugą usługę. Robimy to w poniższy sposób:

    # systemctl mask systemd-backlight@backlight\:intel_backlight

Sprawdzamy też czy ta pożądana przez nas usługa działa poprawnie:

![]({{< baseurl >}}/img/2016/01/1.backlight-systemd-usluga.png)

## Backlight i Xserver

Jeśli podczas fazy boot jesteśmy w stanie zaobserwować zmianę jasności ekranu, oznacza to, że
systemd robi swoje. Niemniej jednak, po zalogowaniu się w sesji graficznej, może nastąpić kolejna
zmiana jasności ekranu. Dzieje się tak dlatego, że Xserver prawdopodobnie korzysta z tego drugiego
sterownika, a że tamten ma ustawione inne wartości, to doświadczamy rozjaśnienie tudzież
przyciemnienia ekranu. By wybrnąć z tej sytuacji, musimy wskazać Xserver'owi, z którego modułu ma
korzystać.

Środowiska graficzne z reguły będą przepisywać ustawienia Xserver'a, dlatego też ta sekcja dotyczy
jedynie minimalistycznych środowisk opartych jedynie o menadżery okien, np. openbox.

Tworzymy zatem plik konfiguracyjny dla Xserver'a, `/etc/X11/xorg.conf.d/20-monitor.conf` i dodajemy
w nim ten poniższy kod:

    Section "Monitor"
          Identifier   "LVDS1"
    EndSection

    Section "Device"
          Identifier  "Device0"
          Driver      "intel"
          Option      "Backlight" "acpi_video0"
          BusID "PCI:0:2:0"
    EndSection

    Section "Screen"
          Identifier     "Screen0"
          Device         "Device0"
          Monitor        "LVDS1"
          DefaultDepth    24
    EndSection

Większość użytych wyżej opcji została wyjaśniona w artykule poświęconym [konfiguracji
monitora]({{< baseurl >}}/post/monitor-i-jego-konfiguracja-pod-linuxem/) i nie będę ich tutaj
ponownie przytaczał. Nas interesuje głównie opcja `Backlight` w sekcji `Device` . Ustawiamy ją w
oparciu o nazwę linku, który możemy znaleźć w katalogu `/sys/class/backlight` . W tym przypadku
określamy `acpi_video0` . W tej chwili zarówno systemd jak i Xserver będą korzystać z jednego
modułu do zmiany jasności matrycy w naszym laptopie.

## Niedziałające klawisze

Jeśli znajdujemy się w tak niefortunnej sytuacji, że klawisze sterujące podświetlaniem ekranu
zwyczajnie nie działają, to zawsze możemy skorzystać z odpowiednich narzędzi, by poziom jasności
sobie dostosować programowo. W debianie mamy dostępny min. pakiet `xbacklight` , który wyśmienicie
się nadaje do tego celu. Instalujemy go i wpisujemy w terminal poniższe polecenie:

    $ xbacklight -set 40%

## Wyłączenie acpi\_video0

Możemy także pokusić się o całkowite wyłączenie modułu ACPI odpowiedzialnego za backlight, a
sterowanie podświetleniem matrycy zostawić sterownikowi karty graficznej. By wyłączyć ten moduł,
musimy dopisać do linijki kernela opcję `acpi_backlight=vendor` . Poniżej przykład dla bootloader'a
extlinux:

    APPEND root=... acpi_backlight=vendor ...

Po ponownym uruchomieniu komputera, w katalogu `/sys/class/backlight/` powinien znajdować się
jedynie link `intel_backlight` . Jeśli zamierzamy korzystać z tego rozwiązania, to pamiętajmy też o
włączeniu odpowiedniej usługi systemd.

Jeśli chodzi jeszcze o wartości backlight'ów sterowników graficznych, to możemy tutaj zaobserwować,
np. 2431 (w stosunku do 5 w ACPI). W przypadku ACPI, mamy 10 punktów jasności, między którymi możemy
się przełączać. Natomiast jeśli chodzi o `intel_backlight` , to mamy możliwość nieco bardziej
płynnego przejścia. Choć moim zdaniem 10 poziomów jasności w zupełności wystarcza.

## Maksymalna i minimalna wartość podświetlenia

Starajmy się unikać granicznych wartości podświetlenia ekranu jakie możemy ustawić. Chodzi o to, że
może wystąpić dość dziwny błąd, który powoduje rozjechanie się ustawień. Nawet jeśli zmiana jasności
działa prawidłowo, to po osiągnięciu, np. maksymalnej jasności ekranu, ta wartość nie będzie już
podlegać dalszym zmianom. W efekcie czego, przy zamykaniu systemu zostanie zapisana maksymalna
wartość podświetlenia, mimo, że ekran był rozjaśniony, np. w 40%. Później wartość odpowiadająca za
100% zostanie wczytana przy starcie systemu.

Jeśli tego typu sytuacja nam się przytrafi, to pamiętajmy, że zawsze możemy deaktywować usługę
systemd i ręcznie zmienić wartość w plikach w katalogu `/var/lib/systemd/backlight/` .
