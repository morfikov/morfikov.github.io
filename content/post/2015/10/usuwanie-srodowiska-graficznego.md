---
author: Morfik
categories:
- Linux
date: "2015-10-18T17:52:42Z"
date_gmt: 2015-10-18 15:52:42 +0200
published: true
status: publish
tags:
- debian
- apt
- tasksel
- środowisko-graficzne
GHissueID: 186
title: Usuwanie środowiska graficznego
---

[Na forum DUG'a][1] znów został poruszony ciekawy wątek, tym razem odnośnie usunięcia całego
środowiska graficznego z systemu. Pozornie niby nic nadzwyczajnego, przecie każdy z nas potrafi
odinstalować szereg pakietów via `apt` czy `aptitude` . Problematyczne za to mogą się okazać
zależne pakiety, które nie zostaną automatycznie usunięte wraz z konkretnym metapakietem od
środowiska graficznego. Jak zatem usunąć te pozostałości?

<!--more-->
## Zbędne środowisko graficzne

Prędzej czy później, część z użytkowników, którzy zaczną zgłębiać w stopniu ponadprzeciętnym
strukturę systemu linux, dostrzeże problemy i ograniczenia, jakie niesie ze sobą używanie coraz to
bardziej rozbudowanych środowisk graficznych. Chodzi generalnie o to, że czasem niekoniecznie
potrzebujemy całej funkcjonalności, którą oferuje dane środowisko. Czasami też korzystamy z
alternatywnych aplikacji, które naszym zdaniem są w jakimś tam stopniu lepsze od tych oferowanych,
np. przez GNOME. W takim przypadku mamy zwykle dwie aplikacje, z których wykorzystujemy tylko jedną,
a druga zwyczajnie zaśmieca nam system. Oprogramowanie, które leży odłogiem, zwykliśmy usuwać z
systemu, ale jako że komponenty danego środowiska graficznego są mniej lub bardziej zakorzenione w
jego strukturze, to nie zawsze jesteśmy w stanie takie zbędne nam pakiety wywalić z systemu bez
rozwalania przy tym całego środowiska graficznego.

Innym problemem mogą być błędy wynikające ze złożoności takiego środowiska graficznego oraz
nadpisywania szeregu niskopoziomowych ustawień. Przykładem może być np. klawiatura, która
skonfigurowana za pomocą plików Xorg'a będzie działać w porządku, natomiast po konfiguracji jej
przez panel sterowania środowiska graficznego już niekoniecznie. Do tego dochodzi jeszcze
rozbieżność tych ustawień w aplikacjach, czego efektem jest to, że jedna aplikacja korzysta z
ustawień Xorg'a, a druga z tych oferowanych przez środowisko graficzne.

Ja generalnie nie mam nic do środowisk graficznych. Są one idealne w przypadku systemów live ale
moim zdaniem nie nadają się one na bardziej zaawansowany desktop. Jest to też dobry start jeśli ktoś
chce przejść z innego systemu operacyjnego na linux'a. Tak czy inaczej w pewnym momencie taki
człowiek zacznie się zastanawiać nad tym jak przeprowadzić usuwanie środowiska graficznego, tak by
usunąć jego wszelkie komponenty i nie pozostawić w systemie śmieci.

## Metapakiety i taski

Wprawdzie dawno nie instalowałem systemu (zwłaszcza z płytki cd/dvd) ale raczej niewiele się w tej
kwestii zmieniło. Instalator zwykle instaluje system podstawowy i wstępnie go konfiguruje
wyświetlając nam szereg okienek z pytaniami. Gdy ten etap dobiega końca, mamy już w pełni
działający system, który możemy uruchomić. Zwykle jednak pojawia się dodatkowe zapytanie o
zainstalowanie środowiska graficznego i właśnie te pakiety chcielibyśmy usunąć.

Oczywiście linux'a, zwłaszcza debiana, można stawiać wewnątrz środowiska chtoot i manualnie
doinstalować wszelkie jego komponenty, wliczając w to też i środowisko graficzne. Jednak
instalowanie paruset czy tysięcy pakietów nie należy do przyjemnych rzeczy, dlatego też takie
pakiety zostały pogrupowane i doklejono na ich szczycie dodatkowy pakiet, który został określony
mianem metapakietu . W taki sposób, możemy podczas instalacji sprecyzować tylko ten jeden
metapakiet, a zawarte w nim zależności pociągną za sobą instalację pozostałych komponentów.

Jeśli chcielibyśmy sprawdzić jakie metapakiety mamy do dyspozycji w debianie, możemy je odszukać
przy pomocy narzędzia `debtags` , przykładowo:

    # debtags search role::metapackage
    ...
    lxde - Metapackage for LXDE
    lxde-core - Metapackage for the LXDE core
    ...

Podobnie sprawa ma się w przypadku tasków. Taski, to też metapakiety, z tym, że są one jeszcze
bardziej obszerne, tzn. pociągają jeszcze więcej zależności. Do instalacji tasków, zwłaszcza przy
instalacji systemu z płytki, wykorzystuje się narzędzie `tasksel` i oferuje ono możliwość
zainstalowania tych poniższych rzeczy:

    # tasksel --list-tasks
    u desktop       Debian desktop environment
    u gnome-desktop GNOME
    u xfce-desktop  Xfce
    u kde-desktop   KDE
    u cinnamon-desktop      Cinnamon
    u mate-desktop  MATE
    u lxde-desktop  LXDE
    u web-server    web server
    u print-server  print server
    u ssh-server    SSH server
    u laptop        laptop

Są też i pomniejsze taski, które zwykle są uwzględniane przy instalacji, np. któregoś ze środowisk
graficznych widocznych powyżej. Możemy je wylisować wydając poniższe polecenie:

    # apt-cache search --names-only ^task-

### Zawartość metapakietów

Zatem mamy nazwy metapakietów i tasków. Przydałoby się zajrzeć w głąb tych pakietów i ustalić
dokładnie jakie zależności są tam wypisane. Możemy to zrobić przy pomocy `tasksel` lub/i
`apt`/`aptitude` , przykładowo dla środowiska graficznego LXDE:

    # tasksel --task-packages lxde-desktop
    task-lxde-desktop

    # aptitude show task-lxde-desktop
    ...
    Depends: tasksel (= 3.33), task-desktop, lightdm, lxde
    Recommends: lxtask, lxlauncher, xsane, libreoffice-gtk, synaptic, iceweasel, libreoffice, libreoffice-help-en-us, mythes-en-us,
                hunspell-en-us, hyphen-en-us, system-config-printer, gnome-orca

W zależności od konfiguracji systemu, pakiety, które zostaną pociągnięte do instalacji mogą odnosić
się tylko do `Depends:` lub też dodatkowo do `Recommends:` . Jak widzimy wyżej, w `Depends:` mamy
jeden dodatkowy task `task-desktop` oraz metapakiet `lxde` . Zatem trzeba by także ustalić ich
zależności i robimy to dokładnie tak samo jak zostało to opisane powyżej, aż uzyskamy dokładną
listę pakietów, które zostaną pociągnięte przy instalacji taska `task-lxde-desktop` .

## Całkowite usuwanie środowiska graficznego

Mamy już mniej więcej pojęcie czym są metapakiety i taski oraz jak odnaleźć ich zależności. Jeśli
chcielibyśmy zrezygnować z korzystania z jakiegoś środowiska graficznego, które zaznaczyliśmy w
instalatorze, w tym przypadku LXDE, to przy pomocy `tasksel` musimy wyrzucić metapakiet
`task-lxde-desktop` :

    # tasksel remove lxde-desktop

Przeprowadzając testy na potrzeby tego artykułu, zauważyłem, że `tasksel remove` przy standardowej
konfiguracji systemu nie usuwa wszystkich rzeczy. W sumie to praktycznie nic nie usuwa, bo ilość
pakietów w systemie zmniejszyła się o 9. Podczas gdy zostało ich zainstalowanych 1168. Jest to
wynikiem nieodpowiedniej konfiguracji `apt`/`aptitude` , a konkretnie chodzi o pakiety zalecane i
sugerowane, których system nie wywala, gdy usuwane są metapakiety.

Możemy temu oczywiście zaradzić przez dodanie odpowiednich wpisów do [pliku apt.conf][2] , który
znajduje się w katalogu `/etc/apt/` . Domyślnie on nie istnieje i trzeba go sobie stworzyć. Wpisy,
które nas interesują, są poniżej:

    APT::Install-Recommends "false";
    APT::Install-Suggests "false";
    APT::AutoRemove::RecommendsImportant "false";
    APT::AutoRemove::SuggestsImportant "false";

Oczywiście, za zachowanie przy usuwaniu pakietów odpowiadają tylko te dwa ostatnie ale dobrze jest
także dodać te dwa pierwsze, bo wtedy przy instalacji, pakiety zalecane i sugerowane nie będą
pociągane w zależnościach.

Gdy teraz spróbujemy wydać w terminalu `tasksel remove` , spowoduje to usunięcie praktycznie
wszystkich pakietów, które zostały zainstalowane za sprawą `lxde-desktop` . Będą tam również
pakiety, których raczej nie chcemy wywalać z systemu, np. pakiety od Xorg'a. Jest to nieuniknione,
ze względu na mechanizm zależności jaki oferuje `apt`/`aptitude` . Najlepszym wyjściem byłoby
zaakceptowanie wyrzucenia tych wszystkich pakietów i późniejsze doinstalowanie tylko tych, które
potrzebujemy. Możemy także spróbować usunięcia pomniejszych metapakietów, które składają się na
`lxde-desktop` , co może nas uchronić nas przed ponowną instalacją szeregu potrzebnych rzeczy.

Trzeba też pamiętać o tym, że instalacja pewnych rzeczy może skutkować odinstalowaniem innych
rzeczy, w efekcie czego, status pakietów po usunięciu środowiska graficznego może się nieznacznie
różnić. Nie jest to jednak jakaś wielka różnica. W moim przypadku było to zaledwie 10 pakietów,
których nie mogłem usunąć ręcznie po tym jak usuwanie środowiska graficznego dobiegło końca. Trzeba
także liczyć się z tym, że im dłużej korzystamy z danego środowiska graficznego i
instalujemy/usuwamy pakiety, to mogą one namieszać nieco w zależnościach i nie zawsze wszystkie
rzeczy zostaną usunięte, tak jak to by miało miejsce tuż po zainstalowaniu czystego środowiska
graficznego. W takim przypadku dobrze jest rozważyć usunięcie określonych metapakietów lub też
pojedynczych pakietów w oparciu o wypisane przez `apt`/`aptitude` zależności.


[1]: https://forum.dug.net.pl/viewtopic.php?id=27813
[2]: /post/konfiguracja-apt-i-aptitude-w-pliku-apt-conf/
