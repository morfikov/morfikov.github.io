---
author: Morfik
categories:
- Linux
date: "2015-10-30T23:45:04Z"
date_gmt: 2015-10-30 21:45:04 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- xserver
title: Xauth i xhost na straży bezpieczeństwa Xserver'a
---

Na debianie Xserver domyślnie ma wyłączoną możliwość nasłuchiwania połączeń zdalnych. Chodzi
oczywiście o kwestie bezpieczeństwa, bo przecie nie od dziś wiadomo, że akurat to oprogramowanie
jest dziurawe jak sito i nikt rozsądny nie chciałby instalować go na swoim serwerze. Należałoby
jednak rozgraniczyć wykorzystywanie podatności jakiegoś oprogramowania od możliwości wejścia z nim w
interakcję. Jakby nie patrzeć, sam Xserver posiada co najmniej trzy mechanizmy ochrony, a do tego
dochodzą jeszcze reguły `iptables` , czy też pliki `/etc/hosts.allow` i `/etc/hosts.deny` .
Prawdopodobnie jest ich jeszcze kilka ale te najczęściej wykorzystywane mechanizmy gdy pojawia się
słowo Xserver, to `-nolisten tcp` (domyślnie aktywowany), `xhost` oraz `xauth` . Pierwszy z nich
wyklucza się z pozostałymi i to tym dwóm ostatnim przyjrzymy bliżej w tym wpisie.

Mechanizmy `xhost` oraz `xauth` w żaden sposób nie zabezpieczają informacji przesyłanych do
Xserver'a. Wobec czego, całą komunikację można bez problemu podsłuchać. Stwarza to zagrożenie
przechwycenia nie tylko obrazu wyświetlanego na monitorze ale także danych dotyczących myszy i
przyciskanych klawiszy na klawiaturze.

<!--more-->
## Mechanizm xhost

Każdy menadżer logowania, jak i sam `startx` mają domyślnie ustawioną opcję `-nolisten tcp` . W
przypadku gdy ją usuniemy i proces `X` zostanie uruchomiony z opcją `-listen tcp` , to Xserver
zacznie nasłuchiwać na porcie TCP 6000. Niemniej jednak, komunikacja z Xserver'em nie będzie dalej
możliwa. Pakiety sieciowe będą, co prawda, trafiać do niego ale żadne połączenie nigdy nie zostanie
ustanowione, a to za sprawą polityki jaką ustawia mechanizm `xhost` , którą to możemy podejrzeć
wpisując w terminalu to poniższe polecenie:

    $ xhost
    access control enabled, only authorized clients can connect
    SI:localuser:morfik

Komunikat informuje nas, że dostęp jest filtrowany oraz, że tylko autoryzowani klienci mogą uzyskać
połączenie. Na liście jest tylko jeden klient, którym jest lokalny użytkownik systemu `morfik` .
Każda inna osoba, która by się spróbowała połączyć z tym Xservere'm, nie uzyska połączenia.

Problem z `xhost` polega na tym, że możemy określać za jego pomocą jedynie adresy IP lub domeny, z
których połączenia mogą być akceptowane. Ten mechanizm miał rację bytu w zamierzchłych czasach,
gdzie było nie wiele komputerów i każdy miał przypisany określony adres IP. Obecnie, przy szerokim
wykorzystaniu technologi NAT, bardzo wielu ludzi ma ten sam adres IP i w ich przypadku możemy
zezwolić im wszystkim na dostęp albo też im go odmówić. Generalnie rzecz biorąc, `xhost` nie
powinien być wykorzystywany w internecie.

Nic jednak nie stoi na przeszkodzie by korzystać z niego w przypadku bezpiecznych sieci lokalnych
lub też [kontenerów LXC](/post/konfiguracja-kontenerow-lxc/) lub innych sposobów
wirtualizacji. W każdym z tych powyższych przypadków musimy nauczyć Xserver, które połączenia ma
akceptować. Najprościej jest posłużyć się znakami `+` i `-` , przykładowo:

    $ xhost +192.168.10.20
    $ xhost -192.168.10.20

