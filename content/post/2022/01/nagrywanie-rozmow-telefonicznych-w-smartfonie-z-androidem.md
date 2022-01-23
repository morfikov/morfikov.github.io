---
author: Morfik
categories:
- Android
date:    2022-01-23 18:11:00 +0100
lastmod: 2022-01-23 18:11:00 +0100
published: true
status: publish
tags:
- audio
- smartfon
- xiaomi
- redmi-9
- aosp
- root
- aplikacje
- f-droid
- magisk
GHissueID: 584
title: Nagrywanie rozmów telefonicznych w smartfonie z Androidem
---

Właściciele smartfonów mają zwykle podpisane umowy ze swoimi operatorami GSM na świadczenie usług
telefonicznych. Pakiety rozmów oferowane przez tych operatorów już od dłuższego czasu są
nielimitowane, co z kolei zachęca abonentów do głosowego komunikowania się przy pomocy tych małych
lecz przy tym bardzo zaawansowanych technologicznie komputerów. Nie ma obecnie chyba sprawy, której
nie można załatwić telefonicznie, a że coraz dłużej wisimy na słuchawce, to też sporo informacji z
takich rozmów nam bezpowrotnie ucieka, np. imię i nazwisko osoby, z którą rozmawialiśmy, nie
wspominając o całej masie innych rzeczy, które mogą paść podczas choćby negocjowania/przedłużania
jakiegoś kontraktu. Gdy w późniejszym czasie zorientujemy się, że coś jest nie tak, to możemy mieć
poważny problem z udowodnieniem swoich racji, np. że zostaliśmy celowo wprowadzeni w błąd. Gdybyśmy
dysponowali nagraniem z takiej rozmowy, to bez problemu można by się do niego odnieść i nikt by nie
miał wątpliwości po czyjej stronie byłaby racja. Niemniej jednak, smartfony z Androidem mają
ogromny problem z nagrywaniem dźwięku audio podczas tego typu konwersacji, czego efektem jest brak
możliwości niezależnego rejestrowania rozmów telefonicznych. Jeśli jednak posiadamy ukorzenionego
Androida (dostęp do praw administratora root), to możemy te obostrzenia obejść zaprzęgając do pracy
aplikację Call Recorder.

<!--more-->
## Po co nagrywać rozmowy telefoniczne

Parę tygodni temu przytrafiła mi się bardzo niemiła sytuacja związana z jedną z umów, które były
zawierane prawie rok temu. Sprawa nie dotyczyła bezpośrednio mnie, a jedynie członka rodziny, przez
co nie do końca miałem pełne rozeznanie w sytuacji. Chodziło generalnie o przedłużenie umowy z
jednym z większych telekomów w Polsce (mniejsza o to z którym). Z informacji, które wstępnie
uzyskałem (od członka rodziny) wynikało, że pewna usługa została włączona przez konsultanta wbrew
świadomej i dobrowolnej woli tej osoby, czego efektem był sporo wyższy abonament.

Jako, że minęło już sporo czasu (około roku), to jedyną opcją rozwiązania zaistniałego problemu
było ustalenie tego, co dokładnie zostało powiedziane w rozmowie telefonicznej. Oczywiście osoby,
które negocjowały kontrakt nie zarejestrowały rozmowy i by z tej sytuacji jakoś wybrnąć, trzeba było
tutaj posłać zgłoszenie do operatora GSM z prośbą o udostępnienie nagrań, które ten operator
posiada właśnie m.in. na takie okoliczności, jak te wspomniane tutaj.

### Forma nagrań i warunki ich odsłuchania

O ile nagrania zostały mi udostępnione, to forma i warunki tego udostępnienia budzą co najmniej
niesmak. Chodzi o to, że operator GSM wymagał ode mnie zadzwonienia pod wskazany numer (w sieci
tego operatora), podania numeru rozmowy oraz PIN'u i tylko w ten sposób można było odsłuchać
rozmowę. Zatem by odsłuchać nagranie musiałem zapłacić za połączenie. Do tego dochodzi fakt, że nad
takim nagraniem nie miało się żadnej kontroli w kwestii tempa odtwarzania czy zwykłego cofnięcia o
kilka sekund, by przesłuchać dany fragment rozmowy jeszcze raz. Jeśli czegoś nie wyłapałem, np. z
faktu dość słabej jakości nagrania audio, to trzeba było jeszcze raz dzwonić i ponosić dodatkowe
koszty. Zdumiewające jest dla mnie to, że operator GSM nie potrafi udostępnić człowiekowi nagrania
do odsłuchania w formie offline i to bez zmuszania tej osoby do ponoszenia dodatkowych kosztów z tym
związanych.

Poza tymi problemami wymienionymi wyżej, czas na odsłuchanie rozmów był szalenie krótki. Operator
GSM dał mi niespełna 12 godzin na zapoznanie się z nagraniami -- mail dotarł do mnie około południa,
a PIN'y do nagrań wygasały jeszcze tego samego dnia (prawdopodobnie o północy). Po wygaśnięciu
PIN'u, rozmowy nie szło już odsłuchać. W takim pośpiechu przesłuchiwanie czegokolwiek nie jest
dobrym pomysłem. Pomyślałem sobie zatem, że nagram te rozmowy po tym jak uzyskam połączenie z tym
numerem, który mi został podany i wpiszę te dane, o które zostanę poproszony.

