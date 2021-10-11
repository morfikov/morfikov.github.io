---
author: Morfik
categories:
- linux
date:    2021-09-04 12:36:00 +0200
lastmod: 2021-09-04 12:36:00 +0200
published: true
status: publish
tags:
- debian
- udev
- backlight
- klawiatura
- lenovo
- thinkpad
- T430
- initramfs
- initrd
GHissueID: 325
title: Kontrola podświetlenia klawiatury via UDEV (backlight)
---

Ostatnio na [polskim forum linux mint pojawił się wątek][1], w którym jeden z użytkowników miał
problem z ogarnięciem podświetlania klawiatury. Chodzi generalnie o start komputera z włączonym
backlight'em klawiatury, przez co trzeba to podświetlenie manualnie wyłączać za każdym razem po
uruchomieniu się systemu. W przypadku mojego laptopa Lenovo ThinkPad T430, taka sytuacja co prawda
nie występuje i system uruchamia się naturalnie ze zgaszoną klawiaturą. Niemniej jednak,
zainteresował mnie ten problem tyle, że w drugą stronę -- chodzi o możliwość startu systemu z
włączonym podświetleniem klawiatury, bo ustawienia BIOS/EFI/UEFI mojego laptopa tego aspektu pracy
komputera nie są w stanie w żaden sposób skonfigurować. Okazało się, że zaimplementowanie tego typu
funkcjonalności nie jest jakoś specjalnie trudne i można bez większego trudu ogarnąć backlight
klawiatury przy pomocy prostej reguły dla UDEV'a.

<!--more-->
## Sterownie podświetleniem klawiatury

Jeśli nasz laptop posiada podświetlaną klawiaturę, to zapewne udostępnia w systemie stosowne pliki
w katalogu `/sys/` , przez zapis których możemy kontrolować fakt zapalenia lub zgaszenia
backlight'a (zakładając oczywiście, że odpowiedni moduł kernela został wcześniej załadowany do
pamięci RAM). W przypadku tego ThinkPad'a T430, te potrzebne nam pliki znajdują się w katalogu
`/sys/devices/platform/thinkpad_acpi/leds/tpacpi::kbd_backlight/` . Nas najbardziej interesuje
pozycja `brightness` oraz też po części `max_brightness` .

Z pliku `max_brightness` możemy odczytać maksymalną wartość jaką możemy zapisać do pliku
`brightness` . W tym przypadku w `max_brightness` mamy wartość `2` . Zatem mamy aż trzy możliwe
kombinacje, które mogą powędrować do pliku `brightness` , tj. `0` , `1` i `2` . W przypadku ustawienia
wartości `0` ,  backlight klawiatury zostanie wyłączony. Jeśli prześlemy wartość `1` , to zostanie
włączone lekkie podświetlenie przycisków. Natomiast w przypadku określenia wartości `2` ,
podświetlenie klawiatury zostanie aktywowane w pełni.

Mój laptop uruchamia się ze zgaszoną klawiaturą i przy starcie systemu nawet nie próbuje jej
zapalić. Takie jest chyba standardowe zachowanie ale w poruszonym wątku na forum, system zdawał
się zachowywać zupełnie inaczej, tj. zapalał podświetlenie klawiatury w momencie uruchamiania się
środowiska graficznego. Ja ze środowisk graficznych nie korzystam, m.in. właśnie z takiego powodu,
tj. teoretycznie wszystko działa jak należy ale jakieś ustawienia środowiska graficznego coś gdzieś
psują i potem mamy takie dziwne problemy.

Tak czy inaczej, ja bardzo często siedzę przy kompie wieczorami/nocami, no i do tego mój system
jest w pełni zaszyfrowany, co wymaga ode mnie wpisywania hasła za każdym razem, gdy Debian się
uruchamia. W nocy zwykle mam zgaszone światełko, no i jak laptop restartuje, to przy wprowadzaniu
hasła muszę klawiaturę sobie zapalić. Do tej pory nawet nie zawracałem sobie głowy tym problemem,
bo co to jest zapalić raz na jakiś czas klawiaturę wciskając `Fn` + `space` ? Niemniej jednak,
zafascynowany problemem użytkownika forum mint'a postanowiłem ogarnąć sobie te światełka pod
przyciskami klawiatury, tak by system właśnie startował z włączoną klawiaturą, a po wpisaniu
hasełka do kontenera LUKS, ten backlight wyłączał.

## Usługa systemd-backlight@.service

W systemach linux, w których zainstalowany jest już domyślnie systemd, to [usługa
systemd-backlight@.service][2] ma odpowiadać za zadanie zapisania przy zamykaniu systemu i
odtwarzania przy jego starcie stanu podświetlenia i to nie tylko klawiatury ale również monitora i
innych jeszcze podzespołów. Ta usługa nie działa w jakiś magiczny sposób, a jedynie operuje ona na
zasadzie zapisania określonej wartości do plików w katalogu `/var/lib/systemd/backlight/` przy
wyłączaniu systemu. W późniejszym czasie, gdy uruchamiamy komputer, te pliki są odczytywane i
system wie jakie podświetlenie klawiatury (czy monitora) ma sobie ustawić.

