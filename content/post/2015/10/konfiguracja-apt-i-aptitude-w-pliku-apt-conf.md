---
author: Morfik
categories:
- Linux
date: "2015-10-18T20:04:30Z"
date_gmt: 2015-10-18 18:04:30 +0200
published: true
status: publish
tags:
- debian
- apt
title: Konfiguracja apt i aptitude w pliku apt.conf
---

Praktycznie każdy z nas korzysta z menadżera pakietów `apt` lub też jego nakładki `aptitude` .
Operowanie na debianie bez tych narzędzi raczej by nam nieco utrudniło życie. Sporo osób ogranicza
się jedynie do podstawowych poleceń, typu `update` , `upgrade` czy `dist-upgrade` , pomijając przy
tym całą konfigurację w/w narzędzi. W tym wpisie zostanie zaprezentowanych szereg opcji, które można
zdefiniować na stałe w pliku konfiguracyjnym `/etc/apt/apt.conf` , tak by nie trzeba było ich ciągle
wpisywać w terminalu ilekroć tylko korzystamy któregoś menadżera pakietów.

<!--more-->
## Plik /etc/apt/apt.conf

Opcji, które mogą zostać określone w pliku `apt.conf` , jest naprawdę sporo. Wszystkie z nich można
znaleźć w [man apt.conf(5)][1] lub też w [dokumentacji aptitude][2]. Tutaj zaś zostaną
przedstawione tylko te opcje, które są używane przeze mnie.

Plik `/etc/apt/apt.conf` domyślnie nie występuje w systemie i trzeba go sobie utworzyć samemu.
Konfiguracja zawarta w tym pliku może dotyczyć nie tylko `apt` i `aptitude` ale ma też wpływ na
pozostałe menadżery pakietów, np. `synaptic` . Można także w nim określić opcje dotyczące menadżera
pakietów `dpkg` . Warto też wiedzieć, że szereg aplikacji umieszcza konfigurację w katalogu
`/etc/apt/apt.conf.d/` , jednak te opcje, które zostaną określone w `apt.conf` zawsze mają
pierwszeństwo.

Jakie opcje zatem możemy dodać do pliku `apt.conf` ?

### Domyślne ścieżki do plików konfiguracyjnych

Na sam początek skonfigurujmy ścieżki odpowiedzialne za położenie takich plików jak `sources.list` ,
który odpowiada za źródła pakietów, czy też katalog, w którym przechowywane są paczki `.deb` pobrane
przy instalacji/aktualizacji pakietów:

    Dir::Etc::SourceList "/etc/apt/sources.list";
    Dir::Etc::SourceParts "/etc/apt/sources.list.d/";
    Dir::Etc::Main "/etc/apt/apt.conf";
    Dir::Etc::Parts "/etc/apt/apt.conf.d/";
    Dir::Etc::Preferences "/etc/apt/preferences";
    Dir::Etc::PreferencesParts "/etc/apt/preferences.d/";
    Dir::Cache "/media/Kabi/backup_packages/apt";
    Dir::Cache::Archives "/media/Kabi/backup_packages/apt/archives/";
    Dir::State::Lists "/media/Kabi/backup_packages/apt/lists/";

Powyższe opcje maja zdefiniowane domyślne wartości i jeśli nie chcemy ich zmieniać, to dobrze jest
je wykomentować. Z kolei te 3 ostatnie pozycje są w stanie przenieść archiwum pakietów do
zewnętrznego katalogu, który może być wykorzystywany przez inny system, lub też może posłużyć do
instalacji nowego systemu, dzięki czemu nie będzie trzeba pobierać pakietów przez sieć.

### Domyślny release

Mamy możliwość także skonfigurowania domyślnego release (wydania) dla naszego systemu. Możemy
ustawić tutaj, np. `testing` czy `sid` , a priorytet dla źródeł pakietów (w pliku `sources.list` ),
które mają ustawiony release określony w tym parametrze, zostanie podbity do 990. Ma to znaczenie
przy pinning'ach, które zostaną opisane w osobnym artykule. Jeśli chcemy ustawić wyższy priorytet
dla release `testing`, dodajemy poniższą linijkę:

    APT::Default-Release "testing";

### Pakiety zalecane i sugerowane

