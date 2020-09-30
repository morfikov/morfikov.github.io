---
author: Morfik
categories:
- Android
date: "2017-01-15T18:20:11Z"
date_gmt: 2017-01-15 17:20:11 +0100
published: true
status: publish
tags:
- smartfon
- twrp
- marshmallow
title: Backup partycji /data/ w smartfonach przez recovery TWRP
---

W artykułach dotyczących przeprowadzania procesu root na smartfonach Neffos
[Y5](/post/android-root-smartfona-neffos-y5-od-tp-link/) oraz
[Y5L](/post/android-root-smartfona-neffos-y5l-tp-link/) był pokazany sposób na
dokonanie backup'u całego flash'a tych urządzeń. Jeśli Android w naszym telefonie jest ukorzeniony
albo chociaż mamy wgrany obraz TWRP na partycję `/recovery/` , to jesteśmy w stanie przeprowadzać
regularny backup wszystkich danych użytkownika z poziomu trybu recovery. Proces takiego backup'u
będzie się nieco różnił w stosunku do tego opisywanego w wyżej podlinkowanych artykułach. W tym
przypadku nie będziemy robić kopii binarnej, a jedynie zgramy sobie wszystkie pliki znajdujące się
na partycji `/data/` . W tym artykule zostanie pokazany sposób na przeprowadzanie procesu kopii
zapasowej w smartfonie Neffos Y5. Niemniej jednak, taki regularny backup można przeprowadzać
praktycznie w każdym smartfonie posiadającym recovery z TWRP.

<!--more-->
## Kopia binarna czy kopia plików

Dysponując obrazem z recovery TWRP w zasadzie jesteśmy w stanie przeprowadzić dwa rodzaje kopii
zapasowych: kopia binarna i kopia zwykłych plików. Nas średnio interesuje kopia binarna, a to ze
względu na fakt zgrywania z partycji `/data/` każdego pojedynczego bita danych.

Partycja `/data/` jest największą partycją w naszych telefonach i zwykle ma ona rozmiar 10-12 GiB
(przy rozmiarze flash'a 16 GiB). W tych bardziej wypasionych smartfonach, ta partycja może mieć
sporo większy rozmiar. Jeśli teraz byśmy stworzyli kopię binarną takiej partycji, to system zrzuci
nam wszystkie dane na niej zawarte wliczając w to wolne miejsce. W efekcie możemy mieć na tej
partycji zajętych 2 GiB, a i tak zostanie utworzony plik o rozmiarze tych 12 GiB. Marnujemy zatem
sporo miejsca, bo przecież taki backup trzeba gdzieś przechowywać.

W przypadku kopii zapasowej plików, system zgrywa poszczególne pliki z partycji `/data/` i robi z
nich skompresowane archiwum, coś jak paczka `.zip` . W porównaniu do kopii binarnej, takie spakowane
archiwum plików zajmuje parokrotnie mniej miejsca, no i oczywiście sam proces backup'u (i
późniejszego kopiowania danych z telefonu na komputer) trwać będzie o wiele krócej.

## Odblokowany bootloader

Technicznie rzecz biorąc, tryby recovery w smartfonach z Androidem mają różne opcje. Niektóre z nich
oferują możliwość zrobienia backup'u partycji `/data/` , a inne takiej funkcji nie posiadają. Jeśli
stock'owy obraz partycji `/recovery/` nie daje nam możliwości przeprowadzenia procesu kopii
zapasowej, to nie pozostaje nam nic innego jak sięgnięcie po obraz recovery z TWRP.

Problem z tym obrazem jest jednak taki, że musimy go albo wgrać na partycję `/recovery/` , albo tez
odpalić go w pamięci telefonu, tak by nie wprowadzać żadnych zmian na flash'u urządzenia. Rzecz w
tym, by móc tego typu zabieg przeprowadzić, trzeba odblokować bootloader, a to, jak zapewne wiemy,
inicjuje proces Factory Reset i czyści wszystkie dane użytkownika zgromadzone na partycji `/data/` .
Mając na uwadze ten fakt, kopia zapasowa plików użytkownika przez tryb recovery z TWRP dotyczy tylko
i wyłącznie telefonów z odblokowanym bootloader'em. Zakładam w tym miejscu, że mamy już odblokowany
bootloader w naszym smartfonie.

