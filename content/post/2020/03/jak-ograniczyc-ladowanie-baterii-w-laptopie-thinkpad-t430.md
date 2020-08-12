---
author: Morfik
categories:
- Linux
date: "2020-03-13T18:30:00Z"
published: true
status: publish
tags:
- debian
- bateria/akumulator
- lenovo
- thinkpad
- t430
title: Jak ograniczyć ładowanie baterii w laptopie ThinkPad T430
---

Czy zastanawialiście się może dlaczego baterie w laptopach zużywają się mimo, że w pewnych
przypadkach nie są one praktycznie wykorzystywane? Rozważmy sobie ideę używania laptopa w roli
przeciętnego komputera stacjonarnego. W takiej sytuacji do laptopa ciągle jest podpięty przewód
zasilający, przez co bateria powinna być używana jedynie w momencie braku zasilania z sieci
energetycznej. Zatem niby pobieramy prąd z gniazdka ale bateria i tak się nam zużyje po pewnym
czasie. Niektórzy radzą, by wyciągnąć akumulator z laptopa i używać takiego komputera bez baterii.
Takie postępowanie ma w moim odczuciu jednak same wady i postanowiłem poszukać jakiegoś bardziej
cywilizowanego rozwiązania wzorowanego na [aplikacji Battery Charge Limit][9] spotykanego w
smartfonach z Androidem. Gdyby udało się ustalić limit ładowania akumulatora w moim ThinkPad T430
na max 40%, to wydłużyłoby dość znacznie żywotność jego baterii. Niekiedy oprogramowanie na windows
umożliwia tego typu funkcjonalność ale co w przypadku linux'a? Czy da radę pod linux powstrzymać
laptopa od degradowania baterii za sprawą ładowania jej ciągle do 100%?

<!--more-->
## Czynniki wpływające na żywotność baterii

Na żywotność akumulatorów opartych o technologię litowo-jonową (Li-ion) w naszych laptopach [ma
wpływ wiele czynników][5]. Jednym z tych, które możemy kontrolować jako użytkownicy komputerów, jest
przede wszystkim ilość cykli ładowania/rozładowania. Laptopy trochę różnią się w tej kwestii od
smartfonów, bo te drugie nie umożliwiają zwykle obejścia układu baterii i musi być ona obecna jeśli
zamierzamy korzystać z telefonu. W przypadku laptopów, użytkownik jest w stanie wyciągnąć
akumulator z obudowy i w dalszym ciągu używać takiego komputera, o ile go tylko podłączy do
gniazdka. Niemniej jednak, w dalszym ciągu laptop może być ładowany i rozładowany w pełni jeśli ma
tylko dość mobilnego właściciela. Każdy taki cykl pełnego ładowania degraduje baterię, która po
około 300-500 cyklach będzie się wykazywać dość sporym ubytkiem pojemności, co z reguły będzie
wiązać się z wymianą akumulatora, a dla nas z dodatkowymi kosztami.

Drugim czynnikiem, na który możemy mieć wpływ, to poziom naładowania baterii. Z reguły w laptopach
system będzie nam ładował akumulator do 100%. W przypadku takiego podejścia chodzi o
zmaksymalizowanie czasu pracy na baterii w momencie, gdy nie mamy dostępu do sieci energetycznej.
Niemniej jednak, odbywa się to kosztem żywotności baterii jako, że baterie litowo-jonowe preferują
bardziej stany pośrednie, niż stan pełnego naładowania. Generalnie to im pełniejszy jest Li-ion,
tym gorzej dla jego żywotności.

