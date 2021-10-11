---
author: Morfik
categories:
- Linux
date: "2015-11-11T22:30:36Z"
date_gmt: 2015-11-11 21:30:36 +0100
published: true
status: publish
tags:
- xserver
- tty
- monitor
GHissueID: 245
title: Jak wyłączyć monitor z linii poleceń
---

Pewnego niezbyt pamiętnego dnia byłem zmuszony skorzystać z windowsa. Po chwili pracy na nim,
musiałem odejść od komputera na dłuższą chwilę. Chciałem zatem wyłączyć monitor bez jednoczesnego
wyłączania całego komputera, czy przełączania go w stan uśpienia. Problem na jaki się natknąłem był
taki, że kompletnie nie miałem pojęcia jak tego dokonać i ostatecznie zakończyło się to
przyciśnięciem fizycznego przycisku na obudowie monitora. Gdybym tego nie zrobił, monitor by się
wyłączył sam ale dopiero po pewnym czasie, który został określony w opcjach zarządzania energią. Nie
miałem za bardzo czasu i chęci szukać rozwiązania tego problemu ale z tego co widziałem na necie, to
ludzie rozpisywali jakieś tutorale na ten temat, co wydało mi się co najmniej dziwne. Czy w
windowsie nie nie ma żadnej opcji by wyłączyć w prosty sposób monitor? Najwyraźniej nie ma,
przynajmniej ja jej nie znalazłem. Czy my na linux'ie też mamy takie problemy? W tym wpisie
postaramy się odpowiedzieć na pytanie czy jest jakiś prosty sposób by na linux'ie wyłączyć monitor z
wiersza poleceń, tak by efekt był praktycznie natychmiastowy.

<!--more-->
## Xserver potrafi wyłączyć monitor

Na naszych maszynach zwykle instalujemy środowiska graficzne. Do do tych bardziej rozbudowanych są
dołączane panele sterowania, w których można konfigurować opcje zasilania. Podobnie jak to można
zrobić na windowsie. Środowiska graficzne opierają się o Xserver. Zatem jeśli środowisko graficzne
nie oferuje nam możliwości szybkiego wyłączenia czy wygaszenia ekranu, to zawsze jesteśmy w stanie
się odwołać bezpośrednio do Xserver'a. To samo tyczy się systemów, które nie korzystają ze środowisk
graficznych, a jedynie z prostych menadżerów okien.

Instalując w systemie metapakiet `Xorg` , w zależnościach dociągany jest między innymi pakiet
`x11-xserver-utils` . Po nazwie możemy łatwo wywnioskować, że są w nim zawarte narzędzia, które
pomogą nam zarządzać Xserver'em. Mamy w nim do dyspozycji `xset` i to za jego pomocą możemy
[skonfigurować sobie szereg parametrów dotyczących
monitora](https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling), klawiatury i
myszy. Jako, że mamy tam opcje odpowiadające za monitor, to jesteśmy także w stanie go w prosty
sposób wyłączyć. Zatem by wyłączyć monitor, odpalamy terminal i wpisujemy w nim to poniższe
polecenie:

    $ xset dpms force off

W zależności od konfiguracji systemu, wywołanie `xset` by wyłączył monitor może także pociągnąć za
sobą blokadę ekranu przy pomocy jakiegoś mechanizmu bezpieczeństwa, np. `light-locker` . W takim
przypadku, po włączeniu monitora trzeba będzie się zalogować do systemu. Odpowiada to mechanizmowi
zwykłego wygaszacza ekranu, który się uruchomia po kilku minutach, gdy nie wykorzystujemy klawiatury
i myszy.

## Jak wyłączyć monitor pod TTY

W przypadku konsoli TTY, nie możemy korzystać z `xset` , bo jest to narzędzie Xserver'a. TTY jest
zaś środowiskiem tekstowym i zupełnie od niego niezależnym. Jak zatem wyłączyć monitor będąc
zalogowany jedynie na TTY? Można to zrobić korzystając z narzędzia `vbetool` . Niemniej jednak, by
móc go użyć, musimy posiadać uprawnienia administratora systemu lub też dodać odpowiednią linijkę
do pliku `/etc/sudoers` . Zakładam, że do tak prostej czynności jak wyłączenie monitora, nie
potrzebujemy hasła. Dodajmy zatem do pliku `sudoers` ten poniższy wpis (korzystając z `visudo` ):

    Host_Alias HOSTY = localhost,morfikownia
    morfik      HOSTY = (root) NOPASSWD: /usr/sbin/vbetool dpms [onf]*

By teraz wyłączyć monitor będąc zalogowanym na TTY, wystarczy wpisać to poniższe polecenie:

    $ vbetool dpms off

Jest jednak problem z włączeniem tak wyłączonego monitora i będzie trzeba wpisywać tekst na ślepo,
bo dopiero po podaniu parametru `on` , monitor się włączy. Nie wiem czy jest jakieś obejście tego
problemu, tak by monitor włączał się po przyciśnięciu jakiegoś klawisza na klawiaturze. W każdym
razie zawsze można skorzystać z klawisza ↑ , by wywołać ostatnie polecenie wprowadzone do terminala.
Następnie można skasować trzy znaki i dopisać `on` .