## Kopia zapasowa z poziomu aplikacji

Inny problem, jaki może mieć dla nas znaczenie przy kopii zapasowej przez tryb recovery, to potrzeba
chwilowego wyłączenia telefonu. Ten sposób nie jest zatem optymalny dla użytkowników, którzy muszą
być ciągle online i nie mogą na te kilkanaście minut rozłączyć się ze światem.

Jest za to kilka programików, które umożliwiają zrobienie tego typu backup'u z poziomu działającego
telefonu. Ja jednak nigdy nie testowałem tego oprogramowania i za bardzo nie mogę nic na jego temat
powiedzieć. Zwykle też oferowane za sprawą takich programików rozwiązania są płatne. Z kolei zaś
obrazy TWRP, które możemy wgrać na swój telefon, mamy za free.

## Uruchamianie smartfona w trybie recovery

Zwykle jak korzystamy z telefonu przez dłuższy czas, to jest wielce prawdopodobne, że sporo rzeczy
zmieniliśmy w konfiguracji takiego urządzenia. Mamy zapewne też wgranych szereg niestandardowych
aplikacji, których ustawienia również zostały przez nas dostosowane do naszych potrzeb. O ile pliki
graficzne, dźwiękowe czy też video można sobie bez problemu zgrać na komputer, o tyle właśnie tych
ustawień telefonu za bardzo nie mamy jak sobie skopiować.

Wejdźmy zatem w tryb recovery naszego telefonu. W przypadku, gdy obraz TWRP jest wgrany na flash
smartfona, to wyłączamy urządzenie i przyciskamy jednocześnie przyciski VolumeUp + Power. Jeśli zaś
chcemy uruchomić TWRP w pamięci RAM telefonu, to musimy uruchomić to urządzenie w trybie
bootloader'a i zaaplikować obraz z partycją `/recovery/` za pomocą narzędzia `fastboot` .
[Instalacja i konfiguracja fastboot pod
linux](/post/android-jak-zainstalowac-adb-i-fastboot-pod-linux/) jest opisana w
osobnym wątku. Niezależnie od wybranego sposobu, naszym oczom powinno ukazać się menu TWRP podobne
do tego poniżej (w opcjach można wybrać język polski):

