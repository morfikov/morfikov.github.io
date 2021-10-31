---
author: Morfik
categories:
- Android
date:    2017-04-02 19:27:56 +0200
lastmod: 2017-04-02 19:27:56 +0200
published: true
status: publish
tags:
- tp-link
- smartfon
- root
- twrp
- neffos
- neffos-x1
- neffos-c5
- neffos-c5-max
- neffos-y5
- neffos-y5-l
- adb
- fastboot
GHissueID: 45
title: Root w smartfonach Neffos od TP-LINK (X1, C5, C5 MAX, Y5, Y5L)
---

Jakiś czas temu opisywałem proces ukorzeniania (root) smartfonów Neffos, a konkretnie były to
modele [C5][1], [C5 MAX][2], [Y5][3] i [Y5L][4]. Od tamtego czasu zdążyłem się nieco bardziej
zagłębić w struktury Androida i udało mi się ze źródeł [OMNI ROM][5] zbudować natwyne obrazy TWRP
dla każdego z tych ww. telefonów. Oczywiście TP-LINK ma w swojej ofercie jeszcze modele [C5L][6],
[Y50][7], [X1][8] oraz [X1 MAX][9] ale póki co nie będę w stanie przygotować obrazu TWRP i opisu
jak ukorzenić Androidy w trzech z tych czterech smartfonów. Chodzi o to, że C5L został wycofany z
produkcji i raczej nie wpadnie on w moje łapki. Natomiast modele Y50 oraz X1 MAX nie są jeszcze
dostępne w polskiej ofercie TP-LINK'a, przez co minie trochę czasu zanim uda mi się do nich dobrać.
Postanowiłem napisać świeży artykuł dotyczący procesu root w smartfonach Neffos C5, C5 MAX, Y5, Y5L
oraz X1. Po co pisać kolejny artykuł o ukorzenianiu Androida w Neffos'ach? Generalnie rzecz biorąc,
w tych poprzednich wpisach było bardzo dużo informacji zbędnych z punktu widzenia przeciętnego
użytkownika, który chce zrootować system w swoim telefonie. Teraz, gdy dysponuję natywnymi obrazami
TWRP własnej roboty i zdobyłem nieco wiedzy z zakresu operowania na Androidzie, to proces root jest
o wiele prostszy i właśnie dlatego przydałoby się to wszystko opisać na nowo.

<!--more-->
## Narzędzia ADB, fastboot (linux)

Przede wszystkim, by zabrać się za proces root'owania smartfonów Neffos C5, C5 MAX, Y5, Y5L i X1,
musimy przygotować sobie odpowiednie narzędzia. Zapewnią one nam możliwość rozmawiania z telefonem.
Będziemy potrzebować `adb` ([Android Debug Bridge][10]) oraz `fastboot` . Za obsługę `adb` w
Debianie odpowiada pakiet `android-tools-adb` . Z kolei jeśli chodzi zaś o `fastboot` , to musimy
doinstalować w systemie pakiet `android-tools-fastboot` :

    # apt-get install \
        android-tools-adb \
        android-tools-fastboot

## Jak odblokować bootloader w smartfonach Neffos

By skorzystać z narzędzia `fastboot` do wgrywania obrazu TWRP, musimy odblokować bootloader. Zanim
jednak odblokujemy bootloader, musimy włączyć opcje programistyczne. Przechodzimy zatem do
Ustawienia =\> Informacje o telefonie i klikamy kilka razy w numer kompilacji do momentu aż
dostaniemy informację o byciu programistą. W tej chwili w Ustawieniach pojawi się dodatkowa pozycja
"Opcje programistyczne". Tam z kolei zaznaczamy opcję "Zdjęcie blokady
OEM".

