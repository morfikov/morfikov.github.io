---
author: Morfik
categories:
- Linux
date: "2015-06-11T19:12:27Z"
date_gmt: 2015-06-11 17:12:27 +0200
published: true
status: publish
tags:
- debian
- pendrive
- live
- luks
- szyfrowanie
GHissueID: 113
title: Persistence, czyli zachowanie zmian w systemie live
---

Systemy live mają jedną ale za to dość dającą się odczuć wadę, mianowicie chodzi o to, że po
wypaleniu takiego obrazu, nie mamy możliwości zachowania zmian. Nawet jeśli doinstalujemy nowy
pakiet czy wyedytujemy jakiś plik, to zmiany wprowadzone przez nas są jedynie tymczasowe, bo
dokonywane w pamięci operacyjnej RAM. W efekcie jeśli uruchomimy taki system ponownie, będziemy
zmuszeni przeprowadzać raz jeszcze wszystkie poprzednie czynności pod kątem jego dostosowania. W
przypadku cd/dvd nie mamy praktycznie żadnego pola manewru. Inaczej jednak ma się sprawa jeśli
chodzi o pendrive, bo tutaj możemy utworzyć osobną partycję, gdzie będą przechowywane wszystkie
zmiany jakich dokonamy.

<!--more-->
## Nakładka na system live

Mowa oczywiście o mechanizmie zwanym [persistence][1] i jak wspomniałem na wstępie, polega on na
utworzeniu osobnej partycji, która po starcie systemu zostanie zamontowana jako nakładka na system
plików obrazu live, tzw. overlay. Mamy tu do czynienia z rozwiązaniem znanym choćby z routerów i
ich alternatywnego oprogramowania OpenWRT, z tym, że tam jest ono stosowane z powodu zbyt małego
flash'a routera, który jest rzędu paru MiB i miłośnicy oprogramowania nie są w stanie się zmieścić
na tak małej przestrzeni, wobec czego zaciągają do pracy pendrive i tam lokują m.in. nowo
instalowane pakiety.

Jeśli korzystamy z [alternatywnego przygotowania nośnika][2] pod system live, możemy bez wahania
utworzyć kolejną partycję. W przypadku gdy wypalamy obraz przy pomocy `dd` , za każdym razem gdy to
robimy, będziemy tracić również i dane zgromadzone na tej drugiej partycji w wyniku nadpisania MBR.
Ja się posłużę tym samym pendrive, który wykorzystywałem w przytoczonym linku.

Nie będę opisywał tutaj jak stworzyć nową partycję przy pomocy `gparted`, bo to raczej każdy z nas
powinien umieć zrobić, nawet jeśli wymaga to skurczenia którejś z istniejących. Poniżej mamy już
przygotowany odpowiedni układ partycji:

![gparted-persistence-live](/img/2015/06/1.gparted-persistence-live.png#big)

W tym przypadku system live rezyduje na partycji drugiej. Na trzeciej zaś będzie persistence.
Montujemy zatem tę trzecią partycję i tworzymy na niej plik `persistence.conf` :

    # mount /dev/sdb3 /mnt
    # touch /mnt/persistence.conf

Plik `persistence.conf` zawierać będzie konfigurację folderów, które życzymy sobie nadpisać, tj.
których zmiany będą zapisywane na tej partycji. Możemy zdefiniować pojedyncze foldery, np `/home/`
albo `/etc/` , możemy także wziąć pod uwagę monitoring wszelkich zmian przez podanie `/` . Wpisy
definiujemy jeden pod drugim w poniższej postaci:

    /home/    union

W przypadku gdy definiujemy katalog `/` , trzeba brać pod uwagę, że przy wgrywaniu nowych obrazów
live będą występować problemy uniemożliwiające start systemu, co jest zrozumiałe, bo macierzysty
system uległ zmianie i wprowadzone zmiany nie dotyczą już niego. Konfiguracja użytkownika jest
relatywnie bezpieczna, podobnie jak i katalog `/etc/`. Oczywiście to nie znaczy, że wprowadzanie
zmian do całego systemu plików jest złe. Zawsze można przecież wyczyścić tę partycję.

Zapisujemy plik i przechodzimy do konfiguracji systemu live. Interesują nas pliki zlokalizowane w
katalogu `/media/morfik/good/extlinux/` (nazwa tego katalogu może się różnić w zależności od
wgranego bootloader'a), a konkretnie `live.cfg` . To tam dopisujemy parametry dla kernela. Tworzymy
zatem dodatkowy wpis uwzględniając opcję `persistence` , przykładowo:

    label live-amd64-persistence
          menu label ^Live-persistence (amd64)
          menu default
          linux /live/vmlinuz
          initrd /live/initrd.img
          append boot=live components persistence quiet splash

By cały mechanizm zadziałał, trzeba dostosować odpowiednio etykietę partycji. Choć jeśli przyjrzymy
się obrazkowi powyżej, to już ten krok powinniśmy mieć za sobą.

Domyślnie system live poszukuje partycji, których etykietą jest `persistence` . Jeśli z jakichś
powodów chcemy mieć inną nazwę, to istnieje także parametr dla kernela, który może skonfigurować
system live by uwzględnił ją przy przeszukiwaniu partycji. W konfiguracji powyżej, w linijce z
`append boot` trzeba dopisać `persistence-label=dane` , gdzie `dane` , to etykieta partycji.

## Szyfrowany persistence

Na partycji persistence, jako że zawiera system plików w trybie do zapisu, mogą znajdować się
poufne dane konfiguracyjne, np. hasła do aplikacji czy kont online. Z tego powodu istnieje też
możliwość jej zaszyfrowania. W tym celu będzie nam potrzebny kontener [LUKS][3].

Nie będę tutaj opisywał tworzenia takiego kontenera na dane, bo to zagadnienie na osobny artykuł.
Poniższe kroki zostaną przeprowadzone przy założeniu, że już taki zaszyfrowany volumin posiadamy i
wiemy jak z niego korzystać.

Po otwarciu kontenera, nadajemy etykietę voluminowi przy pomocy narzędzia `tune2fs` :

    # cryptsetup luksOpen /dev/sdb3 sdb3
    # tune2fs -L persistence /dev/mapper/sdb3

Oczywiście dalszy schemat z tworzeniem i edycją pliku `persistence.conf` jest taki sam, co w
przypadku zwykłego persistence. Jedyna różnica, jaką da się zauważyć, to dodatkowy parametr
`persistence-encryption=luks`, który musimy dopisać do linijki kernela w konfiguracji bootloader'a.
Dla przypomnienia, u mnie to jest plik `/media/morfik/good/extlinux/live.cfg` :

    label live-amd64-persistence
          menu label ^Live-persistence (amd64)
          menu default
          linux /live/vmlinuz
          initrd /live/initrd.img
          append boot=live components persistence persistence-encryption=luks quiet splash

Po tym zabiegu jeśli spróbujemy odpalić system live z tej pozycji, zostaniemy poproszeni o wpisanie
hasła w celu odblokowania kontenera.


[1]: https://live-team.pages.debian.net/live-manual/html/live-manual/customizing-run-time-behaviours.en.html#547
[2]: /post/jak-wgrac-system-live-na-uszkodzony-pendrive/
[3]: https://pl.wikipedia.org/wiki/Linux_Unified_Key_Setup
