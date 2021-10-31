---
author: Morfik
categories:
- Android
date:    2016-10-20 19:41:01 +0200
lastmod: 2016-10-20 19:41:01 +0200
published: true
status: publish
tags:
- smartfon
- aplikacje
- youtube
- reklamy
- spam
GHissueID: 436
title: 'Android: YouTube bez reklam na smartfonie (NewPipe)'
---

Ja generalnie zaliczam się do grona osób bardzo spokojnych ale tylko do czasu aż się zdenerwuję.
Jedną taką bardziej wyprowadzającą mnie z równowagi kwestią są reklamy w serwisie YouTube. Problem
jest o wiele bardziej dotkliwy, gdy w grę wchodzą urządzenia mobilne, np. smartfony. Na komputerze
nie mam większego problemu, bo wszystkie reklamy mogę zablokować stosując adblock/ublock w
przeglądarce lub też korzystać z aplikacji [mpsyt][1]/[minitube][2]. Gdy chcę wesprzeć kogoś, to
odpalam kilka kanałów z reklamami, wyciszam dźwięk i mój linux ogląda za mnie ten cały syf
reklamowy, a ja go ani nie słyszę, ani nie widzę i wszyscy są happy. W przypadku smartfonów
oglądanie serwisu YT jest nieco problematyczne. Nie dość, że nie ma jak obejść tych reklam, to
jeszcze zwykle są one głośniejsze niż ścieżka dźwiękowa materiału video, co bardzo wnerwia w
godzinach nocnych. Przy szukaniu rozwiązania tego problemu natknąłem się na [NewPipe][3]. Jest to
przeglądarkę YT z otwartym kodem źródłowym (OpenSource), która działa podobnie do mpsyt/minitube i
to ten programik zostanie opisany w niniejszym artykule.

<!--more-->
## Instalacja NewPipe

Projekt NewPipe został stworzony przez ludzi (i dla ludzi), którzy widać niezbyt przepadają za
narzucaniem im tego co mają oglądać w danej chwili. Miło, że nie jestem odosobniony w tej kwestii.
Ta aplikacja operuje bezpośrednio na serwisie YouTube przez co jest w stanie wyciągnąć goły materiał
audio/video i udostępnić go odbiorcy. Działa to mniej więcej na takiej zasadzie, jak próba
odtworzenia linku do filmu w zwykłym playerze video. Taki materiał będzie pozbawiony jakichkolwiek
dodatków, które są dołączane przez Google i spółkę, przez co zostanie obcięty cały moduł reklam
pozostawiając nam do obejrzenia czysty film.

Aplikacji NewPipe nie można pobrać ze sklepu GooglePlay. Trzeba się posiłkować [alternatywnym
repozytorium aplikacji jakim jest F-Droid][4]. Po instrukcję do F-Droid'a odsyłam do wyżej
podlinkowanego artykułu. Z kolei, by zainstalować NewPipe, musimy wyszukać tę aplikację w
repozytorium i wcisnąć przycisk "Zainstaluj":