## Call Recorder nie nagrywa dźwięku

Postanowiłem poszukać aplikacji, którymi można by nagrać rozmowy telefoniczne i trafiłem na
[Call Recorder][4] (z repozytorium F-Drdoid). Z informacji o aplikacji można wywnioskować, że jest
ona dedykowana wręcz do nagrywania rozmów telefonicznych. Ustawiłem wszystko jak trzeba, no i
zadzwoniłem. Po podaniu numeru nagrania i PIN'u, rozmowa została odtworzona. Okazało się jednak, że
Call Recorder ma problemy przy nagrywaniu audio, czego efektem było przerywanie procesu nagrywania
chwilę po nawiązaniu połączenia telefonicznego.

Pomyślałem, że może jakieś wejście/wyjście źle ustawiłem. Po paru minutach zabawy doszedłem do
wniosku, że żadne z ustawień tej aplikacji nie było jej w stanie skłonić do nagrania rozmowy
telefonicznej. Nawet już nie chodziło o nagrywanie w obu kierunkach, jedynie o zgranie tego co
było odtwarzane w słuchawce, bo tylko to mnie w zasadzie interesowało.

## Próba przechwycenia audio w Screen Recorder i Audio Recorder

[W Androidzie począwszy od wersji 11][6] została zaszyta możliwość natywnego nagrywania tego co widać
na ekranie smartfona, tj. takie nagrywanie pulpitu/ekranu telefonu. Do tej pory trzeba było ratować
się zewnętrznymi aplikacjami, których głównym problemem było nagrywanie wewnętrznego dźwięku audio
(tego co słychać w głośnikach). W Androidzie 11 już takich niedogodności być nie powinno, a
nagrywanie ekranu/pulpitu dostępne jest z poziomu szybkich ustawień po przyciśnięciu odpowiedniego
kafelka:

|   |   |   |
|---|---|---|
| ![call-recorder-record-audio-dialer-smartphone-screen-record-attempt](/img/2022/01/002.call-recorder-record-audio-dialer-smartphone-screen-record-attempt.jpg#small) | ![call-recorder-record-audio-dialer-smartphone-screen-record-attempt](/img/2022/01/003.call-recorder-record-audio-dialer-smartphone-screen-record-attempt.jpg#small) | ![call-recorder-record-audio-dialer-smartphone-screen-record-attempt](/img/2022/01/004.call-recorder-record-audio-dialer-smartphone-screen-record-attempt.jpg#small) |

Sprawdziłem zatem czy Screen Recorder w moim telefonie działa. Po odpaleniu aplikacji z muzyką,
Screen Recorder nagrywał dźwięk z tej appki bez większego problemu. Niemniej jednak, po nawiązaniu
połączenia głosowego w aplikacji dialer'a, nagrywanie dźwięku było przerywane (obraz w dalszym ciągu
był rejestrowany).

Próbowałem zatem zgrać dźwięku przez mikrofon w trybie głośnomówiącym ale w dalszym ciągu podczas
rozmowy telefonicznej rejestrowana była jedynie cisza. Podobny problem był w przypadku
aplikacji [Audio Recorder][3] pobranej z F-Droid. Niby nagrywanie było aktywne podczas rozmowy
telefonicznej i plik został zapisany na flash smartfona, to i tak zawierał on jedynie samą ciszę.

## Nagrywanie rozmowy telefonicznej przy pomocy dwóch smartfonów

Jako, że miałem niewiele czasu na zapoznanie się z tymi nagraniami udostępnionymi przez operatora
GSM, to jedyne co mi przyszło do głowy, to puszczenie rozmowy na głośnik w jednym telefonie i
nagranie dźwięku z mikrofonu drugiego telefonu, zamykając przy tym oba urządzenia w szczelnym pudle.
Takie zamknięcie odseparowało cały układ od dźwięków otoczenia i jednocześnie poprawiło jakość
nagrania.

W ten sposób udało się wyprowadzić nagrania z serwisu operatora GSM do swobodnego przesłuchania ich
w formie offline, czego ostatecznym efektem było zmuszenie tego operatora do zmiany sposobu
naliczania opłat przez wykazanie nieprawidłowości, które podczas tej rozmowy ze strony konsultanta
zostały poczynione. Reklamacja została rozpatrzona pozytywnie, operator przeprosił za zaistniałą
sytuację, zobowiązał się też do zwrócenia wszystkich dodatkowych kosztów, które zostały przez okres
obowiązywania tej umowy naliczone i ze swojej strony udzielił jeszcze rabatu na abonament na kilka
okresów do przodu.