Ostatnim takim ważnym czynnikiem, na który mamy pośrednio wpływ, jest temperatura baterii
zmieniająca się w zależności od tego jak taki laptop jest wykorzystywany w danym momencie. Im
bardziej intensywnie jest utylizowany procesor lub/i dysk twardy, tym więcej ciepła się wydziela
wewnątrz obudowy komputera. Zdolności laptopa do odprowadzania tego ciepła na zewnątrz są
ograniczone i różne jego elementy mogą się dodatkowo nagrzewać. Jednym z tych elementów jest
właśnie akumulator. Zasada generalnie jest taka, że im wyższa temperatura baterii, tym mniejsza jej
żywotność. Powinniśmy mieć ten fakt na uwadze i nie przegrzewać zbytnio akumulatora. Dodatkowo,
jeśli baterię poddajemy procesowi ładowania, to jej temperatura może wzrosnąć jeszcze bardziej i o
tym też trzeba pamiętać. Warto sobie również zdawać sprawę z zależności jaka istnieje między
stopniem naładowania baterii i jej temperaturą. Im pełniejszy jest akumulator i ma przy tym wyższą
temperaturę, tym gorzej dla jego żywotności. Zatem ta sama temperatura ale przy niższych stanach
naładowania wpłynęła by w o wiele mniejszym stopniu na żywotność akumulatora.

## Laptop bez baterii

Część osób, do których również i ja się zaliczam, odeszła już od korzystania ze stacjonarnych
blaszaków, które mają zasilacze 500+W. W roli desktopowego komputera służy nam w zasadzie już tylko
laptop, u którego pobór prądu rzadko kiedy przekracza 15-20W. O ile w przypadku mobilnych laptopów
chcielibyśmy raczej aby ich bateria starczyła nam na jak najdłużej (zwłaszcza gdy nie mamy dostępu
do sieci energetycznej), to co w przypadku, gdy nasz laptop jest wykorzystywany w roli takiego semi
desktopa, który praktycznie ciągle jest na kablu? Niektórzy starają się wyciągać akumulator z
laptopa i chowają go zwykle do szafy czy podobnego pomieszczenia, tak by czekał on sobie na
nadejście lepszych dla niego czasów, w których będzie potrzebny. Takie zachowanie ma w zasadzie
same wady.

Po pierwsze, pozbawiamy się właściwości UPS, które bateria w laptopie zapewnia. Brak UPS niesie ze
sobą problemy w przypadku krótkich zaników prądu, przy których to wystąpieniu nasza maszyna będzie
się resetować/wyłączać, co z kolei może uszkodzić system plików partycji na dysku twardym. Nawet
jeśli system plików się nie posypie, to zwykle mamy ryzyko stracenia niezapisanych jeszcze
informacji, np. w otwartym edytorze tekstu.

Druga sprawa dotyczy przechowywania baterii, której się nie używa. Chodzi o to, że do
przechowywania baterii trzeba podejść dość rozważnie. W transporcie mocny nacisk kładzie się na to,
by taka bateria [była w określonym stanie naładowania nieprzekraczającym 30%][6]. Ma to na celu
minimalizować ryzyko zagrożenia pożarowego. Może i nam ono nie grozi aż tak bardzo jak w przypadku
przewożenia akumulatorów z jednego końca świata na drugi ale też [niższe stany naładowania potrafią
znacznie zwiększyć żywotności baterii podczas dłuższego przechowywania][7]. Technicznie rzecz
ujmując, to im bardziej jest naładowania bateria (dąży do 100%) i leży nieużywana, tym większe
permanentne straty pojemności będzie notować.

Trzeba też wziąć pod uwagę fakt, że taki akumulator będzie się rozładowywał sam z siebie. Wszystko
zależy od stopnia naładowania i temperatury w jakiej ta bateria jest przechowywana (może to być
nawet kilka-kilkanaście procent w skali miesiąca). Jeśli teraz zapomnimy o tej baterii, by ją od
czasu do czasu wyciągnąć z szafy w celu wykonania kilku pełnych cykli ładowania-rozładowania, to
jest niemal pewne, że ona przejdzie w [stan uśpienia za sprawą zbyt niskiego napięcia][8]. Jeśli
taką baterię któregoś dnia podłączymy do naszego laptopa, to on już nie będzie w stanie jej wykryć
i w zasadzie taki akumulator będzie jedynie do wyrzucenia (ewentualnie trzeba będzie go oddać do
serwisu by tam go przywrócili do życia).

