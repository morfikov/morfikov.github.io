---
author: Morfik
categories:
- Linux
date: "2016-03-08T15:42:59Z"
date_gmt: 2016-03-08 14:42:59 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- rootkit
- debian
GHissueID: 507
title: Rkhunter jako ochrona przed rootkit'ami
---

[Rootkit][1] to takie ustrojstwo, które jest w stanie przebywać niepostrzeżenie w naszym systemie
przez bardzo długi czas. Dzieje się tak ze względu na uśpioną czujność administratora. Teoretycznie
wszystko jest w należytym porządku, nie widać żadnych złowrogich procesów, a szereg narzędzi, które
mają przeciwdziałać rootkit'om, nie zwraca żadnych ostrzeżeń o ewentualnych próbach naruszenia
bezpieczeństwa systemu operacyjnego. Taki rootkit jest w stanie całkowicie przejąc kontrolę nad
linux'em i np. może decydować o tym jakie procesy zostaną nam pokazane, a jakie ukryte. W świetle
takiego zagrożenia, linux'y wypracowały sobie pewne mechanizmy obronne. W tym wpisie przebadamy
sobie jeden z projektów, tj. [rkhunter][2].

<!--more-->
## Rkhunter i dodatkowe narzędzia

W debianie mamy dostępny pakiet `rkhunter` , zatem nie powinno być problemów z instalacją tego
oprogramowania. To, o czym warto wspomnieć dotyczy dodatków, które to narzędzie jest w stanie
wykorzystywać podczas swojej pracy, by jeszcze w lepszym stopniu chroniło nasze maszyny. Jednym z
tych dodatkowych pakietów, który przydałoby się zainstalować, jest `unhide` . Ma on za zadanie
znaleźć ukryte przez rootkit'y procesy i/lub porty TCP/UDP. W debianie mamy także inny pakiet, tj.
`unhide.rb` . Jest on próbą przepisania oryginalnego `unhide` na Ruby, z tym, że póki co nie ma
zaimplementowanej pełnej funkcjonalności. Jest także mniej bezpieczny.

## Baza danych rkhunter'a

Po tym jak zainstalujemy w systemie pakiet `rkhunter` , musimy utworzyć bazę danych, w oparciu o
którą to będziemy w stanie zweryfikować czy aby nasz system nie zawiera jakiegoś rootkit'a. Bazę
danych możemy wygenerować z grubsza na dwa sposoby. Pierwszy z nich zakłada wykorzystanie
informacji, które są dostępne w pakietach pobieranych z repozytoriów debiana. Mamy tam min. plik
`*.md5sums` (do wglądu w `/var/lib/dpkg/info/` ). Zawiera on sumy kontrolne wszystkich plików, które
zostały zainstalowane za pomocą menadżera pakietów. Jako, że pakiety w repozytorium są zabezpieczone
cyfrowo i sumy kontrolne całych plików `.deb` są podpisane, to mamy pewność, że sumy zawarte w
plikach `*.md5sums` można traktować jako punkt wyjścia. Niemniej jednak, jak nazwa wskazuje, są to
sumy MD5 i lepiej na nich za bardzo nie polegać. W każdym razie jeśli chcemy wygenerować bazę danych
dla `rkhunter` w oparciu o te sumy kontrolne znajdujące się w pakietach `.deb` , to logujemy się na
użytkownika root i wpisujemy w terminalu to poniższe polecenie:

    # rkhunter --propupd --pkgmgr DPKG

Innym wyjściem jest utworzenie świeżej bazy danych z tych plików, które aktualnie znajdują się w
naszym systemie. Trzeba być tutaj bardzo uważnym, bo jeśli w tym momencie mamy wgranego rootkit'a,
to takie sumy kontrolne nam się raczej do niczego nie przydadzą. Dlatego musimy być pewni co do
pochodzenia plików, których sumy będą generowane. Zaletą tego rozwiązania z kolei jest fakt, że
możemy określić hash. Do wyboru mamy: `MD5` , `SHA1` , `SHA224` , `SHA256` , `SHA384` oraz
`SHA512` . Pamiętajmy, że im dłuższy hash, tym więcej czasu zajmie wyliczanie sum. Poniżej przykład
generowania bazy danych z hashami `SHA256` :

    # rkhunter --propupd --hash SHA256

Po wygenerowaniu, baza danych będzie znajdować się w pliku `/var/lib/rkhunter/db/rkhunter.dat` . Ta
baza jest podatna na zmiany, dlatego też powinniśmy ją zgrać na inne medium. Jeśli tylko nabierzemy
podejrzeń co do naruszenia integralności systemu, to będziemy mogli go zeskanować posługując się tym
plikiem. Trzeba też pamiętać, że czasem podczas aktualizacji systemu, sumy kontrolne poszczególnych
plików mogą uleć zmianie. W takim przypadku będzie trzeba dokładnie te sumy porównać i wygenerować
nową bazę w przypadku, gdy wszystko będzie w porządku.

## Konfiguracja rkhunter'a

