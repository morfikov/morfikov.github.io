---
author: Morfik
categories:
- Linux
date: "2015-11-04T21:36:58Z"
date_gmt: 2015-11-04 19:36:58 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
GHissueID: 260
title: Port knocking i Single Packet Authorization
---

Poniższy wpis ma na celu zaprezentować jak w prosty sposób dozbroić nieco serwer, tak by znajdujące
się na nim usługi były należycie chronione. Zostanie to pokazane na przykładzie SSH, bo chyba każdy
serwer posiada zdalny dostęp przez shell'a i za bardzo nie godzi się by zostawić tę usługę otwartą
na zewnętrzny świat wirtualny bez jakiegokolwiek nadzoru. Postaramy się tutaj wdrożyć [port
knocking](https://pl.wikipedia.org/wiki/Port_knocking), z tym, że nie będziemy wykorzystywać do tego
celu narzędzia `knockd` . Skorzystamy za to z
[fwknop](http://www.cipherdyne.org/fwknop/docs/fwknop-tutorial.html) , który eliminuje szereg wad
występujących w leciwym już `knockd` .

<!--more-->
## Domyślne porty

Chyba każdy już zdołał wyczytać na necie, że pierwsze co trzeba zrobić w przypadku wrażliwych usług
internetowych, a do takich zalicza się SSH, to zdjęcie ich z domyślnych portów i przypisanie im
portów najlepiej 5-cyfrowych. Ma to na celu zamaskowanie usługi, przez co może być ona bardziej
bezpieczna. W przypadku botów skanujących domyślne porty, może i będzie ale ta zmiana nas nie
uchroni przed świadomą próbą wykrycia nasłuchującej usługi, bo portów jest przecie tylko 65536. Poza
tym, zmiana domyślnych portów może pociągać za sobą dostosowanie konfiguracji szeregu narzędzi
klienckich. Nawet jeśli posłużymy się plikiem `/etc/services` i to w nim określimy domyślne porty,
to zawsze pozostaje problem z użytkownikami, którzy nie lubią zmian lub też za nimi nie nadążają.
Wobec tego mamy w logach pełno zdarzeń, które mogą wskazywać, że dany serwer pada właśnie ofiarą
ataku, a w rzeczywistości może to być użytkownik, który próbuje się zalogować do systemu z płytki
live, bo tam przecie domyślne porty są inne.

Zamiast się bawić z przepisywanie portów, dużo lepszym wyjściem jest odpowiednie skonfigurowanie
[filtru iptables](/post/firewall-na-linuxowe-maszyny-klienckie/), gdzie zezwalamy
na połączenie jedynie określonym klientom. Problem w przypadku dostępu zdalnego jest taki, że wielu
użytkowników może się łączyć z danym serwerem i niekoniecznie musi to robić z pewnego określonego
miejsca na ziemi. Zatem nawet adresy IP mogą ulegać zmianie i stworzenie reguł dla tych użytkowników
może w sporej części przypadków nie być możliwe. Zostawiając przy tym niezbyt filtrowany domyślny
port 22, to proszenie się o kłopoty.

Czy jest port knocking

Port knocking ma na celu otwieranie portów, na których nasłuchują usługi, z tym, że w pewnym
określonym momencie. W jakim? Tuż przed podłączeniem się zdalnego klienta. W taki sposób, mając
demona SSH nasłuchującego na porcie 22, nikt z zewnątrz nie da rady się podłączyć. No bo ten port
będzie zablokowany w iptables. Ci klienci, którzy będą mieli prawo się podłączyć, będą też świadomi
faktu odfiltrowania portu na zaporze. Wobec tego będą musieli podjąć pewne kroki mające na celu
skłonienie firewall'a, by ten im otworzył dany port, wpuścił ich, po czym po chwili ten port
zamknął.

Ten mechanizm działa w oparciu o stany połączeń. Z reguły mamy do czynienia z zaporami stanowymi,
tj. takimi, które rozróżniają pakiety w stanie NEW, ESTABLISHED, itp. Stan ESTABLISHED jest zawsze
akceptowany. Zatem jeśli łączymy się ze zdalnym serwerem, wysyłamy mu pakiet w stanie NEW. Zapora
decyduje czy go przepuścić i jeśli tak się dzieje, to po chwili połączenie przechodzi w stan
ESTABLISHED i to tutaj trafiają pakiety powiązane z tym połączeniem. Port knocking ma na celu
jedynie umożliwienie przejścia tego pakietu w stanie NEW przez zaporę. Odbywa się to przez dodanie
do zapory pewnej określonej reguły. W taki sposób klient będzie miał do dyspozycji kilkusekundowe
okno, wewnątrz którego będzie w stanie się podłączyć. Po tym czasie, ta reguła przepuszczająca
pakiety w stanie NEW zostanie usunięta z zapory automatycznie.

Mechanizm port knocking'u może być jeszcze bardziej uporczywy niż zmiana domyślnego portu, zwłaszcza
w przypadku korzystania z `knockd` . My się nie będziemy zajmować tutaj tym narzędziem, bo jest ono
już przestarzałe. Poza tym, sekwencję portów wykorzystywanych przez `knockd` można bez problemu
podsłuchać. Jest jednak o wiele lepsze oprogramowanie do port knocking'u, które wykorzystuje
techniki kryptograficzne oferujące uwierzytelnianie konkretnego klienta, co czyni cały proces
podłączania praktycznie niewidocznym i bezpiecznym.

## Single Packet Authorization (SPA)

`fwknop` wykorzystuje jeden pakiet (Single Packet Authorization) do otwierania portu/portów i ma za
zadanie wyeliminowanie wad, które ma wspomniany wyżej `knockd`. Proces uwierzytelniania i
autoryzacji odbywa się przy pomocy czegoś na wzór zaszyfrowanego ciasteczka. Dane są generowane dla
każdego klienta osobno i dostarczane na serwer. Później klienci przesyłają już tylko ciasteczka do
serwera i po ich pomyślnym odszyfrowaniu, `fwknop` może dodać określone reguły do iptables, lub/i
wykonywać pewne skrypty. Mając do dyspozycji dwie maszyny, serwer oraz klient, na pierwszej z nich
instalujemy pakiet `fwknop-server` , a na drugiej `fwknop-client` .

### Konfiguracja klienta

Najpierw zajmijmy się konfiguracją klientów. Musimy wygenerować klucze szyfrujące. Wpisujemy zatem w
terminalu tę poniższą linijkę:

    $ fwknop -A tcp/22 -a 192.168.1.20 -D 192.168.1.50 --key-gen --use-hmac --save-rc-stanza

Poniżej znajduje się wyjaśnienie użytych parametrów:

  - `-A` -- określa porty, do których klient chce mieć dostęp. Można podać kilka, np.
    `tcp/22,udp/53` .
  - `-a` -- definiuje adres IP, z którego klient będzie mógł nawiązywać połączenia z serwerem. Ten
    adres zostanie uwzględniony w regułach iptables.
  - `-D` -- precyzuje adres serwera.
  - `--key-gen` -- generuje klucze, które są używane do zaszyfrowania pakietu.
  - `--use-hmac` -- ustawia tryb HMAC w przypadku szyfrowania uwierzytelnionych komunikacji.
  - `--save-rc-stanza` -- zapisuje wprowadzone w linijce opcje w pliku `~/.fwknoprc`

Zajrzymy zatem do pliku `~/.fwknoprc` :

    [192.168.1.20]
    ALLOW_IP                    192.168.1.50
    ACCESS                      tcp/22
    SPA_SERVER                  192.168.1.20
    KEY_BASE64                  OtJlYqHPeROSwpYbeQt6OVkQykThqPZHUf6m+CBJBos=
    HMAC_KEY_BASE64             1OceLkRVoGUIPj7ExOVk7vRZl3Q4uNyusI36ojgx4hJqCL6K6aDys1twjw0o4tFbqE8ZHa6UgTKLVf0aZNf8VQ==
    USE_HMAC                    Y

`KEY_BASE64` oraz `HMAC_KEY_BASE64` to klucze, które musimy przesłać w bezpieczny sposób na serwer.
Opcji do precyzowania jest dość sporo i nie będziemy ich tutaj wszystkich omawiać. Niemniej jednak,
powyższy kawałek to niezbędne minimum.

### Konfiguracja serwera

W przypadku serwera, musimy oczywiście włączyć demona w pliku `/etc/default/fwknop-server` . Mamy
dodatkowo dwa inne pliki, na które musimy rzucić okiem: `access.conf` oraz `fwknopd.conf` , oba
znajdują się w katalogu `/etc/fwknop/` . W pliku `access.conf` musimy umieścić klucze wszystkich
klientów, którzy będą mieć prawo łączyć się do serwera, przykładowo:

    SOURCE                      192.168.1.50
    OPEN_PORTS                  tcp/22
    FW_ACCESS_TIMEOUT           20
    REQUIRE_SOURCE_ADDRESS      Y
    KEY_BASE64                  OtJlYqHPeROSwpYbeQt6OVkQykThqPZHUf6m+CBJBos=
    HMAC_KEY_BASE64             1OceLkRVoGUIPj7ExOVk7vRZl3Q4uNyusI36ojgx4hJqCL6K6aDys1twjw0o4tFbqE8ZHa6UgTKLVf0aZNf8VQ==

Tego typu zwrotek można umieszczać ile się chce. Trzeba jednak pamiętać, że plik jest przeszukiwany
od góry do dołu pod kątem dopasowań i pierwsze z nich zostanie zaakceptowane. Opcja
`FW_ACCESS_TIMEOUT` określa czas (w sekundach) otwarcia okna.

Z kolei w pliku `fwknopd.conf` określamy opcje dla samego demona:

    VERBOSE                 1;
    PCAP_INTF               eth0;
    IPT_INPUT_ACCESS        ACCEPT, filter, tcp, 2, fwknop_input, 1;

Ostatnia linijka z tych powyżej określa, w którym miejscu umieścić łańcuch `fwknop_input`, do
którego to trafiać będą reguły generowane przez `fwknopd` . W tym przypadku `ACCEPT` określa
politykę reguły, `filter` tablicę iptables, `tcp` łańcuch w tej tablicy. Cyfra `2`, definiuje
pozycję, na której umieścić łańcuch `fwknop_input`, z kolei zaś cyfra `1` precyzuje, na której
pozycji będą dodawane reguły.

Jeśli mamy niestandardową konfigurację interfejsów sieciowych, trzeba sprecyzować, który z nich ma
być poddany analizie. W moim przypadku jest to `eth0`. Jeśli podamy błędy interfejs, np. nie będzie
miał adresu IP, demon nie wystartuje. Warto także ustawić tryb verbose i obserwować logi w syslog'u.

### Test port knocking'u

Jeśli ustawiliśmy wszystko zgodnie z powyższym opisem, to po przesłaniu ciasteczka, będziemy mieć
okno otwarte przez 20s. Na ten czas zostanie dopisana reguła do iptables. Odpalmy zatem na serwerze
demona i z maszyny klienckiej przesyłamy ciasteczko do serwera za pomocą poniższej linijki (użyj
`-v` by uzyskać więcej info):

    # fwknop -n 192.168.1.20

Pakiet powinien zostać przechwycony, a reguła dodana do iptables. Po 20s, okno powinno się
automatycznie zamknąć, a reguła iptables zostać usunięta, co spowoduje odcięcie dostępu do serwera
dla nowych połączeń. Dzięki mechanizmowi jaki oferuje `fwknop` możemy spać spokojnie nawet w
przypadku gdy do publicznej wiadomości zostanie podana informacja o jakiejś luce 0day.
