---
author: Morfik
categories:
- Linux
date:    2020-09-27 12:29:00 +0200
lastmod: 2020-09-27 12:29:00 +0200
published: true
status: publish
tags:
- debian
- ntp
- nts
- sntp
- ntpsec
- bezpieczeństwo
- czas
- systemd
- systemd-timesyncd
title: Uwierzytelnianie odpowiedzi z serwerów czasu NTP przy pomocy NTS
---

Przepisując ostatnio stare artykuły dotyczące [zabezpieczenia zapytań DNS za sprawą wdrożenia na
linux dnsmasq i dnscrypt-proxy][1], natknąłem się [na informację][16], że nie tylko komunikacja DNS
w obecnych czasach w sporej mierze nie jest zaszyfrowana. W zasadzie każda maszyna podłączona do
internetu potrzebuje dysponować w miarę dokładnym czasem. By ten czas był dokładny, wymyślono
mechanizm synchronizacji czasu przez wysyłanie zapytań do serwerów NTP ([Network Time Protocol][2]).
Niemniej jednak, odpowiedzi z tych serwerów czasu nie są w żaden sposób zabezpieczone i praktycznie
każdy na drodze tych pakietów może nam zmienić ustawienia czasu w systemie (MITM). W taki sposób
możemy zostać cofnięci w czasie, co z kolei może oznaczać, że system zaakceptuje certyfikaty SSL/TLS
czy też [klucze/bilety sesji][3] (używane do wznawiania sesji TLS), które już dawno temu wygasły
lub/i zostały w jakiś sposób skompromitowane. Pchnięcie nas do przodu w czasie również oznacza
problemy, bo możemy zaakceptować certyfikat, który jeszcze nie zaczął być ważny. To z kolei otwiera
drogę do odszyfrowania połączeń z serwisami WWW, a przecie nie po to szyfrujemy ruch, by go ktoś bez
większego problemu odszyfrował. Dlatego też powinniśmy zadbać o to, by informacje o aktualnym czasie
otrzymywane z sieci docierały do nas z wiarygodnego źródła i były w jakiś sposób uwierzytelnione.
Na Debianie standardowo do synchronizacji czasu wykorzystywany jest `systemd-timesyncd` ale [nie
wspiera on póki co protokołu NTS][4]. Trzeba będzie zatem się go pozbyć i zastąpić go demonem `ntpd`
z pakietu `ntpsec` .

<!--more-->
## Czemu potrzebny jest nam dokładny czas

O protokole NTP raczej każdy z nas już słyszał. Zwykle jest on stosowany do synchronizacji
czasu między maszynami przy wykorzystaniu sieci, gdzie opóźnienia na drodze pakietów nie są stałe.
Dokładny czas jest obecnie dość istotnym elementem pracy komputera podłączonego do internetu, bo
nawet drobne różnice mogą wpływać bardzo niekorzystnie na jego zachowanie. Weźmy na przykład
uwierzytelnianie dwuskładnikowe wykorzystujące kody jednorazowe oparte na czasie ([Time based One
Time Password][12], TOTP). Jak bez dokładnego czasu bylibyśmy w stanie te kody generować i z nich
korzystać? Jeśli czas naszego komputera/smartfona nie byłby zsynchronizowany, to mógłby odbiegać od
czasu serwera usługi, do której próbujemy się zalogować. Inny czas na tych maszynach oznacza inne
kody. Oczywiście, drobne różnice w czasie są dopuszczalne ([~89 sekund][13]), bo w przeciwnym
wypadku mielibyśmy ciągłe problemy z logowaniem, zwłaszcza przy korzystaniu z tokenów sprzętowych,
które do sieci zwykle podłączone nie są, przez co nie mogą synchronizować czasu, a ich bateria przy
tym wieczna nie jest. Niemniej jednak, są sytuacje, które wymagają bardzo precyzyjnych czasów, np.
Bitcoin.

## Jak wygląda synchronizacja czasu w Debianie

Zwykle synchronizacja czasu na Debianie odbywa się automatycznie i tym zadaniem zajmuje się demon
`systemd-timesyncd` przy zestawianiu połączenia sieciowego. W zasadzie nie są od nas wymagane żadne
dodatkowe czynności, by nasz linux miał aktualny czas. Niekiedy jednak ustawienia, które mają za
zadanie automatycznie synchronizować czas mogą być wyłączone. W takim przypadku użytkownik musi
włączyć synchronizację czasu z serwerami NTP przy pomocy `timedatectl` :

    # timedatectl set-ntp true

Po wydaniu tego polecenia dobrze jest jeszcze zweryfikować czy synchronizacja czasu działa:

    $ timedatectl status
                   Local time: Tue 2020-09-22 22:30:16 CEST
               Universal time: Tue 2020-09-22 20:30:16 UTC
                     RTC time: Tue 2020-09-22 20:30:16
                    Time zone: Europe/Warsaw (CEST, +0200)
    System clock synchronized: yes
                  NTP service: active
              RTC in local TZ: no

Wyżej widzimy ustawienia czasu mojej maszyny. To co nas najbardziej interesuje tutaj, to czy usługa
NTP jest aktywna ( `active` ) oraz czy zegar systemowy został zsynchronizowany ( `yes` ). Zatem
system automatycznie dokonał synchronizacji czasu. Jeśli jednak by się tak zdarzyło, że czas nie
byłby zsynchronizowany, to naturalnie możemy dokonać ręcznej synchronizacji restartując demona
`systemd-timesyncd` :

    # systemctl restart systemd-timesyncd.service

Status synchronizacji możemy sprawdzić w `timedatectl` :

    $ timedatectl timesync-status
           Server: 94.154.96.7 (0.pl.pool.ntp.org)
    Poll interval: 1min 4s (min: 32s; max 34min 8s)
             Leap: normal
          Version: 4
          Stratum: 2
        Reference: C21D82FC
        Precision: 1us (-22)
    Root distance: 34.377ms (max: 5s)
           Offset: -12.500ms
            Delay: 40.628ms
           Jitter: 0
     Packet count: 1
        Frequency: -82.819ppm

Poniżej znajduje się krótkie wyjaśnienie tego całego magicznego wyjścia widocznego wyżej:

- `Server` odpowiada za maszynę, z którą nasz linux synchronizuje czas. Jak widzimy wyżej, czas
  został zsynchronizowany z serwerem `0.pl.pool.ntp.org` . Proces tej synchronizacji będzie
  przeprowadzany automatycznie co pewien interwał czasu określony w `Poll interval` .