Jeśli chodzi o system mojej maszyny, to w tym powyższym katalogu mamy takie oto dwa pliki:

    #  ls -al /var/lib/systemd/backlight/
    total 24
    drwxr-xr-x  2 root root 4096 2020-12-09 13:56:54 ./
    drwxr-xr-x 12 root root 4096 2021-08-18 15:30:35 ../
    -rw-r--r--  1 root root    4 2021-09-03 22:18:07 pci-0000:00:02.0:backlight:intel_backlight
    -rw-r--r--  1 root root    2 2021-09-03 22:18:07 platform-thinkpad_acpi:leds:tpacpi::kbd_backlight

Plik `pci-0000:00:02.0:backlight:intel_backlight` jest odpowiedzialny za odtworzenie podświetlenia
monitora, zaś plik `platform-thinkpad_acpi:leds:tpacpi::kbd_backlight` jest od odtworzenia stanu
podświetlenia klawiatury. Podglądając wartości w tych plikach możemy ustalić, czy i z jaką
intensywnością będzie zapalone podświetlenie konkretnego komponentu naszego komputera podczas
startu maszyny.

Ten powyższy katalog powinien być pierwszym miejscem, w które zaglądamy w przypadku problemów z
backlight'em. Jeśli ten katalog jest pusty, prawdopodobnie usługi mające na celu zapisanie i
odtworzenie stanu podświetlenia nie działają albo działają błędnie i powinniśmy tym tropem podążyć.

Usługi do zapisu/odtworzenia stanu backlight'u są generowane automatycznie -- chodzi o tę część po
znaku `@` . W przypadku mojego laptopa są dwie usługi, a ich pliki mają nazwy
`systemd-backlight@backlight:intel_backlight.service` oraz
`systemd-backlight@leds:tpacpi::kbd_backlight.service` . Te wartości po znaku `@` są brane katalogu
`/sys/class/backlight/` oraz `/sys/class/leds/` (pozycje zawierające `backlight` ) . Tak czy
inaczej upewnijmy się, że stosowne usługi zostały uruchomione i nie zgłosiły żadnych błędów:

    # systemctl status systemd-backlight@leds:tpacpi::kbd_backlight.service --no-pager -l
    ● systemd-backlight@leds:tpacpi::kbd_backlight.service - Load/Save Screen Backlight Brightness of leds:tpacpi::kbd_backlight
         Loaded: loaded (/lib/systemd/system/systemd-backlight@.service; static)
         Active: active (exited) since Fri 2021-09-03 22:19:00 CEST; 1h 48min ago
           Docs: man:systemd-backlight@.service(8)
        Process: 1449 ExecStart=/lib/systemd/systemd-backlight load leds:tpacpi::kbd_backlight (code=exited, status=0/SUCCESS)
       Main PID: 1449 (code=exited, status=0/SUCCESS)

    Sep 03 22:19:00 morfikownia systemd[1]: Starting systemd-backlight@leds:tpacpi::kbd_backlight.service...
    Sep 03 22:19:00 morfikownia systemd[1]: Finished systemd-backlight@leds:tpacpi::kbd_backlight.service.

    # systemctl status systemd-backlight@backlight:intel_backlight.service --no-pager -l
    ● systemd-backlight@backlight:intel_backlight.service - Load/Save Screen Backlight Brightness of backlight:intel_backlight
         Loaded: loaded (/lib/systemd/system/systemd-backlight@.service; static)
         Active: active (exited) since Fri 2021-09-03 22:19:00 CEST; 1h 48min ago
           Docs: man:systemd-backlight@.service(8)
        Process: 1448 ExecStart=/lib/systemd/systemd-backlight load backlight:intel_backlight (code=exited, status=0/SUCCESS)
       Main PID: 1448 (code=exited, status=0/SUCCESS)

    Sep 03 22:19:00 morfikownia systemd[1]: Starting systemd-backlight@backlight:intel_backlight.service...
    Sep 03 22:19:00 morfikownia systemd[1]: Finished systemd-backlight@backlight:intel_backlight.service.

Jeśli te usługi działają prawidłowo, to zapisywanie i odtwarzanie wartości podświetlenia klawiatury
czy monitora powinno również działać bez problemu.

## Backlight i UDEV

