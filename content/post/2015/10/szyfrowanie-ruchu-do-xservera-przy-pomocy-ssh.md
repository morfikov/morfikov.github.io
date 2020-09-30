---
author: Morfik
categories:
- Linux
date: "2015-10-31T16:47:10Z"
date_gmt: 2015-10-31 14:47:10 +0100
published: true
status: publish
tags:
- ssh
- xserver
- szyfrowanie
title: Szyfrowanie ruchu do Xserver'a przy pomocy SSH
---

W przypadku zaufanych sieci lokalnych, czy też [kontenerów
LXC](/post/konfiguracja-kontenerow-lxc/), nie musimy zbytnio się troszczyć o
bezpieczeństwo przesyłanych danych. Nikt nam przecież nie założy tutaj podsłuchu. Dlatego też we
wpisie poświęconym konfiguracji Wine nie szyfrowaliśmy praktycznie żadnego ruchu sieciowego. Gdyby
jednak zaszła potrzeba przesłania pakietów do zdalnego Xserver'a przez internet, to takie
rozwiązanie naraziłoby nas na przechwycenie wszystkich danych. By zabezpieczyć się przed tego typu
scenariuszem możemy zaszyfrować ruch do Xserver'a [forward'ując wszystkie zapytania przy pomocy
szyfrowanego tunelu TLS](https://help.ubuntu.com/community/SSH/OpenSSH/PortForwarding). Możemy to
zrobić przy pomocy SSH i w tym wpisie postaramy się skonfigurować ten mechanizm.

<!--more-->
## Konfiguracja SSH

Spróbujmy na chwilę przyjąć, że gdzieś tam w internecie posiadamy jakiś serwer, do którego to
łączymy się zdalnie przy pomocy SSH. W takim przypadku jest to środowisko tekstowe, którym możemy
operować przy pomocy wiersza poleceń. Wielu z nas przywykło jednak do edycji plików tekstowych w
swoich ulubionych graficznych edytorach. Problem w tym, że na takim zdalnym serwerze prawdopodobnie
nie mamy zainstalowanego nawet Xserver'a, a przez to nie możemy tam uruchamiać sesji graficznej, no
i nie możemy korzystać z żadnego GUI.

Brak Xserver'a na zdalnej maszynie nie stanowi praktycznie żadnego problemu by graficzne aplikacje
tam uruchomić. Oczywiście muszą być one zainstalowane na takiej maszynie ale to się rozumie chyba
samo przez się. Jedyne co musimy zrobić, to tak skonfigurować SSH, by wszelkie zapytania do
Xserver'a tunelował do zdalnej lokalizacji. Ten mechanizm jest podobny do tego opisywanego przy
[konfiguracji Xserver'a pod Wine](/post/wine-w-kontenerze-lxc/), z tą różnicą, że
nie musimy praktycznie w ogóle tykać konfiguracji samego Xserver'a. Jedyne pliki jakie będziemy
poddawać edycji, to te od SSH, zarówno na kliencie jak i na serwerze. Cała pozostała konfiguracja
zostanie dostosowana automatycznie i my nie będziemy musieli sobie nią głowy zawracać.

Wymagane jest jednak od nas by na serwerze doinstalować pakiet `xauth` . A to z tego względu, że SSH
przy forward'owaniu zapytań wykorzystuje [mechanizm
xauth](/post/xauth-i-xhost-na-strazy-bezpieczenstwa-xservera/), który opiera się o
ciasteczka Xserver'a.

### Klient

Klienta możemy konfigurować albo globalnie w pliku `/etc/ssh/ssh_config` , albo lokalnie w pliku
`~/.ssh/config` . Połączenia można konfigurować niezależnie, tj. jeśli będziemy korzystać z
Xserver'a tylko w jednym przypadku, to możemy to wyraźnie określić w pliku konfiguracyjnym SSH w
poniższy sposób.:

    Host 192.168.10.20
          ForwardX11 yes
          Compression yes
    #     Ciphers

Te powyższe opcje możemy także sprecyzować bezpośrednio w wierszu poleceń przy wpisywaniu do
terminala `ssh` , gdzie `-X` odpowiada za ForwardX11, a `-C` za Compression . Dobrze jest także
zmienić szyfr przy pomocy opcji `-c` , która w powyższym pliku określana jest przez Ciphers.

### Server

Konfiguracja SSH na serwerze sprowadza się do edycji pliku `/etc/ssh/sshd_config` , gdzie musimy
dostosować te dwa poniższe parametry:

    X11Forwarding yes
    X11DisplayOffset 10

## Zmienna $DISPLAY

By Xserver mógł bez problemu działać, musi wiedzieć na jaki ekran (display) ma przesłać dane.
Odpowiada za to zmienna `$DISPLAY` , którą trzeba ustawić w środowisku na stacji klienckiej. W
przypadku SSH nie musimy tego robić ręcznie, bo SSH automatycznie robi to za nas za każdym razem gdy
podłączamy się do serwera. Poniżej jest skrócony log z połączenia, z którego możemy wywnioskować, że
połączenie z Xserverem leci po SSH:

    $ ssh -vvv 192.168.10.20
    OpenSSH_6.9p1 Debian-2, OpenSSL 1.0.2d 9 Jul 2015
    debug1: Reading configuration data /home/morfik/.ssh/config
    debug1: /home/morfik/.ssh/config line 10: Applying options for 192.168.10.20
    debug1: Reading configuration data /etc/ssh/ssh_config
    debug1: /etc/ssh/ssh_config line 19: Applying options for *
    ...
    morfik@192.168.10.20's password:
    ...
    debug1: Enabling compression at level 6.
    ...
    debug2: x11_get_proto: /usr/bin/xauth  list :0.0 2>/dev/null
    debug1: Requesting X11 forwarding with authentication spoofing.
    debug2: channel 0: request x11-req confirm 1
    ...
    morfik@viper:~$

Zmienna `$DISPLAY` powinna zostać automatycznie weksportowana w środowisku zdalnym. Dodatkowo, plik
`~/.Xauthority` powinien zostać odpowiednio zaktualizowany. Sprawdźmy czy tak jest w istocie:

    morfik@viper:~$ echo $DISPLAY
    localhost:10.0

    morfik@viper:~$ xauth list
    viper/unix:10  MIT-MAGIC-COOKIE-1  be81cf83e8124a805212e6161c1ecdf3

Jako, że zarówno zmienna `$DISPLAY` jak i plik `~/.Xauthority` są ustawiane automatycznie przez SSH,
to bez problemu możemy teraz odpalić jakąś graficzną aplikacje na serwerze, a okienka zostaną
przesłane do naszego lokalnego Xserver'a.

## Test szyfrowania

Przydałoby się sprawdzić czy aby na pewno ruch z Xserver'em jest szyfrowany. Możemy to zrobić albo
za pomocą jakiegoś sniffer'a sieciowego albo też możemy posłużyć się iptables na maszynie
klienckiej, gdzie możemy zablokować ruch przychodzący zostawiając jednocześnie otwarte dwa porty,
tj. jeden dla Xserver'a i drugi dla SSH. Jeśli ruch jest szyfrowany, pakiety nie powinny trafiać do
reguły łapiącej ruch Xserver'a. Poniżej przykład:

![](/img/2015/10/1.szyfrowanie-xserver-ssh-iptables.png#huge)

## Uwagi końcowe

Z tego co wyczytałem na sieci, to połączenie szyfrowane przy pomocy algorytmu AES bez kompresji jest
bardzo wolne i nie nadaje się zbytnio do szyfrowania połączeń z Xserver'em. Oczywiście można jechać
na standardowych ustawieniach SSH ale wtedy te okienka nie będą się płynnie ładować. Dlatego też
dobre jest skonfigurować ten aspekt SSH. Pamiętajmy jednak o tym, że by połączenie mogło zostać
ustanowione, obie strony muszą wspierać wybrany szyfr, a te możemy ustalić wydając w terminalu
poniższe polecenia:

    $ ssh -Q cipher
    $ ssh -Q mac
    $ ssh -Q kex