- `Poll interval` odpowiada za minimalny i maksymalny interwał czasu odpytywania serwera NTP o
  aktualny czas. Z początku po uruchomieniu `systemd-timesyncd` , czas jest synchronizowany z chwilą
  wystartowania tego demona. Następna synchronizacja czasu będzie miała miejsce 32 sekundy później.
  Kolejne odpowiednio w 1:04, 2:08, 4:16, 8:32, 17:04 i 34:08. Po osiągnięciu tego ostatniego czasu,
  linux będzie przeprowadzał synchronizacje w tym najwyższym interwale, czyli co około pół godziny.
  W przypadku gdybyśmy restartowali demona `systemd-timesyncd` czy też połączenie sieciowe (albo i
  cały system), to ten proces zacznie się od nowa, tj. z początku będzie ten czas synchronizowany
  co parę sekund/minut, a później znów co pół godziny.
- `Leap` określa (zgodnie z tym co można wyczytać w [RFC5905][7]) jak traktowana jest sekunda
  przestępna ([leap second][8]), która raz na jakiś czas jest dodawana (lub  odejmowana) do zegara w
  chwili wybicia północy. W takim przypadku zamiast zmiany z 23:59:59 na 00:00:00, będzie zmiana na
  23:59:60 i dalej już standardowo. W linux zaś [przez dwie sekundy będziemy mieć czas 23:59:59][6].
  Wartość `normal` , którą widzieliśmy wyżej w wyjściu `timedatectl` oznacza, że mamy standardowy
  czas, tj. bez dodatkowych sekund przestępnych.
- `Version` odpowiada za wersję protokołu NTP. Obecnie używa się wersji `4` .
- `Stratum` określa z jakim typem maszyny został zsynchronizowany czas. Wyróżnia się `Stratum` od
  `0-255` . W przypadku `0` mamy nieokreślony/błędny typ, `1` odpowiada za główny serwer np.
  wyposażony w odbiornik GPS czy zegar atomowy (bardzo dokładny czas), `2-15` to standardowy typ
  serwera czasu wykorzystywany w protokole NTP, `16` oznacza zaś czas niezsynchronizowany. Pozostałe
  wartości `17-255` są póki co zarezerwowane i nieużywane. Generalnie to im mniejszy numerek tym
  czas otrzymany od takiej maszyny jest dokładniejszy. Więcej o [hierarchii serwerów NTP można
  poczytać tutaj][5].
- `Reference` (brak danych).
- `Precision` określa precyzję zegara maszyny synchronizującej czas, tj. naszego linux'a, a te
  zwykle mają precyzję pokroju jednej mikrosekundy (1 us).
- `Root distance` to połowa `root delay` , który to odpowiada za RTD/RTT (Round-Trip Delay,
  Round-Trip Time) do serwera wzorcowego, plus `root dispersion` , który jest miarą dyspersji tego
  źródła (cokolwiek to znaczy :D).
- `Offset` to różnica między dwoma zegarami, tj. zegarem naszej maszyny i zegarem maszyny, z którą
  czas ma być zsynchronizowany. Wartość tego czasu zawsze będzie ujemna, choć do obliczeń używa się
  wartości absolutnej/bezwzględnej.
- `Delay` to RTD/RTT, czyli czas potrzebny na przesłanie pakietu do serwera i otrzymanie od niego
  pakietu potwierdzającego przybycie tego pakietu cośmy mu wysłali.
- `Jitter` jest miarą wariancji opóźnienia w sieci. Jeśli opóźnienie w sieci jest stałe, to jitter
  jest zerowy.
- `Packet count` odpowiada za zliczanie ilości dokonanych synchronizacji czasu. Z każdą kolejną
  synchronizacją, wartość tego parametru rośnie o `1` .
- `Frequency` nie mam pojęcia od czego jest ten parametr ale mierzony jest w `ppm` (parts per
  million, czyli części na milion), gdzie 1 ppm jest równy 10^(-6), tj. 0.000001 s/s
  (seconds-per-second, czyli sekund na sekundę).

### Brak wsparcia dla NTP w systemd-timesyncd

Synchronizacja czasu z wykorzystaniem protokołu NTP może odbywać się na różne sposoby. Najczęściej
wykorzystywany jest podział na klienta i serwer, gdzie klient przesyła zapytanie w pojedynczym
pakiecie UDP na port `123` do serwera, a ten z kolei zwraca odpowiedź w postaci swojego czasu
zegarowego. Następnie klient oblicza szacunkową różnicę między czasem swojego zegara, a czasem
zegara zdalnego, próbując przy tym skompensować opóźnienia związane z przesyłaniem pakietów przez
sieć. Klient przesyła także zapytania do wielu serwerów jednocześnie, by najlepiej oszacować
dokładny czas i odrzucić przy tym błędne odpowiedzi. Niemniej jednak, w przypadku
`systemd-timesyncd` [nie mamy do czynienia z jako takim protokołem NTP][15], a jedynie SNTP ([Simple
Network Time Protocol][14]). W efekcie czego, synchronizacja czasu przy pomocy tego demona odbywa
się tylko z jednym serwerem i to bez znaczenia ile serwerów czasu skonfigurujemy w pliku
`/etc/systemd/timesyncd.conf` :

    [Time]
    NTP=0.pl.pool.ntp.org 1.pl.pool.ntp.org 2.pl.pool.ntp.org 3.pl.pool.ntp.org
    FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
    #RootDistanceMaxSec=5
    #PollIntervalMinSec=32
    #PollIntervalMaxSec=2048

W takiej konfiguracji jaką mamy wyżej, `systemd-timesyncd` będzie próbował zsynchronizować czas z
serwerem `0.pl.pool.ntp.org` i jeśli ten będzie niedostępny z jakiegoś powodu, to wtedy zostanie
użyty `1.pl.pool.ntp.org` , itd. Serwery w `FallbackNTP` są używane jedynie w momencie braku
skonfigurowania jakiejkolwiek informacji o serwerach czasu. Nie chodzi tutaj o brak odpowiedzi z
serwerów czasu, np. w skutek niedziałającej sieci czy problemów po stronie serwerów, tylko o fakt,
że w systemd można skonfigurować serwer NTP dla każdego interfejsu sieciowego osobno i w takim
przypadku ten serwer będzie miał pierwszeństwo i to on zostanie użyty przy synchronizacji czasu.
Wyżej widoczny parametr `NTP` określa zaś serwery NTP dla wszystkich interfejsów sieciowych w
systemie jednocześnie. Gdyby zdarzyła się taka sytuacja, że w konfiguracji interfejsu nie byłby
określony serwer NTP i dodatkowo parametr `NTP` byłby pusty, to w takim przypadku dopiero zostaną
wykorzystane serwery skonfigurowane w `FallbackNTP` .

### Podejrzenie synchronizacji czasu w wireshark

