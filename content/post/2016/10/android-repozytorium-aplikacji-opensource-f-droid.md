---
author: Morfik
categories:
- Android
date: "2016-10-11T20:47:13Z"
date_gmt: 2016-10-11 18:47:13 +0200
published: true
status: publish
tags:
- tor
- smartfon
- lollipop
- f-droid
- aplikacje
title: 'Android: Repozytorium aplikacji OpenSource (F-Droid)'
---

Przeszukując sklep Google Play za nowymi aplikacjami, które mógłbym wgrać na swojego [Neffos'a
C5](/post/recenzja-smartfon-neffos-c5-od-tp-link/), zawsze staram się zwracać uwagę
co tak naprawdę zamierzam zainstalować. Nie chodzi tutaj tylko o poleconą mi przez kogoś aplikację,
a konkretnie jej nazwę, bo te mogą być przecież bardzo podobne i łatwo zainstalować nie tego app'ka,
którego powinniśmy. Android jest prawie jak windows, no może z tą różnicą, że jest udostępniany na
wolnych licencjach. Z racji swojej popularności musi być bardzo prosty w obsłudze, by nie generować
żadnych błędów i problemów wśród korzystających z niego użytkowników. Z doświadczenia wiem, że
prostota obsługi nie zawsze idzie w parze z bezpieczeństwem, a gdy mamy przed sobą tak popularny
system operacyjny jak Android, to już tylko krok dzieli nas od kompromitacji systemu przez wgranie
jakiejś trefnej aplikacji ze sklepu Google. Nie znam tych wszystkich programików, które są w nim
dostępne ale można zrobić lekki przesiew instalując jedynie aplikacje OpenSource. Przeszukiwanie
Google Play pod tym kątem nie jest zbyt wygodne, dlatego też ktoś postanowił uruchomić [projekt
F-Droid](https://f-droid.org/) zrzeszający wolne aplikacje, które możemy wgrać na swój telefon bez
większego problemu. Ten wpis będzie poświęcony właśnie aplikacji F-Droid.

<!--more-->
## Czym jest F-Droid

Użytkownicy linux'ów, w szczególności dystrybucji Debian, wiedzą jak ważne jest posiadanie z swoim
systemie aplikacji spod znaku Free & OpenSource. Tego typu podejście w dużej mierze zabezpiecza nas
przed podejrzanym oprogramowaniem. Chodzi o to, że za sprawą OpenSource jesteśmy w stanie przebadać
kod i wprowadzane w nim zmiany pod kątem szkodliwości dla systemu operacyjnego. Z kolei Free
sprawia, że rozwój aplikacji zwykle nie jest motywowany nadmierną chęcią zysku i wzbogacenia się.

W sklepie Google jest cała masa aplikacji, które mają dopisek "zawiera reklamy". Może i taki
programik jest nam udostępniany za free ale za to jesteśmy zmuszani do ciągłego oglądania reklam,
czy też taka aplikacja śledzi nasze poczynania i przesyła gdzieś w ten sposób zebrane dane. Reklamy
można co prawda [odfiltrować za sprawą
adblock'a](/post/blokowanie-reklam-adblock-na-domowym-routerze-wifi/) ale tylko pod
warunkiem, że mamy router z wgranym firmware OpenWRT. Niemniej jednak, w przypadku sporej części
aplikacji w tym sklepie nie wiemy praktycznie nic na temat kodu źródłowego.

Nie to bym w każdej aplikacji na Androidzie widział złowrogie oprogramowanie ale jeśli taki program
ma 100-500 milionów użytkowników i do tego nic nie wiadomo na temat mechaniki jego działania, to co
powstrzyma twórców takiej aplikacji przed podejmowaniem zachowań niezbyt korzystnych dla docelowej
grupy odbiorców? Aspekt władzy i pieniądza jeszcze tylko dodatkowo zaognia tę kwestię. Dlatego
właśnie powstał F-Droid.

F-Droid, to repozytorium aplikacji, mniej więcej takie jak Google Play, z tym, że znajdujące się w
nim programiki muszą być na wolnych licencjach, np GPL, Apache czy MIT. Każda taka pozycja na liście
jest bardzo przyzwoicie opisana. Wiemy, kto tworzy dany projekt, gdzie go znaleźć oraz też gdzie
znajdują się źródła tej konkretnej aplikacji:

![](/img/2016/10/1.f-droid-smartfon-android-info-aplikacja.png#medium)

Każdą aplikację można zainstalować sobie w Androidzie za pomocą paczki `.apk` z pominięciem
F-Droid'a. Problem w tym, że w ten sposób zainstalowany program nie będzie otrzymywał aktualizacji.
Ten mechanizm był/jest znany choćby z windowsów, gdzie po pewnym czasie mamy w systemie całą masę
nieaktualnego softu, co stwarza zagrożenie dla bezpieczeństwa. Jako, że F-Droid oferuje nam
repozytorium aplikacji, to każdy app'ek będzie podlegać procesowi aktualizacji, tak jak to ma
miejsce w Google Play, czy każdym linux'ie.

![](/img/2016/10/2.f-droid-smartfon-android-aktualizacja.png#big)

## Instalacja F-Droid (brak w Google Play)

Prawdopodobnie już wiecie, że aplikacji F-Droid nie ma w sklepie Google Play, a to z tego względu,
że narusza ona [umowę z
Google](https://play.google.com/about/developer-distribution-agreement.html). Chodzi o to, że
F-Droid umożliwia instalację w Androidzie aplikacji z pominięciem Google Play. Dlatego też, by móc
zainstalować na swoim smartfonie F-Droid, musimy odwiedzić [oficjalną stronę projektu i pobrać z
niej plik .apk](https://f-droid.org/).

F-Droid pochodzi za zewnętrznego źródła w stosunku do Google Play. Nie damy rady standardowo
zainstalować takiej aplikacji. Przed procesem instalacyjnym musimy włączyć w Androidzie możliwość
instalacji programów z niezaufanych źródeł. W przypadku, gdy ta opcja nie jest zaznaczona, to
zostaniemy o tym fakcie poinformowani podczas instalacji:

![](/img/2016/10/3.f-droid-smartfon-android-wlaczenie-nieznane-zrodla-aplikacji.png#huge)

Po włączeniu powyższej opcji, instalujemy F-Droid i uruchamiamy go:

![](/img/2016/10/4.f-droid-smartfon-android-instalacja.png#huge)

Czekamy chwilę, aż F-Droid zaktualizuje informacje, przechodzimy na zakładkę aktualizacje i
instalujemy update przeznaczony dla F-Droid.

![](/img/2016/10/5.f-droid-smartfon-android-aktualizacja.png#huge)

Po chwili okno F-Droid'a zostanie zamknięte. Czekamy moment i odpalamy tę aplikację skrótem na
pulpicie. Wchodzimy w menu (trzy kropki w prawym górnym rogu) i wybieramy Repozytoria. Tam z kolei
zaznaczamy również [repozytorium The Guardian
Project](https://en.wikipedia.org/wiki/The_Guardian_Project_(software)):

![](/img/2016/10/6.f-droid-smartfon-android-repozytoria.png#big)

Teraz możemy zacząć szukać interesujących nas aplikacji. Warto zaznaczyć, że by korzystać z F-Droid
nie musimy się nigdzie rejestrować, tak jak to ma miejsce, np. w Google Play.

## Instalowanie aplikacji z F-Droid

Aktualnie w repozytorium F-Droid znajduje się około 2100 aplikacji. Nie jest to przytłaczająca
liczba, przynajmniej nie w porównaniu z około 1,5 miliona aplikacji, które można znaleźć w Google
Play. Niemniej jednak, jest z czego wybierać. Pod zakładkami jest ulokowane rozwijane menu. W nim z
kolei mamy różne kategorie oprogramowania:

![](/img/2016/10/7.f-droid-smartfon-android-repozytoria-zawartosc.png#huge)

Wystarczy wybrać jedną z pozycji i zostaną nam wyświetlone informacje o danej aplikacji.

![](/img/2016/10/8.f-droid-smartfon-android-instalacja-aplikacja.png#big)

Oczywiście listę aplikacji możemy przeszukać, wystarczy kliknąć na lupę w prawym górnym rogu i
wpisać w formularzu kilka znaków:

![](/img/2016/10/9.f-droid-smartfon-android-szukanie-repozytorium.png#medium)

### Historia wersji aplikacji

Niewątpliwą zaletą F-Droid'a jest historia wersji aplikacji. Weźmy dla przykładu WiFi Analyzer.
Jeśli z jakiegoś powodu najnowsza wersja tego programu nam nie działa, to możemy zainstalować
starszą wersję. Nie zalecałbym tego typu działań ale skoro F-Droid ma wbudowany taki ficzer, to
wypadałoby o nim wspomnieć. Ostatnie trzy wersje są dostępne w informacjach o aplikacji. Gdy zajdzie
potrzeba dania downgrade do starszej wersji, wystarczy ją wskazać:

![](/img/2016/10/10.f-droid-smartfon-android-starsza-wersja-aplikacji.png#big)

### Aplikacje wymagające root

[W przypadku, gdy nasz Android ma
root'a,](/post/android-root-smartfona-neffos-c5-od-tp-link/) możemy odhaczyć w menu
opcję ukrywającą na liście aplikacji te pozycje, które wymagają do działania praw administracyjnych.
Generalnie nie zalecam instalacji tego typu aplikacji ale jako, że w tym przypadku mamy spełniony
jeden z warunków bezpieczeństwa (aplikacja musi być OpenSource), to nie widzę przeszkód by możliwość
instalacji takich programów w systemie włączyć.

![](/img/2016/10/11.f-droid-smartfon-android-aplikacje-wymagajace-root.png#medium)

Oczywiście samą aplikację przed instalacją wypadałoby jeszcze zweryfikować i obadać ale to już
raczej nie powinno sprawić problemów.

### Aplikacje promujące niewolne usługi

Przy przeszukiwaniu repozytorium F-Droid, natknąłem się na kilka ciekawych aplikacji. Niemniej
jednak, przy części z nich widnieje dość rzucające się w oczy ostrzeżenie:

![](/img/2016/10/12.f-droid-smartfon-android-ostrzezenie-aplikacja.png#medium)

Ta aplikacja jest jak najbardziej Free & OpenSource ale opiera się na usłudze YouTube. [Wszystkie
tego typu usługi są flagowane przez F-Droid](https://f-droid.org/wiki/page/Antifeatures) i oznaczane
mniej więcej na podobnej zasadzie.

### Automatyczne aktualizacje

Na początku wspomniałem o jednej z bardziej użytecznych właściwości F-Droid'a jaką jest możliwość
aktualizacji oprogramowania, które z jego pomocą zainstalujemy na swoim smartfonie. Ten proces może
być dla nas zupełnie niewidoczny, lub też możemy przejąć nad nim całkowitą kontrolę. Stosowne opcje
są w menu:

![](/img/2016/10/13.f-droid-smartfon-android-konfiguracja-aktualizacje.png#medium)

Jak widać powyżej, mamy tutaj mniej więcej standardowe ustawienia aktualizacji. Możemy wymusić
pobieranie nowych pakietów przez WiFi. Możemy także zainicjować pobieranie pakietów w tle, tak by
przy procesie aktualizacji stosowne pliki były już pobrane. Nawet częstotliwość sprawdzania nowych
wersji aplikacji jest do skonfigurowania.

## F-Droid i TOR

Ciekawą rzeczą jest również możliwość przepuszczenia ruchu generowanego za sprawą F-Droid przez sieć
TOR. Jedyna rzecz, która nam jest potrzebna, to stosowny klient TOR, a ten z kolei można pobrać
przez Google Play lub też z repozytorium F-Droid. Mowa oczywiście o
[ORBOT](https://www.torproject.org/docs/android.html.en). Mając skonfigurowanego ORBOT'a, możemy
zaznaczyć poniższą opcję w F-Droid:

![](/img/2016/10/14.f-droid-smartfon-android-orbot-tor.png#big)