Sporo pakietów w debianie ma określone zależności, bo bez nich taki pakiet zwyczajnie nie działa
najlepiej. Ponadto, cześć pakietów ma też dodatkowe zależności, które można podzielić na dwie grupy:
zalecane oraz sugerowane. Różnica między nimi polega na tym, że jeśli dana aplikacja traci na
funkcjonalności dość znacznie w wyniku braku w systemie jakiegoś pakietu, to taka zależność ma
zostać przypisana do tych zalecanych. Wszystkie inne przypadki trafiają do sugerowanych.

Możemy określić czy menadżer pakietów ma instalować/usuwać tego typu pakiety automatycznie, czy też
nie i odpowiadają za to poniższe opcje:

    APT::Install-Recommends "false";
    APT::Install-Suggests "false";
    APT::AutoRemove::RecommendsImportant "false";
    APT::AutoRemove::SuggestsImportant "false";

### Weryfikacja pakietów przed instalacją

Pakiety w repozytoriach są podpisane, by czasem ktoś ich przez przypadek nie podmienił. Jeśli by
taki proceder miał miejsce, to `apt` czy `aptitude` zwyczajnie odmówi współpracy i nie będzie chciał
zainstalować takiego pakietu, albo przynajmniej wyrzuci stosowne ostrzeżenie. Dobrze jest wyraźnie
nakazać menadżerowi pakietów, by nigdy nie instalował pakietów, których nie można zweryfikować:

    APT::Get::AllowUnauthenticated "false";
    Aptitude::CmdLine::Ignore-Trust-Violations "false";

### Domyślne usuwanie plików konfiguracyjnych

Pakiety, które instalujemy w naszych systemach, zwykle wgrywają też pliki konfiguracyjne do katalogu
`/etc/` . W przypadku gdy usuwamy taki pakiet z systemu, pliki w katalogu `/etc/` nie są usuwane, no
bo przecie zawierają konfigurację, nad którą mogliśmy siedzieć długie miesiące. Jeśli notorycznie
usuwamy konfigurację pakietów przy ich wywalaniu z systemu (parametr `purge` ), to możemy
poinstruować menadżer pakietów, by robił to automatycznie. Większość z nas wolałaby jednak, by tego
typu sytuacja nie miała miejsca, dlatego też możemy dodać poniższą linijkę:

    APT::Get::Purge "false";

### Automatyczne czyszczenie zależności

Może się zdarzyć tak, że po odinstalowaniu jakiegoś pakietu, zależności będące obecne w systemie nie
będą zwyczajnie już dłużej potrzebne. W takim wypadku, `apt` zwróci stosowny komunikat. Nie podejmie
jednak akcji usunięcia tych pakietów i to my będziemy musieli ręcznie te pakiety wyrzucić z systemu.
Możemy nakazać `apt` by tego typu zbędne zależności automatycznie usuwał z naszego systemu:

    APT::Get::AutomaticRemove "true";

### Czyszczenie cache ze zbędnych plików

Przy instalowaniu pakietów, te są pobierane do lokalnego cache. Gdy w późniejszym czasie zajdzie
potrzeba reinstalacji takiego pakietu, nie ma potrzeby pobierać go przez sieć, bo jest on już u nas
na dysku. Gdy usuwamy pakiety, lub też dokonujemy ich aktualizacji, wiele wersji tego samego pakietu
może zalegać w cache. W pewnych sytuacjach, takie zachowanie jest pożądane ale w większości
przypadków, te pakiety zwyczajnie zajmują miejsce na dysku. Im częściej dokonujemy aktualizacji
systemu, tym więcej miejsca idzie na zmarnowanie. Poniższe dwa parametry zatroszczą się o to by
cache był zawsze aktualny:

    APT::Get::List-Cleanup "true";
    Aptitude::Autoclean-After-Update "true";

### Formatowanie informacji zwracanych przez menadżer pakietów

Poniżej jest kilka opcji, które mogą pomóc nieco sformatować wyjście jakie jest zwracane przez, np.
`aptitude search` .

    Aptitude::CmdLine::Disable-Columns "true";
    Aptitude::CmdLine::Versions-Group-By "source-package";
    Aptitude::CmdLine::Package-Display-Format "%c%a%M %p# => %d# [%t]";
    Aptitude::CmdLine::Verbose "0";

### Pasek postępu (Progress bar)