Warto też zaznaczyć tutaj fakt, że pewne układy w komputerze (CMOS RAM i RTC) wymagają ciągłego
podtrzymywania zasilania. W ich przypadku mamy zwykle dedykowaną osobną [baterię CMOS][10], której
zadaniem jest podtrzymywanie napięcia w układach na wypadek braku zewnętrznego zasilania. W
przypadku laptopa, zewnętrzne zasilanie może dostarczyć jego akumulator lub zasilacz (w przypadku,
gdy maszyna działa bez akumulatora). Jeśli teraz wyciągniemy z laptopa akumulator i na noc
odłączymy również zasilacz od gniazdka, tak by komputer nie był non stop podłączony od sieci
energetycznej, to ta bateria CMOS będzie stopniowo rozładowywana, aż któregoś dnia padnie, co my
odczujemy resetem czasu i ustawień BIOS.

Jak widać, wyciąganie baterii z laptopa i przechowywanie jej w szafie w celu ochrony przed zużyciem
mija się trochę z celem, zwłaszcza jak się do tego zabierzemy w niezbyt przemyślany sposób.
Optymalnym rozwiązaniem za to byłoby utrzymywanie baterii w stanie naładowania około 30-40% bez
potrzeby wyciągania akumulatora z laptopa. W taki sposób, jeśli pojemność by spadła poniżej 30%, to
system by nam tę baterię automatycznie podładował, jednocześnie nie ładowałby baterii powyżej 40%,
a my mielibyśmy przy tym zapewniony również UPS oraz zasilanie dla zegara i BIOS'u.

## Progi ładowania baterii w laptopach Lenovo

Standardowo [laptopy Lenovo mają ustawione progi][4] 96% dla rozpoczęcia ładowania baterii i 100%
dla zatrzymania ładowania. Czyli naładujemy baterię laptopa do 100% i po osiągnięciu tego progu,
maszyna powinna przełączyć się w tryb zasilania z sieci. W przypadku, gdy z jakiegoś powodu stan
baterii będzie wskazywał na 96% (i niżej), np. pojemność baterii zmniejszy się w skutek samo jej
rozładowania z czasem, to w takim przypadku system nam załączy ładowanie baterii i zostanie ona
doładowana do 100%. Nie jest to jakiś szczególnie optymalny poziom ładowania baterii
litowo-jonowych ale mój poprzedni laptop zdawał się ładować baterię, gdy ta miała 99%, więc jeszcze
gorzej niż w tym ThinkPad T430.

Na szczęście niektórzy producenci laptopów (w tym Lenovo) umożliwiają użytkownikowi konfigurację
progów ładowania za sprawą plików `charge_start_threshold` i `charge_stop_threshold` obecnych w
katalogu `/sys/class/power_supply/BAT0/` (nazwy pewnie mogą się ździebko różnić). Dzięki tym dwóm
plikom można określić zarówno dolną jak i górną granicę stanu naładowania baterii. W taki oto
sposób jeśli mamy laptopa postawionego w roli desktopa i nie chcemy obawiać się krótkich zaników
energii, to możemy skonfigurować baterię określając jej progi 30%-40% w poniższy sposób:

	# echo 30 > /sys/class/power_supply/BAT0/charge_start_threshold
	# echo 40 > /sys/class/power_supply/BAT0/charge_stop_threshold

Dzięki takiemu stanu rzeczy, nie będziemy musieli wyjmować baterii z laptopa, by chronić ją przed
nadmiernym naładowaniem, a jednocześnie będziemy mieli zapewnioną funkcjonalność UPS na wypadek
krótkoterminowych zaników prądu.

Co ciekawe, ustawienie progów baterii zdaje się mieć trwały efekt, tj. po restarcie systemu, czy
też jego wyłączeniu, te wyżej ustawione wartości są zachowywane i odtwarzane, co niweluje potrzebę
zastosowania warstwy programowej, która by nam te progi ustawiała przy każdym starcie komputera.
Wszystko dlatego, że kontrolowaniem procesu ładowania zajmuje się [firmware wbudowanego
kontrolera][13] (EC), któremu my jedynie podajemy wartości (za sprawą plików w katalogu
`/sys/class/power_supply/BAT0/` ). Zatem też bateria w naszym laptopie nie będzie ładowana powyżej
limitów w przypadku podłączenia go do prądu w stanie wyłączonym.

