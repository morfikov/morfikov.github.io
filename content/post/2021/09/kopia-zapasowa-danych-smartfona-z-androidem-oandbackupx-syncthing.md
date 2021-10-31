---
author: Morfik
categories:
- Android
- Linux
date:    2021-09-26 16:44:00 +0200
lastmod: 2021-09-26 16:44:00 +0200
published: true
status: publish
tags:
- debian
- smartfon
- xiaomi
- redmi-9
- root
- backup
- aplikacje
- f-droid
- prywatność
GHissueID: 327
title: Kopia zapasowa danych smartfona z Androidem (OAndBackupX, Syncthing)
---

Jak to zwykło się mawiać w świecie IT: ludzie dzielą się na te osobniki, które robią backup i
te, co backup robić będą. Jakiś czas temu pochyliłem się nad zagadnieniem [czy smartfon z
Androidem bez Google Play Services ma sens][2]. Poruszyłem tam problem tworzenia kopi zapasowej
danych użytkownika zgromadzonych w telefonie. W tym podlinkowanym artykule zostało przestawione jak
przy pomocy TWRP dokonać pełnego backupu całego systemu urządzenia i o ile pod względem technicznym
takie podejście było jak najbardziej w pełni do zaakceptowania, to jednak to rozwiązanie miało dość
istotną wadę, tj. na czas backup'u trzeba było wyłączyć smartfon. Parę dni temu natrafiłem
na [narzędzia OAndBackupX][1], które jest w stanie zrobić kopię zapasową wszystkich zainstalowanych
w Androidzie aplikacji (oraz ich ustawień), wliczając w to nawet appki systemowe. OAndBackupX jest
o tyle lepszym rozwiązaniem w stosunku do TWRP, że można przy jego pomocy robić kopię danych
pojedynczych aplikacji, a nie od razu całego ROM'u, co nie tylko jest czasochłonne ale również
zjada sporo miejsca na dysku komputera, czy gdzie ten backup zamierzamy przechowywać. Niestety
OAndBackupX wymaga uprawnień root, zatem na stock'owym ROM'ie producenta naszego telefonu nie damy
rady z tego narzędzia skorzystać. Jeśli jednak mamy alternatywny ROM na bazie AOSP/LineageOS, to
przydałoby się rzucić okiem na ten kawałek oprogramowania, bo zdaje się ono być wielce użyteczne,
zwłaszcza w przypadku osób mojego pokroju, czyli linux'iarzy, którzy o backup swoich danych chcą
zatroszczyć się tylko i wyłącznie we własnym zakresie, zaprzęgając do pracy choćby [Syncthing][3].

<!--more-->
## Konfiguracja Syncthing do pracy z OAndBackupX

Przede wszystkim trzeba tutaj wyraźnie zaznaczyć, że będziemy potrzebować dwóch narzędzi, by taki
backup danych smartfona utworzyć. OAndBackupX tworzy co prawda backup danych aplikacji ale ich
kopia zapasowa będzie przechowywana lokalnie w telefonie, tj. w jakimś dedykowanym katalogu, który
sobie określimy. Ten katalog naturalnie można by zaciągnąć ze smartfona przy pomocy ADB ale raz, że
trzeba by to robić manualnie za każdym razem, a dwa, to trzeba by podłączać smartfona do komputera
via port USB. Zamiast się bawić w takie prymitywne rozwiązania pokroju ADB, można zaprzęgnąć do
pracy Syncthing, który to jest w stanie przy pomocy sieci WiFi przesłać synchronizowane katalogi
na dowolną maszynę w sieci i to nie tylko w obrębie naszego domowego WiFi.

Zarówno [Syncthing][4] jak i [OAndBackupX][5] są dostępne w repozytorium F-Droid i raczej nie
powinniśmy mieć problemów z ich instalacją na telefonie. By jednak móc zsynchronizować katalogi
backup'u ze smartfona na komputer, na obu tych urządzeniach musimy zainstalować Syncthing. Na
szczęście Syncthing jest dostępny na linux'a i z jego instalacją na Debianie też nie powinno być
żadnego problemu. Wystarczy wgrać pakiet `syncthing` , który standardowo [jest dostępny][6] w
głównym repozytorium tej dystrybucji.

### Konfiguracja Syncthing pod linux'em

