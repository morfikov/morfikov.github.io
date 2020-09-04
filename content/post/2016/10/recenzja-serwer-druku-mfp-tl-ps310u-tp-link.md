---
author: Morfik
categories:
- Hardware
date: "2016-10-01T18:09:35Z"
date_gmt: 2016-10-01 16:09:35 +0200
published: true
status: publish
tags:
- sieć
- drukarka
- recenzja
- tp-link
title: 'Recenzja: Serwer druku MFP TL-PS310U od TP-LINK'
---

Przeglądając ofertę na stronie TP-LINK, zaciekawiło mnie pewne urządzenie. Chodzi o [serwer druku
MFP i pamięci masowej
TL-PS310U](http://www.tp-link.com.pl/products/details/cat-5688_TL-PS310U.html). Niby słyszałem
kiedyś w przeszłości frazę "Print Server" ale nigdy zbytnio się nie zastanawiałem nad tym do czego
takie coś ma w istocie służyć. Po nazwie można jedynie przypuszczać, że chodzi o coś związanego z
sieciowymi urządzeniami drukującymi. Niemniej jednak, ten serwer wydruku, który dotarł do mnie nie
przypomina zbytnio drukarki sieciowej. To co to w takim razie jest?

<!--more-->
## Zawartość opakowania

Podobno obrazek jest warty więcej niż tysiąc słów, zatem zamiast się trudzić przy próbie opisu
TL-PS310U, lepiej jest go zwyczajnie ofotkować, by nie było żadnych niedomówień. Poniżej znajduje
się pudełko jak i jego zawartość:

![]({{< baseurl >}}/img/2016/09/1.TL-PS310U-print-server-serwer-druku-tp-link-pudelko.jpg#huge)

![]({{< baseurl >}}/img/2016/09/2.TL-PS310U-print-server-serwer-druku-tp-link-pudelko-zawartosc.jpg#huge)

Tak, to małe kwadratowe w środku, to właśnie nasz serwer druku. Jego dokładne wymiary to 56 x 52 x
23 mm. No i jak widać, sam zasilacz jest od niego sporo większy. Co ciekawe, ten adapter zasilania
ma 5V/2A (długość przewodu jakieś 1,5 metra), zatem niezły żarłok musi być z tego naszego małego
urządzenia:

![]({{< baseurl >}}/img/2016/09/3.TL-PS310U-print-server-serwer-druku-tp-link-zasilacz.jpg#huge)

Przyjrzyjmy się bliżej temu TL-PS310U:

![]({{< baseurl >}}/img/2016/09/4.TL-PS310U-print-server-serwer-druku-tp-link-rj45-diody-zasilanie.jpg#huge)

Z lewej strony widać gniazdo do podłączenia zasilacza. Mamy też port ethernet 100 mbit/s. Na tym
porcie są dwie diody: zielona z podpisem "100M" i pomarańczowa z podpisem "Link". Obie diody
zapalają się po podłączeniu przewodu ethernet. Świecą one ciągle, tylko ta pomarańczowa dioda miga,
gdy dane są wymieniane z urządzeniem. Nie wiem po co są dwie diody. Ta pomarańczowa powinna chyba
wystarczyć.

Wyżej widzimy także trzecią diodę, na prawo od portu. Ta dioda z kolei sygnalizuje status
podłączenia jakiegoś urządzenia do portu USB 2.0, który jest ulokowany po przeciwnej stronie
obudowy:

![]({{< baseurl >}}/img/2016/09/5.TL-PS310U-print-server-serwer-druku-tp-link-usb.jpg#huge)

Z kolei zaś trzeci z boków TL-PS310U skrywa czarny przycisk reset:

![]({{< baseurl >}}/img/2016/09/6.TL-PS310U-print-server-serwer-druku-tp-link-przycisk-reset.jpg#huge)

Poniżej jest jeszcze spód urządzenia zawierający min. informacje z domyślnym adresem IP, choć ten
mój serwer druku uzyskał adresację po DHCP. Na tej etykiecie znajduje się także adres MAC
urządzenia:

![]({{< baseurl >}}/img/2016/09/7.TL-PS310U-print-server-serwer-druku-tp-link-spod.jpg#huge)

Co ciekawe, na stronie TP-LINK'a jest informacja, że w opakowaniu ma znajdować się także przewód
ethernet ale najwyraźniej go zabrakło.

## Specyfikacja TL-PS310U

TL-PS310U można bardzo łatwo otworzyć. Jedyną przeszkodą na drodze jest ta naklejka, której
naruszenie oznacza utratę gwarancji. W środku obudowy znajduje się mały PCB:

![]({{< baseurl >}}/img/2016/09/8.TL-PS310U-print-server-serwer-druku-tp-link-pcb-EST-E2868M4-B.jpg#huge)

![]({{< baseurl >}}/img/2016/09/9.TL-PS310U-print-server-serwer-druku-tp-link-pcb.jpg#huge)

TL-PS310U ma wbudowany SoC [EST
E2868M4-B](https://www.google.pl/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiI-sGYsrXPAhUKFCwKHc_XA3UQFggeMAA&url=http%3A%2F%2Fjhongtech.com%2FDOWN%2FDs-28xx-12-E.pdf&usg=AFQjCNF_mRUarGHtTlTHmmoY7ajXIJMbgQ&sig2=37tFHz_6NhdhLwxnAxCnlQ),
który jest w stanie obsługiwać do 4 urządzeń USB. Problem tylko w tym, że jak widzieliśmy wyżej na
fotkach, TL-PS310U ma tylko jeden port USB. By mieć możliwość podłączenia więcej niż jednego
urządzenia do tego serwera druku, potrzebny nam będzie HUB USB, a takiego w zestawie zabrakło. Ten
czip wspiera także drukowanie z wykorzystaniem protokołu LPD/LPR przez co mamy dużą szansę obsługi
tego urządzenia na stacjach linux'owych.

Zgodnie z informacjami, jakie znajdują się w tym datasheet, E2868M4-B jest nie tylko w stanie
obsługiwać drukarki ale także skanery, HUB'y USB, pamięć masową (USB storage) i czytniki kard SD.
Na opakowaniu zaś widnieje jeszcze informacja, że damy radę do tego Print Server'a podłączyć
głośniki i kamery oczywiście te na USB.

Próbowałem przeskanować TL-PS310U w poszukiwaniu otwartych portów. Znalazłem jedynie te poniższe:

    # nmap -p 1-65535 192.168.1.132
    ...
    Not shown: 65532 filtered ports
    PORT     STATE SERVICE
    80/tcp   open  http
    515/tcp  open  printer
    4660/tcp open  mosmig
    ...

O ile dwa pierwsze są w miarę logiczne, o tyle ten ostatni kompletnie nic mi nie mówi.

Doszukałem się również [informacji o kolejkowaniu
użytkowników](http://www.tp-link.com.pl/faq-222.html) podczas korzystania z oprogramowania
dostarczonego na płytce dołączonej do zestawu. Wygląda na to, że jeśli korzystamy w danej chwili z
tego softu, to żaden inny użytkownik w sieci przez ten czas nie może korzystać z TL-PS310U.
Oczywiście dla linux'a zabrakło wersji i nas ten problem nie dotyczy, bo będziemy korzystać z
protokołu LPD/LPR.

## Zarządzenie TL-PS310U

Jako, że TL-PS310U jest wyposażony jedynie w port ethernet i port USB do podłączania urządzeń, to
tym serwerem druku zarządzamy w zasadzie przez dedykowane oprogramowanie, które znajduje się na
płytce CD. My linux'iarze musimy sobie inaczej radzić, bo widać w XXI wieku ciągle brakuje wsparcia
dla używanego przez nas systemu operacyjnego. Niemniej jednak, ten Print Server dysponuje webowym
panelem administracyjnym, ubogim bo ubogim ale zawsze to lepsze niż nic.

Na etykiecie znajdującej się na obudowie wiedzieliśmy, że domyślny adres tego urządzenia to
`192.168.0.10` . W przypadku mojej sieci, po podłączeniu przewodu ethernet, TL-PS310U pobrał sobie
adresację z serwera DHCP mojego routera. Zatem faktyczny adres tego urządzenia trzeba odczytać z
lease DHCP routera i to nim się posłużyć przy próbie dostępu do panelu administracyjnego. Sam panel
wygląda mniej więcej tak:

![]({{< baseurl >}}/img/2016/09/10.TL-PS310U-print-server-serwer-druku-tp-link-panel-admina.png#huge)

W zasadzie, to ten panel jest głównie do odczytu. Możemy, co prawda, włączyć/wyłączyć konfigurację
via DHCP, zresetować ustawienia do fabrycznych, zaktualizować firmware do nowszej wersji czy też
zrestartować urządzenie ale nic poza tym. Podłączane do portów USB drukarki, skanery albo dyski są
zwracane w formie widocznej powyżej.

Jeśli chodzi zaś o zarządzenie samymi urządzeniami podpiętymi do TL-PS310U, to konfigurujemy je z
poziomu konkretnego systemu operacyjnego. Na linux'ach działają jedynie drukarki i nie damy rady
zamontować pendrive czy innych zewnętrznych dysków, nie wspominając już o tych wszystkich
pozostałych urządzeniach, które ten serwer druku jest w stanie obsłużyć.

## Serwer druku TL-PS310U pod linux'em

Do konfiguracji drukarki udostępnianej w sieci za pomocą TL-PS310U będziemy wykorzystywać narzędzie
CUPS. Ten proces jest podobny do opisywanej już przeze mnie metody [konfiguracji drukarki pod linux
udostępnianej przez router z
OpenWRT]({{< baseurl >}}/post/cups-konfiguracja-drukarki-pod-linuxem/). Różnica sprowadza się
jedynie do zdefiniowania innego protokołu i odpowiedniej jego konfiguracji.

Na stronie TP-LINK'a jest informacja [jak skonfigurować przykładową drukarkę na przykładzie
Ubuntu](http://www.tp-link.com/en/faq-352.html). Co prawda nie używam Ubuntu ale opis graficznych
aplikacji wykorzystanych w tamtym HOWTO może się przydać także użytkownikom dystrybucji Debian.
Poniższy opis sprowadzać się będzie do bezpośredniej konfiguracji CUPS. Ubuntu dodaje tylko
dodatkową warstwę (GUI) i przez nią CUPS jest zarządzany. My tego nie potrzebujemy i zrobimy użytek
z przeglądarki internetowej.

By mieć możliwość wejścia w interakcję z drukarkami musimy w systemie posiadać odpowiednie
oprogramowanie. Potrzebny nam będzie pakiet `cups` oraz `printer-driver-gutenprint` zawierający bazę
sterowników do całej masy drukarek. Po instalacji tych pakietów, logujemy się do panelu
administracyjnego CUPS'a, który znajduje się pod adresem `{{< baseurl >}}:631/` . Tam z kolei
dodajemy nową drukarkę sieciową z wykorzystaniem protokołu LPD/LPR:

![]({{< baseurl >}}/img/2016/09/11.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

Następnie konfigurujemy protokół LPD/LPR. Nasza drukarka sieciowa będzie miała przypisany adres IP.
Będzie to ten sam adres IP, którym dysponuje Print Server. Ten adres poprzedzamy `lpd://` . Musimy
także określić nazwę kolejki do której będą dodawane drukowane dokumenty. Cały adres może zatem
przyjąć następującą postać `lpd://192.168.1.132/lpr1` . Adres IP oczywiście trzeba sobie dostosować.

![]({{< baseurl >}}/img/2016/09/12.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

Teraz jeszcze opisujemy naszą drukarkę:

![]({{< baseurl >}}/img/2016/09/13.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

Wybieramy jej producenta oraz model:

![]({{< baseurl >}}/img/2016/09/14.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

![]({{< baseurl >}}/img/2016/09/15.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

Po chwili drukarka powinna zostać dodana:

![]({{< baseurl >}}/img/2016/09/16.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

I to w sumie cała robota. Drukarka powinna być widoczna w edytorach tekstu i przy jej pomocy
powinniśmy być w stanie bez problemu drukować dokumenty:

![]({{< baseurl >}}/img/2016/09/17.TL-PS310U-print-server-serwer-druku-tp-link-cups-drukarka.png#huge)

## Czy taki serwer druku jest potrzebny linux'iarzom

Czy linux'iarz powinien sobie zawracać głowę takim Print Server'em? Jeśli ma on tylko nam
udostępniać drukarkę w sieci bez potrzeby zaprzęgania do tego dedykowanego komputera, to raczej
można sobie darować ten TL-PS310U. Powodów jest kilka. Praktycznie każdy router WiFi dysponuje
jednym lub dwoma portami USB, które są w stanie udostępnić drukarkę w sieci. Nawet jeśli nasz router
dysponuje tylko jednym portem, to nic nie stoi na przeszkodzie, by do tego portu podpiąć HUB USB i
rozszerzyć nieco możliwości samego routera. Z tym, że tutaj już jest potrzebny alternatywny firmware
OpenWRT/LEDE.

Jeśli jednak posiadamy już router ale bez portów USB, to inwestowanie w taki serwer druku jest
raczej pozbawione sensu, oczywiście w przypadku sieci zrzeszającej hosty linux'owe. Chodzi
generalnie o cenę, która jest na poziomie ceny routerów WiFi wspieranych przez OpenWRT/LEDE. Zamiast
TL-PS310U można nabyć taki właśnie router i wgrać na niego alternatywne oprogramowanie. Przy tym nie
ma problemów z [przerobieniem takiego routera na serwer
druku]({{< baseurl >}}/post/drukarka-sieciowa-w-openwrt-serwer-wydruku/), który będzie w o wiele
lepszym stopniu nadawał się do sieci linux'owych.
