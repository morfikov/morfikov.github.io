---
author: Morfik
categories:
- OpenWRT
date: "2016-09-19T22:22:00Z"
date_gmt: 2016-09-19 20:22:00 +0200
published: true
status: publish
tags:
- chaos-calmer
- router
- adblock
title: Blokowanie reklam z adblock na domowym routerze WiFi
---

Na sporej części stron internetowych są nam prezentowane reklamy w miej lub bardziej nachalny
sposób. Takie banery są w stanie w dużej mierze przesłonić faktyczną treść serwisu albo też wręcz
uniemożliwić nam spokojne czytanie tekstu, który się w takiej witrynie znajduje. By walczyć z tego
typu praktykami, powstała cała masa dodatków do przeglądarek, np. [adblock][1] czy [ublock][2],
które są w stanie odfiltrować praktycznie wszystkie reklamy. Możemy pokusić się o zaimplementowanie
takiego adblock'a bezpośrednio na naszym routerze WiFi, z tym, że by taki filtr reklam wdrożyć w
naszej sieci domowej, na routerze musimy zainstalować alternatywny firmware OpenWRT/LEDE.

<!--more-->
## Problemy z adblock/ublock

Wszyscy spotkaliśmy się z przeglądarkowym dodatkiem adblock/ublock. Jego największą wadą jest
blokowanie reklam bezpośrednio na kliencie, czyli na tym komputerze, z którego korzystamy
przeglądając strony internetowe. W przypadku posiadania kilku urządzeń, ten dodatek musimy
zainstalować na każdym z nich. Jeśli dodatkowo mamy niezbyt standardową konfigurację filtrów, to te
same czynności konfiguracyjne musimy przeprowadzać kilka razy, tj. osobno na każdym profilu
przeglądarki.

Drugim problemem jaki pojawia się w przypadku korzystania z adblock/ublock, to utylizacja zasobów
komputera. Jeśli teraz przemnożymy sobie ilość mocy obliczeniowej procesora oraz potrzebną pamięć
operacyjną RAM przez wszystkie stacje robocze podłączone w naszej sieci, to dojdziemy do wniosku, że
część z tych zasobów się zwyczajnie marnuje i można by je przeznaczyć na inne rzeczy.

Zamiast zatem konfigurować filtry reklam na końcowych urządzeniach, możemy spróbować zaimplementować
taki mechanizm bezpośrednio na routerze WiFi, przez który to wszystkie połączenia wychodzą do
internetu. W ten sposób będziemy mieli do kontroli tylko jedno urządzenie i jeden filtr, z którego
będą korzystać wszystkie maszyny podłączone do naszej sieci domowej.

## Zalety i wady posiadania adblock'a na routerze

Ten filtr, który zamierzamy sobie zainstalować i skonfigurować na routerze różni się nieco od tego,
który jest dostarczany za sprawą przeglądarkowych dodatków adblock/ublock. W ich przypadku, część
zawartości strony jest blokowana przed wysłaniem żądań o określone jej elementy. W przypadku
adblock'a na routerze, ten filtr będzie blokował jedynie zapytania DNS dotyczące adserwerów. Chodzi
o to, że obecnie cała masa reklam jest nam serwowana ze specjalnie do tego celu zaprojektowanych
serwerów.

Te serwery reklamowe mają przypisane domeny, jak każdy inny zasób www w internecie. By serwis, który
my przeglądamy, był nam w stanie taką reklamę wyświetlić, nasza przeglądarka musi zostać odesłana do
adserwera. Przeglądarka zna domenę ale nie zna adresu IP. Musi ona zatem rozwiązać nazwę DNS
wysyłając zapytanie do serwera DNS. Blokując takie zapytanie, a właściwie przekierowując je,
zablokujemy tym samym komunikację z takim adserwerem i reklama nie zostanie wyświetlona w
przeglądarce.

Taki filtr nie jest skuteczny w 100%. Chodzi o to, że jeśli dany serwis wykorzystuje inny mechanizm
serwowania reklam niż adserwery, to te reklamy nam się wyświetlą. Oczywiście nasz filtr będziemy w
stanie dostosować i zablokować te pojedyncze obrazki ale trzeba mieć na uwadze takie przecieki.

Druga sprawa, to brak możliwości ukrycia pewnych elementów strony. Adblock/ublock dają nam możliwość
określenia, które elementy strony chcielibyśmy ukryć, tj. usunięcie z kodu html pewnej jego części,
tak by nie została ona przetworzona przez przeglądarkę. Adblock na routerze niestety będzie
pozbawiony tego typu mechanizmu.