Początkowo by osiągnąć zadowalające mnie rozwiązanie, kombinowałem coś z własną usługą dla systemd.
W zasadzie wszystkie informacje do skonstruowania takiej usługi posiadamy, tj. namierzyliśmy pliki
sterujące backlight'em klawiatury i określiliśmy jakie wartości one przyjmują oraz co się stanie po
zapisaniu konkretnej wartości. Niemniej jednak, usługa systemd się do sterowania backlight'em
kompletnie nie nadaje. Wystarczy sobie zadać pytanie, co jeśli ktoś nie używa systemd? Albo też, co
jeśli ktoś ma zaszyfrowany system? W obu tych przypadkach backlight klawiatury na wczesnym etapie
startu systemu się nie zapali. Dlatego trzeba było znaleźć lepsze rozwiązanie i tak wróciłem do
pierwszego pomysłu jaki przyszedł mi do głowy, czyli do [reguł UDEV'a][3]. Z początku nie miałem za
bardzo pojęcia jak napisać regułę dla backlight'a klawiatury ale ostatecznie się udało.

Generalnie to trzeba wyjść ze ścieżki do samego podświetlenia klawiatury albo monitora (w
zależności od tego z czym mamy problem), przykładowo:

    # udevadm info -a -p /sys/class/leds/tpacpi::kbd_backlight
    # udevadm info -a -p /sys/class/backlight/intel_backlight

W ten sposób uzyskamy potrzebne informacje do skonstruowania reguły:

    # udevadm info -a -p /sys/class/leds/tpacpi::kbd_backlight
      ...
      looking at device '/devices/platform/thinkpad_acpi/leds/tpacpi::kbd_backlight':
        KERNEL=="tpacpi::kbd_backlight"
        SUBSYSTEM=="leds"
        ...
        ATTR{brightness}=="0"
        ...
      looking at parent device '/devices/platform/thinkpad_acpi':
        ...
        DRIVERS=="thinkpad_acpi"
        ...

Stworzenie reguły przy posiadaniu tych powyższych informacji jest już banalnie proste. Tworzymy
sobie plik `/etc/udev/rules.d/99-kbd_backlight.rules` i wrzucamy do niego poniższą linijkę:

    SUBSYSTEM=="leds", ACTION=="add", KERNEL=="tpacpi::kbd_backlight", DRIVERS=="thinkpad_acpi", \
      ATTR{brightness}="1"

Ta regułka mówi tyle, że gdy zostanie wykryte podświetlenie w klawiaturze naszego laptopa, to ma
zostać mu ustawiona wartość atrybutu `brightness` na `1` , czyli w przypadku mojej klawiatury
będzie to lekkie podświetlenie.

UDEV ma tę przewagę nad systemd, że działa również w fazie initramfs. Zatem jeśli przebudujemy
sobie teraz obraz initrd, to tak napisane reguły zostaną w nim uwzględnione i przy starcie systemu
(zaraz po wypakowaniu obrazu initrd), UDEV wykryje klawiaturę i zapali jej podświetlenie. By
wygenerować nowy obraz initrd, w terminalu wpisujemy poniższe polecenie:

    # update-initramfs -u -k all

W dokładnie taki sam sposób możemy ustawić intensywność podświetlenia dla monitora, przykładowo:

    SUBSYSTEM=="backlight", ACTION=="add", KERNEL=="intel_backlight", DRIVERS=="i915", \
      ATTR{brightness}="665"

## Zapalenie klawiatury jedynie na potrzeby wpisania hasła LUKS

Jako, że w tym artykule chodzi o włączenie podświetlenia klawiatury jedynie we wczesnej fazie
initramfs/initrd, by bardziej komfortowo móc wpisać hasło do kontenera LUKS, to trzeba też było się
zastanowić w jaki sposób na późniejszym etapie fazy rozruchu systemu ten backlight klawiatury
wyłączyć. Na szczęście nie trzeba było tutaj długo kombinować, bo przecie systemowa usługa od
odtworzenia zapisanego stanu podświetlenia klawiatury działa i ma się dobrze.

Zatem jak wygląda obecnie zachowanie się klawiatury? Włączamy komputer, ten się uruchamia
początkowo z wyłączonym podświetleniem klawiatury. Po wyborze systemu operacyjnego (kernela)
ładowany jest do pamięci obraz initramfs/initrd i z niego czytane są reguły UDEV'a przy wykrywaniu
sprzętu, co powoduje zapalenie się klawiatury. Po chwili na ekranie pojawia się prompt, by wpisać
hasło do kontenera LUKS. Po wpisaniu hasła następuje dalsza część rozruchu systemu, gdzie
uruchamiana jest usługa od odtworzenia stanu backlight'u klawiatury, a że przy zamykaniu systemu
podświetlenie klawiatury było wyłączone, to podczas startu ten stan zgaszonego podświetlenia
klawiatury zostanie odtworzony, co efektywnie gasi backlight. Dalej system się uruchamia już przy
zgaszonej klawiaturze. I o takie zachowanie dokładnie nam chodziło.


[1]: https://forum.linuxmint.pl/showthread.php?tid=1739&pid=13156
[2]: https://www.freedesktop.org/software/systemd/man/systemd-backlight@.service.html
[3]: /post/udev-czyli-jak-pisac-reguly-dla-urzadzen/