Jeśli ktoś jest ciekaw jak wygląda proces synchronizacji czasu przy wykorzystaniu
`systemd-timesyncd` na maszynie z Debianem, to wystarczy zainstalować sniffer pakietów `wireshark`
i odpalić go na interfejsie, którym pakiety są przesyłane w świat (w tym przypadku jest to interfejs
`bond0` ):

![]({{< baseurl >}}/img/2020/09/001-debian-linux-time-sync-ntp-sntp-wireshark.png#huge)

Zostały w zasadzie przesłane tylko dwa pakiety. Pierwszy to zapytanie o czas, a drugi to odpowiedź
z serwera z aktualnym jego czasem. Gdyby wykorzystywany był tutaj pełny protokół NTP, to wtedy tych
pakietów byłoby więcej.

Jak widać synchronizacja czasu na linux jest w pełni automatyczna oraz odbywa się w tle, przez co
praktycznie nie zauważamy tego procesu podczas codziennego korzystania z komputera. Niemniej jednak,
synchronizacja czasu z wykorzystaniem protokołu NTP/SNTP (za pośrednictwem internetu) nie jest w
żaden sposób zabezpieczona, w efekcie czego komunikaty otrzymane od serwera NTP mogą zostać
podrobione, co implikuje problemy związane z bezpieczeństwem. By się przed takimi sytuacjami
chronić, trzeba zrezygnować z korzystania z protokołu NTP/SNTP i przejść na protokół NTS.

## Czym jest protokół NTS

[Protokół Network Time Security][9] (NTS) powstał w celu uporania się z problemami natury
bezpieczeństwa, które są obecne w protokole NTP. Zgodnie z tym co możemy [wyczytać na stronie
CloudFlare][10], protokół NTS przy wykorzystaniu kryptografii zabezpiecza protokół NTP umożliwiając
tym samym użytkownikom dokonanie synchronizacji czasu w sposób uwierzytelniony.

[Zasada działania protokołu NTS][20] jest stosunkowo prosta. Proces synchronizacji czasu jest
podzielony na dwie fazy, a każda z tych faz jest obsługiwana przez inny protokół:
`NTS Key Establishment` oraz `NTS Extensions for NTPv4` . Pierwsza faza ma miejsce na początku
komunikacji i jej zadaniem jest negocjacja parametrów oraz wymiana materiału klucza w postaci
plików cookie. W drugiej fazie mamy do czynienia z zabezpieczonym połączeniem NTP. Te ciasteczka,
które serwer przesłał klientowi będą teraz przez tego klienta dołączane do zapytań NTP. Druga faza
trwa do momentu zerwania połączenia lub skończenia się klientowi ciasteczek w skutek powtarzającej
się utraty pakietów. Gdy taka sytuacja nastąpi, to ponownie rozpoczyna się faza pierwsza.

### Szczegółowy opis działania protokołu NTS

Weźmy na przykład serwer CloudFlare. Klient chcący zsynchronizować czas łączy się pierw z serwerem
NTS na adres `time.cloudflare.com` i port `4460/TCP` . Jest to regularne połączenie TLS, w którym
obie strony przeprowadzają proces witania (TLS handshake). Ta procedura zezwala na zastosowanie
infrastruktury klucza publicznego ([PKI][19]) i sprawdzenie wiarygodności serwera, o ile jego
certyfikat jest godny zaufania. Po sprawdzeniu i zweryfikowaniu certyfikatów, tunel TLS zostaje
otwarty.

Przy pomocy TLS Application Data Protocol negocjowane są parametry dla protokołu NTS, po
czym zapisywane są one w rekordach TLS. Te rekordy zawierają między innymi zestaw ciasteczek
(sztuk 8) oraz informacje o adresie i porcie serwera NTP, dla którego te ciasteczka są ważne.
Informacje o dodatkowym adresie i porcie umożliwiają rozdzielone usług serwera NTS i NTP na kilka
maszyn fizycznych, przez co wiele serwerów NTP może być w stanie współdzielić jeden serwer NTS.

Zarówno serwer NTS jak i serwer NTP mogą generować ciasteczka i szyfrować je z wykorzystaniem
sekretnego klucza głównego. Ciasteczka różnią się między sobą, co ma na celu przeciwdziałać
śledzeniu hostów w sieci, np. gdy ich adres IP ulega zmianie. Ciasteczka posiadają także określony
przez serwer czas żywotności. Co jakiś czas serwer generuje nowy klucz główny ale nie oznacza to
automatycznie, że nie można korzystać ze starych ciasteczek. Będą one akceptowane przez serwer
przez jakiś czas (~24h). Takie podejście sprawia, że klucz może być odświeżany w regularnych
interwałach czasu bez potrzeby ponownego połączenia z serwerem NTS i przechodzenia przez fazę
pierwszą. Zastosowanie ciasteczek sprawia, że serwer może pracować bezstanowo, czyli nie przechowuje
informacji o połączeniach z klientami. Te ciasteczka zawierają także materiał klucza wydobyty z
połączenia TLS, który klient wyciąga przy pomocy [TLS key export][11] po otrzymaniu ciasteczek, po
czym kończy połączenie TLS i przechodzi w fazę drugą.

W fazie drugiej przeprowadzana jest faktyczna synchronizacja czasu z serwerem NTP, którego adres i
port został wskazany w poprzedniej fazie. Klient przesyła do serwera pakiet NTP, który zawiera
dodatkowo szereg pól rozszerzeń (zwykle 3-10, choć póki co tylko 4 są zdefiniowane). W skład tych
pól wchodzi też jedno ciasteczko oraz tag uwierzytelniający wyliczony przy wykorzystaniu materiału
klucza wydobytego z TLS handshake.

Ciasteczka są wykorzystywane tylko raz w kolejności od najstarszego (najkrótszy czas życia). Po
wysłaniu zapytania do serwera NTP, takie ciasteczko jest zużywane. By u klienta nie doszło do
wyczerpania ciasteczek, serwer w odpowiedzi odeśle jedno ciasteczko. Gdyby nastąpiła utrata pakietów
i klient by dysponował mniejszą ilością ciasteczek, to serwer uzupełni te brakujące ciasteczka w
następnej odpowiedzi. W ten sposób u klienta zawsze będzie dostępny zestaw 8 ciasteczek. Z kolei tag
uwierzytelniający ma za zadanie zapewnić integralność nagłówka NTP i wszystkich wcześniejszych pól
rozszerzeń.

Po wdrożeniu protokołu NTS, pakiet NTP będzie trochę większy w stosunku do tego standardowego.
Nagłówek niezabezpieczonego pakietu NTP ma rozmiar 48 bajtów. W przypadku pakietu NTP
zabezpieczonego protokołem NTS, ten rozmiar będzie w granicach od 228 do 1468 (nagłówek NTP + pola
rozszerzeń). Może zatem pojawić się problem z fragmentacją pakietów, choć można go uniknąć przez
żądanie mniejszej ilości ciasteczek w pojedynczym zapytaniu.

W momencie, gdy serwer otrzyma pakiet NTP zabezpieczony protokołem NTS, to najpierw odszyfrowywany
jest plik cookie za pomocą klucza głównego. Następnie z tego odszyfrowanego ciasteczka wyodrębniany
jest wynegocjowany algorytm AEAD jak i również zawarte w nim klucze. W ten sposób serwer jest w
stanie sprawdzić integralność pakietu NTP i upewnić się, że zawarte w nim dane nie zostały w żaden
sposób zmienione po opuszczeniu maszyny klienta. W przypadku pozytywnego zweryfikowania pakietu,
serwer generuje jedno lub więcej ciasteczek i tworzy pakiet NTP z odpowiedzią. Serwer zawsze
szyfruje swoje ciasteczka, w przeciwieństwie do klienta.

Klient po otrzymaniu odpowiedzi sprawdza unikalny identyfikator (Unique Identifier EF) by
wyeliminować potencjalne ryzyko przeprowadzenia [ataku typu replay][21]. Następnie sprawdzana jest
integralność pakietu (zarówno klucz główny jak i algorytm AEAD są klientowi znane). Po pomyślnej
weryfikacji, ciasteczka są deszyfrowane i dodawane do puli ciasteczek klienta. Informacje o czasie
zawarte w pakiecie są zaś przekazywane do systemu.

## Rezygnacja z systemd-timesyncd na rzecz ntpd (ntpsec)

Problem z demonem `systemd-timesyncd` jest oczywisty -- nie wspiera ani protokołu NTP jako takiego,
ani też nie potrafi zapewnić bezpiecznej synchronizacji czasu z wykorzystaniem protokołu NTS. Jak
to bywa w świecie linux'ów, wiele różnych aplikacji jest w stanie realizować dokładnie to samo
zadanie i jeśli jakaś aplikacja nie spełnia naszych wymagań, to naturalnie możemy ją zastąpić inną
aplikacją. Może i CloudFlare posiada swój serwer NTS ale póki co jeszcze nie opracowali oni klienta
NTS, który można by bez problemu wykorzystać do synchronizacji czasu. Na szczęście część klientów
NTP dostępnych w repozytorium Debiana wspiera protokół NTS, a jednym z nich jest `ntpd` z pakietu
`ntpsec` .

Przy instalacji tego pakietu trzeba trochę uważać, bo najwyraźniej nie można posiadać w systemie
kilku pakietów z demonami synchronizacji czasu. Dlatego też gdy będziemy instalować `ntpsec` , to
system zażąda od nas usunięcia pakietu `systemd-timesyncd` :

    # aptitude install ntpsec
    The following NEW packages will be installed:
      ntpsec python3-ntp{a}
    The following packages will be REMOVED:
      systemd-timesyncd{a}
    The following packages are SUGGESTED but will NOT be installed:
      certbot ntpsec-doc ntpsec-ntpviz
    0 packages upgraded, 2 newly installed, 1 to remove and 4 not upgraded.
    Need to get 0 B/424 kB of archives. After unpacking 1,042 kB will be used.
    Do you want to continue? [Y/n/?]

Oczywiście potwierdzamy zapytanie. Usługa `systemd-timesyncd` zostanie automatycznie zatrzymana i
usunięta z autostartu, więc nie trzeba tych czynności przeprowadzać wcześniej ręcznie. Po
zainstalowaniu `ntpsec` upewnijmy się, że usługa synchronizacji została dodana do autostartu:

    # systemctl enable ntpsec.service
    Synchronizing state of ntpsec.service with SysV service script with /lib/systemd/systemd-sysv-install.
    Executing: /lib/systemd/systemd-sysv-install enable ntpsec
    Created symlink /etc/systemd/system/multi-user.target.wants/ntpsec.service → /lib/systemd/system/ntpsec.service.

    # systemctl start ntpsec.service

### Kwestia serwerów czasu otrzymywanych w lease DHCP

Standardowym klientem DHCP w Debianie jest `dhclient` i domyślnie jest on tak skonfigurowany, by w
zapytaniu o konfigurację adresacji IP wysyłał również prośbę o przesłanie adresów serwerów czasu.
Możemy ten fakt poznać po obecności parametrów `ntp-servers` oraz `dhcp6.sntp-servers` w linijce z
`request` w pliku `/etc/dhcp/dhclient.conf` , przykładowo:

    request subnet-mask, broadcast-address, time-offset, routers,
        domain-name, domain-name-servers, domain-search, host-name,
        dhcp6.name-servers, dhcp6.domain-search, dhcp6.fqdn, dhcp6.sntp-servers,
        netbios-name-servers, netbios-scope, interface-mtu,
        rfc3442-classless-static-routes, ntp-servers;

Jeśli teraz serwer DHCP ma skonfigurowane jakieś serwery czasu, to prześle je do klienta DHCP, a ten
tak skonfiguruje system, by z tych serwerów czasu korzystał. Jeśli tego typu zachowanie nie jest
pożądane i chcemy statycznie określić z jakich serwerów czasu ma korzystać demon `ntpd` , to możemy
zrobić dwie rzeczy. Pierwszą z nich jest edycja tej powyższej konfiguracji `dhclient` i usunięcie z
niej parametrów `ntp-servers` oraz `dhcp6.sntp-servers` . Drugim rozwiązaniem jest edycja pliku
`/etc/default/ntpsec` i przepisanie w nim tej poniższej opcji:

    IGNORE_DHCP="yes"

W ten sposób skrypt `/etc/dhcp/dhclient-exit-hooks.d/ntpsec` nie będzie konfigurował serwerów czasu.

Oczywiście nic nie stoi na przeszkodzie, by te dwa rozwiązania połączyć.

## Konfiguracja ntpsec

Standardowo usługa `ntpsec` nie jest skonfigurowana, tak by umożliwić nam synchronizację czasu z
wykorzystaniem protokołu NTS. Przeglądając plik `/etc/ntpsec/ntp.conf` możemy zauważyć, że są tam
określone te cztery poniższe serwery czasu (a właściwie pule serwerów):

    pool 0.debian.pool.ntp.org iburst
    pool 1.debian.pool.ntp.org iburst
    pool 2.debian.pool.ntp.org iburst
    pool 3.debian.pool.ntp.org iburst

Zwykle demona NTP konfiguruje się tak, by korzystał z kilku serwerów czasu jednocześnie. Czy
synchronizacja czasu z jednym serwerem nie powinna nam w zupełności wystarczyć? W przypadku,
gdybyśmy synchronizowali czas tylko z jednym serwerem, to jeśli on miałby błędy czas, to po
synchronizacji również i my byśmy mieli błędy czas. W przypadku dwóch serwerów, które mają ustawiony
inny czas, protokół NTP nie jest w stanie oszacować, który z nich jest prawidłowy. W przypadku
trzech serwerów, nawet jeśli któryś z nich ma błędy czas, to NTP będzie w stanie oszacować ten
prawidłowy, przy założeniu, że dwa pozostałe serwery dysponują poprawnym czasem. Może się jednak
zdarzyć sytuacja, że jeden z tych dwóch poprawnie skonfigurowanych serwerów czasu będzie niedostępny
i w takim przypadku wracamy do poprzedniej konfiguracji z jednym błędnym i jednym dobrym czasem.
Dlatego właśnie stosuje się cztery serwery czasu, by te powyżej opisane problemy ograniczyć do
minimum. Oczywiście w dalszym ciągu można tych serwerów skonfigurować więcej, by jeszcze bardziej
zminimalizować ryzyko dokonania synchronizacji nieprawidłowego czasu ale te cztery serwery zwykle
wystarczają.

Warto zwrócić uwagę, że każda z tych powyższych linijek zaczyna się od `pool` i zwykle taki jeden
wpis zapewnia odpytywanie czterech serwerów czasu. Parametr `iburst` ma na celu wysłanie 6 pakietów
do każdego ze skonfigurowanych serwerów w celu dokładniejszej synchronizacji czasu. Jak łatwo można
policzyć, tych zapytań może być nawet 4\*4\*6=96 i tyle samo odpowiedzi. Zatem protokół NTP może być
trochę bardziej rozmowny niż SNTP (gdzie było tylko jedno zapytanie i jedna odpowiedź) ale też
powinien zapewnić dokładniejszy czas i uchronić nas przed ewentualnymi błędami w konfiguracji
serwerów.

### Włączenie protokołu NTS w ntpsec

Nas jednak te domyślne ustawienia synchronizacji czasu niezbyt interesują, bo serwery
`debian.pool.ntp.org` nie wspierają NTS. Dlatego trzeba te powyższe wpisy wykomentować. W ich
miejsce dodajemy zaś te serwery, które wspierają protokół NTS:

	server time.cloudflare.com iburst minpoll 6 maxpoll 10 version 4 nts aead AES_SIV_CMAC_512
	server nts.ntp.se:4443 iburst minpoll 6 maxpoll 10 version 4 nts aead AES_SIV_CMAC_512
	server ntpmon.dcs1.biz iburst minpoll 6 maxpoll 10 version 4 nts aead AES_SIV_CMAC_512
	server ntp1.glypnod.com iburst minpoll 6 maxpoll 10 version 4 nts aead AES_SIV_CMAC_512
	server ntp2.glypnod.com iburst minpoll 6 maxpoll 10 version 4 nts aead AES_SIV_CMAC_512

W powyższej linijce zamiast `pool` został określony `server` . Oznacza to, że zapytania będą
przesyłane tylko do określonego serwera, a nie do puli serwerów. Z informacji, które znalazłem
wynika, że CloudFlare dysponuje tylko jednym serwerem czasu, który obsługuje protokół NTS. Biorąc
pod uwagę specyfikację NTP, tych serwerów musi być co najmniej kilka. Dlatego też gdybyśmy określili
wyżej tylko jeden wpis z serwerem `time.cloudflare.com` , to nasz system nie będzie chciał
przeprowadzić synchronizacji czasu (albo ta synchronizacja zajmie bardzo długo). Dlatego tych
serwerów jest kilka, a ich adresy zostały wzięte z [tej strony][24].

[Domyślnym portem dla NTS jest 4460][22]. Niemniej jednak, CloudFlare początkowo wykorzystywał port
1234 ale [ostatnio zmienił go na ten domyślny][13] określony w specyfikacji NTS. Póki co wygląda na
to, że z serwera NTS oferowanego przez CloudFlare można korzystać zarówno podając port `1234` jak i
`4460` . Trzeba jednak liczyć się z faktem, że ten pierwszy port w końcu przestanie odpowiadać i
dobrze jest już teraz ustawić w konfiguracji tego serwera czasu port `4460` . Możemy także pominąć
określanie portu, przez co zostanie użyty ten domyślny.

Określenie parametru `iburst` znacznie przyśpieszy proces synchronizacji czasu. Jeśli chodzi o
`minpoll` i `maxpoll` , to odpowiadają one za minimalny i maksymalny interwał odpytywania serwera o
czas. Wartości tych parametrów są wyrażone jako potęga dwójki w sekundach, zatem minimalny interwał
to 2^6, a maksymalny to 2^10, czyli interwał synchronizacji wynosi 64-1024 sekund. Warto dodać w tym
miejscu, że ten interwał odpytywania serwera jest nieco inaczej zaimplementowany w `ntpsec` niż
miało to miejsce w przypadku `systemd-timesyncd` . Tutaj [nie wiadomo jak on dokładnie działa][18],
choć w grę mogą wchodzić takie rzeczy jak np. aktualne obciążenie systemu podczas procesu
synchronizacji czasu. Generalnie to przy testach ciągle była wykorzystywana wartość ustawiona w
`minpoll` . Parametr `version` odpowiada za wersję protokołu NTP, natomiast kluczowa w tej powyższej
linijce jest fraza `nts` , która mówi, że komunikacja z tym serwerem czasu ma się odbywać z
wykorzystaniem protokołu NTS. Opcjonalnie można także określić algorytm AEAD ([Authenticated
Encryption with Associated Data][17]), choć możliwość jego zmiany nie jest jeszcze zaimplementowana.

### Polityka kontroli dostępu do ntpd

Demon `ntpd` jest zarazem klientem jak i serwerem. Potrafi wysyłać zapytania do serwerów czasu i
potrafi także je odbierać. Domyślnie ten demon jest tak skonfigurowany, by być w stanie wymieniać
czas z każdym. Niemniej jednak, wszystkie zapytania, które mają na celu zmianę konfiguracji serwera,
są domyślnie zabronione. Generalnie, to polityka [kontroli dostępu][26] jest określona przez wpisy
zaczynające się od `restrict` , które figurują w pliku `/etc/ntpsec/ntp.conf` . Domyślnie widnieją
tam te trzy poniższe wpisy:

    restrict default kod nomodify nopeer noquery limited

    restrict 127.0.0.1
    restrict ::1

Wpis z `default` występuje zawsze (nawet jeśli nie zostanie określony w pliku konfiguracyjnym) i
jego zadaniem jest określenie wszystkich hostów, tj, adres `0.0.0.0` i maska `0.0.0.0` dla IPv4
oraz adres `::` i maska `::` dla IPv6. Naturalnie w miejscu `default` może pojawić się adres IP w
zapisie CIDR (np. 192.168.1.0/24). Kolejne parametry określają co danemu hostowi wolno podczas
połączenia. Jeśli wpis z `restrict` ma jedynie adres IP, to wtedy hostowi wolno wszystko i ta
powyższa konfiguracja zezwala na zarządzanie demonem `ntpd` w dowolny w sposób ale tylko z lokalnego
hosta. Wszystkie pozostałe hosty mają te poniższe ograniczenia:

- `kod` -- przy pogwałceniu polityki dostępu, [pakiet Kiss of Death][25] zostanie przesłany do
  hosta, który tę politykę naruszył.
- `nomodify` -- nie zezwala na zapytania `ntpq` , które są w stanie zmodyfikować stan serwera, tj.
  przekonfigurować go podczas pracy. Jedynie zapytania, które zwracają informacje są dozwolone.
- `nopeer` -- odrzuca pakiety, które spowodowałyby mobilizację nowego powiązania. Działo się
  tak, gdy zdalny klient używał polecenia `peer` w swoim pliku konfiguracyjnym. `ntpsec` nie
  obsługuje tego trybu.
- `noquery` -- nie zezwala na żadne zapytania `ntpq` .
- `limited` -- odmawia usługi, jeśli odstępy między pakietami przekraczają dolne limity określone w
  poleceniu `limit` . Historia klientów jest przechowywana przy użyciu funkcji monitorowania
  `ntpd` . W związku z tym monitorowanie jest zawsze aktywne, o ile istnieje wpis z flagą
  `limited` .

Przy konfiguracji tej powyższej polityki dostępu trzeba uważać, bo wpisy, które mogą zablokować
dostęp do serwera, mogą także wpłynąć na funkcjonalność klienta. W zasadzie nie ma potrzeby ruszać
ten polityki, gdy zamierzamy korzystać z `ntpd` jedynie w roli klienta.

## Test synchronizacji czasu z wykorzystaniem protokołu NTS

Konfiguracja `ntpsec` nie była jakoś skomplikowana i po jej przeprowadzeniu, nasz linux powinien
być w stanie synchronizować czas w sposób bezpieczny z wykorzystaniem protokołu NTS. Zrestartujmy
zatem demona `ntpd` i popatrzy co się dzieje w logu systemowym:

	systemd[1]: Starting ntpsec.service...
	ntpd[3082636]: INIT: ntpd ntpsec-1.1.9 2020-09-25T18:42:15Z: Starting
	ntpd[3082636]: INIT: Command line: /usr/sbin/ntpd -p /run/ntpd.pid -c /etc/ntpsec/ntp.conf -g -N -u ntpsec:ntpsec
	ntp-systemd-wrapper[3082636]: 2020-09-26T12:46:37 ntpd[3082636] INIT: ntpd ntpsec-1.1.9 2020-09-25T18:42:15Z: Starting
	ntp-systemd-wrapper[3082636]: 2020-09-26T12:46:37 ntpd[3082636] INIT: Command line: /usr/sbin/ntpd -p /run/ntpd.pid -c /etc/ntpsec/ntp.conf -g -N -u ntpsec:ntpsec
	systemd[1]: ntpsec.service: Can't open PID file /run/ntpd.pid (yet?) after start: Operation not permitted
	ntpd[3082638]: INIT: precision = 0.063 usec (-24)
	systemd[1]: Started ntpsec.service.
	ntpd[3082638]: INIT: successfully locked into RAM
	ntpd[3082638]: CONFIG: readconfig: parsing file: /etc/ntpsec/ntp.conf
	ntpd[3082638]: CONFIG: restrict nopeer ignored
	ntpd[3082638]: CLOCK: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): good hash signature
	ntpd[3082638]: CLOCK: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): loaded, expire=2020-12-28T00:00Z last=2017-01-01T00:00Z ofs=37
	ntpd[3082638]: INIT: Using SO_TIMESTAMPNS
	ntpd[3082638]: IO: Listen and drop on 0 v6wildcard [::]:123
	ntpd[3082638]: IO: Listen and drop on 1 v4wildcard 0.0.0.0:123
	ntpd[3082638]: IO: Listen normally on 2 lo 127.0.0.1:123
	ntpd[3082638]: IO: Listen normally on 3 bond0 192.168.1.150:123
	ntpd[3082638]: IO: Listen normally on 4 lo [::1]:123
	ntpd[3082638]: IO: Listen normally on 5 bond0 [fd80:56bf:c96:0:2d24:59fb:8003:4e1a]:123
	ntpd[3082638]: IO: Listen normally on 6 bond0 [fe80::8c2a:9317:9f2e:76%2]:123
	ntpd[3082638]: IO: Listening on routing socket on fd #23 for interface updates
	ntpd[3082638]: INIT: MRU 10922 entries, 13 hash bits, 65536 bytes
	ntpd[3082638]: INIT: OpenSSL 1.1.1g  21 Apr 2020, 1010107f
	ntpd[3082638]: NTSc: Using system default root certificates.

	ntpd[3082638]: DNS: dns_probe: time.cloudflare.com, cast_flags:1, flags:21901
	ntpd[3082638]: NTSc: DNS lookup of time.cloudflare.com took 0.557 sec
	ntpd[3082638]: NTSc: connecting to time.cloudflare.com:1234 => 162.159.200.123:1234
	ntpd[3082638]: NTSc: set cert host: time.cloudflare.com
	ntpd[3082638]: NTSc: Using TLSv1.3, TLS_AES_256_GCM_SHA384 (256)
	ntpd[3082638]: NTSc: certificate subject name: /C=US/ST=California/L=San Francisco/O=Cloudflare, Inc./CN=time.cloudflare.com
	ntpd[3082638]: NTSc: certificate issuer name: /C=US/O=DigiCert Inc/CN=DigiCert ECC Secure Server CA
	ntpd[3082638]: NTSc: certificate is valid.
	ntpd[3082638]: NTSc: Good ALPN from time.cloudflare.com
	ntpd[3082638]: NTSc: read 750 bytes
	ntpd[3082638]: NTSc: Using port 123
	ntpd[3082638]: NTSc: Got 7 cookies, length 100, aead=15.
	ntpd[3082638]: NTSc: NTS-KE req to time.cloudflare.com took 1.350 sec, OK
	ntpd[3082638]: DNS: dns_check: processing time.cloudflare.com, 1, 21901
	ntpd[3082638]: DNS: Server taking: 162.159.200.123
	ntpd[3082638]: DNS: dns_take_status: time.cloudflare.com=>good, 0