Może na początek zajmijmy się skonfigurowaniem Syncthing pod linux. Przede wszystkim, jeśli
korzystamy z systemd, to Syncthing ma do zaoferowania usługę `syncthing.service` , którą można
uruchomić jako zwykły użytkownik. Możemy ją naturalnie sobie włączyć, lub też ręcznie odpalać
Syncthing ilekroć tylko będziemy mieli zamiar synchronizować katalog kopi zapasowej na smartfonie.

#### Konfiguracja firewall'a

Syncthing do synchronizacji wykorzystuje protokoły TCP, UDP oraz IP. Jeśli nasz linux ma
restrykcyjny firewall, który blokuje połączenia przychodzące, to ten FW trzeba odpowiednio
skonfigurować, by połączenia do Syncthing mogły być zrealizowane. [Musimy sobie otworzyć na
zaporze][7] porty `22000/TCP` (synchronizacja via TCP) , `22000/UDP` (synchronizacja via QUIC)
oraz `21027/UDP` (local discovery broadcast), przy czym ten ostatni nie do końca jest potrzebny
jeśli będziemy ręcznie określać końcówki synchronizacji. Reasumując, na firewall'u powinniśmy
wpisać mniej więcej takie reguły:

    create chain inet filter input-tcp
    create chain inet filter input-udp

    add rule inet filter INPUT meta l4proto tcp tcp flags & (fin|syn|rst|ack) == syn ct state new counter jump input-tcp
    add rule inet filter INPUT meta l4proto udp ct state new counter jump input-udp

    add rule inet filter input-tcp tcp dport { 22000 } ip saddr 192.168.1.0/24 counter accept comment "syncthing ipv4"
    add rule inet filter input-tcp tcp dport { 22000 } ip6 saddr fe80::/64 counter accept comment "syncthing ipv6"

    add rule inet filter input-udp udp dport { 22000 } ip saddr 192.168.1.0/24 counter accept comment "syncthing ipv4"
    add rule inet filter input-udp udp dport { 22000 } ip6 saddr fe80::/64 counter accept comment "syncthing ipv6"

    add rule inet filter input-udp udp dport { 21027 } ip daddr 255.255.255.255 counter accept comment "syncthing announce ipv4"
    add rule inet filter input-udp udp dport { 21027 } ip daddr 192.168.1.255 counter accept comment "syncthing announce ipv4"
    add rule inet filter input-udp udp dport { 21027 } ip6 daddr ff12::8384 counter accept comment "syncthing announce ipv6"

#### Zarządzanie Syncthing przez panel webowy

Gdy mamy odpowiednio skonfigurowaną zaporę sieciową, możemy uruchomić Syncthing, np. wpisując
`syncthing` w terminalu. Powinniśmy zostać automatycznie przekierowani do przeglądarki internetowej
na adres `http://127.0.0.1:8384/` . To właśnie przy pomocy panelu webowego będziemy konfigurować
Syncthing na linux.

W zasadzie nie mamy tutaj póki co za wiele do roboty. Jedyne co musimy w tej chwili skonfigurować,
to określić w zakładce `General` nazwę urządzenia w `Device Name` oraz domyślną ścieżkę do
katalogu w `Default Folder Path`, w którym będą przechowywane synchronizowane foldery.