Z narzędziem `rkhunter` dostarczany jest obszerny plik konfiguracyjny `/etc/rkhunter.conf` . Mamy
tam szereg opcji dość przyzwoicie opisanych i ze zrozumieniem większości z nich nie powinno być
żadnych problemów. Musimy jednak rzucić okiem na dyrektywy `ENABLE_TESTS` oraz `DISABLE_TESTS` , bo
to one będą sterować w dużej mierze zachowanie samego `rkhunter` . To tutaj włączamy i wyłączamy
określone testy oddzielając spacjami ich nazwy od siebie. Wszystkie dostępne testy możemy wyświetlić
sobie za pomocą tego poniższego polecenia:

    # rkhunter --list tests

Standardowo wyłączony jest `hidden_procs` , który wymaga pakietu `unhide` . Jeśli zainstalowaliśmy
go u siebie w systemie, to możemy ten test usunąć z listy wyłączonych. Zawsze po edycji tego pliku
starajmy się sprawdzić jego poprawność w poniższy sposób:

    # rkhunter -C

Konfiguracja zawarta w pliku `/etc/rkhunter.conf` może nieco przytłoczyć z początku ale warto
wiedzieć, że zawsze można zajrzeć na stronę projektu do [FAQ][3] lub [README][4] i znaleźć tam
użyteczne informacje. Nie zapomnijmy też przejrzeć [man rkhunter][5].

## Skanowanie systemu

Po utworzeniu bazy danych i skonfigurowaniu samego rkhunter'a, możemy przejść do skanowania systemu.
W zależności od tego z jakiego hasha korzystamy oraz jakie testy przeprowadzamy, taka operacja może
zająć mniej lub więcej czasu. Odpalamy zatem terminal i jako root wydajemy polecenie
`rkhunter --check` . Skanowanie wygląda mniej więcej tak:

![rkhunter-debian-rootkit-skan](/img/2016/03/1.rkhunter-debian-rootkit-skan.png#big)

Wszystkie wyrzucone ostrzeżenia niekoniecznie oznaczają problemy z systemem. Jest duże
prawdopodobieństwo, że to tylko fałszywy alarm. Niemniej jednak, musimy mieć pewność i dobrze jest
przebadać log, który standardowo jest zapisywany w `/var/log/rkhunter.log` . To w tym pliku znajdą
się dużo bardziej szczegółowe informacje, które z pewnością pomogą nam ustalić przyczynę pojawienia
się ostrzeżenia. Wszystkie wyjątki, które uda nam się zweryfikować i oznaczyć jako zaufane, dodajemy
do pliku `/etc/rkhunter.conf` w odpowiednich sekcjach.

## Zadanie dla cron'a

`rkhunter` może diagnozować jedynie maszyny, na których został uruchomiony. Zagrożenie
bezpieczeństwa zaś może zostać wykryte dopiero po fakcie, czyli gdy system został już
skompromitowany. Zwykle w takim przypadku będziemy potrzebować zewnętrznego medium, np. system live.
Niemniej jednak, debian dostarcza dwa skrypt dla cron'a, z których jeden jest wywoływany co dzień,
drugi skrypt zaś co tydzień. Od czego te skrypty są? Odpowiedź na to pytanie znajduje się w pliku
`/etc/default/rkhunter` . Przeglądając treść tego pliku, widzimy, że jeden ze skryptów (ten dzienny)
ma za zadanie skanować maszynę, zaś ten drugi skrypt ma dokonywać aktualizacji bazy danych, z tym,
że tylko tej części, która jest dostarczana wraz z pakietem. Nie wiem zbytnio czy skanowanie
maszyny co 24h ma jakiś większy sens, bo przecie jeśli już w niej znajduje się rootkit, to i tak
raczej nie zostanie wykryty. Natomiast jeśli chodzi o aktualizację bazy, to dobrze jest tę czynność
przeprowadzać regularnie.

## A może chkrootkit?

Istnieje jeszcze jedno narzędzie, z którym warto się zaznajomić, tj. [chkrootkit][6]. Różni się ono
nieco od `rkhunter` ale też ma za zadanie wykryć rootkit'a. Jest to zbitka kilku skryptów, które
nie wymagają praktycznie żadnej konfiguracji ze strony użytkownika. Oczywiście, uruchamianie tego
narzędzia w zainfekowanym systemie też raczej nam się na niewiele zda. Niemniej jednak, warto mieć
je w wyposażeniu swojego systemu live i w razie problemów, przeskanować macierzysty system za jego
pomocą.


[1]: https://pl.wikipedia.org/wiki/Rootkit
[2]: http://rkhunter.sourceforge.net/
[3]: http://rkhunter.cvs.sourceforge.net/viewvc/rkhunter/rkhunter/files/FAQ
[4]: http://rkhunter.cvs.sourceforge.net/viewvc/rkhunter/rkhunter/files/README
[5]: https://manpages.ubuntu.com/manpages/xenial/en/man8/rkhunter.8.html
[6]: http://www.chkrootkit.org/