Log jest naturalnie dłuższy, bo w tym powyższym wycinku nie zostały uwzględnione wpisy dotyczące
wszystkich skonfigurowanych serwerów czasu. W zasadzie to ten ostatni blok (ten od pustej linijki)
powtarza się dla każdego serwera. Widzimy zatem, że demon `ntpd` posłał zapytanie do serwera czasu
i nawiązał z nim połączenie. Certyfikat serwera jest ważny, no i otrzymaliśmy także ciasteczka,
które powinny zostać użyte w pakietach NTP. Przy pomocy `wireshark` możemy sprawdzić czy w istocie
tak się dzieje:

![]({{< baseurl >}}/img/2020/09/002-debian-linux-time-sync-ntp-sntp-nts-wireshark.png#huge)

Jak widać, zamiast dwóch standardowych pakietów NTP mamy teraz tych pakietów nieco więcej (wciąż
jednak jest to pojedyncze zapytanie). Większość komunikacji jest szyfrowana ale warto zwrócić tutaj
uwagę, że zapytanie do serwera czasu, jak i odpowiedź od niego otrzymana, leci w formie
niezaszyfrowanej. Widzimy też, że pakiety NTP są wyraźnie większe (374 bajty vs. 90 bajtów w
przypadku `systemd-timesyncd` ). Brak szyfrowania pakietów NTP w niczym nie przeszkadza, bo jesteśmy
w stanie zweryfikować dane zawarte w tych pakietach. Więc nawet jeśli zdarzyłaby się sytuacja, że
odpowiedź z serwera czasu zostałaby zmieniona, to nasz klient NTP by takiej odpowiedzi nie
zaakceptował i zmiana czasu w systemie nie mogła by mieć miejsca.

Jeśli chcemy przekonać się czy faktycznie synchronizacja czasu działa, to możemy cofnąć systemowy
zegar o godzinę czy dwie do tyłu i zrestartować demona `ntpd` :

	# timedatectl set-time "2020-09-26 13:21:00"
	# systemctl restart ntpsec

Po chwili czas powinien zostać zsynchronizowany, co możemy podejrzeć w `timedatectl` :

	$ timedatectl
				   Local time: Sat 2020-09-26 17:58:37 CEST
			   Universal time: Sat 2020-09-26 15:58:37 UTC
					 RTC time: Sat 2020-09-26 15:58:37
					Time zone: Europe/Warsaw (CEST, +0200)
	System clock synchronized: no
				  NTP service: n/a
			  RTC in local TZ: no

Co ciekawe systemd nie potrafi wykryć innych usług synchronizacji czasu z serwerami NTP i zgłasza
ich brak, no i oczywiście również błędnie pokazywany jest stan synchronizacji, choć sam czas jest
prawidłowy. Także trzeba uważać przy interpretacji tych powyższych informacji jeśli nie korzystamy
z `systemd-timesyncd` .

### Zapytania do ntpd przez ntpq

Na koniec przydałoby się jeszcze popatrzeć na jakieś statystyki, które by nam powiedziały co nieco
o stanie synchronizacji. Takie informacje możemy wyciągnąć przez `ntpq` . Tym poleceniem możemy
operować w formie interaktywnej wpisując w terminalu `ntpq` ale możemy również do niego dodać
parametr `-c` i podać polecenie do wywołania. W takim przypadku wyjście tego polecenia zostanie
wypisane standardowo na konsoli, a program się zakończy.

Na początek podejrzyjmy listę serwerów, do których przesyłamy zapytania:

	# ntpq -p
		 remote                                   refid      st t when poll reach   delay   offset   jitter
	=======================================================================================================
	+time.cloudflare.com                     10.75.8.44       3 8   40   64   37  54.8485  -6.1316   9.7063
	 ntsts.sth.ntp.se                        .STEP.          16 0    -   64    0   0.0000   0.0000   0.0001
	+ntpmon.dcs1.biz                         .PPS.            1 8   25   64  177 357.4715 -13.2263  12.4706
	+ntp1.glypnod.com                        204.123.2.72     2 8    6   64  377 207.7296  -1.7988  26.4537
	*ntp2.glypnod.com                        193.62.22.74     2 8   25   64  177  72.1782  -5.3229  40.3431

Kolumna `st` oznacza typ serwera (stratum). W kolumnie `t` mamy ilość ciasteczek, które zostały do
dyspozycji klienta dla konkretnego serwera. Generalnie to powinniśmy widzieć tutaj zawsze wartość
`8` . Każda inna wartość świadczy o utracie pakietów. No może za wyjątkiem `7` , gdzie akurat
trafiliśmy na pakiet w locie. Kolumna `poll` oznacza interwał odpytywania serwera czasu w sekundach,
zaś kolumna `when` mówi kiedy ostatnio serwer został odpytany. W kolumnie `reach` mamy status RR
([Reachability Register][27]), gdzie `377` oznacza, że klient uzyskuje próbki non stop od serwera
czasu przez okres ostatnich 8 interwałów, zaś `0` wskazuje na brak komunikacji z serwerem. Wartości
pomiędzy trzeba interpretować przeliczając je z systemu ósemkowego na system binarny. W ten sposób
widoczny wyżej `0o177` to binarnie `1101 1111` . Jak widać cyferek jest 8, po jednej na każdą próbę.
Zatem wartość `177` oznacza 7 otrzymanych próbek w kolejności, którą oznacza ciąg bitów. Więc 5
ostatnich prób było udanych, potem był problem i dwie najbardziej odległe w czasie próby również
udane. Pozostałe kolumny ( `delay` , `offset` , `jitter` ) zawierają najświeższe dane na temat
wyliczonych opóźnień (w milisekundach). Idealnie jest, gdy `jitter` jest jak najbliższy zeru.

Znaczki po lewej stronie w kolumnie `remote` wskazują na status synchronizacji. Aktualnie wybrany
serwer, z którym czas został zsynchronizowany, jest oznaczony `*` . Serwery mające `+` to dodatkowe
serwery wyznaczone do synchronizacji ale aktualnie nie są one wybrane. Jeden serwer nie ma żadnego
znaczka (został odrzucony w procesie doboru i nie będzie brany pod uwagę). Tych znaczków [może być
więcej][28].

Nieco więcej informacji o tych widocznych wyżej serwerach możemy wyciągnąć z polecenia
`associations` w połączeniu z `rv` :

	# ntpq -c associations

	ind assid status  conf reach auth condition  last_event cnt
	===========================================================
	  1 17767  f43a   yes   yes   ok  candidate    sys_peer  3
	  2 17768  c011   yes    no   bad    reject    mobilize  1
	  3 17769  f41a   yes   yes   ok  candidate    sys_peer  1
	  4 17770  f42a   yes   yes   ok  candidate    sys_peer  2
	  5 17771  f65a   yes   yes   ok   sys.peer    sys_peer  5

	# ntpq -c "rv 17767"
	status=f43a conf, authenb, auth, reach, sel_candidate, 3 events, sys_peer,
	srcadr=162.159.200.123, srcport=123, dstadr=192.168.1.150, dstport=123, leap=00, hmode=3, stratum=3,
	ppoll=99, hpoll=7, precision=-24, rootdelay=7.187, rootdisp=0.458, refid=10.75.8.44,
	reftime=e319eed4.95f85048 2020-09-26T16:46:12.585Z, rec=e319ef76.b2c8337f 2020-09-26T16:48:54.698Z,
	xmt=e319ef76.9ca69d8a 2020-09-26T16:48:54.611Z, reach=377, unreach=0, delay=99.602048, offset=-36.648954,
	jitter=44.845333, dispersion=3.535664, keyid=0,
	 filtdelay =99.60       151.62  249.37  208.90  101.55  200.20  202.57  149.64,
	filtoffset =-36.65      -62.00  -110.77 -87.67  -32.02  -84.52  -86.06  -60.80, pmode=4,
	  filtdisp =0.00        1.95    3.90    5.87    6.89    7.91    8.93    9.95, flash=00 ok, headway=0, srchost="time.cloudflare.com",
	ntscookies=8

Z kolei z poniższej rozpiski możemy wyciągnąć wnioski na temat synchronizacji czasu z wykorzystaniem
protokołu NTS:

	# ntpq -c nts
	NTS client sends:            111
	NTS client recvs good:       96
	NTS client recvs w error:    0
	NTS server recvs good:       0
	NTS server recvs w error:    0
	NTS server sends:            0
	NTS make cookies:            0
	NTS decode cookies:          0
	NTS decode cookies old:      0
	NTS decode cookies too old:  0
	NTS decode cookies error:    0
	NTS KE probes good:          6
	NTS KE probes_bad:           0
	NTS KE serves good:          0
	NTS KE serves_bad:           0

Mamy tutaj w zasadzie same zapytania klienta (nasze własne) do serwera. Od momentu uruchomienia
demona `ntpd` było 6 prób nawiązania połączenia z serwerami NTS (pierwszej fazy). Biorąc pod
uwagę fakt, że mamy 5 serwerów NTS, to przy restarcie demona 5 prób dostajemy z automatu. Ten
dodatkowy pakiet do serwera NTS związany jest zapewne z `ntsts.sth.ntp.se` , który najwyraźniej ma
jakieś problemy. Poza tym, pakiety NTP są przesyłane na linii klient-server i nie widać żadnych
błędów.

## Podsumowanie

Protokół NTP do bezpiecznych nie należy ze względu na fakt, że był on projektowany w połowie lat 80
ubiegłego wieku. Te parę dekad temu internet wyglądał zupełnie inaczej niż ma to miejsce obecnie
(tylko zaufane/wiarygodne podmioty miały do niego dostęp) i w tamtych czasach nikt zbytnio nie
przejmował się bezpieczeństwem przy projektowaniu tego protokołu. Dzisiaj, prawie 40 lat później,
protokół NTP w dalszym ciągu jest niezabezpieczony, przez co stwarza poważne zagrożenie dla innych
protokołów wykorzystywanych do zabezpieczenia komunikacji w sieci, np. SSL/TLS.

Taka prosta wymiana demona realizującego synchronizację czasu w linux'ie przez zastosowanie
protokołu NTS jest w stanie znacznie poprawić bezpieczeństwo naszego systemu, a przez to również i
nas samych przy codziennym korzystaniu z internetu. Póki co, ogromna większość serwerów czasu
wspiera jedynie protokół NTP, przez co serwerów NTS nie ma jeszcze zbyt wiele. Niemniej jednak,
CloudFlare z racji swojej pozycji może zmienić ten san rzeczy, choć pewnie jeszcze trochę czasu
upłynie zanim ludzkość przejdzie z NTP/SNTP na NTS. Trzeba zdawać sobie sprawę jeszcze z faktu, że
sam protokół NTS nie jest jeszcze skończony ale już teraz jego obsługę możemy sami zaimplementować
i to zadanie do jakoś szczególnie trudnych nie należy.


[1]: {{< baseurl >}}/post/szyfrowany-dns-z-dnscrypt-proxy-i-dnsmasq-na-debian-linux/
[2]: https://pl.wikipedia.org/wiki/Network_Time_Protocol
[3]: https://trimstray.github.io/posts/2019-07-21-nginx-optymalizacja_sesji_ssl-tls/#ssl_session_tickets
[4]: https://github.com/systemd/systemd/issues/9481
[5]: https://en.wikipedia.org/wiki/Network_Time_Protocol#Clock_strata
[6]: https://0xstubs.org/systemd-timesyncd-and-leap-seconds/
[7]: https://tools.ietf.org/html/rfc5905#section-7.3
[8]: https://en.wikipedia.org/wiki/Leap_second
[9]: https://tools.ietf.org/html/draft-ietf-ntp-using-nts-for-ntp-28
[10]: https://developers.cloudflare.com/time-services/nts
[11]: https://tools.ietf.org/html/rfc5705
[12]: https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm
[13]: https://tools.ietf.org/html/rfc6238#page-7
[14]: https://en.wikipedia.org/wiki/Network_Time_Protocol#SNTP
[15]: https://github.com/systemd/systemd/issues/2893
[16]: https://blog.cloudflare.com/secure-time/
[17]: https://en.wikipedia.org/wiki/Authenticated_encryption
[18]: https://gitlab.com/NTPsec/ntpsec/-/issues/674
[19]: https://en.wikipedia.org/wiki/Public_key_infrastructure
[20]: https://blog.apnic.net/2019/11/08/network-time-security-new-ntp-authentication-mechanism/
[21]: https://academy.binance.com/pl/security/what-is-a-replay-attack
[22]: https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?skey=9&page=140
[23]: https://lists.ntpsec.org/pipermail/devel/2020-September/009489.html
[24]: https://docs.ntpsec.org/latest/NTS-QuickStart.html
[25]: http://doc.ntp.org/4.2.6/rate.html#kiss
[26]: https://docs.ntpsec.org/latest/accopt.html
[27]: https://tools.ietf.org/html/rfc1305
[28]: http://doc.ntp.org/4.1.0/ntpq.htm
