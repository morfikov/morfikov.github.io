---
author: Morfik
categories:
- Android
date:    2021-09-03 20:53:00 +0200
lastmod: 2021-09-03 20:53:00 +0200
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
GHissueID: 324
title: Optymalizacja procesu ładowania baterii via ACC w smartfonie z Androidem
---

[Odblokowanie bootloader'a][1] w smartfonie Xiaomi Redmi 9 (Galahad/Lancelot) mamy już z głowy.
Podobnie sprawa wygląda z [wgrywaniem na partycję recovery obrazu TWRP][2] i uzyskiwaniem praw
administratora systemu root za sprawą Magisk'a. Te dwa kluczowe procesy otworzyły nam drogę do
nieco bardziej zaawansowanych prac jeśli chodzi o konfigurację samego urządzenia. W tym artykule
nie będziemy się jeszcze bawić we wgrywanie alternatywnych ROM'ów na bazie AOSP/LineageOS ale za to
zainteresujemy się nieco bardziej procesem ładowania samej baterii w takim telefonie. Chodzi
generalnie o fakt przedłużenia żywotności baterii smartfona za sprawą limitowania maksymalnej
wartości, do jakiej można taką baterię podładować podłączając telefon czy to do portu USB
komputera, czy też do pełnowymiarowej ładowarki. Taki stopień konfiguracji można osiągnąć
zaprzęgając do pracy zaawansowany kontroler ładowania ([Advanced Charging Controller][3], ACC),
który manipuluje niskopoziomowymi ustawieniami kernela linux. W standardowym Androidzie tego typu
funkcjonalności nie uświadczymy, przez co bateria zużywa się parokrotnie szybciej niż powinna, co
przekłada się na wymianę urządzenia na nowe po niecałym roku czy dwóch jego użytkowania. Jeśli nie
uśmiecha nam się wydawać hajsu co roku na nowy telefon tylko dlatego, że nie można w nim wymienić
baterii, to powinniśmy rozważyć rozwiązanie jakie nam daje ACC.

<!--more-->
## Problematyczne baterie w urządzeniach mobilnych

Jeden problem z urządzeniami mobilnymi, takimi jak smartfony czy laptopy, od wielu lat pozostaje w
zasadzie niezmienny, tj. z biegiem czasu bateria takiego urządzenia ulega degradacji w skutek m.in.
ciągłych cykli ładowania/rozładowania. W przypadku laptopów można z tym nieuchronnym procesem
wałczyć, np. wyjmując z niego baterię, ale co w przypadku telefonów z Androidem, w których nie
dość, że takiego zabiegu się nie da przeprowadzić, to jeszcze samo urządzenie wymaga obecności
baterii, by w ogóle chciało się uruchomić (podłączenie pod ładowarkę nic nie da)?

Generalnie rzecz biorąc, to praktycznie każde urządzenie, które może być zasilane przy pomocy
baterii powinno oferować możliwość kontroli jej procesu ładowania. Nie wiedzieć czemu, producenci
takich urządzeń mobilnych nie chcą udostępniać końcowemu użytkownikowi opcji zarządzania procesem
ładowania baterii. Choć zapewne wszyscy wiemy, że takie zachowanie producentów sprzętu
elektronicznego podyktowane jest chęcią zysku, przez co urządzenia, które technicznie rzecz biorąc
są w pełni sprawne i mogłyby nam posłużyć jeszcze wiele lat stają się złomem, który trzeba wyrzucić
na śmietnik, a wszystko to z powodu braku kawałka dobrej baterii.

Jakiś już czas temu opisywałem na przykładzie mojego laptopa Lenovo Thinkpad T430 możliwość
[ustawienia progów ładowania jego baterii][4], by niepotrzebnie jej nie stresować i tym samym
wydłużyć jej żywotność dość znacznie. Tego typu zabieg można również przeprowadzić w części
smartfonów zaprzęgając do pracy wspomniany już zaawansowany kontroler ładowania (Advanced Charging
Controller, ACC). Problem z ACC jest jednak taki, że wymaga on praw administratora systemu root w
telefonie, by móc z powodzeniem wypełniać powierzone mu funkcje sprawowania kontroli nad procesem
ładowania baterii.

Jak zapewne sporo osób już wie, telefony z Androidem praktycznie nie oferują dostępu root ze
względów bezpieczeństwa, albo bardziej po ludzku, by chronić użytkownika telefonu przed jego
własną głupotą. Są jednak urządzenia, takie jak np. Xiaomi Redmi 9, w których bez zbytniego wysiłku
można uzyskać prawa administracyjne i jeśli posiadamy takie urządzenie (albo też inne, na którym
dostęp root udało nam się uzyskać), to możemy spróbować poskromić proces ładowania baterii w
naszym telefonie i narzucić mu ograniczenia, z którymi zapewne do tej pory się jeszcze nie spotkał.

## ACC, ACCA i Magisk

Na necie można spotkać się z dwoma projektami: [ACC][3] i [ACCA][7]. Czym one się różnią i czy
musimy mieć zainstalowane obie te rzeczy, czy też wystarczy samo ACC lub ACCA?

ACC to jest właściwa aplikacja, którą trzeba zainstalować w smartfonie z Androidem. W interakcję z
tym oprogramowaniem wchodzi się przy pomocy wiersza poleceń, czyli potrzebna jest jakaś aplikacja
terminala. Dodatkowo, nasz system musi wspierać mechanizm uruchamiania skryptów init.d, by ACC był
się w stanie uruchomić automatycznie z systemem.

Z kolei ACCA, to graficzna nakładka na ACC, przy pomocy której możemy kontrolować zachowanie ACC.
Dodatkowo ta aplikacja może być uruchamiana w regularny sposób ze startem systemu (zwykły autostart
Androida), przez co żadnych dodatkowych kroków nie musimy przeprowadzać, by ACC nam działał.

Technicznie rzecz biorąc ACCA ułatwia cały proces konfigurowania i zarządzania kontrolerem ACC.
Jeśli nie przepadamy za aplikacjami konsolowym, to powinniśmy sobie zainstalować ACCA (obok ACC),
który jest dostępny standardowo w [repozytorium F-Droid][8] i to z niego konfigurować ACC.

Mając ukorzeniony system za sprawą Magisk'a, ACC możemy zainstalować w Androidzie wgrywając moduł
`Advanced Charging Controller (ACC)` . Dla pełnej funkcjonalności, możemy także doinstalować moduł
`Daily Job Scheduler (DJS)` , który ma na ceku uruchamianie poleceń m.in. przy starcie systemu.

![acc-acca-android-battery-controller-magisk-modules](/img/2021/09/005-acc-acca-android-battery-controller-magisk-modules.jpg#small)

## Jak działa zaawansowany kontroler ładowania ACC

Standardowe kontrolery ładowania działają na zasadzie monitorowania stanu napięcia i jeśli ten
osiągnie jakiś zdefiniowany poziom, to proces ładowania jest przerywany. Standardowo ten poziom
napięcia w smartfonach wynosi około 4,35V. Analogicznie, jeśli poziom napięcia spadnie poniżej tej
określonej wartości, to proces ładowania jest załączany ponownie.

Taki stan rzeczy dość mocno stresuje baterię, ze względu na fakt, że baterie litowo-jonowe (li-ion)
niezbyt kochają stany pełnego naładowania, a jeszcze gorzej znoszą ciągłe doładowywanie do 100%
będąc już blisko tego progu. Dlatego też nie powinno się zostawiać podłączonego telefonu na noc do
ładowarki.

Wielu użytkowników stara się korzystać z różnych aplikacji pokroju [AccuBattery][5], który jest nas
w stanie powiadomić o osiągnięciu określonego stopnia naładowania baterii. Gdy taką notyfikację
usłyszymy, wiemy, że jest to czas by odłączyć naszego smartfona od ładowarki, co efektywnie
przerywa proces ładowania. Problem z tego typu aplikacjami jest taki, że wymagają od nas świadomej
interakcji i pilnowania procesu ładowania, co nie zawsze jest możliwe, a poza tym, mamy raczej
ważniejsze sprawy na głowie. Właśnie po to powstał projekt ACC, który ma na celu przejęcie pełnej
kontroli nad procesem ładowania baterii w telefonie i zautomatyzowanie go w pełni, tak by końcowy
użytkownik nie musiał sobie tym procesem głowy zaprzątać.

### Widełki pojemnościowe

Ten zaawansowany kontroler ładowania działa na kilku płaszczyznach. Pierwszą z nich jest ustawienie
dwóch progów, tj. niskiego i wysokiego, dla przykładu weźmy sobie 70% i 75%. Gdy po całym dniu
pracy telefonu podłączymy go pod ładowarkę, to ACC podładuje go do 75% i odetnie go od prądu. Zatem
niby telefon jest podłączony pod ładowarkę ale po osiągnięciu tych 75%, proces ładowania zostanie
przerwany. By uniknąć ciągłego załączania się procesu ładowania (w sytuacji, gdy stopień
naładowania baterii spadnie np. do 74%), wprowadzono ten drugi próg, który ma za zadanie załączyć
proces ładowania dopiero w momencie, gdy stopień naładowania baterii spadnie poniżej 70%. Gdy ten
próg zostanie osiągnięty, to kontroler załączy proces ładowania i znów podładuje baterię do 75%.

![acc-acca-android-battery-controller-capacity](/img/2021/09/001-acc-acca-android-battery-controller-capacity.jpg#small)

Wyżej w pierwszej kolumnie mamy także wartość określająca przy jakim stopniu naładowania baterii,
ACC powinien odciąć zasilanie, tj. wyłączyć telefon. Powinniśmy unikać sytuacji, w których telefon
rozładowuje nam się do 0%, a w szczególności powinniśmy unikać sytuacji, w których tak rozładowany
telefon pozostaje przez dłuższy czas. Dlatego dobrze jest określić pewien minimalny próg poniżej
którego ACC odetnie zasilanie.

W zasadzie taka logika procesu ładowania wystarczy ogromnej większości użytkowników smartfonów z
Androidem ale to nie jest wszystko na co stać ACC,

### Widełki temperaturowe

Kolejny aspekt procesu ładowania baterii, który możemy kontrolować za sprawą kontrolera ACC,
dotyczy temperatury baterii. Podobnie jak w przypadku pełnego stanu naładowania, baterie li-ion
niezbyt lubią gorące klimaty. Niemniej jednak, podczas procesu ładowania (zwłaszcza tego szybkiego)
wydziela się sporo ciepła. Im cieplejsza staje się bateria, tym gorzej dla jej żywotności.
Technicznie rzecz biorąc, [nie powinno się wychodzić poza próg 30°C][6], a szybkie ładowanie jest w
stanie rozgrzać telefon nawet i powyżej 45°C, co efektywnie degraduje pojemność baterii, gdy taka
temperatura utrzymuje się przez dłuższy okres czasu.

Jako ciekawostkę można dodać, że gdyby utrzymywać smartfon w temperaturze 40°C, to po roku takiego
użytkowania, pojemność baterii spadłaby do około 60% wartości nominalnej. Dlatego też temperatura
baterii czy to podczas standardowej pracy telefonu, czy podczas procesu ładowania powinna być
możliwe niska, najlepiej poniżej tych wspomnianych wyżej 30°C.

![acc-acca-android-battery-controller-temperature](/img/2021/09/002-acc-acca-android-battery-controller-temperature.jpg#small)

Powinniśmy zatem unikać szybkiego ładowania oraz nie powinniśmy korzystać z telefonu, gdy ten jest
podłączony pod ładowarkę. Niemniej jednak, w sytuacji, gdy nasz telefon zacznie się nagrzewać i
bateria osiągnie zdefiniowany wyżej próg temperatury, to ACC przerwie proces ładowania na pewien
określony czas (w tym przypadku będzie to czas 90 sekund) lub też, gdy temperatura spadnie poniżej
zdefiniowanej wartości (tutaj 25°C). W jednym lub drugim przypadku, kontroler rozpocznie ponownie
proces ładowania baterii i tak w kółko. Oczywiście taki stan rzeczy może dość znacznie wydłużyć
proces ładowania, zwłaszcza w miesiącach letnich.

### Widełki prądowo-napięciowe

Kolejna kwestia dotyczy napięcia na baterii oraz natężenia prądu w procesie ładowania baterii. Te
dwie rzeczy również możemy kontrolować przy pomocy ACC. Jakie zatem powinno być napięcie na baterii
i jakim prądem powinniśmy ładować telefon?

Standardowo baterie w telefonach mają około 4,35V w stanie pełnego naładowania. Czy to dużo czy
mało? Jeśli sobie popatrzymy po ogniwach li-ion 18650, to takie ogniwa mają z reguły napięcie
nominalne 3,6V/3,7V i można je podładować zwykle do 4,2V. Te baterie w telefonach mają standardowo
napięcie nominalne 3,7V ale w pełni naładowane mają już 4,35V, zatem więcej o te 0,15V. Ta różnica
nie wydaje się być duża ale to tylko pozory. Te 0,15V daje pojemność około +20% w stosunku do
baterii z napięciem 4,2V. Zatem z jednej strony mamy dość spory wzrost pojemności baterii w
telefonie, co cieszy. Niemniej jednak, każdy wzrost napięcia na baterii o 0,1V (powyżej tych 4,2V)
powoduje utratę żywotności takiej baterii o połowę, co już niezbyt cieszy.

O ile sporadyczne podładowanie smartfona do pełna czasami może nam uratować skórę (np. jak się
zgubimy gdzieś na jakimś zadupiu), to jeśli faktycznie zależy nam na żywotności baterii, to nie
powinniśmy jej do tych 100% ładować w codziennym użytkowaniu urządzenia. Dlaczego zatem producenci
smartfonów nas o tym fakcie nie informują? Zapewne wszyscy znamy odpowiedź na to pytanie, choć też
sami jesteśmy sobie winni. Jeśli mielibyśmy do wyboru dwóch producentów telefonów, z których każdy
ma w swoim urządzeniu baterię 10000 mAh, ale jeden z nich ogranicza sprzętowo możliwość ładowania
baterii widełkami 20%-80%, czyli wykorzystywane  jest 60% z tych 10000 mAh, co daje 6000 mAh, to
telefon którego producenta byśmy kupili? I dlatego właśnie te baterie się tak ostro przekręca pod
względem napięciowym, by tylko na papierku wyglądało, że ma trochę większą pojemność, a żywotnością
takich baterii nikt sobie głowy nie zawraca, przecie klient przyjdzie za rok i tak kupi nowe
urządzenie.

Technicznie rzecz biorąc, powinniśmy unikać napięcia na baterii większego od 4.05V, co przekłada
się na około 70-75% stanu naładowania baterii. Wciąż jednak, bateria w tym Xiaomi Redmi 9 ma
pojemność około 5000 mAh, więc nie jest znowu najgorzej, nawet przy tych 75%. Lepiej jest też
kupić smartfon z pojemniejszą baterią i ładować ją do tych 75%, niż zakupić urządzenie z mniejszą
baterią i utylizować ją w pełni.

Jeśli zaś chodzi o sam prąd ładowania, którym powinniśmy ładować telefon, to tutaj sprawa zdaje się
być również oczywista, tj. stosuje się zasadę 1C, gdzie C oznacza pojemność baterii ale wyrażona w
mA. Zatem prąd ładowania będzie się różnił, w zależności od tego jakiej pojemności baterię
posiadamy w smartfonie. Tutaj jest bateria około 5000 mAh, co dawałoby 5000 mA lub 5A. Nie oznacza
to, że powinniśmy tyle prądu w tę baterię wpuszczać. By przedłużyć żywotność baterii, z tego 1C
powinniśmy zjechać do około 0,3C-0,5C, co efektywnie dawałoby nam 1,5A-2,5A. Generalnie, to im
mniejszy będzie prąd ładowania, tym mniej będziemy stresować baterię ale nie powinniśmy schodzić
poniżej 0,1C, co w tym przypadku dawałoby 0,5A (tyle co port USB2). Jeśli nie potrzebujemy
natychmiast podładować telefonu, to trzymajmy się tych niższych wartości natężenia prądu.

Biorąc pod uwagę te powyższe informacje, napięcie do którego ACC ma podładować mój telefon
ustawiłem na 4,1V, zaś prąd ładowania na maksymalnie 1A:

![acc-acca-android-battery-controller-voltage-current](/img/2021/09/003-acc-acca-android-battery-controller-voltage-current.jpg#small)

### Ładowanie z cyklami chłodzenia

ACC posiada dość ciekawy algorytm samego ładowania baterii, tj. ładowanie z cyklami chłodzenia. Ten
mechanizm jest załączany od pewnego stopnia naładowania baterii, np. 50% czy 60%. Następnie bateria
ładowana jest przez pewien określony przedział czasu, np. 50 sekund. Po tym okresie, ładowanie jest
przerywane na jakiś czas, np. 10 sekund i w takich właśnie cyklach ten proces ładowania baterii
przebiega.

![acc-acca-android-battery-controller-cool-down-charging](/img/2021/09/004-acc-acca-android-battery-controller-cool-down-charging.jpg#small)

Takie zachowanie jest w stanie nie dopuścić do zbytniego rozgrzania się baterii w procesie
ładowania, zwłaszcza szybkiego ale naturalnie cały proces będzie trwał sporo dłużej.

Minusem tego trybu ładowania jest zaś włączanie się ekranu za każdym razem gdy załączany jest
proces ładowania. Próbowałem jakoś temu zaradzić ale najwyraźniej potrzebne są dodatkowe zabiegi (w
tym custom ROM'y), by takie włączanie się wyświetlacza telefonu po podaniu zasilania być w stanie
skonfigurować, a Xiaomi Redmi 9 takiej funkcjonalności standardowo zwyczajnie nie oferuje. Niemniej
jednak, ten tryb ładowania z cyklami chłodzenia jest bardzo użyteczny, gdy mamy włączony ekran, np.
oglądamy YT, czy ogólnie operujemy na smartfonie podczas jego ładowania, choć powinniśmy unikać
takich sytuacji.

### Jednorazowe ładowanie z pominięciem restrykcji

Czasami zapewne przytrafi nam się sytuacja, w której trzeba będzie w pośpiechu się zebrać i wyjść z
domu. Co, gdy się okaże, że po zerknięciu na ekran smartfona, ten nam wskaże, że bateria jest na
wyczerpaniu? W takich momentach nie mamy za dużo czasu na zabawę telefonem i przekonfigurowanie go
pod kątem zmiany parametrów procesu ładowania. ACC przewiduje tego typu sytuacje i pozwala
użytkownikowi na aktywację prostej opcji, za sprawą której możemy przez jedno ładowanie pominąć
wszystkie zdefiniowane restrykcje i przeprowadzić proces ładowania baterii w telefonie w tradycyjny
sposób.

![acc-acca-android-battery-controller-avoid-restrictions](/img/2021/09/006-acc-acca-android-battery-controller-avoid-restrictions.jpg#small)

## Problemy z kontrolerem ACC

O ile ACC to bardzo przydatna zabawka, o tyle trzeba sobie zdawać sprawę, że może ona być dość
niebezpieczna, zwłaszcza w przypadku smartfonów, które nie są kompatybilne z tego typu kontrolerami
ładowania. Spora część telefonów raczej powinna z ACC działać bez problemu, niemniej jednak jeśli
chodzi o moje urządzenia z Androidem, to ten Xiaomi Redmi 9 jest pierwszym, który jest w stanie
współpracować z ACC bez większego problemu. Moje poprzednie telefony miały z tym niesamowity
problem.

Nawet gdy nasze urządzenie zdaje się działać poprawnie z ACC, to mogą wystąpić drobne problemy przy
jego użytkowaniu, wliczając w to też uszkodzenie telefonu. Na stronie projektu widnieje takie
ostrzeżenie:

> Some devices, notably Xiaomi phones, have a buggy PMIC (Power Management Integrated Circuit) that
> can be triggered by acc. The issue blocks charging. Ensure your battery does not discharge too
> low. Using acc's auto shutdown feature is highly recommended.

Zatem najwyraźniej pewne urządzenia, [w tym telefony Xiaomi][9], mają jakieś błędy w układzie
zarządzania zasilaniem PMIC, które ACC może aktywować, czego efektem będzie blokada możliwości
zainicjowania procesu ładowania baterii w telefonie lub też dość mocne samorozładowanie się baterii.
Takie rozładowanie się telefonu bez żadnej kontroli może z kolei doprowadzić do śmierci baterii
(system jej już nie wykryje) i w efekcie do uszkodzenia telefonu.

By ochronić się przed tym wyżej opisanym problemem, ACC wprowadziło zabezpieczenie (w opcjach
widełek pojemnościowych), gdzie mamy możliwość skonfigurowania wyłączenia się telefonu, np. gdy
stopień naładowania baterii spadnie poniżej 5%, co powinno uniemożliwić wytransferowanie z niej
ładunku poniżej progu wykrywalności przez elektronikę.

Jeśli po zainstalowaniu/skonfigurowaniu ACC obserwujemy problemy z ładowaniem telefonu lub zbyt
szybką utratą pojemności baterii, to powinniśmy ACC odinstalować, by uniknąć ewentualnego
uszkodzenia urządzenia. Można też spróbować uruchomić telefon w trybie bootloader'a (np. via `adb
reboot bootloader` ), by zresetować układ zarządzania zasilaniem PMIC.

Czasami też niektóre rzeczy oferowane przez ACC nie są wspierane przez kernel w danym urządzeniu,
np. limitowanie natężenia prądu czy ładowanie do określonej wartości napięcia. Posiadanie uprawnień
root za bardzo nam tutaj nie pomoże.

W przypadku limitowania wartości napięcia do jakiego bateria może być naładowana, trzeba też mieć
na uwadze, że na niektórych urządzeniach może ulec zmianie stan naładowania takiej baterii (odchyły
od stanu faktycznego). Niemniej jednak, system zarządzania baterią samoczynnie się nieustannie 
kalibruje i nie ma się co przejmować tutaj, bo odtworzenie domyślnych limitów napięcia naprawi ten
problem. Jeśli zaś chodzi o limity prądowe, to ten ficzer jest z reguły wspierany przez większość
urządzeń i nawet ten Xiaomi Redmi 9 bez problemu adaptuje się do ustawionych wartości w czasie
rzeczywistym bez potrzeby restartu ACC -- wystarczy podłączyć jakiś miernik USB i obserwować czy
zmiany wartości natężenia prądu będą odzwierciedlane we wskazaniach miernika.

## Podsumowanie

Zaawansowany kontroler ACC i jego graficzna nakładka ACCA są w stanie zoptymalizować proces
ładowania baterii w smartfonie z Androidem i uczynić ten proces praktycznie transparentny z naszego
użytkowego punktu widzenia. Raz skonfigurowany kontroler nie wymaga dalszych zabiegów z naszej
strony, a jego działanie jest w pełni automatyczne. Niekoniecznie musimy korzystać ze wszystkich
funkcji ACC w tym samym czasie. Przeciętny użytkownik z powodzeniem zadowoli się jedynie widełkami
pojemnościowymi, a ci bardziej zaawansowani mają możliwość także skorzystać z widełek
temperaturowych czy też prądowo-napięciowych. Jedyny problem jaki pozostaje do rozwiązania, to
zapalanie się ekranu telefonu po podłączeniu zasilania, co najbardziej odczuwalne jest przy
ładowaniu z cyklami chłodzenia, gdzie ekran włącza się praktycznie co niecałą minutę, przez co w
stock'owych ROM'ach korzystanie z tego trybu ładowania mija się z celem.


[1]: /post/jak-wgrac-twrp-recovery-i-magisk-w-xiaomi-redmi-9-galahad-lancelot/
[2]: /post/jak-odblokowac-bootloader-w-xiaomi-redmi-9-galahad-lancelot/
[3]: https://github.com/VR-25/acc
[4]: /post/jak-ograniczyc-ladowanie-baterii-w-laptopie-thinkpad-t430/
[5]: https://play.google.com/store/apps/details?id=com.digibites.accubattery
[6]: https://batteryuniversity.com/article/bu-808-how-to-prolong-lithium-based-batteries
[7]: https://github.com/MatteCarra/AccA
[8]: https://f-droid.org/packages/mattecarra.accapp/
[9]: https://forum.xda-developers.com/t/rom-official-arrowos-11-0-android-11-0-vayu-bhima.4267263/page-14#post-85119331