Pierwsza linijka zezwala hostowi o adresie `192.168.10.20` na podłączenie do Xserver'a. Po wpisaniu
drugiej, to prawo zostanie mu odebrane. To w zasadzie cała funkcjonalność `xhost` . Dodawanie hostów
ma tylko efekt w czasie trwania sesji graficznej. Po zresetowaniu Xserver'a trzeba ponownie dodać
pożądane hosty do białej listy. Jeśli jakiś host ma łączyć się z naszym Xserver'em cały czas, to
trzeba będzie dodać taki wpis do autostartu środowiska graficznego. Więcej informacji można znaleźć
w [manualu xhost](http://manpages.ubuntu.com/manpages/xenial/en/man1/xhost.1.html) .

## Mechanizm xauth

Innym rozwiązaniem i przy tym o wiele bezpieczniejszym są ciasteczka Xserver'a trzymane z reguły w
katalogu użytkownika w pliku `~/.Xauthority` . Mechanizm `xauth` różni się poprzednika tym, że nie
operuje na adresach IP czy domenach, przynajmniej nie tak jak robił to `xhost` . Zamiast tego,
generowana jest informacja, na podstawie której klient może zostać wpuszczony przy podłączeniu do
zdalnego Xserver'a. Dane w takim ciasteczku mogą zostać wydobyte na serwerze i przesłane, np. przy
pomocy ssh, do klienta i tam dołączone do jego pliku `~/.Xauthority` .

By skorzystać z tego mechanizmu, musimy wygenerować ciasteczko oraz odpalić proces `X` z opcją
`-auth "$HOME/.Xauthority"` . Najprościej to zrobić edytując plik `/etc/X11/xinit/xserverrc` i
umieszczając w nim ten poniższy wpis:

    exec /usr/bin/X -auth "$HOME/.Xauthority" -listen tcp "$@"

Jeśli korzystamy z graficznego menadżera logowania, to edycja powyższego pliku raczej nic nam nie
da. Każdy menadżer logowania ma swoje opcje i to on podnosi proces `X` . Może mu również podać
określone parametry. Dla przykładu, poniżej jest plik konfiguracyjny LightDM,
`/etc/lightdm/lightdm.conf` :

    xserver-command=X -auth "$HOME/.Xauthority" -listen tcp
    xserver-allow-tcp=true

Po tym jak ustawimy parametr odpowiedzialny za ciasteczko, Xserver będzie akceptował jedynie
połączenia od tych klientów, którzy posiadają pewne informacje autoryzacyjne. Ważne jest przy tym,
by taki klient miał zainstalowany w swoim systemie pakiet `xauth` . Zawiera on bowiem narzędzia,
które operują na tych ciasteczkach.

W przypadku gdybyśmy nie mieli jeszcze tego ciasteczka (pliku `$HOME/.Xauthority` ), możemy utworzyć
je ręcznie:

    $ mcookie|sed -e 's/^/add :0 . /'|xauth -q

Jednak to, że go nie mamy jest mało prawdopodobne, bo program startujący Xserver, czy to `startx` ,
czy też DM, tworzą ten plik automatycznie. Jedyny problem przed jakim teraz stoimy, to przesłanie
informacji autoryzujących połączenie do Xserver'a na drugą maszynę. Jak wspomniałem wyżej, możemy to
zrobić przy pomocy ssh:

    $ xauth extract - morfikownia.mhouse.lh:0.0 | ssh -x morfik@192.168.10.20 xauth merge -

Fraza `morfikownia.mhouse.lh` to domena zdalnego Xserver'a. Można zamiast niej określić adres IP. Z
kolei `:0.0` określa z którego ekranu zamierzamy korzystać.

W przypadku korzystania z LightDM, `xauth extract - morfikownia.mhouse.lh:0.0` zwraca komunikat: "No
matches found, authority file "-" not written". Nie wiem czemu tak się dzieje, bo na sesji odpalonej
za pomocą `startx` , wszystko działa jak należy. Alternatywą może być skorzystanie ze zmiennej
`$DISPLAY` i przesłanie ciasteczka w ten poniższy sposób:

    $ xauth extract - $DISPLAY | ssh -x morfik@192.168.10.20 xauth merge -

Jeśli ciasteczko zostało pomyślnie przesłane, na maszynie zdalnej powinniśmy być je w stanie
podejrzeć:

    $ xauth list
    morfikownia.mhouse.lh:0  MIT-MAGIC-COOKIE-1  a578170842011b9f496ab3d8d91456bc
