---
author: Morfik
categories:
- Android
date:    2022-02-06 17:52:00 +0100
lastmod: 2022-02-06 17:52:00 +0100
published: true
status: publish
tags:
- xiaomi
- redmi-9
- root
- smartfon
- bateria
- akumulator
- acc
- f-droid
- aplikacje
- aosp
GHissueID: 588
title: Tryb bezczynności baterii (IDLE mode) w smartfonach z Androidem
---

Gdy zapytamy użytkowników smartfonów z Androidem o to, czy taki sprzęt jest w stanie pracować z
pominięciem układu baterii, zapewne wiele z tych osób odpowiedziałoby, że nie ma takiej opcji, bo
przecież w tych telefonach od lat baterii się już nie da wyciągnąć. Nawet w tych starszych
modelach, po wyjęciu baterii i podłączeniu ładowarki, system w takim urządzeniu nie chciał się
uruchomić. Okazuje się jednak, że nie trzeba wyjmować akumulatora ze smartfona, by to urządzenie
było w stanie działać z pominięciem układu baterii, tj. w tzw. trybie bezczynności baterii (battery
IDLE mode), coś na wzór rozwiązania stosowanego od dekad w laptopach. Niemniej jednak, takiej
funkcjonalności zwykle nie uświadczymy w stock'owych Androidach. Możemy się jednak o nią postarać
ale do tego celu niezbędne będzie nam uzyskanie praw administratora systemu root, choć lepiej wgrać
sobie na smartfon ROM na bazie AOSP/LineageOS. Bez ukorzenionego Androida nie mamy nawet co
podchodzić do implementacji tego mechanizmu. Dodatkowo będzie nam potrzebny [zaawansowany kontroler
ładowania baterii ACC][3] (w postaci modułu do Magisk'a) i opcjonalnie też [graficzna nakładka
ACCA][4] (do pobrania z F-Droid). W poniższym artykule postaramy się odpowiedzieć na pytanie czy
takie rozwiązanie na bazie IDLE mode w przypadku baterii w smartfonach z Androidem ma w ogóle sens
i czy może nam się ewentualnie do czegoś przydać.

<!--more-->
## Nieunikniona degradacja baterii za sprawą ciągłych cykli ładowania

Każde urządzenie elektroniczne, by było w stanie w ogóle funkcjonować, potrzebuje jakiejś formy
zasilania. Smartfony zwykle czerpią energię z wbudowanej w te urządzenia baterii, która z kolei
ładowana jest za sprawą dedykowanej ładowarki z gniazdka sieci elektrycznej. Zatem ładujemy baterię
w telefonie do pełna, odłączamy ładowarkę i przez dzień czy dwa korzystamy ze smartfona, aż poziom
baterii osiągnie niski stan naładowania, po czym podłączamy ładowarkę, ładujemy do full i tak w
kółko.

Dla osób, które nie chcą co roku wymieniać telefonu na nowy, taka cykliczna praca baterii jest w
zasadzie nie do zaakceptowania, bo bardzo szybko się nam to ogniwo zużyje. Jakiś czas temu
opisywałem [w jaki sposób można zaradzić zjawisku degradacji baterii w smartfonach z Androidem][1]
przy zastosowaniu zaawansowanego kontrolera ładowania ACC/ACCA. W tamtym artykule główny nacisk
został położony na ograniczenie wartości prądu ładowania, napięcia na baterii i jej temperatury,
których nie powinniśmy przekraczać, by bateria mogła się cieszyć zwiększoną żywotnością. Niemniej
jednak, takie rozwiązanie posiada jedną zasadniczą wadę, tj. proces ładowania jest bardzo powolny,
a z czasem i tak ogniwo za sprawą cykli straci swoje właściwości chemiczne.

## Czym jest tryb bezczynności baterii (battery IDLE mode)

ACC ma w swoim wachlarzu możliwości do zaoferowania tryb bezczynności baterii, czyli tzw. battery
IDLE mode. Ten tryb umożliwia zasilanie elektroniki telefonu bezpośrednio z sieci elektrycznej, tj.
przy pomocy dedykowanej ładowarki.