Niemniej jednak, taki sposób z pudłem i dwoma smartfonami nie przystoi cywilizowanym osobnikom w 21
wieku i przydałoby się wypracować nieco rozsądniejsze rozwiązanie, które można by zastosować będąc
nawet dzikiej głuszy.

## Dlaczego Androidy mają problemy z nagrywaniem audio

Gdy emocje opadły, mogłem na spokojnie poszukać rozwiązania problemu nagrywania rozmów
telefonicznych, których doświadczyłem w moim smartfonie mającym wgrany ROM na bazie AOSP/LineageOS
z Androidem 11.

Technicznie rzecz biorąc, [Android 9+ powinien posiadać wbudowaną w aplikację dialer'a możliwość
nagrywania rozmów][1]. I faktycznie dialer w moim Andku ma taką opcję, choć po przyciśnięciu
przycisku nagrywania rozmowy nic się nie dzieje. Sprawdzając w innym telefonie, udało się co prawda
załączyć nagrywanie ale jedyne co szło usłyszeć na tym nagraniu to dźwięcząca cisza. Być może w
mocy są jakieś restrykcje prawne i Google nie zezwala w tej części świata na nagrywanie rozmów
telefonicznych, a może to kwestia samego urządzenia. Nawet gdyby ta opcja nagrywania w stock'owej
aplikacji dialer'a od Google działała, to wygląda na to, że [rozmówca jest powiadamiany][2] o tym,
że druga końcówka rozpoczęła nagrywanie rozmowy, a ja zbytnio nie chciałbym się tą informacją
chwalić.

Ze sporej liczby artykułów, które ostatnimi dniami udało mi się na internetach przejrzeć, wychodzi
na to, że te wszystkie problemy związane z przechwytywaniem dźwięku w Androidach biorą się z troski
Google o prywatność użytkowników... Dlatego też, gdy aplikacja dialer'a zaczyna nadawać dźwięk, to
wszystkie inne aplikacje próbujące rejestrować audio (czy to z głośników, czy z mikrofonu)
automatycznie zostają tej możliwości pozbawione, czego efektem jest albo plik bez dźwięku, albo
proces nagrywania audio ulega zatrzymaniu.

## Call Recorder z repo Magisk'a

Jako, że ja swojego Androida mam ukorzenionego i mam dostęp do konta root, to mogłem nieco głębiej
poszperać i zobaczyć, co np. Magisk ma w swoim repozytorium modułów. Okazało się, że w tym
repozytorium widnieje pozycja [Axet's Call Recorder][5].

![call-recorder-record-audio-dialer-smartphone-magisk-module](/img/2022/01/005.call-recorder-record-audio-dialer-smartphone-magisk-module.jpg#small)

Jest to dokładnie ta sama aplikacja, co poprzednio pobrana z F-Droid, z tym że ten moduł (jak można
wyczytać w jego opisie) instaluje Call Recorder jako aplikację systemową i nadaje jej prawa
[CAPTURE_AUDIO_OUTPUT][7]. Warto też tutaj wspomnieć, że aktualizacje tej aplikacji z poziomu
F-Droid jak najbardziej będą możliwe.

Odinstalowałem zatem wgraną przez F-Droid aplikację Call Recorder i zainstalowałem ten moduł
Magisk'a. Po ponownym uruchomieniu telefonu zaktualizowałem przez F-Droid appkę Call Recorder i co
się okazało? Tym razem Call Recorder zaczął nagrywać rozmowy bez najmniejszego problemu i to w obu
kierunkach.

|   |   |   |
|---|---|---|
| ![call-recorder-record-audio-dialer-smartphone-fixed-app](/img/2022/01/006.call-recorder-record-audio-dialer-smartphone-fixed-app.jpg#small) | ![call-recorder-record-audio-dialer-smartphone-settings](/img/2022/01/007.call-recorder-record-audio-dialer-smartphone-settings.jpg#small) | ![call-recorder-record-audio-dialer-smartphone-settings](/img/2022/01/008.call-recorder-record-audio-dialer-smartphone-settings.jpg#small) |

Po odpowiednim skonfigurowaniu aplikacji, to nagrywanie rozmów może odbywać się automatycznie
ilekroć ktoś do nas dzwoni lub to my dzwonimy do kogoś, przez co żadna rozmowa już nam nie umknie.
Naturalnie druga strona nie jest powiadamiana o tym, że my mamy załączone nagrywanie, no i też
możemy nagrywać bez potrzeby zbytniego kombinowania z kilkoma smartfonami.


[1]: https://www.xda-developers.com/how-to-record-calls-android/
[2]: https://support.google.com/phoneapp/thread/107601233/how-to-off-call-recording-alert?hl=en&msgid=112333094
[3]: https://f-droid.org/en/packages/com.github.axet.audiorecorder/
[4]: https://f-droid.org/en/packages/com.github.axet.callrecorder/
[5]: https://github.com/di72nn/callrecorder-axet
[6]: https://www.xda-developers.com/android-11-screen-recorder-internal-audio/
[7]: https://developer.android.com/reference/android/Manifest.permission#CAPTURE_AUDIO_OUTPUT
