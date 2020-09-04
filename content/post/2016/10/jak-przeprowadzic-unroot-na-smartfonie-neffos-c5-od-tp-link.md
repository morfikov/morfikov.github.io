---
author: Morfik
categories:
- Android
date: "2016-10-20T22:58:57Z"
date_gmt: 2016-10-20 20:58:57 +0200
published: true
status: publish
tags:
- tp-link
- smartfon
- lollipop
- root
- neffos
title: Jak przeprowadzić unroot na smartfonie Neffos C5 od TP-LINK
---

[Proces root na smartfonie Neffos
C5]({{< baseurl >}}/post/android-root-smartfona-neffos-c5-od-tp-link/) od TP-LINK można
przeprowadzić w miarę bez większych problemów, choć nie jest to rozwiązanie działające OOTB.
Niemniej jednak, taki root telefonu czyni go bardziej podatnym na zagrożenia ze strony wrogich
aplikacji. Ponadto, kasując czy też zmieniając pliki systemowe, możemy sprawić, że nasze urządzenie
zwyczajnie przestanie nam działać, tj. już się nie uruchomi. Niektórzy użytkownicy smartfonów nie
zdają sobie z tego sprawy i ukorzeniają Androida bez głębszego zastanowienia się. Mi jako
linux'iarzowi, root jest niezbędny do pracy ale czy aby na pewno każdy musi go mieć? Ci z was,
którzy taki root systemu przeprowadzili i nie korzystają z niego praktycznie wcale, zastanawiają
się pewnie czy istnieje sposób, by cofnąć wprowadzone zmiany i przywrócić Androida do stanu
pierwotnego. Krótka odpowiedź brzmi: "oczywiście, że tak" i temu procesowi przyjrzymy się w
niniejszym artykule.

<!--more-->
## Czy potrzebny mi jest root Android'a

Standardowo w Androidzie każda aplikacja zainstalowana w telefonie ma przypisane indywidualne
UID/GID (użytkownika i grupę). Żadna aplikacja nie jest w stanie odczytać danych innych programów,
które zainstalowaliśmy w systemie. Zaprzęgając mechanizm root dajemy możliwość pewnym aplikacjom na
dostęp do danych każdego innego programu.