Warto tutaj zaznaczyć, że te [ustawienia resetują się do domyślnych][12] za każdym razem jak tylko
wyciągniemy baterię z laptopa. Jeśli tego nie robimy zbyt często, to te wyżej ustawione wartości
będą honorowane ale jak tylko wyciągniemy baterię, to trzeba je na nowo ustawić, o czym należy
pamiętać.

Naturalnie nic nie stoi na przeszkodzie by doinstalować sobie jakieś narzędzia pokroju [TLP][1]/
[TLPUI][2] czy [tpacpi-bat][3], tak by umożliwiły one nam łatwiejszą konfigurację tego aspektu
pracy naszego linux'a.

## Test ustawień progów ładowania baterii

Wypadałoby przetestować te nowe ustawienia baterii. Odpalmy zatem `upower` i zobaczmy jak on widzi
baterię w naszym laptopie:

    # upower /org/freedesktop/UPower/devices/battery_BAT0  -d
    ...
    Device: /org/freedesktop/UPower/devices/battery_BAT0
    ...
      battery
        present:             yes
        rechargeable:        yes
        state:               charging
        warning-level:       none
        energy:              7.89 Wh
        energy-empty:        0 Wh
        energy-full:         19.94 Wh
        energy-full-design:  56.16 Wh
        energy-rate:         12.8887 W
        voltage:             10.915 V
        time to full:        56.1 minutes
        percentage:          39%
        capacity:            35.5057%
        technology:          lithium-ion
        icon-name:          'battery-good-charging-symbolic'
    ...

Mamy tutaj `energy-full-design` , który dość znacznie różni się od `energy-full` , zatem ta bateria
ma 35% nominalnej pojemności (widać ktoś dał jej mocno w kość). Mamy też `energy` i `percentage` ,
które określają aktualny stan naładowania akumulatora i jest to około 40% -- więcej nie będzie za
sprawą ustawionego limitu. Jak w późniejszym czasie stan naładowania baterii spadnie poniżej 30%,
to system znów podładuje baterię do tych 40%, co rozwiązuje w zasadzie wszystkie problemy. Jeśli
zaś chodzi o `state` , który ciągle pokazuje `charging` (ładowanie), to mamy tutaj do czynienia
jedynie [z błędem][11] i ta wartość nie do końca potrafi określić dokładny stan baterii po
zastosowaniu limitów.

Tak traktowana bateria powinna zwiększyć swoją żywotność parokrotnie (5-8 razy), a o ile dokładnie
to zależy jeszcze od paru czynników, o których była mowa wcześniej.


[1]: https://linrunner.de/en/tlp/tlp.html
[2]: https://github.com/d4nj1/TLPUI
[3]: https://github.com/teleshoes/tpacpi-bat
[4]: https://support.lenovo.com/ca/en/solutions/ht078208
[5]: https://batteryuniversity.com/learn/article/how_to_prolong_lithium_based_batteries
[6]: https://batteryuniversity.com/learn/article/how_to_transport_batteries
[7]: https://batteryuniversity.com/learn/article/how_to_store_batteries
[8]: https://batteryuniversity.com/learn/article/low_voltage_cut_off
[9]: https://forum.xda-developers.com/android/apps-games/root-battery-charge-limit-t3557002
[10]: https://en.wikipedia.org/wiki/Nonvolatile_BIOS_memory
[11]: https://linrunner.de/en/tlp/docs/tlp-faq.html#panel-applet
[12]: https://linrunner.de/en/tlp/docs/tlp-faq.html#no-tp-bat-func
[13]: https://linrunner.de/en/tlp/docs/tlp-faq.html#thresholds-powered-off
