---
author: Morfik
categories:
- Linux
date: "2019-03-15T17:05:42Z"
published: true
status: publish
tags:
- hdd
- ssd
GHissueID: 297
title: Jak optymalnie podzielić dysk HDD/SSD na partycje pod linux
---

Ostatnio przeglądając nowe wpisy na forach trafiłem
na [pytanie o podział dysku](https://forum.linuxmint.pl/showthread.php?tid=112) pod instalację
linux'a. Autorowi wątku chodziło o jak najlepszy podział dysku HDD ale najwyraźniej pogubił się on
w tym całym bałaganie informacyjnym, który tyczy się procesu partycjonowania nośnika pod kątem jego
optymalnego wykorzystania przez linux. Zagadnienie podziału dysków HDD/SSD nie jest zbytnio jakoś
skomplikowane, a mimo to wciąż pojawiają się pytania o poprawne przeprowadzenie tego procesu i to
pomimo faktu, że mamy obecnie już dość sporych rozmiarów dyski. To pytanie powoli przestaje mieć
jakikolwiek sens, przynajmniej jeśli chodzi o przeciętnego Kowalskiego instalującego linux'a u
siebie na kompie (desktop/laptop). Postanowiłem jednak napisać parę słów na temat tego całego
podziału dysku na partycje, tak by dobrać optymalny ich rozmiar przy instalowaniu dowolnej
dystrybucji linux'a.

<!--more-->
## To jak podzielić ten dysk na partycje pod linux

Odpowiedź na pytanie "jak podzielić dysk na partycje pod linux?" albo bardziej konkretnie "jakie
rozmiary powinny mieć poszczególne partycje pod linux?" zawsze będzie względna i będzie miało na
nią wpływ bardzo dużo czynników. W zasadzie nie ma dwóch takich samych linux'ów i jeśli naprawdę
chcemy w nieco bardziej zaawansowany sposób podzielić dysk na partycje, to wypadałoby przemyśleć
kilka najistotniejszych kwestii, bo zakres wykorzystania całego spektrum możliwości komputera jest
wręcz nieograniczony. Nie tylko zmienia się użytkownik i jego stopień utylizacji zasobów maszyny
(chodzi o rzeczy, do których komputer jest wykorzystywany), to jeszcze podzespoły w naszym
komputerze mogą się znacząco różnić.

Gdy chodzi o podział dysku na partycje, to kluczowe znaczenie ma ilość zainstalowanej w systemie
pamięci operacyjnej RAM oraz wielkość i rodzaj dysku twardego (SSD czy HDD). Te trzy czynniki mogą
mieć spore przełożenie na to w jaki sposób podzielić dysk. Niemniej jednak, jeśli chodzi o zwykłe
maszyny desktopowe, od których się nie wymaga za wiele, to w zasadzie cały ten proces podziału jest
zbędny i tylko niepotrzebnie komplikuje ludziom życie.

W tym podlinkowanym we wstępie wątku na forum, użytkownik miał do dyspozycji maszynę mającą
zainstalowane 4G RAM oraz dysk HDD o pojemności 320G. Nie jest to jakaś zaawansowana konfiguracja w
obecnych czasach, bo nawet mój 8 letni laptop ma dysk o pojemności 1T. To czy mamy do dyspozycji
320G, czy 10T, w zasadzie nie ma większego znaczenia. Po prostu im większy będzie dysk, tym więcej
miejsca będzie można przeznaczyć na partycje niepowiązane bezpośrednio z systemem, a ilość pamięci
RAM oraz rodzaj dysku w zasadzie wpływa jedynie na dobór rozmiaru przestrzeni wymiany SWAP.

## Tablica partycji MS-DOS (MBR) oraz GPT

Podział dysku trzeba rozpocząć od doboru rodzaju tablicy partycji. Do wyboru jest przestarzała już
tablica partycji MS-DOS (nazywana też [MBR](https://en.wikipedia.org/wiki/Master_boot_record)) i
nowsza [GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table) (GUID Partition Table). Te dwie
tablice partycji różnią się budową, m.in. ilością obsługiwanych partycji (4 + dyski logiczne vs
128) i wielkością obsługiwanych dysków (2 TiB vs 8 ZiB, przy 512 bajtowych sektorach). Do tego
dochodzi jeszcze EFI/UEFI, który wymaga obecności tablicy partycji GPT. Te rzeczy w dużej mierze
warunkują wybór jednego lub drugiego rodzaju tablicy partycji. Linux bez większego problemu jest w
stanie działać na GPT od lat i gdy zamierzamy instalować na dysku tylko linux'a, to z tego rodzaju
partycji powinniśmy skorzystać. Nowsze windows'y już chyba też potrafią działać na GPT, np. mój
windows7 tego nie potrafił i wymagał, by na dysku, na którym się go instaluje, była tablica
partycji MS-DOS.

Zmiana typu tablicy partycji niekoniecznie wiąże się z utratą wszelkich danych zawartych na
nośniku. [Migracja z MS-DOS na GPT](/post/konwersja-tablicy-partycji-ms-dos-na-gpt/)
jest w zasadzie bezproblemowa, choć zostawiane są małe wolne obszary jako pozostałość po strukturze
dysków logicznych, których obsługa w tablicy MS-DOS była zaimplementowana na
zasadzie [EBR](https://en.wikipedia.org/wiki/Extended_boot_record). Podobnie też i w drugą stronę,
tj. gdy
zamierzamy [zmienić rodzaj partycji z GPT na MBR](/post/konwersja-tablicy-partycji-gpt-na-ms-dos/),
choć nie zawsze ta operacja będzie możliwa.

## LVM

Wielu użytkowników jest przyzwyczajonych do tworzenia fizycznych partycji na dysku. Jednak w
pewnych konfiguracjach linux'a, zwłaszcza tych zaawansowanych, fizyczne partycje mogą być dość 
problematyczne. Chodzi generalnie o późniejszy proces zarządzania partycjami, np. zmiana ich
rozmiaru, usuwanie obecnych partycji lub tworzenie nowych. Wykorzystując np. pełne szyfrowanie
dysku na bazie LUKS trzeba by się nieco napracować, by zmienić układ partycji dysku. Dlatego też
zamiast się bawić fizycznymi partycjami, można stworzyć jedna dużą fizyczną partycję na dysku i ją
już logicznie podzielić w zależności od upodobań. Zastosowanie LVM eliminuje też ograniczenia,
które niesie ze sobą tablica partycji MS-DOS, gdzie mamy limit 4 partycji podstawowych i trzeba się
ratować partycją rozszerzoną. LVM jest też w stanie pomóc nam przy ewentualnych problemach z
dyskami logicznymi, bo wszystkie zmiany dokonywane w obrębie struktury LVM są rejestrowane i w
przypadku błędów zawsze będziemy mieli jakiś punkt odniesienia.

## Linux i windows na jednym dysku

Nie jestem zbytnio fanem posiadania dwóch systemów (lub więcej) na jednym nośniku fizycznym,
zwłaszcza jeśli są to różne systemy jak linux i windows. Każdy z nich realizuje pewne rzeczy
inaczej i próba pogodzenia ich prędzej czy później będzie owocować problemami, np. przepisywanie
kodu bootloader'a w MBR, co z kolei może prowadzić do problemów z aktualizacją windowsa, czy też
problemy związane z odpaleniem windowsa na dysku mającym tablicę partycji GPT.

Ja jestem zwolennikiem zasady, że jeden dysk, to jeden system operacyjny, a mając jeden OS, to tego
typu dylematów moralnych tyczących się kwestii pogodzenia wielu różnych systemów operacyjnych w
obrębie jednego nośnika zwyczajnie nie doświadczam.

## Kwestia SWAP

Ostatnimi laty pojawiło się sporo niedopowiedzeń lub też błędnych informacji dotyczących
konfiguracji przestrzeni wymiany SWAP pod linux. Mechanizm SWAP miał uzasadnione zastosowanie lata
temu, gdy komputery miały relatywnie mało i do tego dość wolnej pamięci operacyjnej RAM. Przestrzeń
wymiany SWAP w obecnych czasach ma w zasadzie spełniać jedynie dwie role.

Pierwszą z nich jest niedopuszczenie do powieszenia się systemu w skutek wyczerpania się pamięci
RAM (za sprawą mechanizmu memory overcommit). Jak ma się mało RAM, to przy zbyt agresywnym
wykorzystaniu zasobów maszyny, system albo się powiesi, albo aplikacje będą ubijane przez kernel
(za sprawą OOM-killer'a), tak by do tego powieszenia nie dopuścić. Obie te rzeczy są niepożądane,
dlatego też SWAP jest w stanie kosztem wydajności nas przed nimi uchronić. Gdy SWAP zacznie się
zapełniać w dość energicznym tempie, to taka sytuacja powinna nam dać do myślenia i w miarę
możliwości powinniśmy zacząć zamykać zbędne aplikacje. Jeśli się tego nie zrobi, to dane
programów (tych najrzadziej używanych z perspektywy kernela, np. przeglądarka internetowa) będą
zrzucane na dysk, a podnoszenie zminimalizowanego okna Firefox'a w takiej sytuacji może trwać nawet
i paręnaście sekund. Zatem im więcej danych zostanie przeniesionych z pamięci RAM do SWAP, tym
wolniej będzie działał nasz system.

Drugim zdaniem dla SWAP jest umożliwienie hibernacji systemu, czyli zapisu aktualnego snapshot'a
pamięci RAM na dysk w celu późniejszego jego odtworzenia wraz z kolejnym startem systemu, co
przyśpiesza dość znacznie czas, w którym maszyna jest gotowa do pracy. W przypadku, gdy nie
korzystamy z hibernacji, to ten aspekt może nas w ogóle nie dotyczyć. Warto jednak pamiętać o nim,
gdy korzystamy z trybu uśpienia systemu, pod który można podpiąć hybrydowe usypanie, na wypadek
utraty zasilania.

### SWAP i dyski SSD

Gdy mamy do czynienia z dyskami SSD, to z racji, że liczy się u nich w dużej mierze ilość cykli
zapisu komórek zanim one padną, to nie zaleca się tworzenia na tego typu nośnikach partycji wymiany
SWAP. Jeśli mamy za mało pamięci RAM, to SWAP będzie dość ekstensywnie wykorzystywany. Zatem taki
nośnik bardzo szybko nam się zużyje (komórki pozdychają), przez co dysk będzie do wyrzucenia.

W przypadku dysków SSD lepiej jest zainwestować w dodatkową kość RAM, by ograniczyć potrzebę
wykorzystania SWAP do minimum lub też ją całkowicie wyeliminować. Na dyskach SSD system uruchamia
się bardzo szybko, podobnie jak i aplikacje użytkownika, dlatego też zwykle w przypadku tych dysków
nie ma po co hibernować systemu.

Mamy zatem sytuację, w której możemy przyoszczędzić na RAM i wyniszczyć komórki nośnika SSD katując
je cyklami zapisu za sprawą SWAP. Takie podejście przełoży się na wymianę dysku mniej więcej co rok
albo nawet i częściej. Alternatywą zaś jest dokupienie większej ilości RAM, co wyeliminuje potrzebę
stosowania SWAP, a my będziemy przy tym do przodu finansowo (wystarczy jeden padnięty dysk, by
stracić środki na zakup 8-16G dodatkowej pamięci RAM). No i oczywiście odejdzie nam stres związany
z ewentualną migracją danych ze starego dysku na nowy.

### Wielkość SWAP, a ilość RAM

Jeśli zaś chodzi o dyski talerzowe, to mają one do siebie to, że są bardzo wolne w stosunku do
nośników SSD. Taki pospolity HDD nie cierpi, co prawda, z powodu cykli zapisu ale ma mechanizm
głowicy, która musi się fizycznie przemieszczać, a to z kolei generuje spore opóźnienia, co jest
dość mocno odczuwalne przy starcie systemu lub też przy uruchamianiu programów. By przyśpieszyć
operację przygotowania systemu do pracy, można przeznaczyć część dysku na obraz pamięci RAM.

Mając do dyspozycji przykładowo 4G pamięci RAM, nie musimy tworzyć przestrzeni SWAP, której
wielkość zrówna się z ilością dostępnej w systemie pamięci operacyjnej lub też ją przekroczy dwu
lub i nawet trzykrotnie. Jest to działanie bez sensu i marnuje nam tylko cenne miejsce na dysku. W
przeciętnym środowisku, z jakim można się spotkać na desktopie czy laptopie, nigdy nawet nie
zbliżymy się w połowie do tej wartości 4G, nie mówiąc już o jej przekroczeniu. No może 3G SWAP by
się udało jeszcze zapełnić ale system będzie już tak mało responsywny, że w zasadzie korzystać z
niego będą w stanie jedynie osoby o naprawdę mocnych nerwach, zwłaszcza jak system wejdzie w fazę
SWAP-storm.

Duży SWAP znajduje jedynie zastosowanie w przypadku środowisk programistycznych przy kompilacji
programów, przynajmniej sporadycznie, bo nie wyobrażam sobie kompilowania kodu, gdy trzeba ciągle
korzystać ze SWAP na HDD z racji niewystarczających zasobów pamięci RAM.

Może się wydawać, że skoro mamy w systemie zainstalowane 4G RAM, to SWAP powinien również tyle
wynosić, bo w przeciwnym razie nie uda nam się zahibernować systemu. To, ile SWAP jest potrzebne,
by umożliwić proces hibernacji zależy od ilości aktualnie zaalokowanych danych w pamięci RAM. Jeśli
wszystkie uruchomione aplikacje w systemie zjadają nam w sumie 1,5G RAM, to w SWAP musi być minimum
1,5G wolnego miejsca, by hibernacja była możliwa. Zatem niby mamy 4G RAM, a hibernację można
przeprowadzić na SWAP o wielkości 2G bez żadnego problemu (oczywiście przy założeniu, że w SWAP nie
znajduje się aktualnie więcej niż 0,5G danych). W przypadku, gdy ilość danych w RAM przekroczy
wielkość SWAP, to kernel nam odmówi możliwości zahibernowania systemu i trzeba będzie jakieś
aplikacje pierw pozamykać -- zwykle wystarczy zamknąć przeglądarkę internetową.

Mając na uwadze powyższe informacje, nie ma co tworzyć przestrzeni wymiany SWAP wychodzącej poza
rozmiar 4G.

### Przestrzeń SWAP jako plik

Linux umożliwia także
stworzenie [przestrzeni wymiany SWAP w regularnym pliku](/post/czy-w-linux-plik-swap-jest-lepszy-niz-partycja-wymiany/)
wewnątrz systemu
plików. Takie rozwiązanie sprawia, że nie potrzebna nam jest dedykowana partycja. W przypadku, gdy
SWAP nam nie będzie potrzebny, to plik można zwyczajnie skasować odzyskując natychmiastowo miejsce,
które zajmował. W przypadku fizycznych partycji ten zabieg ze zwolnieniem miejsca może być bardziej
skomplikowany.

## Pozycja partycji na dysku HDD

Dyski HDD mają to do siebie, że ich transfer danych i czas dostępu do plików zmienia się w
zależności od miejsca, z którego te dane są czytane lub zapisywane. Im dalej od początku dysku, tym
szybkość transferu danych spada, a czasy dostępu rosną. Dlatego też kluczowe znaczenie ma miejsce,
w którym konkretna linux'owa partycja (w tym przestrzeń wymiany SWAP) zostanie ulokowana.

Niektórzy zalecają tworzenie przestrzeni SWAP zaraz na początku dysku celem zwiększenia jej
wydajności. Może to mieć zbawienny skutek w sytuacji, gdy faktycznie cierpimy na niedostatek
pamięci RAM. W każdym innym przypadku, ta najbardziej cenna przestrzeń dysku (pierwsze 10%) będzie
szła na zmarnowanie degradując wydajność systemu. Tutaj mamy do czynienia z dyskiem 320G, zatem te
pierwsze 30G ma dla nas ogromne znaczenie.

Windows czasem upiera się by na dysku go zainstalować na pierwszej partycji i to tej zaraz na
początku dysku. Jeśli ten system ma być używany jedynie sporadycznie, to powinniśmy rozważyć
wrzucenie go na koniec dysku, tak by nie tracić tych cennych z punktu widzenia wydajności
pierwszych gigabajtów przestrzeni. Poniżej jest fotka przykładowego dysku obrazująca spadek
wydajności wraz z oddalaniem się od początku dysku.

![](/img/2019/03/001-hdd-disk-performance-degradation-linux-partition.jpg#big)

Dlatego też krojąc dysk na partycje na potrzeby linux'a dobrze jest mieć ten stan rzeczy na uwadze
i umieścić na samym początku dysku partycję `/` lub `/home/` , tak by pliki systemowe/użytkownika,
których jest cała masa i są często odpytywane przez system, miały spory bonus, co z kolei przełoży
się na szybkość działania aplikacji.

## Optymalny podział dysku na partycje

Pora przejść do kluczowej kwestii, czyli podziału dysku na partycje. W tym miejscu wypadałoby sobie
zadać pytanie czy zamierzamy z tego linux'a korzystać na co dzień, czy ma to być jedynie swojego
rodzaju system awaryjny lub chcemy się na niego logować od czasu do czasu, np. mając obok windows'a.
W każdym z tych dwóch przypadków będą nam potrzebne co najmniej dwie partycje fizyczne. Jedna z
nich na pliki bootloader'a ( `/boot/` ), druga na linux'a ( `/` ), choć w zależności od
konfiguracji może być nawet i jedna partycja, choć ja nigdy nie tworzyłem takich konfiguracji ze
względu na portowalność rozwiązania -- osobna partycja `/boot/` zawsze daje spore możliwości
konfiguracyjne, a przez to bezpieczeństwo systemu może dość znacznie wzrosnąć. Jak dojdzie do tego
windows, to też ze dwie dodatkowe partycje trzeba liczyć, jedna na system, a druga na jakieś dane
"stałe" przechowywane w systemie plików NTFS. Więc tych partycji będzie minimum 4, czyli dokładnie
tyle ile obsługuje tablica MS-DOS.

### Partycja /

Szereg rzeczy, które domyślnie idą na partycję `/` , powinien zostać rozdzielonych, raz ze względów
bezpieczeństwa, a dwa ze względu na fragmentację plików i ogólnie przez rzeczy związane z
wydajnością systemu (mniejsze partycje to krótsza droga przy ruchu głowicy). Na partycji `/`
powinien nam zostać w zasadzie sam system. Powinno się odseparować katalogi tymczasowe ( `/tmp/` i
`/var/tmp/` ). Podobnie trzeba postąpić z konfiguracją bootloader'a ( `/boot/` ) oraz konfiguracją
użytkowników ( `/home/` ). Obecnie nie powinno się umieszczać na osobnej partycji katalogu `/usr/`
ze względu na
pewne [problemy jakie niesie ze sobą to rozwiązanie](https://freedesktop.org/wiki/Software/systemd/separate-usr-is-broken/).

Partycja `/` powinna być na tyle duża, by system nie cierpiał z powodu zbyt małej ilości wolnego
miejsca. Jej rozmiar zależy w sporej mierze od przeznaczenia systemu. Absolutne minimum to 20G,
choć powinno się mierzyć raczej w próg 30G. Pamiętajmy, że na tej partycji przechowywany jest cache
pod pakiety menadżera pakietów, który może zjadać sporo miejsca.

### Partycja /boot/

Osobna partycja `/boot/` dane nam możliwość ukrycia szeregu newralgicznych elementów systemu (przez
odmontowanie tej partycji po starcie systemu), np. obraz initramfs/initrd, który można poddać
edycji (wygenerować na nowo ze zmienionymi plikami), czy podejrzeć plik `System.map` , który
posiada użyteczne info dla całego syfu, który w systemie się zagnieździ.

Partycja `/boot/` nie powinna być duża ale powinna mieć wystarczająco miejsca, by aktualizacje
systemu (głównie kernela) były możliwe. Czasami można mieć więcej kerneli, czasami mniej. Możemy
też budować własne kernele i wszystkie one wraz z konfiguracją bootloader'a muszą się bez problemu
na tej partycji zmieścić. Ja bym nie przeznaczał mniej niż 1G na partycję `/boot/` .

### Partycje /tmp/ i /var/tmp/

Wydzielenie partycji pod pliki tymczasowe umożliwia zamontowanie ich z opcją `noexec`, co ździebko
poprawia bezpieczeństwo systemu, choć pewne dodatkowe kroki trzeba poczynić przy konfiguracji
systemu by zniwelować negatywny efekt związany z tą flagą. Dla przykładu, pewne rzeczy mogą nie
działać. Jedną z nich może być aktualizacja systemu, która wymaga, by flaga `exec` na partycji
`/tmp/` i `/var/tmp/` była obecna, przynajmniej na czas aktualizacji. Podobnie sprawa wygląda, przy
generowaniu obrazu initramfs/initrd.

W linux mamy dwa katalogi, w których przechowywane są pliki tymczasowe. Są to `/tmp/` i
`/var/tmp/` . Zadaniem katalogu `/tmp/` jest przechowywanie tymczasowych plików krótkoterminowych,
tj. takich, które aplikacja zwykle tworzy przy uruchamianiu i usuwa po jej zamknięciu. Z kolei
katalog `/var/tmp/` ma na celu przechowywać długoterminowe pliki tymczasowe, tj. takie, które
powinny przetrwać przykładowo restart systemu. Domyślnie w linux pliki w katalogu `/var/tmp/`
figurują fizycznie na dysku, a zawartość `/tmp/` jest wrzucana do pamięci RAM, by poprawić
wydajność aplikacji, które tworzą całą masę malutkich plików.

Ilość pamięci RAM pod pliki tymczasowe jest stała i wynosi 50% rozmiaru pamięci, choć to nie
oznacza, że na aplikacje będzie do dyspozycji tylko 2G RAM (przy zainstalowanych 4G). Po prostu
pliki tymczasowe będą mogły zająć maksymalnie 2G, a im ich będzie więcej, to tym mniej miejsca
będzie w pamięci RAM do obsługi uruchamianych programów. Systemd posiada również mechanizm
czyszczenia katalogu `/tmp/` co jakiś czas i usuwa z niego pliki, które są nieużywane (czas
dostępu/zmiany jest notowany na kilka godzin wstecz), co zapobiega gromadzeniu się plików
tymczasowych w pamięci RAM.

To, ile miejsca potrzeba na pliki tymczasowe zależy od używanych aplikacji. Niektóre programy
potrzebują więcej miejsca, a inne miej. Na pliki w katalogu `/var/tmp/` powinno się przeznaczyć
więcej przestrzeni niż na katalog `/tmp/` (jeśli nie chcemy go trzymać w RAM), choć większość
programów i tak wrzuca wszystkie pliki do katalogu `/tmp/` bez patrzenia na wytyczne odnośnie tego
do czego ten katalog ma służyć. Na necie można też spotkać się z linkowaniem jednego katalogu do
drugiego i tworzeniem tylko jednej partycji ale nie powinno się tego robić.

Ja aktualnie mam u siebie po 4G na `/tmp/` i `/var/tmp/` , z tym, że niekoniecznie muszą one być
takie duże, bo zawsze można aplikacji podać zmienną `$TMPDIR` i zmienić lokalizację jej plików
tymczasowych (zwykle appki wspierają tę zmienną).

### Partycja /home/

Osobna partycja `/home/` daje nam opcję prostszego formatu systemu, przez co pliki użytkowników
pozostają nietknięte i po zainstalowaniu świeżego linux'a od razu możemy zalogować się na swojego
użytkownika i przystąpić do pracy.

Na partycji `/home/` powinny znajdować się głównie dane powiązane z konfiguracją kont użytkowników
systemu. Małe partycje `/home/` (2-5G) nie są wskazane, bo dość mocno ograniczają dowolność przy
zapisie plików przez aplikacje i trzeba posiłkować się dodatkowymi partycjami, np. przy pobieraniu
większych plików przez internet z poziomu przeglądarki. Optymalny rozmiar na `/home/` to około
30-60G.

Gdy zachodzi potrzeba, by mieć większy `/home/` , np. kończy nam się wolne miejsce na tej partycji,
to znaczy, że aktualny układ partycji jest nie do końca przemyślany. Część plików z partycji
`/home/` powinna powędrować na osobną partycję, np. filmy czy inne szeroko rozumiane duże pliki.
Ja mam u siebie osobną partycję na tego typu rzeczy i na nią jest przeznaczona praktycznie cała
pozostała cześć przestrzeni dyskowej.


## Ext2, ext3 czy ext4

Generalnie rzecz biorąc, każdą partycję, którą stworzymy na dysku, trzeba sformatować jakimś
systemem plików. Linux obsługuje ich całą masę ale jego takim głównym i najczęściej używanym
systemem plików jest EXT (Extended File System) w wersji 4 -- EXT4.

Część użytkowników w dalszym ciągu chce używać EXT3 ale obecnie nie ma ku temu żadnych powodów.
System plików EXT4 jest już od lat stabilny i wykorzystywany domyślnie przez chyba wszystkie
większe dystrybucje linux'a, w tym Debian i Ubuntu.

Niektórzy też upierają się przy formatowaniu pewnych partycji, np. `/tmp/` czy `/boot/` , systemem
plików EXT2. Chodzi tutaj głównie o fakt, że ta druga wersja systemu plików EXT nie zawiera
dziennika (journal), a zapisywanie w nim informacji wiąże się ze spowolnieniem pracy całego systemu.
W pewnych sytuacjach zastosowanie dziennika faktycznie mija się z celem, bo nie daje nam on żadnego
bonusu w kwestii bezpieczeństwa danych przechowywanych na takiej partycji, np. wspomniany wyżej
katalog `/tmp/` , gdzie pliki tymczasowe są zawsze czyszczone podczas startu systemu, więc co nam
za różnica czy się one uszkodzą w wyniku twardego resetu maszyny czy też nie -- i tak zostaną one
wszystkie usunięte.

Może i EXT4 tworzy dziennik ale jeśli jest on nam zbędny, to możemy go zwyczajnie usunąć (w
dowolnym momencie) przy pomocy poniższego polecenia:

    # tune2fs -O ^has_journal /dev/sda1

Podobnie można postąpić w przypadku utworzenia nowego dziennika:

    # tune2fs -j /dev/sda1

Reasumując, jeśli zdecydujemy się na system plików z rodziny EXT, to tylko w wersji 4.

## Bezpieczeństwo linux'a i jego systemu plików

Trzeba wspomnieć jeszcze o jednej rzeczy, a konkretnie o bezpieczeństwie naszego linux'a oraz
danych, które przechowuje on w systemie plików na różnych partycjach. W zasadzie nic nie stoi na
przeszkodzie, by zainstalować linux'a na jednej partycji. Niemniej jednak, trzeba wziąć pod uwagę,
że system plików może się zapełnić, np. w skutek eksensywnego logowania, czy też tworzenia się
plików tymczasowych albo zwyczajnie przez nieuwagę użytkowników, których mamy w systemie. Jeśli `/`
oraz `/home/` znajdują się w obrębie jednego systemu plików, to łatwo jest nawet zwykłemu
użytkownikowi (nie root) doprowadzić do bardzo niebezpiecznych sytuacji, w których zacznie
systemowi brakować miejsca na dysku, a to z kolei będzie powodować problemy w działaniu usług,
wliczając w to problemy z uruchomieniem komputera.

W przypadku korzystania z systemu plików EXT4 można co prawda zarezerwować trochę przestrzeni pod
użytkownika root podając w `mkfs.ext4` przełącznik `-m` i precyzując % przestrzeni partycji jaka ma
zostać do dyspozycji administratora. Niemniej jednak, wszystkie procesy, które nie działają z
uprawnieniami root, mogą ucierpieć w przypadku zapełnienia się dysku.

Dlatego też warto w miarę możliwości oddzielić od głównego systemu plików logi ( `/var/log/` ),
pliki tymczasowe ( `/tmp/` i `/var/tmp/` ) oraz katalogi domowe zwykłych użytkowników ( `/home/` ),
tak by ten problem z brakiem miejsca ograniczyć do minimum albo też zupełnie wyeliminować. Trzeba
tylko pomyśleć trochę nad odpowiednim rozmiarem partycji pod te wyżej wymienione katalogi, tak by
czasem nie utworzyć ich byt małych albo dużych, co z jednej strony będzie generować więcej szkody
niż pożytku, a z drugiej marnować nam jakże cenne miejsce na dysku.

Warto też wspomnieć, że każdy podmontowany w trybie zapisu (RW) system plików może ulec uszkodzeniu
w przypadku powieszenia się komputera czy innej awarii, np. braku prądu. Tego typu sytuacja może
wiązać się ze stratami danych. Nie jest to jakieś częste zjawisko i zwykle linux potrafi się
podnieść po zawale maszyny tracąc jedynie tylko ostatnie zmiany w otwartych/edytowanych
dokumentach/plikach. Niemniej jednak, ryzyko zawsze istnieje.

Korzystając z odpowiedniego podziału na partycje, można część systemów plików odmontować, gdy z
nich nie korzystamy, np. `/boot/` , który znajduje zastosowanie jedynie w przypadku rozruchu
systemu (i czasem też podczas aktualizacji systemu), albo też zamontować niektóre systemy plików w
trybie tylko do odczytu (RO), dla przykładu `/usr/` , w którym pliki pozostają w zasadzie
niezmienione między kolejnymi aktualizacjami systemu, przez co trzymanie tego systemu plików w
trybie do zapisu mija się z celem.

## Fragmentacja plików i wolne miejsce

Sumując te wszystkie wartości wielkości poszczególnych partycji, o których była mowa wyżej,
wychodzi na to, że nasz linux potrzebuje około 70G. Czy nie jest to lekką przesadą? Przecie nawet
windows zjada mniej. O ile w przypadku windows'a, to faktycznie tam pliki systemu zjadają dość
sporo miejsca, o tyle w przypadku instalacji linux'a spora część poszczególnych partycji pozostaje
w formie nieużywanej.

Wolne miejsce ma ogromne znaczenie w przypadku wykorzystywania dysku HDD, gdzie fragmentacja danych
bardzo spowalnia proces zapisu i odczytu plików z racji ruchów głowicy magnetycznej. Dlatego też
powinno się zostawiać tak około 33%-50% wolnego miejsca na danej partycji, by dość mocno ograniczyć
proces fragmentowania się plików, a jeśli te pliki ulegną fragmentacji, to zawsze będzie można je
zdefragmentować przy pomocy `e4defrag` . Dokładny procent wolnej przestrzeni zależy od rozmiaru
partycji oraz od wielkości samych plików, które będą na takiej partycji przechowywane. Chodzi tutaj
bardziej już o fragmentacje wolnej przestrzeni, a nie samych plików. Wgrywając nawet najmniejszy
plik fragmentujemy tym samym wolną przestrzeń uniemożliwiając przy tym wgranie większych plików w
jednym kawałku. A to z kolei jeszcze bardziej fragmentuje wolną przestrzeń i mamy takie dodatnie
sprzężenie zwrotne, które ostateczne spowolni nam dość znacznie system. Lepiej nie dopuszczać do
sytuacji, w której zaczyna brakować miejsca na partycji i dobrze jest sobie zrobić jego zapas.
Podobnie też nie powinno się mieszać dużych plików z małymi.

W przypadku dysków SSD proces fragmentacji ma z kolei marginalne znaczenie jeśli chodzi o wydajność
przy odczycie/zapisie danych ale znowu liczy się wolne miejsce patrząc z perspektywy całego nośnika,
tak by proces równoważenia obciążenia zużycia komórek był w stanie wykonywać swoje zadanie. I jeśli
chodzi o te nośniki to nie powinniśmy schodzić poniżej tych 20-30% wolnego miejsca w skali dysku.
Nie powinniśmy też zbytnio przechowywać na tych dyskach większych plików, których nie mamy zamiaru
w ogóle ruszać miesiącami albo nawet latami, np. filmy czy obrazy `.img` . Ogólnie chodzi o
wykorzystywanie dysku SSD jako storage. Jeśli będziemy ten dysk wykorzystywać w taki sposób, to
komórki będą się zużywać nierównomiernie, co dodatkowo skróci żywotność samego dysku.

## TL;DR

Mając na uwadze powyższe informacje dotyczące rozsądnego krojenia dysku na partycje na potrzeby
maszyny mającej działać pod kontrolą dowolnej dystrybucji linux'a, optymalny podział dysku
prezentuje się w następujący sposób:

	SWAP        --     4G, SWAP, umieścić między 20-30% licząc od początku dysku
	/           --    30G, EXT4, umieścić na początku dysku
	/boot/      --     1G, EXT4
	/tmp/       --     4G, EXT4
	/var/tmp/   --     4G, EXT4
	/home/      --    30G, EXT4, umieścić po partycji /
	/windows/   --    60G, NTFS, umieścić na końcu dysku
	/dane_nfts/ --    30G, NTFS, umieścić na końcu dysku
	/dane_ext4/ -- reszta, EXT4

Oczywiście liczby widoczne wyżej można dość mocno skompresować jeśli wydają się one za duże ale
powinno się tego zabiegu unikać. Jeśli już, to te wartości powinny zostać jedynie zwiększone.
