---
author: Morfik
categories:
- Linux
date: "2015-06-21T00:33:38Z"
date_gmt: 2015-06-20 22:33:38 +0200
published: true
status: publish
tags:
- bootloader
- grub
title: Błędy bootloadera grub
---

Grub to najpopularniejszy bootloader w systemach linuxowych. Dorobił się tego miejsca na podium
głównie ze względu na swoją pełną automatyzację. Potrafi obsłużyć pokaźną ilości systemów plików,
no i również nie zostaje w tyle w stosunku do aktualnych standardów partycjonowania dysków -- mowa
oczywiście o tablicy partycji GPT. Przeglądając internet w poszukiwaniu odpowiedzi na temat jednego
z błędów jaki grub wyrzucił mi podczas startu systemu, znalazłem [ten oto
artykuł](http://www.uruk.org/orig-grub/errors.html). Zawarte są tam dokładnie wszystkie możliwe
błędy jakie grub potrafi zwrócić wraz z krótkim wyjaśnieniem ich przyczyny. Postanowiłem sobie je
przejrzeć i dorobić do nich polskie tłumaczenie.

<!--more-->
## Grub i jego trzy etapy

Proces uruchamiania systemu przy wykorzystaniu bootloadera grub podzielony jest z grubsza na dwa
etapy. Popatrzmy na fotkę poniżej (źródło
[wikipedia](https://en.wikipedia.org/wiki/GNU_GRUB)):

[![1.linux-mbr-keyfile]({{< baseurl >}}/img/2015/06/1.linux-mbr-keyfile-1024x576.png)]({{< baseurl >}}/img/2015/06/1.linux-mbr-keyfile.png)

Pierwszy etap (stage 1) ma na celu załadowanie `boot.img` , który jest przechowywany w MBR
(ewentualnie VBR). Zadaniem tego kodu jest wskazanie położenia `core.img` bootloadera, który z
reguły jest zlokalizowany bezpośrednio za sektorem MBR i przed pierwszą partycją, chyba, że mamy do
czynienia z tablicą partycji GPT. Podczas etapu pośredniego (stage 1.5) ładowany jest kod obecny w
`core.img` , gdzie przechowywana jest konfiguracja i niezbędne moduły w szczególności sterowniki
systemu plików. W `core.img` są także zawarte dalsze instrukcje dla bootloadera grub. W etapie
drugim (stage 2) są ładowane pliki z katalogu `/boot/grub/` . Gdy ten proces dobiegnie końca,
powinniśmy zobaczyć ekran wyboru systemu operacyjnego.

## Komunikaty błędów

Poniżej jest zamieszczonych 28 komunikatów, które mogą zostać wyrzucone w przypadku gdy występują
jakieś problemy z etapem 2 (stage 2). Dla czytelności i lepszego zrozumienia, postanowiłem zachować
oryginalną treść wiadomości i dorobić do nich kilka słów wyjaśnienia bazując na przytoczonym we
wstępie wpisie.

### 1: "Selected item won't fit into memory"

Ten błąd jest zwracany w przypadku gdy kernel, moduł, lub pliku z danymi w formie surowej (raw)
próbują załadować dane, które nie mieszczą się w pamięci.

### 2: "Selected disk doesn't exist"

Ten błąd może się pojawić w przypadku gdy urządzenie, jego część lub też i pełna nazwa pliku odnosi
się do dysku lub urządzenia BIOS, które nie jest obecne w danej chwili lub nie może z jakiegoś
powodu zostać rozpoznane przez BIOS w systemie.

### 3: "Disk read error"

Ten komunikat pojawia się w sytuacji gdy występują błędy podczas próby odczytu danych z określonego
dysku.

### 4: "Disk write error"

Ten błąd jest zwracany gdy pojawiają się problemy przy zapisie danych na określony dysk. Zwykle może
się on pojawić podczas instalacji zestawu poleceń partycjonowania.

### 5: "Disk geometry error"

Ten błąd pojawia się w sytuacji gdy czytany adres bloku znajduje się poza końcem zrozumiałego przez
BIOS obszaru. Zdarza się to głównie wtedy gdy dysk jest większy niż BIOS jest w stanie obsłużyć.

### 6: "Attempt to access block outside partition"

Ten komunikat może pojawić się gdy adres bloku jest poza obszarem partycji. Może to być wynikiem
uszkodzonego systemu plików na dysku lub też jakiegoś błędu w kodzie samego gruba.

### 7: "Partition table invalid or corrupt"

Ten błąd pojawia się gdy testy stanu integralności tablicy partycji nie zakończą się powodzeniem.

### 8: "No such partition"

Ten błąd dotyczy sytuacji gdy wybrana partycja nie może zostać odnaleziona na odpytywanym dysku.

### 9: "Bad filename (must be absolute pathname or blocklist)"

Ten błąd jest zwracany w przypadku gdy odpytywana nazwa pliku nie pasuje pod względem składni czy
też reguł określonych w [opisie systemu plików](http://www.uruk.org/orig-grub/filesystem.txt).

### 10: "Bad file or directory type"

Ten błąd jest zwracany gdy żądany plik nie jest zwykłym plikiem, tylko dowiązaniem symbolicznym,
katalogiem czy też urządzeniem FIFO.

### 11: "File not found"

Ten komunikat może pojawić się w przypadku gdy określony plik nie może zostać odnaleziony ale
wszystko inne, jak np. informacje o dysku/partycji, jest w porządku.

### 12: "Cannot mount selected partition"

Ten błąd pojawia się gdy żądana partycja istnieje ale jej typ systemu plików nie może zostać
rozpoznany przez gruba.

### 13: "Inconsistent filesystem structure"

Ten błąd jest zwracany przez kod systemu plików w celu oznaczenia wewnętrznego błędu spowodowanego
przez testy stanu struktury systemu plików na dysku. Zwykle jest to wynikiem uszkodzonego systemu
plików lub błędu w kodzie gruba.

### 14: "Filesystem compatibility error, can't read whole file"

Część kodu odpowiadającego za czytanie systemu plików w grubie ma ograniczenia co do długości
plików, które mogą zostać przez niego przeczytane. Ten błąd pojawia się w momencie, gdy użytkownik
wychodzi poza ten określony limit.

### 15: "Error while parsing number"

Ten błąd jest zwracany w przypadku gdy grub spodziewał się odczytać jakąś liczbę, a zamiast tego
napotkał inne dane.

### 16: "Device string unrecognizable"

Ten komunikat może pojawić się w przypadku gdy oczekiwany ciąg znaków określający urządzenie nie
spełniała wymagań co do składni określonej w [opisie systemu plików](http://www.uruk.org/orig-grub/filesystem.txt).

### 17: "Invalid device requested"

Ten błąd jest zwracany w momencie gdy ciąg znaków określający urządzenie został rozpoznany ale nie
mieści się w ramach innych błędów dotyczących urządzeń.

### 18: "Invalid or unsupported executable format"

Ten błąd zwracany jest w przypadku gdy obraz kernela, który jest aktualnie ładowany, nie jest
rozpoznawany jako Multiboot, ani również jako jeden z pozostałych wspieranych natywnych formatów,
tj. Linux zImage lub bzImage, FreeBSD, czy też NetBSD.

### 19: "Loading below 1MB is not supported"

Ten błąd występuje w przypadku gdy najniższy adres w kernelu znajduje się poniżej 1MiB granicy.
Linuxowy format zImage jest specjalnym przypadkiem i może zostać obsłużony ponieważ ma on stały
adres i maksymalny rozmiar.

### 20: "Unsupported Multiboot features requested"

Ten błąd jest zwracany gdy Multiboot wymaga czegoś co jest nierozpoznawalne. Chodzi o to, że kernel
wymaga specjalnego traktowania, którego grub nie jest w stanie zapewnić.

### 21: "Unknown boot failure"

Ten błąd jest wynikiem próby załadowania systemu, która zakończyła się niepowodzeniem z przyczyn
nieznanych.

### 22: "Must load Multiboot kernel before modules"

Ten błąd jest zwracany gdy polecenie załadowania modułu zostało użyte przed załadowaniem samego
kernela Multiboot. To ma tylko sens w tym przypadku, bo grub nie ma pojęcia jak powiązać ze sobą
obecność takich modułów z kernelem, który nie obsługuje Multiboot.

### 23: "Must load Linux kernel before initrd"

Ten błąd może się pojawić gdy polecenie initrd zostało użyte przed załadowaniem się linuxowego
kernela. Podobnie jak w przypadku `22` , to ma tylko sens w tym przypadku.

### 24: "Cannot boot without kernel loaded"

Ten błąd jest wynikiem nakazania grubowi wykonania sekwencji startu bez uprzedniego załadowania się
kernela.

### 25: "Unrecognized command"

Ten błąd jest zwracany gdy nierozpoznane polecenie zostało wprowadzone do terminala lub w sekcji
sekwencji startu w pliku konfiguracyjnym.

### 26: "Bad or incompatible header on compressed file"

Ten komunikat może pojawić się gdy nagłówek skompresowanego pliku jest nieodpowiedni.

### 27: "Bad or corrupt data while decompressing file"

Ten komunikat jest zwracany gdy długotrwała dekompresja kodu napotka wewnętrzny błąd. Zwykle w
przypadku uszkodzonego pliku.

### 28 : "Bad or corrupt version of stage1/stage2″

Ten błąd jest zwracany w przypadku gdy polecenie instalacji wskazuje na niekompatybilną lub
uszkodzoną wersję kodu pierwszego lub drugiego etapu. Ogólnie rzecz biorąc, to nie potrafi on wykryć
problemu ale to jest test numerów wersji, które powinny być poprawne.