## Optymalny router pod filtr reklam

Oryginalny firmware producentów nie daje nam możliwości przerobienia routera na swojego rodzaju
bloker reklam. Jeśli chcemy się w takie rzeczy bawić, to musimy postarać się o taki router, który
jest wspierany przez alternatywny firmware OpenWRT/LEDE. Na potrzeby tego artykułu wykorzystywany
był [router Archer C2600][3] od TP-LINK z wgranym najnowszą wersją LEDE.

Jeśli już planujemy sprawić sobie jakiś router, na którym zamierzamy zainstalować adblock'a, to
pamiętajmy, że potrzebne nam oprogramowanie i jego zależności swoje ważą. Dlatego też routery z 4
MiB flash odpadają. Dodatkowo, przetwarzanie list, w oparciu o które router będzie blokował
komunikację z serwerami reklam, dość mocno utylizuje procesor. Listy adserwerów trzeba także pobrać
do pamięci RAM i tam je przetworzyć, a to wymaga wolnego miejsca w pamięci. Routery z mniej niż 64
MiB RAM raczej też odpadają.

## Konfiguracja adblock'a w OpenWRT/LEDE

To mniej więcej tylko jeśli chodzi o aspekty techniczne blokowania reklam. Logujemy się zatem na
router i instalujemy w systemie te poniższe pakiety:

    # opkg update
    # opkg install adblock uhttpd wget

Te pakiety i ich zależności ważą około 1,2 MiB, więc upewnijmy się czy nasz router dysponuje taką
ilością wolnego miejsc na flash. Po zainstalowaniu tych rzeczy musimy sobie odpowiednio
skonfigurować adblock'a. Praktycznie cała konfiguracja sprowadza się do edycji pliku
`/etc/config/adblock` .

Przede wszystkim, musimy sobie dostosować jeden parametr w sekcji globalnej. Mowa o
`adb_restricted` , który umożliwia generowanie statystyk. Problem w tym, że takie statystyki są
zapisywane w pliku `/etc/config/adblock` , a więc na flash'u routera. Musimy zatem upewnić się, że
taki zapis nie następuje. Edytujemy zatem ww. plik i dodajemy do niego poniższy parametr:

    config adblock 'global'
        ...
        option adb_restricted '1'
        ...

Kolejna sprawa jeśli chodzi o konfigurację naszego filtra reklam, to listy adserwerów, które
zamierzamy blokować. W tym pliku konfiguracyjnym mamy już kilka zdefiniowanych list i możemy je
włączyć lub wyłączyć przez dostosowanie opcji `enabled` , przykładowo:

    config source 'adaway'
        option enabled '1'
        ...

### Polska lista adserwerów adblock'a

Ostatnia sprawa, to lista polskich adserwerów. Nie ma jej domyślnie w pliku `/etc/config/adblock` i
musimy ją sobie zrobić. Nie jest to jakoś specjalnie trudne. Musimy tylko znaleźć listę z takimi
serwerami i zaimportować ją na routerze. Możemy posłużyć się [polską listą do adblock/ublock][4].
Edytujemy zatem plik `/etc/config/adblock` i dodajemy w nim poniższy blok kodu:

    config source 'polish'
        option enabled '1'
        option adb_src 'http://adblocklist.org/adblock-pxf-polish.txt'
        option adb_src_rset '{FS=\"[|^]\"} \$0 ~/^\|\|([A-Za-z0-9_-]+\.){1,}[A-Za-z]+\^$/{print tolower(\$3)}'
        option adb_src_desc 'focus on polish ads'

Jeśli kogoś interesuje format tej zwrotki, to jest on [dokładnie rozpisany tutaj][5]. Warto jednak
wiedzieć, że ten powyższy blok jest w stanie przetworzyć wszystkie listy, które są importowane w
przeglądarkowym adblock'u. Jedyne co, to adres listy trzeba dostosować w `adb_src` , no i
ewentualnie opis w `adb_src_desc` .

Po zaimportowaniu tej listy, w logu systemowym możemy doszukać się takiego komunikatu:

    info : => processing source 'polish'
    info :    no online timestamp
    info :    source download finished (45 entries)
    info :    domain merging finished

Tak, z tej obszernej polskiej listy adblock'a, zostało wyciągniętych jedynie 45 adserwerów. Nie
powinno to dziwić, bo pozostała część listy skupia się na blokowaniu określonych elementów danych
witryn internetowych, z reguły `div` z odpowiednimi atrybutami `id`/`class` .