Z tego co udało mi się znaleźć na necie, [tryb IDLE działa na zasadzie oszukania ładowarki][2] i
zwrócenia jej nieprawdziwych danych odnośnie stanu baterii w telefonie. Chodzi o temperaturę samego
ogniwa, w oparciu o którą ładowarka podejmuje decyzję czy rozpocząć proces ładowania baterii. Jeśli
warunki są sprzyjające, tj. jeśli temperatura baterii nie jest za wysoka albo za niska, to [może
przykładowo zostać załączone szybkie ładowanie][6]. Z drugiej strony zaś, jeśli warunki nie są
optymalne, to proces ładowania może zostać przerwany zupełnie ze względu na fakt, że dalsza jego
kontynuacja (albo w ogóle rozpoczęcie) może grozić uszkodzeniem baterii albo wręcz wybuchem/pożarem
telefonu.

Zatem jeśli ładowarka uzna, że warunki nie nadają się do ładowania baterii, to po podłączeniu
smartfona do źródła zasilania, bateria nie będzie  ładowana. Jeśli zaś samo urządzenie będzie
wykazywało jakieś zapotrzebowanie na energię (np. będziemy z niego korzystać), to wtedy ta
potrzebna moc będzie czerpana z ładowarki i dostarczana bezpośrednio elektronice, co efektywnie
eliminuje baterię z procesu zasilania i sprowadza ją do roli UPS'a.

## Czy mój smartfon z Androidem wspiera tryb bezczynności baterii

Na pytanie czy dane urządzenie z Androidem wspiera tryb bezczynności baterii nie da się
jednoznacznie odpowiedzieć bez przeprowadzenia małego testu, który jest oferowany przez kontroler
ACC. By przeprowadzić test, potrzebny nam będzie jakiś emulator terminala, który zainstalujemy w
Androidzie, np. [Termux][5], oraz źródło zewnętrznego zasilania. Teoretycznie można by ten test
przeprowadzić z poziomu ADB (po podłączeniu smartfona do portu USB komputera) ale w moim przypadku
wyniki przy podłączeniu via USB różniły się trochę od tych, które uzyskałem, gdy smartfon był
podłączony pod dedykowaną ładowarkę. Dlatego też lepiej skorzystać z ładowarki.

Po uruchomieniu terminala, logujemy się na root'a i wpisujemy polecenie `acc` :

    galahad:/ $ su
    galahad:/ # acc

    Advanced Charging Controller v2022.2.3 (202202030)
    Copyright 2017-2022, VR25
    GPLv3+

    (i) accd is running (PID 894)

    1) Language
    2) All commands
    3) Documentation
    4) Start/restart daemon
    5) Stop daemon
    6) Export logs
    7) Charge once to #%
    8) Uninstall
    9) Edit config.txt
    a) Reset battery stats
    b) Test charging switches
    c) Check for update
    d) Flash zips
    e) Battery info
    f) Undo upgrade
    g) Exit

Zostało nam wyświetlone krótkie menu, z którego musimy wybrać `Test charging switches` . Po
wybraniu opcji `b` rozpocznie się test, który potrwa chwilę:

    #? b

    (!) Charger must be plugged to continue...
    (i) This may take a minute or so...

    (i) [/proc/mtk_battery_cmd/current_cmd 0::0 0::1 /proc/mtk_battery_cmd/en_power_path 1 0] works
    - battIdleMode=true

    (!) [battery/charge_control_limit 0 battery/charge_control_limit_max] won't work

    (i) [battery/input_suspend 0 1] works
    - battIdleMode=false

    (i) [battery/input_suspend 0 1 /proc/mtk_battery_cmd/en_power_path 1 1] works
    - battIdleMode=false

    (i) [battery/constant_charge_current_max 3000000 0] is blacklisted; won't be tested

    (i) [charger/constant_charge_current_max -1 0] works
    - battIdleMode=true

    (i) [main/constant_charge_current_max 300000 0] works
    - battIdleMode=false

    (i) [usb/current_max 500000 0] is blacklisted; won't be tested

    (i) [main/voltage_max 4450000 voltage_now] is blacklisted; won't be tested

    (i) Press any key to continue...

    Advanced Charging Controller v2022.2.3 (202202030)
    Copyright 2017-2022, VR25
    GPLv3+