![neffos-smartfon-tp-link-root-unlock-bootloader](/img/2017/03/001.neffos-smartfon-tp-link-root-unlock-bootloader.png#huge)

Zdjęcie blokady OEM umożliwi odblokowanie bootloader'a. Pamiętajmy jednak, że proces odblokowania
bootloader'a usuwa wszystkie dane użytkownika, tj. podczas odblokowywania jest inicjowany Factory
Reset, który przywraca ustawienia telefonu do fabrycznych. Dane na karcie SD pozostają nietknięte.

### Odblokowanie bootloader'a w Neffos Y5 i Y5L

Po zdjęciu blokady OEM wyłączamy urządzenie i włączamy je trzymając przyciski Volume Down + Power.
Smartfon włączy się w trybie bootloader'a, a na jego ekranie będzie widoczne logo TP-LINK i Android.
Następnie na komputerze w terminalu wpisujemy poniższe polecenie:

    # fastboot devices
    8a8f289 fastboot

Wynik zwrócony wyżej świadczy o tym, że nasz Neffos działa w trybie bootloader'a i możemy z
urządzeniem się porozumieć za pomocą narzędzia `fastboot` .

Bootloader odblokowujemy przy pomocy poniższego polecenia:

    # fastboot oem unlock-go

Na ekranie smartfona powinien nam się pokazać zielony robocik informujący o przeprowadzaniu Factory
Reset. Po chwili ten proces dobiegnie końca, a smartfon uruchomi się ponownie na ustawieniach
fabrycznych.

### Odblokowanie bootloader'a w Neffos C5, C5 MAX i X1

W przypadku smartfonów Neffos C5, C5 MAX X1, jako że te urządzenia posiadają SoC od MediaTek, nie
trzeba zdejmować blokady bootloader'a, by wgrać obraz TWRP. Możemy zatem ten obraz wgrać bez
uprzedniego przywracania smartfona do ustawień fabrycznych. Niemniej jednak, na potrzeby
ujednolicenia kroków w tym HowTo, my tę blokadę ściągniemy.

Wyłączamy urządzenie i włączamy je trzymając przyciski Volume Up + Power. Z menu wybieramy "fastboot
mode". Następnie na komputerze w terminalu wpisujemy poniższe polecenia:

    # fastboot devices
    TSL7DA69OBSO49PJ        fastboot

    # fastboot oem unlock

Po wydaniu tej drugiej komendy, na ekranie smartfona powinno nam pokazać się poniższe ostrzeżenie:

    Unlock bootloader?

    If you unlock the bootloader, you will be able to install custom operating system software on this phone.

    A custom OS is not subject to the same testing as the original OS, and can cause your phone and installed application to stop working properly.

    To prevent unauthorized access to your personal data, unlocking the bootloader will also delete all personal data from your phone "factory reset".

    Press the Volume UP/Down buttons to select Yes/No.

Wciskamy Volume Up, by potwierdzić chęć odblokowania bootloader'a, po czym restartujemy smartfon
poniższym poleceniem:

    # fastboot reboot

## Tworzenie backup'u flash'a smartfona

Użytkownicy smartfonów pomijają jedną niezwykle ważną rzecz podczas ukorzeniania Androida. Jest nią
backup danych zgromadzonych w pamięci flash takiego urządzenia. Posiadając backup flash'a, jesteśmy
w stanie wybrnąć z 99,9% krytycznych sytuacji, które zwykle objawiają się zapętleniem startu systemu
telefonu. Jeśli tylko działa bootloader/preloader/loader, to posiadając kopię zapasową możemy bez
większego problemu odratować swój smartfon.

Backup flash'a można przeprowadzić z poziomu trybu recovery (TWRP) w przypadku każdego modelu
Neffos. Jeśli chodzi o Neffos Y5, Y5L i X1, to obraz TWRP można załadować bezpośrednio do pamięci
smartfona, nie czyniąc tym samym żadnych zmian w pamięci flash. Niemniej jednak, potrzebny nam jest
odblokowany bootloader.

Niestety w przypadku Neffos C5 i C5 MAX nie da rady załadować obrazu TWRP recovery bezpośrednio do
pamięci telefonu. Możemy natomiast posłużyć się narzędziem SP Flash Tool i w zasadzie do zrobienia
backup'u przy jego pomocy nie trzeba przeprowadzać żadnych dodatkowych zabiegów. Nie trzeba nawet
odblokowanego bootloader'a. Jedyne co nam potrzebne, to odpowiedni plik `scatter.txt` . Z SP Flash
Tool możemy korzystać także w przypadku Neffos'a X1.

Nie będę tutaj opisywał procesu przeprowadzania backup'u każdego z modelów Neffos, bo to w zasadzie
mija się z celem. Raz, że w starych wątkach ten proces został dość szczegółowo opisany, a dwa, że
bardzo rzadko będzie nam potrzebny cały backup, tj. wszystkie jego partycje.

Zwykle podczas procesu root są zmieniane jedynie partycje `/boot/` , `/recovery/` oraz `/system/` i
w zasadzie backup tylko tych obszarów flash'a telefonu należy przeprowadzić. Osoby, które są
zainteresowane dokonaniem ręcznej kopi zapasowej flash'a czy też kilku partycji urządzenia, odsyłam
do starych wątków opisujących proces root: [Neffos C5][11], [Neffos C5 MAX][12],[Neffos Y5][13] i
[Neffos Y5L][14].

## Obrazy TWRP recovery dla Neffos C5, C5 MAX, Y5, Y5L i X1

Smartfony Neffos C5, C5 MAX, Y5, Y5L i X1 nie są jeszcze oficjalnie wspierane przez TWRP recovery.
Niemniej jednak, na moim GitHub'ie znajduje się konfiguracja, która umożliwia zbudowanie obrazów
TWRP recovery ze źródeł Androida, a konkretnie ze źródeł OMNI ROM. [Pod tym linkiem są zamieszczone
gotowe obrazy TWRP recovery][15] dla poszczególnych
modeli smartfonów Neffos, które można pobrać i z powodzeniem wgrać na telefon via `fastboot` czy też
SP Flash Tool. Niżej zaś są także linki do repozytoriów z konfiguracją obrazów.

[Repo z konfiguracją dla Neffos Y5][16]

[Repo z konfiguracją dla Neffos Y5L][17]

[Repo z konfiguracją dla Neffos C5][18]

[Repo z konfiguracją dla Neffos C5 MAX][19]

[Repo z konfiguracją dla Neffos X1][20]

## Wgrywanie obrazu TWRP na smartfon Neffos

Mając odblokowany bootloader oraz pobrany stosowny plik TWRP recovery, możemy przejść do etapu
flash'owania smartfona. Ten proces w zasadzie nie różni się co do zasady ale w przypadku modeli Y5,
Y5L i X1 mamy możliwość załadowania obrazu TWRP do pamięci smartfona, np. gdy chcemy jedynie
ukorzenić Androida ale jednocześnie chcemy pozostać przy stock'owym recovery.

Przełączamy zatem telefon w tryb bootloader'a i przy pomocy narzędzia `fastboot` wgrywamy obraz TWRP
na partycję `/recovery/` wskazując w ostatnim argumencie ścieżkę do obrazu:

    # fastboot flash recovery tp-link-neffos-twrp-recovery.img

Po wgraniu obrazu musimy uruchomić smartfon w trybie recovery przez przyciśnięcie przycisków
VolumeUP + Power. Trzeba to zrobić natychmiast po procesie flash'owania. Gdyby nam się telefon
uruchomił ponownie tuż po wgraniu obrazu, to system weryfikując partycję `/recovery/` stwierdzi, że
znajduje się na niej nieoczekiwany kontent. Android podejmie próbę odtworzenia tej partycji podczas
startu systemu i w efekcie wygeneruje on sobie nowy obraz i wgra go na partycję `/recovery/`
przywracając stock'owy tryb recovery w naszym smartfonie. Jeśli do tego dojdzie, to trzeba będzie
jeszcze raz ponowić proces flash'owania obrazem TWRP.

Po wejściu w tryb recovery, naszym oczom powinien pokazać się poniższy obrazek:

![](/img/2017/03/002-neffos-smartfon-tp-link-root-twrp-zmiany.png#small)

Mamy tutaj zapytanie odnośnie wprowadzania zmian przez TWRP na partycji `/system/` . Zmiana jest w
zasadzie tylko jedna i polega ona na przepisaniu nazwy pliku `/system/recovery-from-boot.p` na
`/system/recovery-from-boot.bak ` . W ten sposób Android nie będzie w stanie przeprowadzić procesu
weryfikacji partycji `/recovery/` i nie będzie jej próbował przepisać podczas startu smartfona.
Jeśli nie zezwolimy TWRP na wprowadzenie tej zmiany, to jak tylko zresetujemy smartfon, Android
przywróci stock'ową partycję `/recovery/` i w późniejszym czasie trzeba będzie jeszcze raz wgrywać
obraz TWRP.

Trzeba tutaj wyraźnie zaznaczyć, że wyłączenie mechanizmu sprawdzania zawartości partycji
`/recovery/` może utrudnić odratowanie smartfona w przypadku, gdy coś stanie się z oprogramowaniem
rezydującym na tej partycji. Zwykle jednak nic złego się nie dzieje i można bez problemu zezwolić
TWRP na wprowadzenie wymaganych zmian.

#### Problemy w przypadku Neffos X1

Nie wiem dlaczego Neffos X1 ma pewne problemy, gdy TWRP wprowadzi swoje zmiany na partycji
`/system/` . Zezwalając TWRP na dokonanie zmian praktycznie uwalamy telefon (system nie chce
wystartować) i trzeba ratować się wgrywaniem stock'owego ROM'u via ADB Sideload. Oczywiście
późniejsze zmiany, które wprowadzimy, np. za sprawą procesu root, czy też instalując aplikacje
wymagające praw administratora systemu już takiego przykrego efektu nie wywołują. Dlatego też w
przypadku Neffos X1 nie zezwalajmy TWRP na dokonywanie zmian. Po zainstalowaniu SuperSU
prawdopodobnie nie będzie trzeba jeszcze raz wgrywać obrazu TWRP na partycję `/recovery/`.

## Instalacja SuperSU

Ostatnią rzeczą na drodze do ukorzenienia Androida na smartfonach Neffos jest wgranie aplikacji
umożliwiającej korzystanie różnym programom z praw administratora systemu w telefonie. W obrazach
TWRP recovery znajduje się odpowiedni moduł w pełni automatyzujący proces root w smartfonie. Nie
trzeba zatem manualnie pobierać paczki z [SuperSU][21]. Jedyne co musimy zrobić to z menu TWRP
wybrać Reboot =\> System. Przed zresetowaniem urządzenia, TWRP wyrzuci informację, że to urządzenie
nie ma jeszcze root'a i zapyta nas czy chcemy zainstalować SuperSU (w przypadku Neffos X1 niestety
trzeba instalować manualnie za pomocą [paczki dostępnej tutaj][21]):

![neffos-smartfon-tp-link-root-twrp-instalacja-supersu](/img/2017/03/003.neffos-smartfon-tp-link-root-twrp-instalacja-supersu.png#big)

TWRP zainstalował również swoją aplikację, która ma na celu umożliwić łatwą aktualizację obrazów
TWRP, gdy zostanie wypuszczona nowa wersją tego trybu recovery (można ją też pobrać z Google
Play).

![neffos-smartfon-tp-link-root-instalacja-supersu](/img/2017/03/004.neffos-smartfon-tp-link-root-instalacja-supersu.png#small)

Na tych dodatkowych ikonkach widocznych wyżej trzeba kliknąć i postępować zgodnie z instrukcjami
aktualizacji:

![neffos-smartfon-tp-link-root-instalacja-supersu](/img/2017/03/005.neffos-smartfon-tp-link-root-instalacja-supersu.png#huge)

![neffos-smartfon-tp-link-instalacja-twrp](/img/2017/03/006.neffos-smartfon-tp-link-instalacja-twrp.png#huge)

Po zainstalowaniu, restartujemy smartfon.

## Test root na smartfonach Neffos

Po zresetowaniu urządzenia możemy sprawdzić czy Android w naszym Neffos'ie został już ukorzeniony,
np. doinstalowując sobie [jedną z aplikacji][22], która nam ten sam rzeczy oznajmi. Poniżej są fotki
obrazujące root smartfonów Neffos C5, C5 MAX, Y5, Y5L i X1:

![neffos-smartfon-tp-link-sprawdzenie-root](/img/2017/03/007.neffos-smartfon-tp-link-sprawdzenie-root.png#huge)

![](/img/2017/05/008-neffos-x1-root-check.png#small)

## Unroot

Standardowo w Androidzie każda aplikacja zainstalowana w telefonie ma przypisane indywidualne
UID/GID (użytkownika i grupę). Żadna aplikacja nie jest w stanie odczytać danych innych programów,
które zainstalowaliśmy w systemie. Zaprzęgając mechanizm root dajemy możliwość pewnym aplikacjom na
dostęp do danych każdego innego programu.

Jeśli teraz wgramy podejrzaną aplikację, to może ona wykorzystać fakt ukorzenienia Androida i
uzyskać dostęp do poufnych danych czy nawet przejąc całkowitą kontrolę nad systemem operacyjnym
telefonu, wliczając to podsłuch z mikrofonu, kamery i klawiatury. Dlatego też w pewnych sytuacjach
root Androida nie jest wskazany.

Podczas ukorzeniania Androida poczyniliśmy zmiany na partycji `/recovery/` (wgrane TWRP) oraz
`/system/` i `/boot/` (na potrzeby SuperSU). By teraz powrócić do fabrycznego firmware, te zmiany
trzeba cofnąć. Jeśli nie instalowaliśmy żadnych dodatkowych aplikacji wymagających praw
administratora root, to wystarczy odinstalować SuperSU oraz przywrócić stock'owe obrazy partycji
`/recovery/` oraz `/boot/` .

Natomiast w przypadku instalowania dodatkowego oprogramowania, to niestety trzeba już wgrać
stock'owy obraz na partycję `/system/` . Najprościej jest po prostu wgrać wszystkie te trzy ww.
obrazy i wtedy będziemy mieć pewność, że powrócimy do fabrycznego oprogramowania. Obrazy wgrywamy
przy pomocy narzędzia `fastboot` :

    # fastboot flash system orig-system.img
    # fastboot flash recovery orig-recovery.img
    # fastboot flash boot orig-boot.img

Następnie czyścimy cache:

    # fastboot format cache

No i na koniec zakładamy blokadę na bootloader, co zainicjuje również proces Factory Reset,
czyszcząc tym samym wszystkie dane użytkownika:

    # fastboot oem lock

### Proces Root Integrity Check

Smartfony Neffos są wyposażone w mechanizm, który jest w stanie zweryfikować integralność danych w
systemie. W przypadku powracania ze zrootowanego Androida do stock'owego firmware TP-LINK'a, dobrze
jest przeprowadzić [proces Root Integrity Check][23] z poziomu trybu recovery tak, by upewnić się,
że faktycznie powróciliśmy do oryginalnego oprogramowania oraz, że nie będzie problemów z
ewentualnymi aktualizacjami systemu telefonu w późniejszym czasie.

## Brak wolnego miejsca i zmiana układu partycji flash'a

Neffos C5 i C5 MAX mają wydzielone 4 GiB na partycję `/system/` . Jest to dość sporo biorąc pod
uwagę fakt, że TP-LINK'owy ROM jest w stanie się zmieścić na około 2 GiB. Z kolei w przypadku
Neffos'ów Y5 i Y5L, partycja `/system/` ma około 1,8 GiB i w zasadzie zostaje nam do dyspozycji
bardzo niewiele wolnego miejsca. Jest to rząd wielkości 30-50 MiB.

We wszystkich tych modelach smartfonów, układ partycji na flash'u jest do wymiany. Oczywiście nic
nie stoi na przeszkodzie by w Neffos C5 i C5 MAX mieć przeznaczone 4 GiB na partycję `/system/` ale
wtedy tracimy trochę cennej przestrzeni, którą można by dołączyć do partycji `/data/` , a tym samym
mieć więcej miejsca na dane użytkownika.

Poważny problem za to zaczyna się w przypadku Neffos Y5 i Y5L, gdzie mało miejsca na partycji
`/system/` może prowadzić do niestabilności systemu lub niemożliwości jego uruchomienia się. Tutaj
mamy po prostu idealnie wykrojoną część flash'a pod stock'owy ROM. Natomiast, gdy przychodzi do
zabaw z prawami administratora root i wgrywaniem aplikacji, które operują na partycji `/system/` ,
to niestety musimy się liczyć z faktem, że w przypadku tych dwóch smartfonów może nam tego wolnego
miejsca zwyczajnie zabraknąć.

Rozwiązaniem jest naturalnie fizyczne usunięcie szeregu partycji z pamięci flash telefonu i
stworzenie ich od podstaw. To zadanie jednak wykracza poza ramy tego artykułu. Niemniej jednak,
planuję tego typu zabieg przeprowadzić i dokładnie go opisać. (link FIXME).

Informacje na temat [zmiany układu partycji na flash'u w smartfonach Neffos C5 i C5 MAX można
znaleźć tutaj][24].

## Problemy i niebezpieczeństwa związane z procesem root

Ja posiadam kilka smartfonów, które mają ukorzenionego Androida, tj. został na tych urządzeniach
przeprowadzony proces root. Tego typu zabieg wyłącza praktycznie wszystkie (albo znaczną większość)
mechanizmów obronnych naszego telefonu. Biorąc pod uwagę fakt, że cała masa użytkowników smartfonów
(nie tylko Neffos'ów od TP-LINK) ukorzenia te urządzenia bez wiedzy co tak naprawdę robi, to
postanowiłem napisać kilka słów odnośnie problemów, którym użytkownik ukorzenionego systemu będzie
musiał stawić czoła.

Przede wszystkim, muszę tutaj zaznaczyć, że samymi smartfonami, a właściwie systemem Android,
interesuję się od kilku miesięcy i w zasadzie nie poznałem go jeszcze w pełni. Niemniej jednak, od
czasu do czasu rozpracowuje sobie pewne rzeczy w oparciu o dwie wersje Androida: 5.1 (Lollipop) oraz
6.0 (Marshmallow). W tym miejscu chciałbym zebrać wszystkie swoje artykuły, które pokazują jak
proces root wpływa na bezpieczeństwo systemu oraz które z jego funkcji przestają działać lub tez są
w znacznym stopniu upośledzone.

To, że akurat ja korzystam z ukorzenionego Androida, nie znaczy, że i ty powinieneś, zwłaszcza w
przypadku, gdy bezpieczeństwo danych przechowywanych w telefonie ma dla ciebie nadrzędne znaczenie.
W zasadzie wszystkie z czterech modeli smartfonów Neffos dostępnych na polskim rynku, tj. C5, C5
MAX, Y5 i Y5L, można zrootować bez większego problemu, co widzieliśmy wyżej.

Jeśli chodzi o mnie, to proces root bardzo ułatwia mi rozpracowanie samego systemu i sprawia, że mam
wgląd w miejsca, w które standardowy użytkownik telefonu zajrzeć nie może, bo Android odmawia mu
dostępu właśnie ze względów bezpieczeństwa. Dlatego też jeśli nie potrzebujesz rootować systemu w
telefonie, to tego po prostu nie rób.

### Problemy z lokalizacją skradzionego smartfona

Jednym z bardziej podstawowych mechanizmów ochronnych, które oferuje Google w Androidzie, to usługa
lokalizacji smartfona na wypadek jego utraty czy kradzieży. No w przypadku zwykłego zawieruszenia
się naszego telefonu raczej nic nam nie grozi ale, gdy takie urządzenie zostanie nam skradzione, to
wtedy mamy bardzo poważny problem.

Przede wszystkim, mając odblokowany bootloader, który jest wymagany do ukorzenienia Neffos'ów,
dajemy złodziejowi narzędzie zresetowania smartfona i obejścia tym samym [blokady Factory Reset
Protection Lock][25] (FRP Lock). Gdy złodziej obejdzie tę blokadę jest w stanie przywrócić system
urządzenia do ustawień fabrycznych, np. w celu odsprzedania telefonu komuś trzeciemu. W takim
przypadku nasz smartfon nie będzie już dłużej powiązany z konkretnym kontem Google i zlokalizowanie
go przez ww. usługę będzie zwyczajnie niemożliwe.

[Więcej informacji na temat lokalizacji zagubionych/skradzionych smartfonów można znaleźć w osobnym
wątku.][26]

### Możliwość obejścia blokady ekranu

Standardowo każdy z nas korzysta z blokady ekranu w swoich telefonach. Ja akurat mam opcję "Przesuń
palcem" ale jakby nie patrzeć, to też blokada. Ci z was, którzy wykorzystują PIN, wzór albo hasło,
mogą nieco się zawieść w przypadku, gdy mają ukorzenionego Androida. Cały ten mechanizm blokady
ekranu opiera się o ustawienia stosownej aplikacji i klucz zabezpieczający. Wszystkie te dane są
przechowywane w plikach na flash'u smartfona. Jeśli teraz mamy zdjętą blokadę bootloader'a, bo
chcieliśmy sobie ukorzenić system, to dostęp do tych kluczy i ustawień pozostaje niechroniony i
można zresetować blokadę ekranu przez tryb recovery.

[Więcej informacji na temat resetowania ustawień blokady ekranu można znaleźć w osobnym wątku][27]
(ostatni nagłówek).

### Odszyfrowanie zawartości karty SD sformatowanej jako pamięć wewnętrzna

Nowsze wersje Androidów (6.0+) są w stanie rozbudować pamięć flash w smartfonach przy wykorzystaniu
[Adoptable Storage][28]. Ten mechanizm zakłada sformatowanie karty SD systemem plików, który daje
możliwość wykorzystania uprawnień do plików w celu poprawienia bezpieczeństwa systemu i poufności
danych przechowywanych na samej karcie SD. Domyślnie dane na tej karcie SD są szyfrowane i nie da
rady do tych informacji uzyskać dostępu z poziomu innego urządzenia. Możemy w zasadzie korzystać z
tej karty na smartfonie, gdzie została ona sformatowana jako pamięć wewnętrzna i nic poza tym.

Do szyfrowania zawartości karty jest wykorzystywany losowy klucz szyfrujący, który jest tworzony w
procesie formatowania karty SD. Ten klucz nie jest zabezpieczony żadnym hasłem (np. tym od blokady
ekranu) i leży sobie jak gdyby nigdy nic na flash'u smartfona. Mając przeprowadzony proces root, ten
klucz jest dostępny praktycznie dla każdego, przez co jakiekolwiek szyfrowanie danych na karcie SD
jest tylko złudzeniem bezpieczeństwa i niepotrzebnie obciąża procesor telefonu.

[Więcej informacji na temat odszyfrowania zawartości karty SD sformatowanej jako pamięć można
znaleźć w osobnym wątku.][29]

### Inne problemy

Oczywiście, te powyżej wypisane niedogodności nie są jedynymi. Prawdopodobnie jest ich jeszcze cała
masa ale, jako że mam ciągle "niewielkie" doświadczenie z Androidem, to jeszcze nie wszystkie
rzeczy udało mi się wyłapać. Niemniej jednak, jak tylko coś ciekawego znajdę, to naturalnie opiszę i
dodam tutaj stosowny nagłówek, tak by ta lista była możliwie rozbudowana, co może przyczyni się do
większej świadomości osób korzystających z Androida i zaowocuje zastanowieniem się nad tym, czy
faktycznie dany użytkownik potrzebuje ukorzenionego Androida w swoim smartfonie.


[1]: http://www.neffos.pl/product/details/C5
[2]: http://www.neffos.pl/product/details/C5-Max
[3]: http://www.neffos.pl/product/details/Y5
[4]: http://www.neffos.pl/product/details/Y5L
[5]: https://omnirom.org/
[6]: http://www.neffos.pl/product/details/C5L
[7]: http://www.neffos.com/en/product/details/Y50
[8]: http://www.neffos.com/en/product/details/X1
[9]: http://www.neffos.com/en/product/details/X1Max
[10]: https://developer.android.com/studio/command-line/adb.html
[11]: /post/android-root-smartfona-neffos-c5-od-tp-link/
[12]: /post/android-root-smartfona-neffos-c5-max-od-tp-link/
[13]: /post/android-root-smartfona-neffos-y5-od-tp-link/
[14]: /post/android-root-smartfona-neffos-y5l-tp-link/
[15]: https://app.box.com/v/tp-link-neffos-twrp-recovery
[16]: https://github.com/morfikov/android_device_tp-link_tp802a
[17]: https://github.com/morfikov/android_device_tp-link_tp801a
[18]: https://github.com/morfikov/android_device_tp-link_tp701a
[19]: https://github.com/morfikov/android_device_tp-link_tp702a
[20]: https://github.com/morfikov/android_device_tp-link_tp902a
[21]:https://forum.xda-developers.com/apps/supersu/stable-2016-09-01supersu-v2-78-release-t3452703
[22]: https://play.google.com/store/apps/details?id=com.jrummyapps.rootchecker
[23]: /post/root-integrity-check-w-smartfonach-z-androidem/
[24]: /post/repartycjonowanie-flash-w-neffos-c5-i-c5-max-od-tp-link/
[25]: /post/factory-reset-protection-frp-w-smartfonach-z-androidem/
[26]: /post/jak-zlokalizowac-skradziony-zagubiony-smartfon-z-androidem/
[27]: /post/backup-partycji-data-w-smartfonach-przez-recovery-twrp/
[28]: https://source.android.com/devices/storage/adoptable
[29]: /post/jak-odszyfrowac-zawartosc-karty-sd-w-smartfonie-z-androidem/