![newpipe-instalacja-f-droid](/img/2016/10/1.newpipe-instalacja-f-droid.png#huge)

## Konfiguracja NewPipe

Po uruchomieniu aplikacji przywita nas niezbyt wyrafinowany interfejs. Jest on bardzo prosty i na
wiele nie pozwala. Natomiast wszystkie bardziej użyteczne rzeczy są jak najbardziej
zaimplementowane. Poniżej fotka głównego interfejsu oraz listy opcji, które możemy skonfigurować:

![newpipe-interfejs-opcje](/img/2016/10/2.newpipe-interfejs-opcje.png#huge)

Możemy tutaj ustawić domyślną rozdzielczość oglądanego materiału video. Mamy do wyboru 144p, 240p,
360p i 720p. Nie ma opcji na 1080p ale chyba nikt w takiej jakości nie ogląda YT na smartfonie.
Jesteśmy w stanie też dostosować sobie format audio, do wyboru WebM i m4a. Jest też kilka bardzo
ciekawych funkcji.

### Zewnętrzny player audio/video

Materiały w serwisie YouTube możemy oglądać bezpośrednio za pomocą NewPipe. Możemy także zaprzęgnąć
do tego celu zewnętrzny odtwarzać, oczywiście w przypadku, gdy z takiego korzystamy. Ja mam u siebie
zainstalowanego VLC, który raczej powinien być wszystkim dobrze znany.

![newpipe-zewnetrzny-odtwarzacz-video](/img/2016/10/3.newpipe-zewnetrzny-odtwarzacz-video.png#medium)

Jeśli nie chce nam się wybierać odtwarzacza za każdym razem, to zawsze możemy ustawić jeden z nich
jako domyślny i wtedy proces przesyłania filmu, np. do VLC, będzie transparentny dla nas.

### Odtwarzaj w Kodi

[Kodi][5] to bardzo zaawansowany odtwarzacz audio/video, którego zadaniem jest ogarnięcie całej
kolekcji multimediów jaką mamy w swoim... domu. Raczej średnio się nadaje ta aplikacja na smartfona
ale jakby nie patrzeć klient na Androida jest. NewPipe potrafi zrobić użytek z Kodi jeśli zamierzamy
uczynić go naszym domyślnym odtwarzaczem.

### Treści tylko dla dorosłych

W aplikacjach, w których możemy przeglądać serwis YouTube bez rejestracji, często nie mamy dostępu
do materiałów zawierających treści tylko dla dorosłych. NewPipe potrafi obejść to obostrzenie i
zezwolić na przeglądanie takich filmów. Bardzo przydatna funkcja.

### Możliwość zapisu ścieżki audio/video

Innym bardzo ciekawym ficzerem jest możliwość zapisu ścieżek audio i video bezpośrednio na pamięć
telefonu, ew. kartę SD (wymagany [root smartfona][6] oraz [zdjęcie pewnych zabezpieczeń][7]).
Jeśli chcemy pobrać jakiś materiał by go obejrzeć w trybie offline, np. mamy dostęp do WiFi w domu
ale za moment musimy wyjść, to bez większego problemu możemy zapisać sobie taki film na flash
telefonu. Jeśli interesuje nas tylko ścieżka audio, to również i ta opcja nie stanowi większego
wyzwania, przez co możemy pobrać sam dźwięk oszczędzając tym samym miejsce w pamięci smartfona:

![newpipe-pobieranie-audio-video](/img/2016/10/4.newpipe-pobieranie-audio-video.png#big)

Wątki widoczne wyżej to nic innego jak tylko podział pliku na części w celu szybszego jego pobrania.

### Odtwarzanie filmu w tle

W przypadku standardowej przeglądarki YT, po zminimalizowaniu aplikacji, proces odtwarzania
materiału video jest automatycznie pauzowany. Nie wiem czy tę opcję da się jakoś wyłączyć, bo
czasem ona trochę przeszkadza. W NewPipe możemy zaznaczyć opcję ExoPlayer i w ten sposób film będzie
odtwarzany w tle, nawet jak zminimalizujemy aplikację. Możemy naturalnie korzystać z zewnętrznego
player'a, który z kolei rządzi się własnymi prawami co do odtwarzania materiału video/audio w tle.

## Przeglądanie YouTube w NewPipe

Przede wszystkim, NewPipe nie wymaga od nas rejestracji w żadnym serwisie, nawet w YouTube czy
Google. Możemy korzystać z tej aplikacji nie mając w Androidzie nawet skonfigurowanego konta Google.
Jedyny problem jaki mam z NewPipe to brak możliwości subskrybowania kanałów. Możemy naturalnie
wyszukiwać kanały i przeglądać ich zawartość posortowaną od najnowszego filmu. Nie będziemy mieli
jednak żadnych notyfikacji czy ulubionych kanałów, tak jak w przypadku standardowej przeglądarki
YouTube w naszym smartfonie. Idąc dalej, nie mamy możliwości komentowania czy nawet czytania
komentarzy. Mamy natomiast dostęp do informacji o filmie. By wyszukać kanał/film, wystarczy wpisać
interesującą nas frazę w szukajce u góry:

![newpipe-funkcjonalnosc-youtube](/img/2016/10/5.newpipe-funkcjonalnosc-youtube.png#big)

Jeśli natomiast chcemy mieć dostęp do ostatniej aktywności kanału, to trzeba kliknąć w nazwę kanału,
która pokaże nam się po wybraniu jednej z pozycji dostępnych na liście filmów:

![newpipe-funkcjonalnosc-youtube](/img/2016/10/6.newpipe-funkcjonalnosc-youtube.png#medium)

W przypadku, gdy nie możemy żyć bez komentarzy i całej funkcjonalności, jaką dostarcza nam serwis
YouTube, to możemy korzystać z przeglądarki, czy to tej domyślnej, czy zwykłej przeglądarki www, np.
Firefox. Trzeba tylko sobie odpowiednio skonfigurować domyślne aplikacje. Wtedy całą funkcjonalność
serwisu będziemy mieli dostępną w Firefox'ie, a oglądane filmy będziemy przesyłać bezpośrednio do
NewPipe za pomocą tapnięcia w robocika, który figuruje w pasu adresu przeglądarki www:

![newpipe-youtube-firefox](/img/2016/10/7.newpipe-youtube-firefox.png#medium)

Jeśli chcemy, aby film po przesłaniu do NewPipe automatycznie się zaczął odtwarzać, to możemy
zaznaczyć stosowną opcję w ustawieniach tejże aplikacji. Miłego oglądania YouTube bez reklam na
smartfonach.

![newpipe-youtube-bez-reklam](/img/2016/10/8.newpipe-youtube-bez-reklam.png#big)


[1]: https://github.com/mps-youtube/mps-youtube
[2]: http://flavio.tordini.org/minitube
[3]: https://github.com/TeamNewPipe/NewPipe
[4]: /post/android-repozytorium-aplikacji-opensource-f-droid/
[5]: https://kodi.tv/
[6]: /post/android-root-smartfona-neffos-c5-od-tp-link/
[7]: /post/android-brak-mozliwosci-zapisu-danych-na-karcie-sd-neffos-c5/