![](/img/2017/01/001.backup-kopia-zapasowa-smartfon-data-android-twrp-kopia.png#medium)

Jak widać na obrazku, mamy dwie pozycje: `Kopia` i `Przywróć`. Jeśli zamierzamy dokonać backup'u, to
naturalnie przechodzimy do pozycji `Kopia`. Jeśli zaś zamierzamy uprzednio utworzoną kopię zapasową
odtworzyć, to przechodzimy do `Przywróć` .

### Tworzenie kopii zapasowej partycji /data/ przez TWRP

Po przejściu do pozycji `Kopia` zostaną nam zaprezentowane opcje wyboru poszczególnych partycji,
których kopie zamierzamy przeprowadzić. To jakie partycje znajdziemy w tym okienku zależy głównie
od pliku `fstab` , który znalazł się w obrazie TWRP. Niektóre są bardziej rozbudowane, a inne
ograniczają się jedynie do podstawowych wpisów. W każdym razie, partycja `/data/` powinna być
widoczna na liście:

![](/img/2017/01/002.backup-kopia-zapasowa-smartfon-data-android-twrp-kopia.png#medium)

W tym przypadku na liście mamy dwie pozycje z nazwą `Data` . Ta zaznaczona pozycja odpowiada za
zrobienie backup'u samych plików na partycji `/data/` . Ta druga opcja umożliwia naturalnie
zrobienie kopii binarnej ale nie będziemy z niej korzystać. Niemniej jednak, warto zauważyć różnice
w ilości kopiowanych danych. Same pliki w tym przypadku mają nieco ponad 100 MiB, podczas gdy cała
partycja ma ponad 12 GiB. Różnica jest ogromna.

Po zaznaczeniu odpowiedniej pozycji upewniamy się jeszcze, że wybraliśmy stosowną pamięć do zapisu
pliku kopii zapasowej, tj. Kartę SD.

![](/img/2017/01/003.backup-kopia-zapasowa-smartfon-data-android-twrp-kopia.png#medium)

Jako, że tutaj nie ma dużo danych do zapisu, to można wykorzystać nawet małe karty SD, które mają
rozmiar 1-2 GiB. Obraz partycji na taką małą kartę SD by nam się nie zmieścił, a same dane wejdą bez
większego problemu.

Warto tutaj jeszcze dodać, że w przypadku, gdy rozmiar danych na partycji `/data/` jest duży, to
plik backup'u zostanie podzielony automatycznie na mniejsze kawałki (~1,5 GiB). Nie ma zatem obawy o
zapis takich plików na kartę SD sformatowaną system plików z rodziny FAT. Obraz całej partycji
`/data/` , jako, że przekracza on limit 4 GIB, trzeba by umieścić na karcie SD sformatowanej innym
systemem plików, np. linux'owym EXT4.

By zrobić backup, przesuwamy strzałki na prawą stronę:

![](/img/2017/01/004.backup-kopia-zapasowa-smartfon-data-android-twrp-kopia.png#big)

Po zakończeniu całego procesu, na karcie SD zostaną utworzone następujące pliki: `data.ext4.win`
(archiwum TAR), `data.ext4.win.md5` (zawiera sumę kontrolną archiwum), `recovery.log` (zawiera log z
backup'u) oraz `data.info` (zawiera info na temat rozmiaru, typu i liczby plików archiwum). Lepiej
nie kasować żadnego z tych plików, bo inaczej TWRP będzie miało prawdopodobnie problemy z
odtworzeniem backup'u.

### Odtwarzanie kopii zapasowej partycji /data/ przez TWRP

Kopię zapasową partycji `/data/` można również odtworzyć. Nie zaleca się jednak wgrywania takiego
backup'u w momencie, gdy był on sporządzany na innej wersji Androida. Dla przykładu załóżmy, że
stworzyliśmy sobie kopię na Androidzie 5.1 Lollipop, zaktualizowaliśmy system do nowszej wersji i
chcemy tę kopię odtworzyć na Androidzie 6.0 Marshmallow. Ja generalnie nie robiłbym tego, a to z
tego względu, że różnice między tymi systemami są znaczne. Tak odtworzona kopia mogłaby uszkodzić
system, w sensie takim, że nie uruchomiłby nam się on ponownie i trzeba by przeprowadzić proces
Factory Reset.

Odtworzenie kopii zapasowej w każdym innym przypadku, tj. na tym samym urządzeniu po uprzednim
przywróceniu jego ustawień do fabrycznych, czy też na innych smartfonach mających tę samą wersję
Androida, nie powinno raczej nam zaszkodzić. Choć w przypadku wgrywania backup'u na inne telefony
również bym uważał.

Zakładając jednak, że coś namieszaliśmy w systemie naszego smartfona i wiemy, że nie obędzie się bez
przywracania go do ustawień fabrycznych, możemy naturalnie po zainicjowaniu Factory Reset wgrać
uprzednio zrobiony backup również przez tryb recovery.

Będąc w trybie recovery, w głównym menu TWRP wybieramy pozycję `Przywróć` . Tam z kolei mamy listę
plików kopii zapasowych, które możemy przywrócić.

![](/img/2017/01/005.backup-kopia-zapasowa-smartfon-data-android-twrp-przywracanie-kopii.png#big)

Po kliknięciu na interesującą nas pozycję zostanie nam zwrócona informacja na temat danych, które w
takim pliku się znajdują, tj. partycji, które zostały w tej kopii zawarte. W tym przypadku mamy dane
z partycji `/data/` :

![](/img/2017/01/006.backup-kopia-zapasowa-smartfon-data-android-twrp-przywracanie-kopii.png#medium)

Na dole mamy również opcję `Włącz weryfikację MD5 kopii zapasowych` . Tę opcję można naturalnie
zaznaczyć ale trzeba mieć na uwadze, że proces odtwarzania backup'u będzie trwał dłużej, zwłaszcza w
przypadku, gdy danych w kopii jest dość sporo. Po zaznaczeniu stosownych opcji, przeciągamy strzałki
na prawą stronę:

![](/img/2017/01/007.backup-kopia-zapasowa-smartfon-data-android-twrp-przywracanie-kopii.png#big)

Jak widać z komunikatów na fotce, nie ma potrzeby przeprowadzania wcześniej procesu Factory Reset,
bo partycja przed wgraniem na nią backup'u jest automatycznie czyszczona.

## Problem z odblokowaniem ekranu po odtworzeniu backup'u

Wiele osób ma zabezpieczony dostęp do telefon za pomocą kodu PIN. W takich przypadkach, by móc
używać tego urządzenia w innych celach niż połączenia alarmowe trzeba podać cztery cyferki. Jeśli
tego nie zrobimy, nie damy rady zdjąć blokady ekranu i nie dostaniemy się do systemu.

Jeśli kopia partycji `/data/` była przeprowadzana na telefonie, który miał włączoną blokadę ekranu,
to w pewnych sytuacjach po odpaleniu systemu nie damy rady ściągnąć tej blokady nawet podając
poprawny PIN:

![](/img/2017/01/008.backup-kopia-zapasowa-smartfon-data-android-problem-pin.png#big)

Problem naturalnie można poprawić ale trzeba odpalić telefon ponownie w trybie recovery. Tam z menu
TWRP przechodzimy do Zaawansowane => Menadżer Plików:

![](/img/2017/01/009.backup-kopia-zapasowa-smartfon-data-android-twrp-usuwanie-pin.png#huge)

W menadżerze plików przechodzimy do katalogu `/data/system/` , z którego to musimy skasować kilka
plików. Kasujemy wszystko to co ma w nazwie `.key` oraz `locksettings.` :

![](/img/2017/01/010.backup-kopia-zapasowa-smartfon-data-android-twrp-usuwanie-pin.png#huge)

W tym przypadku skasowanych zostało 5 plików: `gatekeeper.password.key` , `gatekeeper.pattern.key` ,
`locksettings.db` , `locksettings.db-shm` oraz `locksettings.db-wal` . Po ich usunięciu restartujemy
telefon i już powinniśmy być w stanie zalogować się w systemie. Jedyna różnica jest taka, że teraz
nie byliśmy pytani o PIN, bo usunęliśmy wcześniej zarówno klucze jak ustawienia blokady.

W przypadku, gdy blokada ekranu jest dla nas dość ważna, to naturalnie możemy ją ustawić sobie
ponownie w tradycyjny sposób. Jak tylko blokada zostanie włączona, to system wygeneruje nowe klucze
i nimi zabezpieczy nasz telefon.

Pamiętajmy jednak, że mając TWRP na partycji `/recovery/` , czy w ogóle odblokowany bootloader, to
te klucze/ustawienia możemy usuwać bez problemu i w ten sposób obchodzić mechanizm blokady ekranu w
telefonach z Androidem. Warto mieć zatem świadomość, że blokada ekranu w takich urządzeniach nie
chroni nas zupełnie przed niczym. Oczywiście można nieco minimalizować zagrożenie przez wgrywanie
TWRP tylko do pamięci telefonu, a na partycji `/recovery/` trzymać stock'owy soft, choć i tak lepiej
jest nie zostawić telefonu bez nadzoru na dłuższy czas.
