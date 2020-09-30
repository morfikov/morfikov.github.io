---
author: Morfik
categories:
- Linux
date: "2016-04-20T15:31:46Z"
date_gmt: 2016-04-20 13:31:46 +0200
published: true
status: publish
tags:
- lte
- ussd
- debian
- modem
- huawei
- e3372
title: Obsługa kodów USSD w modemach LTE
---

Każdy, kto ma lub miał prepaid'a, prędzej czy później musiał nauczyć się obsługi [kodów
USSD](https://pl.wikipedia.org/wiki/Unstructured_Supplementary_Service_Data). To za ich pomocą
jesteśmy w stanie sprawdzić stan konta czy też aktywować poszczególne usługi. Co się jednak stanie,
gdy taki prepaid zostanie umieszczony w modemie LTE? Teoretycznie modem powinien nam zapewnić
połączenie LTE ale to jest nieco inna technologia niż GSM czy UMTS, a to za ich pomocą mogą być
przesyłane zarówno kody USSD i SMS. Niby modemy LTE potrafią operować również na UMTS i GSM ale pod
linux'em przesyłanie kodów USSD może być nieco problematyczne. Jedynym oprogramowaniem będącym w
stanie operować na tych kodach był `modem-manager-gui` . Problem w tym, że zajmuje on praktycznie
cały modem dla siebie, co w pewnych sytuacjach może nie być pożądane. Zatem jakie alternatywy nam
pozostają? W jaki sposób operować na tych kodach USSD pod linux'em?

<!--more-->
## Interfejsy modemu

Modemy LTE mogą pracować w kilku trybach. Ja póki co spotkałem się z dwoma z nich: [RAS (demon PPP i
wvdial)](/post/wvdial-ppp-czyli-modem-lte-w-trybie-ras/) i [NDIS
(NCM)](/post/konfiguracja-modemu-lte-w-trybie-ndis-ncm/). Różnic między tymi
trybami jest kilka ale nas interesuje głównie jedna z nich. Chodzi o operowanie na interfejsach
`/dev/ttyUSB*` . W przypadku trybu RAS, jedno z tych urządzeń jest wykorzystywane cały czas.
Natomiast jeśli chodzi o tryb NDIS (NCM), to jeden interfejs jest wykorzystywany tylko przy
nawiązywaniu połączenia, po czym jest zwalniany. W efekcie jeśli mamy do dyspozycji dwa interfejsy,
to po nawiązaniu połączenia w trybie RAS zostanie nam wolny jeden z nich, a w przypadku NDIS (NCM)
będą do dyspozycji dwa. Ma to znaczenie, tylko gdy korzystamy z jakichś egzotycznych usług, np.
[gammu-smsd do odbierania i wysyłania
SMS'ów](/post/gammu-smsd-czyli-wysylanie-odbieranie-sms/). Każda z takich usług
zajmuje sobie jeden z interfejsów i logiczne jest, że jeśli mamy do dyspozycji tylko dwa urządzenia
w katalogu `/dev/` do obsługi modemu LTE, to ilość nasłuchujących demonów usług jest ograniczona do
dwóch lub jednego. Jeśli dwie usługi będą chciały korzystać z tego samego interfejsu, to tylko jedna
z nich będzie mogła to zrobić. Oczywiście nic nie stoi na przeszkodzie, by na chwilę dezaktywować
jedną usługę na czas korzystania z drugiej, choć nie jest to jakoś zbytnio wygodne.

W przypadku tego modemu, tj Huawei E3372s-153 w wersji NON-HiLink, mamy tryb NDIS (NCM) i przy
odpowiedniej konfiguracji możemy zestawić połączenie LTE i mieć do dyspozycji standardowo dwa
interfejsy `/dev/ttyUSB0` oraz `/dev/ttyUSB1` .

## USSD-GUI

Szukając na sieci jakiegoś przyjaznego narzędzia do obsługi kodów USSD, natrafiłem na
[ussd-gui](https://github.com/mhogomchungu/ussd-gui). Ten programik posiada w miarę prosty interfejs
QT i wygląda mniej więcej tak:

![](/img/2016/04/1.ussd-gui-kod.png#big)

W polu `Input` wpisujemy kod USSD i wciskamy przycisk `Send` . Po chwili powinna zostać nam zwrócona
odpowiedź z sieci, tak jak to widzimy wyżej. `ussd-gui` potrafi też odbierać wiadomości SMS ale nie
potrafi ich wysyłać.

![](/img/2016/04/3.wysylanie-sms-ussd-gui.png#big)

Niemniej jednak, [wysyłanie i obieranie SMS lepiej zostawić
wammu](/post/wysylanie-odbieranie-sms-w-wammu/). On chyba najlepiej ma opanowaną tę
kwestię.

Warto też wiedzieć, że `ussd-gui` można sobie lekko skonfigurować w pliku
`~/.config/ussd-gui/ussd-gui.conf` . Poniżej przykład konfiguracji:

    [General]
    autowaitInterval=2
    decodeType=0
    device=/dev/huawei-E3372-1
    gsm7Encoded=true
    history=*111*480*1#\n*105#\n*101#\n*107#\n*111#\n*121#\n*111*480*3#
    initCommand="AT+CMGF=1;^CURC=0;^USSDMODE=0"
    no_history=
    source=internal
    terminatorSequence=68
    timeout=15
    ussdInfo=*111# - RBM: zarzadzanie kontem\n*121# - RBM: wlasny numer\n*107# - RBM: stan internetu\n*101# - RBM: stan konta\n*111*480*1# - RBM: LTE FREE\n*111*480*3# - RBM: LTE FREE status\n

W [FAQ](https://github.com/mhogomchungu/ussd-gui/wiki/Frequently-Asked-Questions) zabrakło
wyjaśnienia opcji `autowaitInterval` i na dobrą sprawę nie mam pojęcia za co ona odpowiada.
Podobnie sprawa ma się w przypadku `gsm7Encoded` . W oparciu o parametry `history` i `no_history`
generowana jest lista wpisanych kodów USSD w oknie aplikacji. Opcja to `timeout` odpowiada za czas
przez jaki aplikacja oczekiwać będzie odpowiedzi na przesłany kod USSD. Standardowo ten czas jest
bardzo długi i waha się w granicach około 1-2 minut. Tu nie chodzi o to, że tyle czasu zajmuje
odpowiedź na żądanie, bo ona jest zwracana w kilka sekund ale z jakiegoś powodu szereg aplikacji
czeka ponad 1 minutę ze zwróceniem wyniku. W przypadku `ussd-gui` odpowiedź jest zwracana prawie
natychmiastowo. Do wyboru mamy dwa backend'y, które możemy podać w opcji `source` . Są to `libgammu`
oraz `internal` . Pierwszy z nich wykorzystuje bibliotekę `libgammu` , drugi zaś operuje na
poleceniach AT. Chodzi generalnie o to, że nie wszystkie urządzenia są wspierane w `libgammu` , w
efekcie czego nie działają za dobrze z tym backend'em. W `device` określamy interfejs modemu, z
którego ma korzystać `ussd-gui`. W `ussdInfo` zaś są przechowywane wszystkie ulubione kody USSD
wraz z ich opisami. Kody USSD jak i opisy można definiować bezpośrednio w oknie `ussd-gui` , co
wygląda mniej więcej tak:

![](/img/2016/04/2.hostoria-kodow-ussd-gui.png#big)

Mamy także dwie dość enigmatyczne opcje: `initCommand` oraz `terminatorSequence` . Obie z nich są
wykorzystywane, gdy w grę wchodzi backend `source=internal` . W `initCommand` podajemy sekwencję
poleceń AT, które mają zostać przesłane do modemu przed nawiązaniem z nim połączenia. Chodzi głównie
o ustawienie poprawnego trybu pracy modemu niezbędnego do prawidłowej pracy urządzenia. Z kolei
jeśli chodzi zaś o `terminatorSequence` to wartość tego parametru trzeba ustalić w oparciu o
zwracane przez modem komunikaty. Przykładowo, przy przesyłaniu kodu USSD do modemu możemy otrzymać
poniższy wynik:

    $ cat /dev/ttyUSB1
    AT+CUSD=1,"*121#",15
    OK

    +CUSD: 0,"Twoj nr: 600123456",68

Wartość `68` na końcu komunikatu zwrotnego, to właśnie jest nasz `terminatorSequence` i to tę
wartość musimy wpisać w pliku konfiguracyjnym. W przeciwnym razie mogą wystąpić problemy w
działaniu `ussd-gui` .

## Kody USSD w gammu

Jako, że `ussd-gui` wykorzystuje `gammu` , to możemy spróbować przesłać te kody również z wiersza
poleceń. Tak dla przykładu, by przesłać kod `*111#` , wpisujemy `gammu getussd '*111#'` . Poniżej
przykład:

![](/img/2016/04/4.gammu-kod-ussd.png#big)

Niestety czas oczekiwania na odpowiedź jest bardzo długi. Nie udało mi się także odpowiedzieć na
otrzymany komunikat i to pomimo faktu, że wyraźnie widnieje tam `Action needed` . Być może było to
za sprawą zbyt długiego oczekiwanie na zwrócenie komunikatu przez aplikację. Nie mam pojęcia jak
dostosować `timeout` w `gammu` , dlatego też nie uśmiecha mi się korzystanie bezpośrednio z niego.

## Przesyłanie kodów USSD przy pomocy poleceń AT

Ostatnia opcja, która nam zostaje, to przesyłanie kodów USSD bezpośrednio na interfejsy modemu przy
pomocy poleceń AT. Oczywiście, możemy to zrobić na dwa sposoby. Pierwszy z nich zakłada
wykorzystanie oprogramowania typu `cu` lub `minicom` . Drugi zaś przesyła kody przy pomocy `echo`
bezpośrednio na interfejsy modemu w katalogu `/dev/` .

Czas reakcji na kody w obu przypadkach jest bardzo szybki. Niemniej jednak, trzeba nauczyć się
poleceń AT i ręcznie je wpisywać za każdym razem do terminala, co jest bardzo niewygodne. Poniżej
przykład:

![](/img/2016/04/5.kod-ussd-cu-terminal.png#huge)

Sposób z `echo` jest nieco wygodniejszy, bo wykorzystywana jest historia shell'a. Jeśli mamy `zsh`
zamiast domyślnego `bash` , to operowanie na tych kodach może być naprawdę proste. Wymagane będą też
dwa okna terminala. W jednym z nich będą wpisywane polecenia, w drugi będą zwracane wyniki. Poniżej
przykład w oparciu o multiplekser `tmux` :

![](/img/2016/04/6.tmux-kod-ussd-echo-cat.png#huge)

W górnym okienku przy pomocy `echo` wpisaliśmy dwa polecenia:

    $ echo -e "AT+CMGF=1;^CURC=0;^USSDMODE=0\r" > /dev/huawei-E3372-0
    $ echo -e "AT+CUSD=1,\"*101#\",15\r" >/dev/huawei-E3372-0

Pierwsze z nich konfiguruje odpowiednio modem, drugie zaś wysyła faktyczny kod USSD. W tym przypadku
jest to `*101#` . Ważne jest by kod ująć w `" "` oraz by zakończyć polecenie za pomocą `\r` .
Odpowiedzi są zwracane w przeciągu 2-3 sekund.

W ten sposób również jesteśmy w stanie odpowiadać na komunikaty, które się pojawiają w dolnym oknie.
Dla przykładu, w górnym oknie prześlijmy kod `*111#` .

![](/img/2016/04/7.tmux-kod-ussd-echo-cat.png#huge)

Widzimy, że pojawiło się kilka opcji do wyboru. Wybierzmy `Kontro` w odpowiedzi wysyłając `1` :

![](/img/2016/04/8.tmux-kod-ussd-echo-cat.png#huge)

Teraz sprawdźmy stan konta również przesyłając `1` :

![](/img/2016/04/9.tmux-kod-ussd-echo-cat.png#huge)

Teraz możemy zakończyć pracę przesyłając `0` :

![](/img/2016/04/10.tmux-kod-ussd-echo-cat.png#huge)

W ten sposób przy wykorzystaniu historii i możliwości shell'a ZSH oraz poleceń `echo` i `cat` możemy
komunikować się z siecią przesyłając jej kody USSD.