W tym przypadku zostało przetestowanych kilka przełączników. Niektóre z nich są na czarnej liście
(prawdopodobnie niezbyt dobrze współgrają z kontrolerem ACC). Pozostałe przełączniki zdają się
działać ale tylko dwa z nich zwracają `battIdleMode=true` . W przypadku, gdyby ten test nie zwrócił
nam żadnego przełącznika z `battIdleMode=true` , to niestety jądro operacyjne systemu (albo sam
smartfon) nie wspiera trybu IDLE i nic tutaj raczej nie poradzimy, przynajmniej nie bez
rekompilacji kernela.

Jak widać wyżej mój smartfon Xiaomi Redmi 9 (lancelot/galahad) jak najbardziej wspiera tryb IDLE
baterii. Pytanie zatem, który z tych dwóch przełączników wybrać. Teoretycznie każdy z nich powinien
się nadać i ja określiłem ten mający w ścieżce `/proc/mtk_battery_cmd/` , jako że mój telefon ma
SoC od MTK. Jeśli, wybrany przełącznik nie będzie działał, to możemy spróbować następny, aż do
skutku.

Konfigurację przełącznika możemy sobie określić przez `acc` (edytując plik `config.txt` ) albo też
korzystając z graficznej nakładki ACCA, poniżej przykład:

![battery-idle-mode-android-phone-acc-acca-config](/img/2022/02/001.battery-idle-mode-android-phone-acc-acca-config.jpg#small)

Po wybraniu odpowiedniego przełącznika, trzeba także zaznaczyć opcję `Prioritize battery idle mode`
(widoczną wyżej na fotce).

## Jak sprawdzić czy tryb bezczynności baterii działa

Jeśli chcielibyśmy bardziej wizualnie zobaczyć jak (o ile w ogóle) tryb IDLE w przypadku baterii
naszego smartfona działa, to przydałoby się zaopatrzyć w zwykły miernik A/V na USB i podłączyć go
do ładowarki. Jeśli tryb bezczynności baterii działa, to przy zgaszonym ekranie telefonu na
wyświetlaczu miernika nie powinien zostać w zasadzie zarejestrowany pobór prądu (minimalne chwilowe
wskazania mogą się pojawić, tj. 0,01A-0,02A):

![battery-idle-mode-android-phone-acc-acca-power-test-offline](/img/2022/02/002.battery-idle-mode-android-phone-acc-acca-power-test-offline.jpg#huge)

Gdy włączony zostanie ekran telefonu, to wskazania naturalnie będą wyższe i odpowiadać będą one
faktycznemu zapotrzebowaniu urządzenia na energię elektryczną:

![battery-idle-mode-android-phone-acc-acca-power-test-online](/img/2022/02/003.battery-idle-mode-android-phone-acc-acca-power-test-online.jpg#huge)

Gdy ekran zgaśnie, to wskazanie aktualnego zużycia prądu z powrotem spadnie do 0A. Podobnie sprawa
będzie wyglądać podczas użytkowania telefonu, gdzie czasem tej energii potrzeba więcej, a czasem
mniej. Jeśli obserwujemy tego typu zależności, to znaczy, że tryb IDLE działa prawidłowo.

## Samorozładowanie się baterii

Trzeba jednak pamiętać, że ogniwa li-ion nie przechowują ładunku w sposób doskonały i tracą go z
czasem. Jest to niewielka wartość rzędu kilku procent miesięcznie, niemniej jednak zjawisko
samorozładowania występuje, zatem trzeba mieć na uwadze, że od czasu do czasu trzeba będzie ten
telefon podładować.

## Optymalne napięcie na baterii

Mając stale podłączony telefon pod ładowarkę, jego bateria nie będzie ładowana/rozładowywana w
czasie (samorozładowanie można tutaj pominąć). Trzeba zatem zatroszczyć się o to, by napięcie na
baterii było optymalnie dobrane, tak by niepotrzebnie jej nie stresować.

Generalnie utrzymywanie napięcia powyżej 4,05V przez dłuższy czas jest dla baterii szkodliwe.
Optimum, w które mierzymy, to 3,92V. Czasami może to być 80% stanu pełnego naładowania, czasami
mniej. Generalnie sam procent nas tutaj nie interesuje zbytnio, ino samo napięcie ma dla nas
ogromne znaczenie. W przypadku mojego Xiaomi Redmi 9, te 3,92V przelicza się na około 60%
pojemności.

![battery-idle-mode-android-phone-voltage-limit](/img/2022/02/004.battery-idle-mode-android-phone-voltage-limit.jpg#small)

Napięcie 3,92V jest tą górną granicą, do której możemy ogniwo litowo-jonowe naładować jeśli chcemy
dość znacznie ograniczyć jego degradację. Widełki, których powinniśmy się trzymać, to 40-60%.

Jeśli wiemy, że telefon będzie zwykle używany w domu, czy innym miejscu, w którym mamy dostęp do
sieci elektrycznej i praktycznie cały czas będzie to urządzenie podłączone do ładowarki, to możemy
z powodzeniem ustawić widełki 40-50%. Niemniej jednak, jeśli chcemy z takiego telefonu korzystać
również i poza domem, to lepiej jest się trzymać blisko progu 60%, np. ustawić widełki 59-60%.

### Dokupienie powerbank'u

Warto też pamiętać, że bez większego problemu możemy zaopatrzyć się w powerbank (albo w ładowarkę
do akumulatorów li-ion 18650 potrafiącą robić za powerbank) i w podbramkowych sytuacjach podładować
telefon z takiego źródła energii.

## Parę słów o starzeniu się ogniw li-ion z upływem czasu

Nawet jeśli spełnimy wszystkie możliwe wymagania, by bateria litowo-jonowa miała najdogodniejsze
warunki pracy, to i tak nie uda nam się zatrzymać procesu starzenia, który trawi to ogniwo od
chwili wyprodukowania go w fabryce. Chodzi o to, że elektrody takiego ogniwa li-ion ulegają z
czasem oksydacji (utlenianiu), w skutek czego rośnie jego rezystancja wewnętrzna. Większa
rezystancja akumulatora powoduje większe spadki napięcia przy takim samym zapotrzebowaniu na prąd.
Przy pewnym granicznym napięciu, elektronika smartfona odmawia współpracy i telefon nam się wyłącza,
domagając się by go podładować. W efekcie niby bateria będzie mieć dalej swoją pojemność ale nie
będziemy w stanie z takiego starego ogniwa już jej wyciągnąć. Nie można o tym fakcie zapominać.

## Podsumowanie

Posiadając odpowiedni ROM w naszym telefonie z Androidem możemy dość znacznie wydłużyć żywotność
akumulatorów litowo-jonowych za sprawą trybu bezczynności baterii, np. wykorzystując do tego celu
zaawansowany kontroler ładowania ACC. Trzeba jednak pamiętać, że nawet diamenty nie są wieczne, a
co dopiero ogniwa li-ion. Niemniej jednak, nawet przy sporej degradacji takiej baterii za sprawą
upływu czasu, smartfon może pozostać sprawny, bo do podstawowych jego funkcji aż tak dużo energii
nie jest potrzebne (od 0,2A do 0,4A). Utrzymując zatem stan naładowania baterii w okolicach 50-60%,
jesteśmy w stanie odwlec w czasie na dobre kilka (jeśli nie kilkanaście) lat wymianę urządzenia na
nowe.


[1]: /post/optymalizacja-procesu-ladowania-baterii-via-acc-w-smartfonie-z-androidem/
[2]: https://android.stackexchange.com/questions/223812/dont-charge-the-battery-but-use-connected-power-to-run-the-phone
[3]: https://github.com/VR-25/acc
[4]: https://github.com/MatteCarra/AccA
[5]: https://f-droid.org/en/packages/com.termux/
[6]: https://batteryuniversity.com/article/bu-410-charging-at-high-and-low-temperatures
