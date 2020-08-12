---
author: Morfik
categories:
- OpenWRT
date: "2016-04-24T00:00:51Z"
date_gmt: 2016-04-23 22:00:51 +0200
published: true
status: publish
tags:
- ssh
- chaos-calmer
- router
title: Dostęp do routera OpenWRT (telnet, ssh, sshfs)
---

Standardowa instalacja OpenWRT nie zawiera w sobie żadnego trybu graficznego czy też panelu www.
Wszelkie operacje trzeba przeprowadzać przy pomocy terminala. Mimo to, OpenWRT daje nam kilka
możliwości na uzyskanie dostępu do routera. Ten firmware ma zaimplementowaną
obsługę [protokołu telnet](https://pl.wikipedia.org/wiki/Telnet), który swoją drogą nie należy do
bezpiecznych. Oprócz niego, mamy możliwość logowania się za
pomocą [protokołu SSH](https://pl.wikipedia.org/wiki/Secure_Shell). Tutaj sprawa bezpieczeństwa ma
się o wiele lepiej. Poza tym, można bardzo łatwo zaimplementować klucze SSH eliminując tym samym
dostęp oparty o wprowadzanie hasła przy logowaniu. Czy takie zabezpieczenia nam są potrzebne w
domowych warunkach? Tę kwestię niech sobie każdy użytkownik rozważy sam. Dodatkowo, jeśli już
wspomnieliśmy o protokole SSH, to warto poruszyć
kwestię [protokołu SSHFS](https://pl.wikipedia.org/wiki/SSHFS), czyli możliwości zamontowania
systemu plików routera lokalnie na komputerze. Daje nam to możliwość przeglądania takiego systemu
plików jak zwykłego katalogu. No i mamy też uproszczoną edycję plików, która może odbywać się w
trybie graficznym przy pomocy narzędzi, z których zwykle korzystamy na swoim PC. W tym wpisie
rzucimy okiem na te poszczególne metody i przy pomocy każdej z nich spróbujemy uzyskać dostęp do
routera.

<!--more-->
## Dostęp przez telnet

Po wgraniu OpenWRT, na router możemy się zalogować jedynie przy pomocy protokołu telnet. Dzieje się
tak ze względu na fakt, że użytkownik root standardowo nie posiada hasła. W efekcie usługa
odpowiedzialna za nasłuch na porcie 22 nie jest w stanie nas wpuścić. Zwykle z protokołu telnet
będziemy korzystać w dwóch sytuacjach. Pierwsza z nich dotyczy wstępnego skonfigurowania routera.
Zwykle ta czynność jest przeprowadzana po flash'owaniu routera nową wersją firmware. Druga sytuacja
występuje w momencie, gdy router nam odmawia posłuszeństwa
i [musimy się ratować trybem failsafe]({{< baseurl >}}/post/tryb-ratunkowy-failsafe-w-openwrt/).
Poniżej jest fotka obrazująca dostęp do routera za pośrednictwem protokołu telnet:

![]({{< baseurl >}}/img/2016/04/1.openwrt-dostep-telnet-router.png)

Widzimy, że router dostępny jest pod adresem 192.168.1.1, a komenda, która zainicjowała połączenie
to:

    # telnet 192.168.1.1

Z chwilą, gdy zalogujemy się na router po raz pierwszy, będziemy mieli możliwość zmiany hasła
użytkownika root. Robimy to przez wpisanie w terminalu polecenia `passwd` :

![]({{< baseurl >}}/img/2016/04/2.openwrt-telnet-zmiana-hasla-router.png)

Po tej operacji telnet zostanie automatycznie dezaktywowany i od tej pory będziemy w stanie logować
się za pomocą protokołu SSH.

## Dostęp przez SSH

Za realizację logowań z wykorzystaniem protokołu SSH na OpenWRT odpowiada `dropbear` . Nasłuchuje
on na porcie 22 i akceptuje jedynie połączenia od strony LAN. By zalogować się na router po SSH, w
terminalu wpisujemy `ssh 192.168.1.1` . Po wpisaniu tej komendy, zostanie nam wyrzucony poniższy
komunikat:

![]({{< baseurl >}}/img/2016/04/3.openwrt-dostep-ssh-hash-weryfikacja.png)

Mamy tutaj informację na temat odciska palca (fingerprint) klucza SSH. Zwykle ten hash jest
unikatowy i jego weryfikacja zabezpiecza komunikację przed podsłuchem. W tym przypadku nie ma to
większego znaczenia, bo przecie między nami i naszym domowym routerem raczej nikt podsłuchu nie
założy. Zresztą, to i tak jest pierwsze połączenie, a klucze zostały wygenerowane przez OpenWRT po
flash'owaniu routera. Dlatego też akceptujemy ten klucz i po chwili powinien się nam pojawić monit
proszący o podanie hasła do konta administratora root. Po jego podaniu zostaniemy zalogowani na
router:

![]({{< baseurl >}}/img/2016/04/4.openwrt-dostep-ssh-router.png)

### Konfiguracja dopbear'a

Narzędzie `dropbear` posiada swój plik konfiguracyjny zlokalizowany w `/etc/config/dropbear` .
Możemy tam określić m.in. te poniższe parametry:

    config dropbear
          option PasswordAuth     'on'
          option RootPasswordAuth 'on'
          option RootLogin        '1'
          option Port             '22'
          option BannerFile       '/etc/banner'
          option Interface        'br-lan'
          option IdleTimeout      '300'

Opcja `PasswordAuth` zezwala na logowanie się użytkownikom przy pomocy hasła. Nie jest to zbytnio
bezpieczne ale przynajmniej proste w obsłudze i jeśli zezwalamy na dostęp do routera jedynie
użytkownikom z sieci domowej, to raczej nie potrzebujemy tutaj dodawać dodatkowych obostrzeń.
Natomiast w przypadku, gdy usługa SSH jest otwarta na świat, to przydałyby się pewne dodatkowe
mechanizmy zabezpieczające, np. w postaci [fwknop](http://www.cipherdyne.org/fwknop/) czy kluczy
uwierzytelniających RSA. Póki co, zostańmy jedynie przy zwykłym haśle. W OpenWRT mamy dostępnego
jednego użytkownika i jest nim administrator systemu root. Można stworzyć wszak innych użytkowników
i przy pomocy `sudo` zrzucić uprawnienia w celu dobezpieczenia routera i jeśli ktoś się zdecydował
na ten krok i używa innego konta do logowania się na SSH, może skorzystać z opcji `RootLogin` i
ustawić ją na `0` . Spowoduje to wyłączenie możliwości logowania się użytkownika root w systemie.
Każdy inny użytkownik będzie mógł się zalogować bez problemu. Dalej w pliku konfiguracyjnym mamy
`Port` odpowiadający za port, na którym nasłuchuje `dropbear` . Warto go zmienić jeśli usługa SSH
jest udostępniana na interfejsie WAN. Opcja `BannerFile` precyzuje ścieżkę do pliku, który będzie
wyświetlany użytkownikom logującym się na router. Następnie mamy `Interface` odpowiadający za
interfejs, na którym będzie nasłuchiwać usługa SSH. Jeśli nie zamierzamy logować się do routera z
internetu, możemy ustawić tutaj `br-lan` (domyślny interfejs sieci lokalnej). Ostatni parametr, tj.
`IdleTimeout` , precyzuje czas bezczynności w sekundach, po którym użytkownik zostanie wylogowany z
routera.

### Kopiowanie plików

By wgrać jakiś plik na router (lub go pobrać z niego), posługujemy się poleceniem `scp` ,
przynajmniej w przypadku linux'ów. Cała procedura sprowadza się do wydania poniższych poleceń:

    $ scp ~/plik.txt root@192.168.1.1:/test/plik.txt

lub:

    $ scp root@192.168.1.1:/test/plik.txt ~/plik.txt

W przypadku pierwszego z nich, skopiowaliśmy lokalny plik `plik.txt` na router do katalogu `/test/`
. Druga komenda zaś pobrała z routera ten plik i zapisała go we wskazanej lokalizacji na dysku
komputera. Taka wymiana plików za pomocą `scp` jest szyfrowana i obciąża router, z czego trzeba
sobie zdawać sprawę. Fraza `root@192.168.1.1:/plik.txt` określa kolejno użytkownika ( `root` ), na
którego chcemy się zalogować przy kopiowaniu plików. Następnie po znaku `@` mamy adres IP, z którym
chcemy się połączyć. Potem występuje `:` sygnalizujący katalog domowy użytkownika, na którego się
logujemy na zdalnym serwerze. Jeśli po nim dodamy `/` , zaczniemy tym samym precyzować ścieżkę w
drzewie katalogów, a te z kolei muszą istnieć. W przeciwnym wypadku, przesyłanie plików się nie
powiedzie.

## SSHFS

Istnieje także bardziej wygodna metoda interakcji z routerem. Co prawda, nie mamy może na routerze
podpiętego monitora ale możemy zamontować systemu plików routera u siebie w systemie. Z tym, że nie
mam pojęcia czy taki myk jest możliwy do przeprowadzenia spod poziomu windowsa. Natomiast działa
wyśmienicie jeśli chodzi o linux'y. By z tego sposobu skorzystać, musimy doinstalować na routerze
pakiet `openssh-sftp-server` . Robimy to w poniższy sposób:

    # opkg update
    # opkg install openssh-sftp-server

Linux'y domyślnie mają już zainstalowane potrzebne oprogramowanie, które umożliwia zamontowanie
systemu plików routera w lokalnym katalogu i nie musimy przeprowadzać żadnych dodatkowych czynności,
by ten mechanizm nam zaczął działać. Jeśli jednak byśmy się znaleźli w gronie osób, które jakimś
sposobem nie mają odpowiednich pakietów w systemie, to musimy w nim doinstalować pakiet `sshfs` .
System plików routera montujemy wydając w terminalu to poniższe polecenie:

    $ sshfs root@192.168.1.1:/ /home/morfik/Desktop/router/

Zamontuje ono katalog główny routera we wskazanej lokalizacji na komputerze. Od tego momentu mamy
dostęp do każdego folderu na routerze i możemy używać graficznych edytorów, by operować na plikach.
Możemy też przy pomocy menadżera plików przesyłać pliki między tymi dwiema maszynami bez większego
problemu, co ułatwi nam znacznie konfigurację routera. Poniżej
fotka:

[![5.openwrt-dostep-sshfs-gui]({{< baseurl >}}/img/2016/04/5.openwrt-dostep-sshfs-gui-1024x549.png)]({{< baseurl >}}/img/2016/04/5.openwrt-dostep-sshfs-gui.png)
