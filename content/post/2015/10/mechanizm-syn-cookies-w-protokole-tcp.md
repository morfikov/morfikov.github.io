---
author: Morfik
categories:
- Linux
date: "2015-10-24T20:22:02Z"
date_gmt: 2015-10-24 18:22:02 +0200
published: true
status: publish
tags:
- tcp
- sysctl
title: Mechanizm SYN cookies w protokole TCP
---

[Atak SYN flood](https://pl.wikipedia.org/wiki/SYN_flood) to rodzaj ataku DoS, którego celem jest
wyczerpanie zasobów serwera uniemożliwiając mu tym samym poprawne realizowanie danej usługi, do
której został oddelegowany. Jest to dość popularne zjawisko i w przypadku, gdy mamy postawioną
jakąś maszynę na publicznym adresie IP, przydałoby się nieco zainteresować tym problem, który może
wystąpić w najmniej oczekiwanym momencie. W tym wpisie rzucimy okiem na mechanizm SYN cookies.

<!--more-->
## Jak wygląda atak SYN flood

Atak SYN flood polega na wysłaniu przez klienta do serwera wielu pakietów z ustawioną flagą `SYN` ,
po czym taki klient nie udziela odpowiedzi `ACK` na wysłane pakiety `SYN-ACK` , które otrzymał od
serwera. W ten sposób na serwerze tworzy się wiele niezamkniętych sesji, które w końcu zapełnią całą
wolną pamięć. Dzieje się tak, bo dane o każdym z klientów muszą być trzymane tak długo, aż w końcu
się on podłączy. W tym przypadku, jako, że klient nie ma na to zwyczajnie ochoty, to te informacje
będą trzymane w nieskończoność. Atak SYN flood skutecznie redukuje przepustowość łącza, lub też
uniemożliwia innym klientom na podłączenie się do atakowanego serwera.

## Zwiększenie ilości wpisów w tablicy conntrack'a

Podstawowy problem w przypadku SYN flood dotyczy kernelowskiego modułu `conntrack` , który odpowiada
za śledzenie połączeń. Dane dotyczące wszystkich nawiązywanych przez nasz serwer połączeń są
trzymane pod `/proc/net/ip_conntrack` lub też w `/proc/net/nf_conntrack` , w zależności od
wykorzystywanego modułu. Każdy wpis w tablicy conntrack'a zajmuje od około 190 do ponad 350 bajtów,
w zależności od pliku (faktyczną wartość można odczytać z pliku `/proc/slabinfo` ). Te dane nigdy
nie zostaną zrzucone do SWAP, zatem zawsze będą rezydować w pamięci operacyjnej i to niesie ze sobą
spore zagrożenie dla maszyn, które padną ofiarami ataku SYN flood. Jeśli zabraknie pamięci, to
serwer się zwyczajnie powiesi.

W przypadku gdy nasz serwer dysponuje sporą ilością pamięci RAM, powyższy scenariusz raczej nam nie
grozi. Niemniej jednak, konfiguracja modułu `conntrack` nie pozwala na nawiązywanie zbyt wielu
połączeń. Może to być poniekąd ochrona przed wyczerpaniem się zasobów pamięci ale w przypadku gdy
mamy jej nadmiar, może godzić w wydajność serwera, która objawiać się będzie niemożliwością
nawiązywania nowych połączeń, póki nie wygasną stare wpisy w tablicy conntrack'a. Ilość wpisów jest
konfigurowalna i możemy to zrobić przy pomocy pliku `/etc/sysctl.conf` . W tym celu wystarczy dodać
do tego pliku te poniższe linijki:

    net.ipv4.netfilter.ip_conntrack_max = 32768
    net.netfilter.nf_conntrack_max = 32768
    net.nf_conntrack_max = 32768

Musimy także dostosować wielkość tablicy hashów, za którą odpowiada wartość `hashsize` w module
`nf_conntrack` i najlepiej to zrobić przy ładowaniu tego modułu wraz ze startem systemu. W tym celu,
do pliku `/etc/modprobe.d/modules.conf` dodajemy tę poniższą linijkę:

    options nf_conntrack hashsize=4096

Pamiętajmy jednak by `nf_conntrack_max` jak i `hashsize` dobrać z głową. Gdy wartości tych
parametrów będą zbyt duże, to będą jedynie marnować dostępną pamięć operacyjną. Domyślne wartości
są ustawiane w oparciu o dostępną ilość pamięci RAM, natomiast optimum, które musimy dostosować
sobie według potrzeb, może się bardzo wahać. Przy przeładowanych serwerach, wartość
`nf_conntrack_max` powinna zostać podniesiona, a `hashsize` należałoby dostosować [w oparciu o tę
formułę](https://wiki.khnet.info/index.php/Conntrack_tuning): HASHSIZE = CONNTRACK\_MAX / 8 .

## Zwiększenie kolejek

Istnieją też dwa pomniejsze parametry, które mogą mieć wpływ na to jak zachowuje się nasz serwer
przy zbyt dużym obciążeniu. Pierwszy z nich to `tcp_max_syn_backlog` i odpowiada on za maksymalną
ilość pamiętanych żądań połączeń, które nadal nie zostały potwierdzone przez klienta. Są to
połączenia w stanie półotwartym (HALF-OPEN). Minimalna wartość dla tego parametru to 128 ale jest
on ustawiany na starcie systemu w oparciu o ilość dostępnej pamięci RAM. W każdym razie możemy go
zmienić dopisując do pliku `/etc/sysctl.conf` tę poniższą linijkę:

    net.ipv4.tcp_max_syn_backlog = 256

Drugi parametr jaki możemy dostosować to `somaxconn` . Jest to rozmiar kolejki nasłuchu (listen
queue), czyli gniazd w stanie LISTEN . Mówiąc po ludzku, jest to liczba jednoczesnych połączeń,
które serwer stara się skonfigurować. Gdy połączenie zostanie ustanowione, nie występuje ono już w
tej kolejce i ta liczba nie ma już większego znaczenia. Jeśli kolejka nasłuchu zostanie zapełniona w
wyniku zbyt wielu jednoczesnych prób połączeń, kolejne próby będą zrzucane. Mając na uwadze
powyższe, możemy dopisać do pliku `/etc/sysctl.conf` tę poniższą linijkę:

    net.core.somaxconn = 256

## SYN cookies

[Mechanizm SYN cookies](https://pl.wikipedia.org/wiki/SYN_cookies) nieco się różni od tych powyżej
opisanych, bo został specjalnie zaprojektowany by walczyć z atakami DDoS. O ile powyższe ustawienia
mogą ulżyć nieco przeładowanemu serwerowi, to raczej tylko odwloką w czasie nieuchronność zawału
systemu. Z kolei SYN cookies powinien do niego nie dopuścić. Ten mechanizm zadziała dopiero w
momencie gdy próg określony w `tcp_max_syn_backlog` zostanie przekroczony.

Trzeba też sobie zdać sprawę, że SYN cookies wykorzystuje opcję nagłówka TCP odpowiedzialną za
[znaczniki czasu (TCP timestamp)](/post/znacznik-czasu-timestamp-w-protokole-tcp/).
Dlatego też, by móc skorzystać z tego mechanizmu ochrony, trzeba również włączyć w kernelu opcję
`tcp_timestamps` . Reasumując, w pliku `/etc/sysctl.conf` powinny się znaleźć dodatkowo te poniższe
opcje:

    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_timestamps = 1
    net.netfilter.nf_conntrack_timestamp = 1

Przy tak skonfigurowanym kernelu, atak SYN flood raczej nam nie grozi. A jeśli już wystąpi, to jego
efekt będzie bardzo ograniczony.