## Test adblock'a na routerze

Sprawdźmy czy ten adblock cośmy go zaimplementowali na routerze w ogóle działa. Odpalmy zatem goły
profili jakiejś przeglądarki, np. Firefox, i odwiedźmy jakieś większe serwisy internetowe, np. onet,
wp czy interia. Zanim jednak je odwiedzimy, zalogujmy się na router i wyłączmy adblock'a:

    # /etc/init.d/adblock stop

Odwiedźmy teraz jakiś serwis i po chwili włączmy nasz filtr. Poniżej jest fotka pokazująca różnice w
wyglądzie przykładowej strony:

![](/img/2016/09/1.adblock-reklamy-wp-router-openwrt.png#huge)

Jak widzimy, różnica jest dość spora. Na onecie efekt jest podobny. Z kolei na interii są przecieki.
Wygląda na to, że filtr nie uwzględnia pewnych domen i przydałoby się je dopisać ręcznie. Po chwili
udało mi się wyselekcjonować adres `hub.com.pl` . Tę domenę trzeba dopisać do pliku
`/etc/adblock/adblock.blacklist` . Następnie w pliku `/etc/config/adblock` włączamy tę listę:

    ...
    config source 'blacklist'
            option enabled '1'
    ...

Po zresetowaniu adblock'a, reklamy powinny zniknąć, poniżej porównanie:

![](/img/2016/09/2-adblock-reklamy-interia-router-openwrt.png#huge)

Podobnie trzeba postąpić ze wszystkimi reklamami, które nie są blokowane przez domyślne filtry lub
też poszukać list, które te domeny uwzględniają. Akurat w tym przypadku, ta problematyczna domena
znajduje się na jeden z list adblock'a, tj. sysctl. Zatem nie musimy jej dodawać ręcznie, tylko
wystarczy włączyć te listę w ustawieniach.

Im więcej list dodamy, tym nasz filtr sprawniej będzie blokował reklamy. Problem w tym, że wraz z
ilością list rośnie dość znacznie ilość potrzebnej pamięci RAM. W moim przypadku przekroczyłem
granicę 130 MiB. I jeśli faktycznie chcemy mieć sporą ilość wpisów w filtrze, to potrzebny będzie
nam router z minimum 128 MiB pamięci RAM.

Problem pojawia się także w przypadku filtrowania adresów w oparciu o dużą listę domen. W przypadku
wykonywania dużej ilości zapytań DNS, proces `dnsmasq` może zjadać praktycznie cały procesor.
Poniżej fotka z mojego routera Archer C2600:

![](/img/2016/09/3.adblock-openwrt-router-utylizacja-procesor-archer-c2600.png#huge)

Im bardziej rozbudowana jest lista adresów, tym w większym stopniu jest utylizowany procesor, a wiec
i mocniej odczujemy spadek wydajności przesyłu danych przez router.

## Regularna aktualizacja list adblock'a

Listy adserwerów są aktualizowane regularnie, zwykle co dzień lub co tydzień. Dlatego też na
routerze musimy zadbać o to, by te listy były również aktualizowane na bieżąco. Ten proces możemy
sobie zautomatyzować za pomocą narzędzia `cron` . Musimy tylko dodać stosowną akcję do tablicy
cron'a. Logujemy się zatem na router i w terminalu wydajemy polecenie `crontab -e` i dopisujemy
poniższą linijkę:

    30 04 * * *    /etc/init.d/adblock start

Od tej chwili isty będą pobierane i aktualizowane automatycznie o godzinie 4:30.

## Adblock i szyfrowany DNS

Jako, że nasz mechanizm blokujący reklamy opiera się o przekierowywanie zapytań DNS, to w pewnych
sytuacjach może on powodować problemy z konfiguracją innych usług na routerze WiFi. Na szczęście
[szyfrowanie zapytań DNS za pomocą dnscrypt-proxy][6] nie koliduje zupełnie z adblock'iem i obie te
usługi mogą działać równolegle.


[1]: https://adblockplus.org/
[2]: https://www.ublock.org/
[3]: http://www.tp-link.com.pl/products/details/Archer-C2600.html
[4]: http://www.adblocklist.org/
[5]: https://github.com/openwrt/packages/tree/master/net/adblock/files
[6]: /post/konfiguracja-dnscrypt-proxy-w-openwrt/
