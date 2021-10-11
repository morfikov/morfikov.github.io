---
author: Morfik
categories:
- Linux
date: "2016-02-12T02:39:29Z"
date_gmt: 2016-02-12 01:39:29 +0100
published: true
status: publish
tags:
- debian
- sieć
- ftp
GHissueID: 510
title: Konfiguracja vsftpd w Debianie
---

Serwery FTP umożliwiają przesyłanie plików przez sieć za pomocą protokołu TCP. Raczej wszyscy
mieliśmy z nimi już do czynienia. Może niekoniecznie zarządzaliśmy takimi serwerami ale na pewno
zdarzyło nam się pobierać pliki za ich pomocą. W tym wpisie jednak postaramy się skonfigurować taki
serwer FTP w oparciu o oprogramowanie [vsftpd](https://security.appspot.com/vsftpd.html).

<!--more-->
## Instalacja vsftpd

By postawić sobie FTP'a, w Debianie musimy doinstalować pakiet `vsftpd` . Standardowo w paczce jest
dostępny plik konfiguracyjny `/etc/vsftpd.conf` , który będziemy musieli sobie nieco dostosować.
Mamy także plik `/etc/ftpusers` , w którym są zawarte nazwy użytkowników, którzy [nie będą w stanie
się logować na serwer FTP](http://manpages.ubuntu.com/manpages/wily/en/man5/ftpusers.5.html). Z
pakietem `vsftpd` są także dostarczane przykładowe pliki konfiguracyjne. Są one ulokowane w katalogu
`/usr/share/doc/vsftpd/examples/` . My jednak stworzymy sobie całą konfigurację od podstaw i
postaramy sobie wyjaśnić większość definiowanych opcji.

Przede wszystkim musimy utworzyć sobie odpowiednią strukturę katalogów. Logujemy się zatem na
użytkownika root i wydajemy w terminalu te poniższe polecenia:

    # mkdir /var/run/vsftpd/empty/
    # mkdir /etc/vsftpd/users/
    # touch /etc/vsftpd/userlist
    # touch /etc/vsftpd/chroot_list
    # touch /etc/vsftpd/banned_emails
    # touch /etc/vsftpd/email_passwords
    # touch /etc/vsftpd/banner

Katalogu `/var/run/vsftpd/empty/` raczej nie będziemy musieli tworzyć ręcznie, bo systemd stworzy go
automatycznie przy starcie systemu za pomocą [mechanizmu
systemd-tmpfiles](https://www.freedesktop.org/software/systemd/man/tmpfiles.d.html). W katalogu
`users/` będzie trzymana konfiguracja specyficzna dla konkretnych użytkowników FTP. Można przy jej
pomocy nadpisać konfigurację główną. W pliku `userlist` będą definiowani użytkownicy, który będą
mogli (ew. nie będą mogli) uzyskać dostęp do FTP'a. Dalej mamy plik `chroot_list` , który będzie
zawierał listę chroot'owanych użytkowników lokalnych. W pliku `email_passwords` będą trzymane hasła
dla użytkowników anonimowych. Z kolei w pliku `banned_emails` będą hasła, których użytkownik
anonimowy nie będzie w stanie używać do zalogowania. Ostatnim plikiem jest `banner` , gdzie może być
przechowywana informacja zwracana użytkownikom tuż po zalogowaniu się na serwer.

Tworzymy także strukturę katalogów pod samego FTP'a:

    # mkdir /media/ftp/
    # mkdir /media/ftp/anon/
    # chown ftp:ftp /media/ftp/anon/
    # mkdir /media/ftp/ftp/
    # chown morfik:ftp /media/ftp/ftp/
    # chmod -w /media/ftp/

Katalog `/media/ftp/anon/` będzie przeznaczony dla klientów anonimowych. Natomiast w
`/media/ftp/ftp/` będzie faktyczny FTP. Ściągamy także prawa do zapisu głównego katalogu FTP. Musimy
to zrobić na potrzeby chroot lokalnych użytkowników.

Oczywiście nie potrzebujemy wszystkich tych plików, a to, które z nich wykorzystamy będzie zależeć
głównie od tego jakie parametry umieścimy w pliku `/etc/vsftpd.conf` . W niczym jednak te puste
pliki i katalogi nie przeszkadzają. Dlatego też można je sobie stworzyć i wykorzystać w przyszłości
jeśli zajdzie taka potrzeba.

## Konfiguracja serwera vsftpd w pliku vsftpd.conf

Wszystkie poniższe parametry musimy umieścić w pliku `/etc/vsftpd.conf` . Zawierać może on dość
sporo opcji i w przypadku tych, których nie będziemy używać, zostaną ustawione wartości domyślne.

### Demon vsftpd

Demon `vsftpd` może nasłuchiwać zarówno w protokole IPv4 jak i IPv6. Może także być uruchamiany w
trybie "standalone" lub za pomocą inetd. Dodajmy zatem tę poniższą zwrotkę do konfiguracji:

    listen=YES
    listen_address=127.0.0.1
    listen_ipv6=NO
    #listen_address6=
    listen_port=21
    background=NO
    run_as_launching_user=NO
    nopriv_user=ftp

Trzeba wiedzieć, że parametry `listen` oraz `listen_ipv6` wzajemnie się wykluczają i nie możemy obu
z nich ustawić na `YES` . Jeśli chcemy, by serwer FTP nasłuchiwał w obu protokołach, to wystarczy,
że ustawimy tę drugą opcję. Nie możemy jednak podawać żadnego adresu IPv6 w `listen_address6` .
Jeśli jednak zamierzamy to zrobić, to będziemy musieli rozbić tę konfigurację na dwa pliki i
uruchomić dwa osobne procesy `vsftpd` . My jednak ograniczymy się do protokołu IPv4 i odpalimy FTP
na adresie określonym przez `listen_address` .

Domyślny port to 21 ale możemy go zmienić dostosowując odpowiednio parametr `listen_port` . Opcja
`background` odpowiada za odpalenie demona w tle. Nie ustawiamy jej ze względu na systemd. Dalej
mamy jeszcze opcję `run_as_launching_user` , która określa czy demon `vsftpd` ma być uruchomiony
jako ten użytkownik, który wywołuje proces. Nie jest to dobrym pomysłem i dużo lepszym rozwiązaniem
jest wskazanie użytkownika w parametrze `nopriv_user` w celu zrzucenia uprawnień.

### Aktywny i pasywny tryb FTP

Serwer FTP może operować w trybie aktywnym lub pasywnym. Chodzi o to, w jaki sposób zostanie
nawiązane połączenie służące do przesyłu danych. W obu przypadkach, klient chcąc nawiązać
połączenie z serwerem FTP wysyła mu żądanie na port 21. To połączenie zwykle zostaje otwarte bez
większego problemu. Niemniej jednak, tutaj są przesyłane jedynie polecenia kontrolne i potrzeba
zestawić drugie połączenie, by przesył danych plików był możliwy.

W trybie aktywnym, klient zaczyna nasłuchiwać na losowym porcie i oczekuje połączenia od serwera FTP
zwykle z portu 20. W tym celu klient wysyła do serwera polecenie `PORT` zawierające informacje na
temat tego losowego portu. W trybie pasywnym zaś, klient przesyła polecenie `PASV` , na które serwer
odpowiada swoim adresem i portem. W oparciu o te informacje, klient tworzy drugie połączenie, którym
dane mogą zostać przesłane.

Widzimy zatem, że w przypadku trybu aktywnego mogą pojawić się problemy z połączeniem, bo większość
stacji roboczych stoi zwykle za NAT'em czy też innym firewall'em i nie może zaakceptować połączenia,
które zainicjował serwer FTP. Dlatego też możemy ten tryb zwyczajnie wyłączyć i akceptować jedynie
połączenia w trybie pasywnym. Dopiszmy zatem ten poniższy blok parametrów do pliku konfiguracyjnego
`vsftpd` :

    port_enable=NO
    connect_from_port_20=NO
    ftp_data_port=20
    port_promiscuous=NO
    pasv_enable=YES
    pasv_min_port=50000
    pasv_max_port=50999
    pasv_promiscuous=NO

Za włączenie trybu aktywnego odpowiada opcja `port_enable` . Jeśli zostanie ustawiona na `NO` , to
serwer FTP nie będzie akceptował polecenia `PORT` podczas próby nawiązywania połączenia. Jeśli
jednak chcielibyśmy włączyć obsługę trybu aktywnego to musimy określić w `connect_from_port_20` czy
te połączenia będą inicjowane z portu 20 na serwerze. Jeśli chcemy określić inny port, to możemy to
zrobić w `ftp_data_port` . Trzeba przy tym pamiętać, że część klientów FTP wymaga portu 20 i
przestawienie go na inny może powodować problemy z połączeniem. Opcję `port_promiscuous` ustawiamy
na `NO` , co zapewni nas, że połączenie nawiązane przez serwer trafi jedynie do klienta, który
wyraził chęć transferu danych.

Pozostałe opcje dotyczą już tylko trybu pasywnego. Który możemy włączyć lub wyłączyć przy pomocy
`pasv_enable` . W parametrach `pasv_min_port` i `pasv_max_port` określamy zakres portów, które
serwer będzie wykorzystywał, gdy będą przychodzić żądania transferu danych. Niekoniecznie musimy je
tutaj podawać, bo te dwa parametry mają głównie na celu ułatwienie konfiguracji zapory sieciowej.
Domyślnie serwer będzie korzystał z wolnych portów. Opcja `pasv_promiscuous` odpowiada za mechanizm
bezpieczeństwa zapewniający, że połączenie rozpoczynające transfer danych pochodzi z tego samego
adresu IP, co połączenie kontrolne na port 21.

### Ograniczenie ilości połączeń

Każdy serwer jest w stanie obsłużyć skończoną ilość jednoczesnych połączeń. W przypadku, gdy nie
dysponujemy za mocnym sprzętem, to możemy zmniejszyć nieco parametr `max_clients` . Możemy też
zmniejszyć limit połączeń nawiązywanych z jednego adresu IP przy pomocy `max_per_ip` . Oczywiście
nic nie stoi na przeszkodzie, by zaimplementować obydwa limity w tym samym czasie. Dodajmy zatem te
dwa poniższe wpisy do pliku konfiguracyjnego `vsftpd` :

    max_clients=100
    max_per_ip=5

### Nieudane próby logowania

Ograniczenie ilości prób logowania może spowolnić zautomatyzowane ataki siłowe na hasło. Przy pomocy
parametru `max_login_fails` możemy ustawić maksymalną ilość prób, po przekroczeniu której połączenie
zostanie unicestwione przez serwer FTP. Mamy także do dyspozycji opcje `delay_failed_login` oraz
`delay_successful_login` , które są w stanie opóźnić ponownie próby logowania (czas w sekundach):

    max_login_fails=3
    delay_failed_login=1
    delay_successful_login=0

### Ograniczenie transferu

By nasz serwer FTP nie wysiadł pod naporem rządnych plików klientów, możemy wprowadzić ograniczenie
transferu. Służą do tego dwa parametry: `anon_max_rate` oraz `local_max_rate` . Definiujemy w nim
zarówno prędkość pobierania jak i wysyłania (w bajtach). Wartość 0 wyłącza ograniczenie:

    anon_max_rate=0
    local_max_rate=0

### Pobieranie i wysyłanie plików

Za globalne ustawienia pobierania oraz wysyła plików z i do serwera FTP odpowiadają te poniższe
cztery parametry:

    write_enable=NO
    download_enable=YES
    ascii_upload_enable=NO
    ascii_download_enable=NO

Nie ma raczej potrzeby rozpisywać się nad opcjami `write_enable` oraz `download_enable` .
Problematyczne za to mogą okazać się te dwa ostatnie parametry. Kontrolują one włączenie trybu
ASCII, który może czasem prowadzić do ataku DoS (polecenie `SIZE /duzy/plik` ). Najlepiej jest ten
tryb po prostu wyłączyć.

### Komunikaty zwracane przez serwer

Po zalogowaniu się użytkownika, serwer FTP może zwrócić mu określoną wiadomość, którą ustawimy w
konfiguracji. Jeśli to ma być w miarę krótka informacja, to możemy ją określić bezpośrednio w
parametrze `ftpd_banner` . W przypadku, gdy chcemy się nieco wysilić i stworzyć bardziej rozbudowaną
wiadomość, to możemy wskazać plik tekstowy przy pomocy opcji `banner_file` :

    ftpd_banner=Welcome to Morfikownia.
    #banner_file=

Dodatkowo, w każdym katalogu możemy umieścić plik `.message` , który będzie czytany po włączeniu
opcji `dirmessage_enable` . Jeśli taki plik zostanie odnaleziony w katalogu, do którego przeszedł
klient, to zostanie mu wyświetlona jego zawartość. Nazwę samego pliku również możemy zmienić za
pomocą parametru `message_file` :

    dirmessage_enable=YES
    message_file=.message

### Strefa czasowa

Czas modyfikacji plików zwracany przez `vsftpd` jest standardowo ustawiony w oparciu o UTC. W Polsce
mamy czas zimowy UTC+1 i czas letni UTC+2, co będzie powodować podawanie nieprawidłowego czasu
zwracanego w listingach plików i katalogów. Przy pomocy opcji `use_localtime` możemy wymusić, by ten
czas był podawany w oparciu o naszą lokalną strefę czasową:

    use_localtime=YES

By to powyższe ustawienie było w ogóle możliwe, trzeba wyeksportować zmienną `TZ` . Nie możemy
jednak tego robić globalnie za pomocą pliku `/etc/environment` i musimy to zrobić albo w skrypcie
startowym, albo w usłudze systemd. W tym przypadku demon `vsftpd` jest odpalany za pomocą systemd,
zatem edytujemy plik usługi przy pomocy `systemctl edit --full vsftpd` i dopisujemy w nim tę
poniższą linijkę:

    [Service]
    ...
    Environment="TZ=Europe/Warsaw"

Pamiętajmy jednak, że klienty FTP są w stanie ustawić odpowiedni czas automatycznie i raczej nie ma
potrzeby korzystania z parametru `use_localtime` . Jeśli jednak zdecydowaliśmy się na dopisanie go
do pliku konfiguracyjnego, to w pewnych przypadkach listing plików i katalogów może zwracać czas z
przyszłości, w zależności od różnicy czasu w stosunku do strefy UTC.

### Timeout'y

Użytkownik po zalogowaniu się na FTP wykonuje zwykle jakieś działania. Niemniej jednak, po pobraniu
czy wysłaniu plików, sesja pozostaje przez dłuższy czas otwarta w trybie IDLE. Czas bezczynności
możemy dostosować za pomocą parametru `idle_session_timeout` (czas w sekundach):

    idle_session_timeout=300

Istnieje także opcja, która jest w stanie rozłączyć klienta podczas przesyłania danych. Znajduje ona
zastosowanie jedynie w przypadku, gdy transfer z jakiegoś powodu został zablokowany i nie ma
widocznego postępu przy przesyłaniu pliku. Jeśli taka sytuacja nastąpi, to sesja użytkownika
zostanie zakończona po czasie określonym w parametrze `data_connection_timeout` :

    data_connection_timeout=60

Mamy także możliwość określenia timeout'u w przypadku ustanawiania połączenia, w którym mają być
wymieniane dane. Czas oczekiwania na połączenie w trybie pasywnym definiujemy w parametrze
`accept_timeout` . Natomiast jeśli chodzi o tryb aktywny, to odpowiada za niego opcja
`connect_timeout` :

    accept_timeout=60
    connect_timeout=60

### TCP wrapper

<span>TCP wrapper</span> jest w stanie regulować dostęp do usług sieciowych za pomocą plików
`/etc/hosts.allow` oraz `/etc/hosts.deny` . Jeśli w `vsftpd` zostało wkompilowane wsparcie dla tego
mechanizmu, to będzie można z niego skorzystać przez ustawienie opcji `tcp_wrappers` . Jak możemy
wyczytać [na stronie Achrlinux'a](https://www.archlinux.org/news/dropping-tcp_wrappers-support/),
ten mechanizm jest bardzo stary i lepiej nie zawracać sobie nim głowy. Dlatego też wyłączmy jego
ewentualną obsługę:

    tcp_wrappers=NO

### Wsparcie dla UTF-8

Opcja `utf8_filesystem` nie jest nigdzie udokumentowana ale za jej pomocą możemy włączyć w `vsftpd`
wparcie dla kodowania UTF-8. Chodzi generalnie o fakt błędnego interpretowania nazw plików w różnych
klientach FTP, m.in. w Firefox'e. Bez określenia tej opcji, przeglądarka domyślnie w miejscu
polskich znaków będzie pokazywać krzaki. Wszystko za sprawą wykrycia nieprawidłowego kodowania
ISO-8859-1. Po ustawieniu tej opcji, polecenie `FEAT` będzie zwracać m.in. `UTF8` sygnalizując
klientom, by korzystały z tego kodowania znaków:

    utf8_filesystem=YES

### Odmawianie dostępu oraz ukrywanie plików i ich atrybutów

Na serwerze FTP możemy posiadać pliki i katalogi, które nie zawsze chcemy udostępniać wszystkim
logującym się użytkownikom. Opcje `hide_file` oraz `deny_file` włączają ciekawy mechanizm, który
sprawia, że przeciętny użytkownik nie będzie w stanie zobaczyć i pobrać pewnych rzeczy. Gdy dany
plik jest umieszczony w pierwszej dyrektywie, to klienci nie zobaczą określonych plików i katalogów
na listingu. Będą jednak w stanie uzyskać dostęp do tych zasobów ręcznie wpisując odpowiednie
ścieżki. Jeśli taki schemat nie jest pożądany, możemy te wzory dopasowań umieścić także w drugiej
dyrektywie. Pamiętajmy też, że wpisanie pliku jedynie do `deny_file` wprawdzie uniemożliwi jego
pobranie czy usunięcie ale nie ukryje go i będzie on widoczny cały czas na listingu. Poniżej
przykład zastosowania:

    hide_file={*.mp3,.hidden,hide*,h?}
    deny_file={*.mp3}

Jeśli listing plików nie jest pożądany, to możemy go całkowicie wyłączyć przy pomocy opcji
`dirlist_enable` . To ustawienie jest przydatne jedynie w przypadku, gdy chcemy aby ktoś wrzucał nam
pliki na FTP'a bez udzielania mu informacji co do tego jakie pliki już na tym serwerze się znajdują.
W każdym innym przypadku lepiej jest włączyć tę opcję:

    dirlist_enable=YES

Listing plików i katalogów zawiera numery identyfikacyjne użytkowników (UID) i grup (GID). Możemy
zamaskować tę informację korzystając z parametru `hide_ids` . W takim przypadku, numery ID będą
wskazywać na użytkownika i grupę `ftp` :

    hide_ids=NO

Serwer `vsftpd` standardowo nie pokazuje plików ukrytych, tj. tych, których nazwy zaczynają się od
`.` . Jest to standardowy mechanizm spotykany w linux'ach. Jeśli jednak chcemy, by te pliki były
pokazywane, to możemy włączyć opcję `force_dot_files` :

    force_dot_files=YES

### Dozwolone i zabronione polecenia

W `vsftpd` mamy dwa parametry, tj. `cmds_allowed` oraz `cmds_denied` , które są w stanie odpowiednio
wyprofilować polecenia, które zalogowany użytkownik będzie mógł przesyłać do serwera FTP. [Wszystkie
polecenia używane w protokole FTP można znaleźć na
wiki](https://en.wikipedia.org/wiki/List_of_FTP_commands).

    #cmds_allowed=PASV,RETR,QUIT
    #cmds_denied=

### Lista zablokowanych klientów

Mechanizm listy użytkowników, który można włączyć w `vsftpd` za pomocą parametru `userlist_enable`
działa na dobrą sprawę na dwa sposoby. Pierwszy z nich ma na celu zablokować próby logowania
wszystkich użytkowników, który zostaną zdefiniowani w pliku określonym przez opcję `userlist_file` .
To jest standardowe zachowanie i można je wykorzystać do blokowania próby przesłania haseł w postaci
czystego tekstu. Z kolei drugi tryb możemy włączyć przy pomocy parametru `userlist_deny` poprzez
przestawienie jego wartości na `NO` . W takim przypadku, tylko te loginy, które znajdują się na
liście będą mieć możliwość zalogowania się na FTP:

    userlist_enable=YES
    userlist_file=/etc/vsftpd/userlist
    userlist_deny=YES

### Nadpisywanie konfiguracji użytkowników

Konfiguracja globalna, którą w tej chwili cały czas dostosowujemy może zostać nadpisana za pomocą
osobnych plików, które trzeba umieścić w katalogu określonym przez parametr `user_config_dir` :

    user_config_dir=/etc/vsftpd/users/

W tym folderze umieszczamy zwykle pliki tekstowe o nazwach kont użytkowników, którzy mogą logować
się na FTP'a. Gdy taki użytkownik próbuje się zalogować, system może mu przypisać niestandardowe
ustawienia. W taki sposób, można, np. wyłączyć globalnie tryb zapisu dla wszystkich użytkowników, po
czym w tych plikach przepisać ten sam parametr ale tylko w przypadku tych klientów, którym chcemy
zezwolić na zapis danych. Trzeba jednak mieć na uwadze, że nie wszystkie parametry można dostosować
osobno dla każdego klienta.

### Prawa dostępu do plików (umask)

Pliki wysyłane przez użytkowników na serwer FTP muszą posiadać jakieś domyślne uprawnienia, które są
aplikowane podczas ich tworzenia na maszynie zdalnej. W `vsftpd` odpowiadają za to trzy parametry:
`file_open_mode` , `anon_umask` oraz `local_umask` . Wartość `file_open_mode` wskazuje zwykle `0666`
lub `0777` . To, którą z nich wybierzemy zależy głównie od tego czy chcemy, aby przesyłane na FTP
pliki posiadały domyślnie atrybut wykonywalności. Lepiej jest tutaj ustawić zatem `0666` . Pozostałe
dwa parametry odpowiadają za umask. Bezpieczne prawa do plików wyglądają zatem tak:

    file_open_mode=0666
    anon_umask=0022
    local_umask=0022

Oczywiście ten umask nie musi wskazywać na taką samą wartość i może być inny w przypadku
użytkowników lokalnych jak i anonimowych. Niemniej jednak, w tym przypadku wszystkie pliki
tworzone na FTP'ie będą posiadać prawa 644 (0666-0022). [Więcej informacji na temat umask można
znaleźć tutaj](http://www.arturpyszczuk.pl/commands-umask.html).

### Konta z błędnym shell'em

Konta FTP nie muszą posiadać poprawnego shell'a. Możemy tę sytuację wykorzystać i wszystkim takim
użytkownikom zablokować możliwość logowania się w systemie, co znacznie poprawi bezpieczeństwo
systemu. Jest tylko jeden problem. Chodzi o to, że `vsftpd` weryfikuje poprawność shell'a i gdy
konto ma ustawiony, np. `/bin/false` , to zostanie zgłoszony błąd. Lista poprawnych shell'ów jest
dostępna w pliku `/etc/shells` . Można by oczywiście dopisać tam kilka pozycji ale jest prostszy
sposób. Możemy posłużyć się opcją `check_shell` , która wyłączy sprawdzanie shell'a i tym samym
umożliwi logowanie się użytkownikom mającym `/bin/false` :

    check_shell=NO

Ten mechanizm znajduje zastosowanie jedynie w przypadku, gdy `vsftpd` nie został skompilowany z
obsługą PAM.

### Użytkownicy anonimowi

Na FTP mogą logować się przeróżni użytkownicy. W sporej części przypadków będziemy mieli do
czynienia z użytkownikami anonimowymi. Są to tacy klienci, którzy identyfikują się loginem `ftp` lub
`anonymous` . W zależności od przeznaczenia FTP'a możemy skonfigurować go tak, by wpuszczał takich
użytkowników bez podania hasła lub też możemy założyć im hasło. Ustawiamy zatem odpowiednio
parametry `anonymous_enable` oraz `no_anon_password` :

    anonymous_enable=YES
    no_anon_password=YES

W przypadku, gdy `no_anon_password` jest ustawiony na `NO` , anonimowi klienci będą musieli podać
jakieś hasło. Nie jest dobrym wyjściem nadawanie każdemu takiemu użytkownikowi tego samego hasła. Po
pierwsze, system będzie ich w stanie odróżnić jeśli będą się oni posługiwać innymi hasłami. A po
drugie, jeśli jedno z haseł zostanie skompromitowane, to zawsze można je zablokować. Ten mechanizm w
`vsftpd` konfigurujemy za pomocą parametrów `deny_email_enable` i `secure_email_list_enable` :

    deny_email_enable=YES
    banned_email_file=/etc/vsftpd/banned_emails

    secure_email_list_enable=YES
    email_password_file=/etc/vsftpd/email_passwords

Hasła, z których użytkownicy nie mogą skorzystać wpisujemy do pliku określonego przez opcję
`banned_email_file` . Podobnie postępujemy w przypadku haseł dozwolonych, z tym, że wpisujemy je do
pliku, który widnieje w parametrze `email_password_file` .

Jeśli chcemy, by użytkownik anonimowy był w stanie wysyłać pliki na FTP, to musimy jeszcze dopisać
do konfiguracji parametr `anon_upload_enable` :

    anon_upload_enable=YES

Definiujemy też podstawowe uprawnienia do plików dla klienta anonimowego. Po pierwsze musimy
oddelegować konkretne konto w systemie, na którym będą operować ci klienci. Zwykle do tego celu jest
wyznaczany `nobody` . Niemniej jednak, jest on używany do szeregu innych rzeczy i raczej nie
powinniśmy z niego korzystać. Zamiast tego powinniśmy w systemie utworzyć dedykowanego użytkownika,
np. `ftp` i określić go w parametrze `ftp_username` :

    ftp_username=ftp

Użytkownik anonimowy po zalogowaniu się na FTP'a musi znaleźć się w określonym katalogu roboczym.
Ten katalog jest określany przez parametr `anon_root` . Pamiętajmy, że użytkownik `ftp` musi być w
stanie zapisywać określony katalog. W przeciwnym razie klienci anonimowi nie będą mógi wysyłać
plików. Nie jest zatem dobrym pomysłem zmiana uprawnień głównego katalogu FTP'a. Lepiej stworzyć
wewnątrz niego podkatalog i to jemu nadać prawa do zapisu przez anonimowych klientów:

    anon_root=/media/ftp/

Jeśli użytkownicy anonimowi mają być w stanie tworzyć nowe katalogi na FTP'ie, musimy to wyraźnie
określić za sprawą opcji `anon_mkdir_write_enable` . Trzeba też pamiętać, że użytkownik taki, musi
mieć prawa do zapisu danego katalogu oraz, że opcja `write_enable` musi być aktywna:

    anon_mkdir_write_enable=YES

Przy pomocy parametru `anon_other_write_enable` jesteśmy w stanie rozszerzyć nieco uprawnienia
anonimowych użytkowników. Standardowo nie mogą oni zmieniać nazw czy usuwać plików z serwera. Mogą
oni jedynie zapisywać pliki i ewentualnie tworzyć nowe katalogi:

    anon_other_write_enable=YES

W przypadku, gdybyśmy chcieli uniemożliwić użytkownikom anonimowym pobieranie plików, które mają
uprawnienia, np. 640, to możemy pokusić się o przestawienie parametru `anon_world_readable_only` na
`YES` :

    anon_world_readable_only=NO

Wszystkie pliki, które są przesyłane przez anonimowych użytkowników mogą podlegać pod przepisanie
uprawnień. Ten mechanizm możemy włączyć przy pomocy opcji `chown_uploads` . Natomiast jeśli chodzi o
nowego właściciela plików i prawa to nich, to określamy to w `chown_username` oraz
`chown_upload_mode` , przykładowo:

    chown_uploads=YES
    chown_username=morfik
    chown_upload_mode=0644

### Użytkownicy lokalni

Poza użytkownikami anonimowymi, do FTP'a mogą mieć także dostęp użytkownicy lokalni, tj. ci, którzy
mają fizyczne konto w systemie. By zalogować się na FTP przy pomocy takiego konta, trzeba podać jego
login i hasło. Tych użytkowników możemy włączyć lub wyłączyć za pomocą opcji `local_enable` :

    local_enable=YES

Podobnie jak w przypadku użytkowników anonimowych, musimy określić odpowiedni katalog roboczy, do
którego ci użytkownicy zostaną przeniesieni po zalogowaniu. Standardowo jest to folder domowy.
Możemy go przepisać, tak by każdy użytkownik po zalogowaniu znalazł się w konkretnym katalogu przy
pomocy opcji `local_root` :

    local_root=/media/ftp/

Problem z takim rozwiązaniem jest taki, że użytkownicy lokalni mogą sobie biegać praktycznie po
całym drzewie katalogów, co nie jest zbytnio bezpieczne. Przydałoby się zatem zastosować mechanizm
chroot, tak by nie mogli oni opuścić głównego katalogu FTP'a. Możemy to zrobić przy pomocy parametru
`chroot_local_user` . Musimy jednak mieć gdzie ten chroot wykonać. Ścieżkę do pustego katalogu,
który nie może być zapisywalny przez użytkownika `ftp` podajemy w `secure_chroot_dir` :

    chroot_local_user=YES
    secure_chroot_dir=/run/vsftpd/empty/

Nie zawsze jednak wszyscy użytkownicy muszą podlegać temu mechanizmowi ochrony. Opcje
`chroot_list_enable` oraz `chroot_list_file` aktywują listę, która może zawierać pewne określone
nazwy kont systemowych. W takim przypadku, jeśli użytkownik zaloguje się na konto znajdujące się na
liście, to nie będzie chroot'owany:

    chroot_list_enable=YES
    chroot_list_file=/etc/vsftpd/chroot_list

#### Parametr allow\_writeable\_chroot

`vsftpd` nie zezwala na dokonanie chroot'a w przypadku, gdy główny katalog serwera FTP posiada prawa
zapisu. Jeśli spróbowalibyśmy się podłączyć do takiego serwera, to zobaczymy poniższy komunikat:

    Response:    500 OOPS: vsftpd: refusing to run with writable root inside chroot()

Możemy oczywiście ściągnąć te uprawnienia zapisu, tak jak to zostało zrobione w tym artykule. Na
sieci jednak można się spotkać z parametrem `allow_writeable_chroot` , który, jak nazwa wskazuje,
umożliwia dokonanie chroot w przypadku, gdy główny katalog FTP jest zapisywalny. Nie jestem
przekonany co do tej opcji i nigdzie nie mogę znaleźć na jej temat żadnej dokumentacji. Na dobrą
sprawę to nie wiem czym grozi przestawienie tego parametru na `YES` , dlatego też lepiej jest go
zostawić w spokoju.

    allow_writeable_chroot=NO

### Logi systemowe

Logi to bardzo ważna rzecz, zwłaszcza w fazie testów i dostrajania konfiguracji FTP'a. `vsftpd` może
zostać przełączony w tryb logowania za pomocą opcji `syslog_enable` . Jeśli nie odpowiada nam
domyślny plik logu, możemy go zmienić w parametrze `vsftpd_log_file` . W przypadku, gdy
potrzebujemy bardziej szczegółowych komunikatów dotyczących wymiany informacji w protokole FTP, to
możemy także włączyć opcję `log_ftp_protocol` .

    syslog_enable=YES
    vsftpd_log_file=/var/log/vsftpd.log
    log_ftp_protocol=YES

Cały plik konfiguracyjny `vsftpd.conf` znajduje się [na moim
github'ie](https://github.com/morfikov/files/blob/master/configs/etc/vsftpd.conf).