Czasem dokonujemy instalacji czy aktualizacji wielu pakietów. Ta czynność może zając sporo czasu.
Tekstowe narzędzia mają jednak to do siebie, że nie zawsze informują nas o tym jak długo będzie
trwała dana operacja i zwykle też nie mamy informacji na temat jej przebiegu. Istnieją opcje, które
są w stanie wskazać, w którym miejscu (mnij więcej) znajduje się proces instalacji czy aktualizacji
przeprowadzany za pomocą tekstowego menadżera pakietów i jeśli interesuje nas taka informacja,
dodajemy te poniższe linijki:

    Dpkg::Progress "true";
    Dpkg::Progress-Fancy "true";

### Częściowe pobieranie listy pakietów

Za każdym razem gdy wydajemy polecenie `aptitude update` , pobierane są listy pakietów dostępnych w
repozytoriach. Im większe jest takie repozytorium lub też im więcej repozytoriów mamy określonych w
pliku `sources.list` , tym więcej danych trzeba pozyskać przy odświeżaniu tych list. Może to
skutkować pobraniem nawet i 100MiB. Możemy odciążyć nieco sieć (w tym, też i serwer) i nakazać
menadżerowi pakietów, by pobrał jedynie różnice w listach pakietów i tylko te kawałki dociągnął. W
taki sposób, zamiast 100MiB, zostanie pobranych kilkaset KiB. Oczywiście, odświeżanie listy pakietów
będzie trwało nieco dłużej ale czego to się nie robi by odciążyć i tak już przeładowane serwery:

    Acquire::PDiffs "true";

### Limit transferu

Domyślnie, menadżery pakietów pobierają pliki z prędkością taką, jaka jest dostępna aktualnie na
łączu. Ma to na celu przyśpieszenie całego procesu instalacji/aktualizacji. Może to jednak
powodować wzrost opóźnień w przypadku innych połączeń, dlatego też możemy ustawić sobie limit (w
kilobajtach na sekundę):

    Acquire::http::Dl-Limit "512";
    Acquire::https::Dl-Limit "512";
    Acquire::ftp::Dl-Limit "512";

### Określenie timeout'a

W przypadku gdy połączenie nie jest najlepszej jakości albo zwyczajnie serwer zdechł z bliżej
nieznanych nam przyczyn, `apt`/`aptitude` będzie próbował się łączyć z nim przez dość długi czas.
Ten czas możemy sobie dostosować (w sekundach):

    Acquire::http::Timeout "2";
    Acquire::https::Timeout "2";
    Acquire::ftp::Timeout "2";

### Wymuszenie protokołu IPv4

Jeśli nasz ISP nie potrafi obsłużyć protokołu IPv6, w pewnych przypadkach `apt`/`aptitude` może
napotkać problemy i próbować pobierać pakiety przez ten właśnie protokół. Jeśli mamy do dyspozycji
jedynie IPv4, powinniśmy nakazać menadżerowi pakietów, by używał tylko tego protokołu:

    Acquire::ForceIPv4 "true";

### Katalog /usr/ w trybie tylko do odczytu

W bardzo rzadkich przypadkach, ludzie instalują system z wydzieleniem katalogu `/usr/` . Ma to na
celu poprawę bezpieczeństwa, przez umożliwienie zamontowania tej partycji w trybie tylko do odczytu,
co zapobiega wprowadzaniu zmian w jej systemie plików. Problem zaczyna się w momencie gdy chcemy
aktualizować czy instalować oprogramowanie. W takim przypadku trzeba pierw przemontować tę partycję
w tryb do zapisu, po czym dokonać instalacji/aktualizacji i znów przemontować ją w tryb tylko do
odczytu. Jest to mało wygodne ale na szczęście możemy dopisać w pliku `apt.conf` tę poniższą
zwrotkę, która przy wywoływaniu `apt` czy `aptitude` zajmie się przemontowaniem partycji za nas:

    DPkg
    {
    Pre-Invoke { "mount /usr -o remount,rw" };
    Post-Invoke { "mount /usr -o remount,ro" };
    };

## Plik apt.conf

Aktualny plik konfiguracyjny dla `apt` znajduje się zawsze [na moim gicie][3].


[1]: http://manpages.ubuntu.com/manpages/wily/en/man5/apt.conf.5.html
[2]: https://www.debian.org/doc/manuals/aptitude/
[3]: https://github.com/morfikov/files/tree/master/configs/etc/apt