![oandbackupx-syncthing-backup-android-phone-settings-linux](/img/2021/09/001.oandbackupx-syncthing-backup-android-phone-settings-linux.png#huge)

W zakładce `Connections` przydałoby się określić adres (i port), na którym Syncthing będzie
nawiązywał połączenia. [Ustawienia domyślne wskazują na default][8] ale lepiej jest je nieco
zawęzić. Możemy też wyłączyć mechanizm `Global Discovery` , jako że nie zamierzamy wykorzystywać
internetu do synchronizacji, a jedynie będziemy ją przeprowadzać w obrębie naszej domowej sieci
WiFi. Jeśli nie chcemy korzystać z lokalnego mechanizmu rozgłaszania, to też możemy go wyłączyć.

![oandbackupx-syncthing-backup-android-phone-settings-linux](/img/2021/09/002.oandbackupx-syncthing-backup-android-phone-settings-linux.png#huge)

To w zasadzie wszystko, co musimy ustawić pod linux, by Syncthing nam zadziałał. Teraz trzeba
jeszcze skonfigurować drugą końcówkę synchronizacji, tj. nasz smartfon z Androidem.

### Konfiguracja Syncthing pod Androidem

Konfiguracja Syncthing pod Androidem nie różni się zbytnio od tej przeprowadzanej pod linux'em. W
zasadzie trzeba dokonać tych samych zmian w ustawieniach, co w przypadku Syncthing na linux, tylko
tutaj możemy to zrobić z poziomu dedykowanej aplikacji (jest też opcja przełączenia do panelu
webowego):

|   |   |
|---|---|
| ![](/img/2021/09/003.oandbackupx-syncthing-backup-android-phone-settings-phone.jpg#small) | ![](/img/2021/09/004.oandbackupx-syncthing-backup-android-phone-settings-phone.jpg#small) |

|   |   |
|---|---|
| ![](/img/2021/09/005.oandbackupx-syncthing-backup-android-phone-settings-phone.jpg#small) |![](/img/2021/09/006.oandbackupx-syncthing-backup-android-phone-settings-phone.jpg#small) |

Warto tutaj też wspomnieć, że Syncthing jest w stanie pracować z prawami root. Niemniej jednak, ta
funkcja jest bardzo niestabilna i powoduje bardzo dziwne problemy i nie zalecałbym z niej korzystać.
Jeśli jednak chcemy uruchamiać Syncthing na prawach root'a, to w przypadku, gdy mamy na Androidzie
zainstalowany jakiś firewall, np. [AFWall][10], to upewnijmy się, że połączenia do Syncthing nie są
w żaden sposób blokowane. Poza tym, to Syncthing zdaje sobie radzić bez tej funkcji dość dobrze,
choć naturalnie nie będziemy mieli możliwości robienia kopii zapasowych plików systemowych ale nas
ten problem nie będzie dotyczył, bo zarówno appki, jak i ich ustawienia, wyciągniemy sobie przez
OAndBackupX i katalog backup'u już będzie można bez większego problemu zaciągnąć na komputer przez
Syncthing bez uciekania się do uprawnień root.

## Co potrafi OAndBackupX

OAndBackupX niestety (albo stety) do prawidłowego swojego działania wymaga uprawnień root. Jeśli
nie mamy ukorzenionego Androida, to trzeba się raczej będzie zainteresować innym oprogramowaniem
pokroju [Seedvault][9], który to potrafi robić kopię zapasową aplikacji według wytycznych AOSP,
czyli nie wymaga root i respektuje brak robienia backup'u w przypadku aplikacji, które sobie tego
nie życzą. OAndBackupX robi backup wszystkich danych, w tym tych prywatnych (które Seedvault
zostawia w spokoju). Dzięki temu będziemy mieli kopię zapasową całej aplikacji, a nie tylko tych
ustawień, które twórca danej appki chciałby do backup'u wrzucić. Oczywiście jedno i drugie
rozwiązanie ma swoje wady i zalety ale głównym czynnikiem determinującym wybór jednego lub drugiego
rozwiązania będzie posiadanie praw administratora systemu.

Przechodząc do rzeczy, OAndBackupX nie tylko potrafi zrobić kopię zapasową konkretnych aplikacji
(wliczając w to też te systemowe) ale również możemy go zaprogramować w taki sposób, by cyklicznie
taki backup przeprowadzał, np. raz w tygodniu.

Poniżej znajdują się poglądowe fotki ustawień OAndBackupX, które po części zostaną omówione poniżej:

|   |   |
|---|---|
| ![](/img/2021/09/007.oandbackupx-syncthing-backup-android-phone-oabx-settings.jpg#small) | ![](/img/2021/09/008.oandbackupx-syncthing-backup-android-phone-oabx-settings.jpg#small) |

|   |   |
|---|---|
| ![](/img/2021/09/009.oandbackupx-syncthing-backup-android-phone-oabx-settings.jpg#small) | ![](/img/2021/09/010.oandbackupx-syncthing-backup-android-phone-oabx-settings.jpg#small) |

Te bardziej interesujące nas funkcje OAndBackupX, z którymi mogliśmy się zapoznać patrząc na
powyższe fotki, to `Backup folder` , który określa katalog przechowywania kopi zapasowych. To
właśnie ten folder będziemy synchronizować sobie na komputer via Syncthing.

### Czy szyfrować kopię zapasową

Kolejna sprawa, to szyfrowanie kopi zapasowej. Ta opcja nie jest wymagana ale bardzo zalecana.
Chodzi o to, że każda appka w Androidzie ma przypisanego innego użytkownika i tylko ta konkretna
aplikacja ma wgląd w swoje pliki. Gdy teraz te ustawienia poszczególnych aplikacji wyciągniemy do
zewnętrznego katalogu, to potencjalnie każda appka w systemie może mieć dostęp do tego folderu i
uzyskać poufne informacje, co może skończyć się tragicznie jeśli złapiemy na fonie jakiś syf.
Dlatego szyfrowanie kopi zapasowej powinniśmy sobie ustawić.

#### Jak odszyfrować backup na linux

Szukając informacji na temat ewentualnego odszyfrowania backup'u zrobionego via OAndBackupX
natrafiłem na szereg postów, w których użytkownicy zakładali wykorzystanie OpenSSL. Problem jednak
z OpenSSL jest taki, że [nie wspiera szyfrów AEAD][13] (np. AES-GCM), z których [OAndBackupX robi
użytek][14]. Deweloperzy OpenSSL zaznaczyli też, że wsparcie dla szyfrów AEAD w tej bibliotece pod
kątem szyfrowania/deszyfrowania plików nie będzie wspierane, bo niesie ze sobą całą masę problemów
natury bezpieczeństwa. Póki co, nie jest mi znany sposób odszyfrowania takiego backup'u z poziomu
linux'a. Być może w stosownym czasie pojawi się [HOWTO na ten temat][15].

### Które dane aplikacji możemy wrzucić do backup'u

Idąc dalej, OAndBackupX umożliwia nam zrobienie kopi zapasowej danych aplikacji w telefonie takich
jak:

 - cache (zbędne i domyślnie wyłączone)
 - chronione pliki aplikacji (katalog `/data/user_de/0/` )
 - pliki trzymane w pamięci zewnętrznej:
   + katalog `/storage/emulated/0/Android/data/`
   + katalog `/storage/emulated/0/Android/media/`
   + katalog `/storage/emulated/0/Android/obb/`

Dowolny miks tych powyższych można sobie określić, co naturalnie wpłynie na rozmiar kopi zapasowej.
Najlepiej wybrać wszystkie opcje za wyjątkiem cache, by mieć pełny backup danych aplikacji.

Warto tutaj zaznaczyć, że wykonanie kopi zapasowej wszystkich tych powyższych elementów nie zmusza
nas do ich odtworzenia w takiej formie w jakiej je zrobiliśmy. Będziemy w stanie odtworzyć, np.
dane z katalogów `/data/user_de/0/` i `/storage/emulated/0/Android/data/` . Wystarczy, że w
OAndBackupX na zakładce `Main` wybierzemy sobie konkretną appkę, której dane zamierzamy odtworzyć i
wybrać jeden z utworzonych wcześniej backup'ów:

|   |   |
|---|---|
| ![](/img/2021/09/018.oandbackupx-syncthing-backup-android-phone-restore.jpg#small) | ![](/img/2021/09/019.oandbackupx-syncthing-backup-android-phone-restore.jpg#small) |

### Rewizje kopi zapasowych

Użyteczniejszą opcją jest także możliwość przechowywania kilku rewizji backup'u, tj. np. trzech
ostatnich kopi zapasowych. Ja jednak preferuję tylko jedną kopię zapasową, a ewentualne rewizje
backup'ów będę robił już z poziomu samego linux'a, tak by nie nadwyrężać pamięci flash w telefonie.

Jeśli mamy mało dostępnej pamięci flash w smartfonie, to powinniśmy też ustawić sobie usunięcie
poprzedniej rewizji backup'u przed próbą tworzenia nowej kopi zapasowej. W ten sposób zredukujemy
potrzebne miejsce na backup o połowę, bo na flash'u telefonu nie będzie przechowywana stara kopia w
chwili tworzenia nowej.

### Specjalne kopie zapasowe

Zgodnie z tym [co można wyczytać w FAQ][11], specjalne kopie zapasowe odnoszą się do procesu
tworzenia backup'u danych systemowych, które nie wchodzą w skład zainstalowanych aplikacji.
Niestety póki co OAndBackupX nie wspiera tworzenia kopi zapasowych tych obszarów telefonu w pełni i
korzystanie z tej funkcji jest póki co wysoce eksperymentalne. Potencjalne uszkodzenie telefonu w
postaci zapętlenia się procedury startu Androida może nam się przytrafić. Dlatego też lepiej nie
korzystać z tej opcji, przynajmniej na razie.

### A co z aplikacjami, które aktualnie działają

OAndBackupX jest w stanie wykonać backup danych aplikacji, które są uruchomione. To zadanie jest
realizowane przez [wysyłanie odpowiednich sygnałów][12] do aplikacji. Sygnał `SIGSTOP` jest
wysyłany tuż przed kopiowaniem jej plików, przez co ta aplikacja jest wstrzymywana (procesor
przestaje wykonywać jej kod) i nie dokonuje w tych plikach żadnych zmian. Jak tylko pliki zostaną
skopiowane, to wysyłany jest do aplikacji sygnał `SIGCONT` i aplikacja może ponownie wznowić swoją
pracę i działać tak jak przedtem. Nie trzeba zatem zamykać czy siłowo ubijać aplikacji na czas
wykonywania kopi zapasowej jej danych.

|   |   |
|---|---|
| ![](/img/2021/09/013.oandbackupx-syncthing-backup-android-phone-oabx-apps.jpg#small) | ![](/img/2021/09/014.oandbackupx-syncthing-backup-android-phone-oabx-apps.jpg#small) |

Najwyraźniej system Androida jest w stanie sam z siebie wysłać do aplikacji sygnał `SIGCONT`
(zwłaszcza do tych systemowych aplikacji), gdy aktywnie korzystamy z telefonu. W takim przypadku
OAndBackupX może mieć problem ze zrobieniem backup'u. Zwykle jednak kopia zapasowa większości
aplikacji zostanie utworzona. Gdy jednak trafią nam się takie problematyczne appki, to możemy
próbować manualnie utworzyć kopię zapasową tych pozostałych aplikacji. Niemniej jednak, na czas
tworzenia kopii zapasowej dobrze jest zostawić telefon w spokoju.

### Harmonogram tworzenia kopi zapasowych

Ręczne tworzenie kopi zapasowych może bywać męczące i nie chodzi tutaj o backup każdej pojedynczej
aplikacji z tych 200 czy 300, które aktualnie mamy zainstalowane w systemie. OAndBackupX umożliwia
robienie zbiorczych backup'ów, tj. wszystkich aplikacji jedna po drugiej, choć taki proces potrafi
zająć nawet i kilkanaście czy kilkadziesiąt minut, w zależności od ilości aplikacji, ilości danych,
no i też szybkości procka czy flash'a w telefonie. Problematyczna może być raczej nasza
systematyczność w tworzeniu takich kopi zapasowych, bo zwykle o ich utworzeniu możemy zwyczajnie
zapomnieć. Dlatego też OAndBackupX posiada wbudowany harmonogram, przy pomocy którego jesteśmy w
stanie skonfigurować automatyczne tworzenie backup'ów co pewien określony czas.

|   |   |
|---|---|
| ![](/img/2021/09/011.oandbackupx-syncthing-backup-android-phone-oabx-sched.jpg#small) | ![](/img/2021/09/012.oandbackupx-syncthing-backup-android-phone-oabx-sched.jpg#small) |

Jak widać na powyższych fotkach, mamy możliwość określenia co dokładnie ma podlegać pod backup oraz
też kiedy ten proces tworzenia kopi zapasowej ma być przeprowadzany. U mnie co 7 dni w okolicy
północy, bo wtedy raczej z fona nie będę jakoś bardziej aktywnie korzystał. Rano jak włączę
komputer, to dane z fona automatycznie zostaną przestałe na kompa i tak to sobie będzie działać.

Jeśli nie chcemy robić kopi zapasowej wszystkich aplikacji, to naturalnie możemy określić tylko te
appki, które nam odpowiadają. Podobnie sprawa wygląda w sytuacji, gdzie chcemy zrobić backup'u
większości aplikacji za wyjątkiem paru zbędnych appek. Takie aplikacje możemy dodać na czarną
listę i wtedy OAndBackupX zrobi nam kopię zapasową bez tych appek, które na nią wpisaliśmy.

## Tworzenie kopi zapasowej via OAndBackupX

Bez znaczenia czy manualnie przeprowadzimy proces tworzenia kopi zapasowej, czy też przy pomocy
harmonogramu, pewne pliki zostaną skopiowane do katalogu, który określiliśmy jako lokalizację
backup'u w ustawieniach OAndBackupX.

|   |   |   |
|---|---|---|
| ![](/img/2021/09/015.oandbackupx-syncthing-backup-android-phone-files.jpg#small) | ![](/img/2021/09/016.oandbackupx-syncthing-backup-android-phone-files.jpg#small) | ![](/img/2021/09/017.oandbackupx-syncthing-backup-android-phone-files.jpg#small) |

Jak widać, mamy tutaj plik `base.apk` , który umożliwia instalację danej aplikacji w Androidzie. To
jest ten sam plik, który jest pobierany ze sklepu Google Play. Mamy też pliki `data.tar.gz` oraz
`external_files.tar.gz` , które zawierają już dane/ustawienia aplikacji. W moim przypadku te dwa
ostatnie pliki mają sufiksy `.enc` , który informuje nas, że te dwa pliki są zaszyfrowane.

## Parowanie komputera ze smartfonem w Syncthing

Jeśli nie sparowaliśmy wcześniej komputera ze smartfonem, to musimy to zrobić teraz. Najprościej
jest wykorzystać do tego celu kod QR i zeskanować telefonem kod QR komputera. Kod QR w Syncthing na
linux znajduje się pod Actions (w prawym górnym rogu panelu webowego) > Show ID. Po zeskanowaniu
kodu QR, w aplikacji Syncthing na smartfonie powinno zostać uzupełnione pole od Device ID i po to
nam ten kod QR był, tj. by nie trzeba było tego ID ręcznie wpisywać.

Nadajemy nazwę urządzeniu oraz określamy jego adres IP, lub pozostawiamy domyślnie `dynamic` , w
przypadku gdy korzystamy z mechanizmu rozgłoszeniowego.

|   |   |   |
|---|---|---|
| ![](/img/2021/09/020.oandbackupx-syncthing-backup-android-phone-sync-device.jpg#small) | ![](/img/2021/09/021.oandbackupx-syncthing-backup-android-phone-sync-device.jpg#small) | ![](/img/2021/09/022.oandbackupx-syncthing-backup-android-phone-sync-device.jpg#small) |

Na komputerze, w panelu webowym Syncthing powinien pojawić się monit, że jakieś urządzenie wyraziło
chęć parowania z naszą maszyną. Sprawdzamy ID i dodajemy to urządzenie do powiązanych.

![oandbackupx-syncthing-backup-android-phone-sync-device](/img/2021/09/023.oandbackupx-syncthing-backup-android-phone-sync-device.png#huge)

![oandbackupx-syncthing-backup-android-phone-sync-device](/img/2021/09/024.oandbackupx-syncthing-backup-android-phone-sync-device.png#huge)

![oandbackupx-syncthing-backup-android-phone-sync-device](/img/2021/09/025.oandbackupx-syncthing-backup-android-phone-sync-device.png#medium)

## Synchronizacja kopi zapasowej OAndBackupX via Syncthing

Mając już pliki kopi zapasowej OAndBackupX w określonym katalogu na flash'u telefonu, pozostało nam
wskazanie w Syncthing tego właśnie katalogu do synchronizacji:

|   |   |   |
|---|---|---|
| ![](/img/2021/09/026.oandbackupx-syncthing-backup-android-phone-sync-folder.jpg#small) | ![](/img/2021/09/027.oandbackupx-syncthing-backup-android-phone-sync-folder.jpg#small) | ![](/img/2021/09/028.oandbackupx-syncthing-backup-android-phone-sync-folder.jpg#small) |

Upewnijmy się, że udostępniamy ten katalog wcześniej sparowanej maszynie, tj. tej, której wcześniej
kod QR skanowaliśmy, w tym przypadku `Morfikownia` .

Ponownie zaglądamy do panelu webowego na komputerze. Powinien tam widnieć monit dotyczący prośby o
zaakceptowanie synchronizacji nowego katalogu:

![oandbackupx-syncthing-backup-android-phone-sync-folder](/img/2021/09/029.oandbackupx-syncthing-backup-android-phone-sync-folder.png#huge)

Potwierdzamy i określamy położenie synchronizowanego katalogu w obrębie systemu plików naszego
linux'a.

![oandbackupx-syncthing-backup-android-phone-sync-folder](/img/2021/09/030.oandbackupx-syncthing-backup-android-phone-sync-folder.png#huge)

Po dodaniu synchronizowanego katalogu, po chwili powinien rozpocząć się proces synchronizacji. Jak
widać na poniższej fotce, do synchronizacji jest około 6,5 GiB. Byłoby tych danych więcej ale pliki
backup'u są skompresowane.

![oandbackupx-syncthing-backup-android-phone-sync-data](/img/2021/09/031.oandbackupx-syncthing-backup-android-phone-sync-data.png#medium)

Proces synchronizacji możemy także podglądać ze smartfona.

![oandbackupx-syncthing-backup-android-phone-sync-data](/img/2021/09/032.oandbackupx-syncthing-backup-android-phone-sync-data.jpg#small)

Po dłuższej chwili proces synchronizacji dobiegnie końca. Gdy w przyszłości OAndBackupX sporządzi
nową kopię zapasową, to zostanie ona w dokładnie ten sam sposób automatycznie zsynchronizowana i
nawet nie będziemy musieli przeprowadzać żadnych dodatkowych czynności. Zatem cały ten proces
synchronizacji kopi zapasowych będzie dla nas całkowicie transparentny.

## Podsumowanie

OAndBackupX w połączeniu z Syncthing oferują pierwszorzędny mechanizm tworzenia kopi zapasowej
danych zgromadzonych w smartfonie z Androidem, który jest w pełni kompatybilny z komputerami
działającymi pod kontrolą systemu operacyjnego linux. Cały proces przeprowadzania backup'u jest
realizowany z poszanowaniem naszej prywatności i do tego może być przeprowadzany lokalnie bez
potrzeby korzystania z usług zewnętrznych podmiotów oraz ich infrastruktury sieciowej.

Podczas synchronizacji, połączenie między smartfonem a komputerem jest w pełni szyfrowane E2E
przez Syncthing, zatem nie musimy się martwić o ewentualny podsłuch i wyciek poufnych informacji,
bez względu na to czy synchronizację przeprowadzamy lokalnie w obrębie naszej domowej sieci WiFi,
czy też globalnie przez internet do jakiegoś zdalnego VPS'a. Katalogi ze smartfona są udostępniane
tylko określonym maszynom, także niepowołane osoby nie będą mieć do nich wglądu.

Posiadanie kopi zapasowej ustawień aplikacji jest wielce użyteczne, zwłaszcza w przypadku, gdy z
jakiegoś powodu trzeba będzie system telefonu całkowicie zaorać. Taki backup przyda się nam również
w sytuacji, gdy będziemy przesiadać się na nowsze urządzenie, bo odejdzie nam potrzeba
konfigurowania wszystkich aplikacji od początku, co zwykle jest mniej lub bardziej czasochłonne.

OAndBackupX ma też jedną wadę. Mianowicie, nie ma póki co prostego sposobu, by do zaszyfrowanych
danych tworzonej przez niego kopi można było uzyskać łatwy dostęp z poziomu dowolnej dystrybucji
linux'a, oczywiście po uprzednim podaniu hasła. Być może w niedługim czasie stosowne rozwiązanie
uda się wypracować, choć też zwykle nie ma potrzeby, by na komputerze dostęp do tych danych był
zapewniony.


[1]: https://github.com/machiav3lli/oandbackupx
[2]: /post/czy-smartfon-z-androidem-bez-google-apps-services-ma-sens/
[3]: https://github.com/syncthing/syncthing
[4]: https://f-droid.org/en/packages/com.nutomic.syncthingandroid/
[5]: https://f-droid.org/en/packages/com.machiav3lli.backup/
[6]: https://packages.debian.org/pl/sid/syncthing
[7]: https://docs.syncthing.net/users/firewall.html#firewall-setup
[8]: https://docs.syncthing.net/users/config.html#listen-addresses
[9]: https://github.com/seedvault-app/seedvault
[10]: https://github.com/ukanth/afwall
[11]: https://github.com/machiav3lli/oandbackupx/blob/main/FAQ.md#what-are-special-backups
[12]: https://man7.org/linux/man-pages/man7/signal.7.html
[13]: https://github.com/openssl/openssl/issues/12220
[14]: https://github.com/machiav3lli/oandbackupx/blob/main/app/src/main/java/com/machiav3lli/backup/utils/CryptoUtils.kt#L61-L66
[15]: https://github.com/machiav3lli/oandbackupx/issues/257#issuecomment-925958659