![]({{< baseurl >}}/img/2016/10/001.neffos-c5-unroot-tp-link-prawa-aplikacji-andriud.png#huge)

Jeśli teraz wgramy podejrzaną aplikację, to może ona wykorzystać fakt ukorzenienia Androida i
uzyskać dostęp do poufnych danych czy nawet przejąc całkowitą kontrolę nad systemem operacyjnym
telefonu, wliczając to podsłuch z mikrofonu, kamery i klawiatury. Dlatego też w pewnych sytuacjach
root Androida nie jest wskazany.

Trzymając się pewnych zdroworozsądkowych zasad można uniknąć zagrożeń, które niesie ze sobą root
systemu. Niemniej jednak, jeśli niezbyt rozważnie korzystamy z telefonu, np. pobieramy aplikacje bez
ich uprzedniej weryfikacji, to root może tylko przyczynić się do kompromitacji zabezpieczeń
telefonu, przez co potencjalny atakujący może bez problemu uzyskać dostęp, np. do danych konta
bankowego.

## Jakie zmiany po przeprowadzaniu procesu root można cofnąć

Standardowo na smartfonie z Androidem mamy min. partycję `/system/` oraz `/data/` . Na `/system/`
znajduje się fabryczny Android, czyli ten system, który w tym przypadku został wypuszczony przez
TP-LINK. Tej partycji nie można zapisać bez przeprowadzenia procesu ukorzeniania systemu. Z kolei
zaś na partycji `/data/` są przechowywane wszystkie zmiany jakie użytkownik telefonu wprowadził,
np. za sprawą wgrania plików, zmiany konfiguracji/ustawień czy aktualizacji poszczególnych
aplikacji.

W przypadku posiadania smartfona, który przeszedł proces root, to na takim urządzeniu zostały
poczynione pewne zmiany. Przede wszystkim, aplikacje SuperSU i BusyBox wgrały nam pliki do katalogu
`/system/xbin/` (przynajmniej na Android 5.1 Lollipop). Dodatkowo, każda aplikacja wymagająca
uprawnień administratora mogła również wprowadzić jakieś zmiany na partycji `/system/` .

Wszelkie niestandardowe zmiany (edycja/dodanie/usunięcie plików systemowych) wprowadzane czy to
przez nas czy przez aplikacje wymagające root mogą zostać cofnięte przez wgranie wcześniej
zrobionego backup'u flash'a telefonu. Oczywiście nie musimy wgrywać całego backupu, a jedynie tę
część, na której znajduje się partycja `/system/` oraz `/data/` .

O ile przywrócenie partycji `/system/` jest wielce niezbędne, o tyle przywrócenie partycji `/data/`
może trwać sporo czasu. Biorąc pod uwagę, że są tam jedynie dane użytkownika, których nie ma za
wiele tuż po wyjęciu smartfona z pudełka, to przydałoby się wyczyścić również i tę przestrzeń przed
odkorzenieniem Androida. Ten proces możemy przeprowadzić korzystając z [Factory
Reset]({{< baseurl >}}/post/android-reset-ustawien-do-fabrycznych-factory-defaults/) z poziomu
systemu. Powinien nam on efektywnie te dane bardzo szybko usunąć.

Trzeba także pamiętać, że zmiany wprowadzone w procesie ukorzeniania telefonu nie ograniczają się
jedynie do partycji `/system/` . Została zmieniona przecież także partycja `recovery` , bez której
nie byłby możliwy root Neffos'a C5. Tę partycję również musimy przywrócić.

## Odinstalowanie SuperSU (unroot)

Od czego zatem zacząć powrót do stock'owego oprogramowania naszego Neffos'a C5? To pytanie trzeba
rozważyć pod kątem wprowadzanych zmian po dokonaniu procesu root. Jeśli nie były one zbyt
zaawansowane, np. instalowaliśmy jedynie kilka aplikacji wymagających praw administracyjnych w celu
odczytu plików systemowych, to możemy skorzystać z opcji dostępnej w SuperSU, tj. `Pełny unroot` :

![]({{< baseurl >}}/img/2016/10/002.neffos-c5-unroot-tp-link-supersu.png#huge)

Pamiętajmy tylko, by usunąć wszelkie aplikacje wymagające root przed usunięciem SuperSU.

W przypadku mojego Neffos'a C5, opcja unroot widoczna w SuperSU nie zadziałała z początku. Po
przyciśnięciu na ekranie pojawiła się jedynie informacja o czyszczeniu, a proces się zawiesił.
Jeśli też natrafiliśmy na tego typu problem, to trzeba zresetować smartfon i ponowić proces unroot
bezpośrednio po włączeniu urządzenia. Po odinstalowaniu SuperSU trzeba uruchomić smartfon ponownie.

Spróbujmy się teraz zalogować na użytkownika root z poziomu jakiegoś terminala. Powinniśmy zobaczyć
poniższy komunikat:

![]({{< baseurl >}}/img/2016/10/003.neffos-c5-unroot-tp-link-termux-su.png#huge)

Nie musimy się obawiać o dane zgromadzone na partycji `/data/` , bo nie zostaną one ruszone w żaden
sposób. Podobnie sprawa ma się w przypadku karty SD. No i nie zostaną cofnięte żadne zmiany na
partycji `/system/` , oczywiście w przypadku, gdy coś zmienialiśmy.

Zatem widzimy, że proces likwidacji root w Neffos C5 jest praktycznie automatyczny. Niemniej jednak,
co w przypadku, gdy mamy jakieś zmiany na partycji `/system/` lub też wgraliśmy niestandardowy ROM?
Odpowiedź jest prosta: trzeba przeprowadzić Factory Reset, który wyczyści dane na partycji `/data/`
oraz przywrócić partycję `/system/` z wcześniej utworzonego backupu flash'a smartfona.

Trzeba będzie także przywrócić partycję `recovery` , bo nie została ona odtworzona w procesie unroot
przeprowadzonym z poziomu SuperSU.

## Factory Reset

W przypadku wprowadzenia zmian na partycji `/system/` (innych niż wgranie SuperSU i BusyBox) dobrze
jest pierw usunąć wszystkie dane znajdujące się na partycji `/data/` . Chodzi o to, że pliki
zgromadzone na tej partycji mogą generować różne problemy w sytuacji, gdy przywraca się oryginalny
ROM. Nie jest to regułą ale jeśli chcemy uniknąć błędów, to zalecane jest przeprowadzić pierw
Factory Reset.

Oczywiście możemy ten krok całkowicie pominąć. W przypadku ewentualnych problemów po flash'owaniu
telefonu, Factory Reset będziemy mogli przeprowadzić z poziomu trybu recovery (przyciski Power +
Volume UP trzymane podczas startu telefonu). Jeśli jednak chcemy wyczyścić wszystkie dane na
partycji `/data/` przed flash'owaniem telefonu, to możemy to zrobić z poziomu działającego systemu
przechodząc do Ustawienia => Kopia i kasowanie danych => Ustawienia fabryczne:

![]({{< baseurl >}}/img/2016/10/004.neffos-c5-unroot-tp-link-factory-reset.png#huge)

## Przywrócenie partycji /system/ na Neffos C5

Jak już zostało wspomniane wyżej, na partycji `/system/` znajduje się ROM TP-LINK z Androidem 5.1.
By powrócić do niego musimy przywrócić całą partycję. Możemy to zrobić z poziomu [aplikacji SP Flash
Tool](http://spflashtool.com/), oczywiście zakładając, że pierw utworzyliśmy backup flash.
Zamontujmy ten backup w systemie przy pomocy poniższego polecenia:

    # losetup /dev/loop0 /media/Kabi/neffos/backup_phone/NeffosC5-orig.img

W systemie powinniśmy mieć dostęp do szeregu partycji tego obrazu. Jeśli się tak nie stało to musimy
odpowiednio [skonfigurować moduł
loop]({{< baseurl >}}/post/obsluga-wielu-partycji-w-module-loop/). Podejrzymy także w `gdisk` jak
prezentuje się tablica partycji samego obrazu. Interesuje nas generalnie pozycja `system` :

    # gdisk -l /media/Kabi/neffos/backup_phone/NeffosC5-orig.img

    Number  Start (sector)    End (sector)  Size       Code  Name
    ...
      20          360448         8749055   4.0 GiB     0700  system
    ...

Widzimy przy niej numer 20. Teraz w systemie odszukujemy urządzenie loop mające numer 20. W moim
przypadku jest to `/dev/loop0p20` . Możemy także zamontować tę partycję, by się upewnić, że
faktycznie znajduje się na niej stock'owy system:

    # mount /dev/loop0p20 /mnt

Jeśli nie ma żadnego błędu i możemy przeglądać katalog `/mnt/` bez problemu, oznacza to, że jest to
ta partycja, której dane musimy przesłać na smartfona. Odmontujmy ją zatem i zrzućmy dane z tego
urządzenia loop do pliku przy pomocy `dd` :

    # dd if=/dev/loop0p20 of=./orig_system.img bs=2M
    2048+0 records in
    2048+0 records out
    4294967296 bytes (4.3 GB, 4.0 GiB) copied, 216.991 s, 19.8 MB/s

Mając wyodrębnioną partycję `/system/` możemy ją wgrać na smartfon przy pomocy SP Flash Tool.
Potrzebna nam jest tylko mapa przestrzeni flash, a ta siedzi w [pliku
mt6735-neffos-c5-tp-link-scatter.txt]({{< baseurl >}}/img/manual/mt6735-neffos-c5-tp-link-scatter.txt).
Jest tam również pozycja dotycząca partycji `/system/` . Odpalamy zatem SP Flash Tool i przechodzimy
na zakładkę Download, gdzie wskazujemy nasz plik `scatter.txt` :

![]({{< baseurl >}}/img/2016/10/005.neffos-c5-unroot-tp-link-sp-flash-tool-scatter.png#huge)

Mamy tutaj wyszczególnione obszary pamięci flash w Neffos C5, które możemy zapisać. Nas interesuje w
tej chwili tylko pozycja `system` . Zaznaczamy ją i upewniamy się, że nad tabelką wybraliśmy
`Download Only` . Może i tutaj jest słówko Download ale trzeba patrzyć na ten proces z perspektywy
telefonu, czyli to on będzie pobierał dane z komputera.

Podłączamy teraz Neffos'a C5 do portu USB komputera. Następnie w SP Flash Tool przyciskamy przycisk
Download i wyłączamy telefon. Następnie próbujemy go uruchomić w trybie recovery przyciskając
przycisk Power + Volume UP jednocześnie. Smartfon się nie uruchomi ale rozpocznie się proces
flash'owania. Sam proces powinien zakończyć się powodzeniem.

![]({{< baseurl >}}/img/2016/10/006.neffos-c5-unroot-tp-link-sp-flash-tool-flash.png#huge)

Teraz można wyciągnąć smartfona z portu USB i uruchomić.

## Przywrócenie partycji recovery

W przypadku partycji `recovery` możemy postąpić dokładnie w taki sam sposób, tj. wgrać stock'owy
obraz przy pomocy SP Flash Tool. Niemniej jednak, nie musimy tego robić po wgraniu obrazu partycji
`/system/` . W moim przypadku, partycja `recovery` została automatycznie odtworzona po wgraniu
obrazu partycji `/system/` .

## Sprawdzenie czy Neffos C5 ma root'a

Ostatnią rzeczą, która nam została to na własne oczy przekonanie się czy wszystkie zmiany zostały
cofnięte. Factory reset powinien nam wyczyścić wszystkie dane na partycji `/data/` . Powinniśmy mieć
także stock'ową partycję `/system/` oraz `recovery` . Z kolei root możemy sprawdzić za pomocą Root
Check:

![]({{< baseurl >}}/img/2016/10/007.neffos-c5-unroot-tp-link-root-check.png#medium)

## Zablokowanie bootloader'a

Ostatnią rzeczą, którą musimy zrobić, to zablokowanie bootloader'a. Odpalamy zatem smartfon w trybie
fastboot (Power + VolUp). Podłączamy także urządzenie do portu USB komputera i wpisujemy w terminalu
poniższe polecenie:

    # fastboot oem lock
    ...
    (bootloader) Start lock flow

    OKAY [4.132s]
    finished. total time: 4.132s

Po wydaniu tej komendy, na ekranie smartfona pojawi się poniższa
    informacja:

    If you lock the bootloader you will need to install official operating system software on this phone.

    To prevent unauthorized access to your personal data, locking the bootloader will also delete all personal data from your phone (a "Factory data reset").

    Press the Volume Up/Down button to select Yes or No.

    Yes (Volume Up) Lock bootloader
    No (Volume Down) Do not lock bootloader

Proces zablokowania bootloader'a usuwa wszystkie dane użytkownika (Factory Reset), zatem upewnij
się, że zrobiliśmy ewentualny backup. Przyciskamy teraz Volume Up. Po chwili zostaniemy
przeniesieni do menu wyboru. Restartujemy telefon wpisując w terminalu poniższe polecenie:

    # fastboot reboot

Smartfon zrestartuje się parokrotnie podczas procesu blokowania bootloader'a ale ostatecznie system
powinien się bez większego problemu załadować na domyślnych
ustawieniach.

![]({{< baseurl >}}/img/2016/12/008.neffos-c5-unroot-smartfon-tp-link-box.png#medium)
