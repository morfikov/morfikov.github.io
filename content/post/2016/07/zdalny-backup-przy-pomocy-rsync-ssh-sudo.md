---
author: Morfik
categories:
- Linux
date: "2016-07-28T23:15:55Z"
date_gmt: 2016-07-28 21:15:55 +0200
published: true
status: publish
tags:
- ssh
- sudo
- rsync
title: Zdalny backup przy pomocy rsync, ssh i sudo
---

Mój VPS, jako że jest dość tani, nie zawiera całej masy wynalazków. Jedną z tych bardziej
użytecznych rzeczy jest backup danych na dysku VPS'a. OVH liczy sobie trochę grosza za usługę
snapshot'ów. Dlatego też byłem zmuszony poszukać innego rozwiązania, które sprawiłoby, że kopia
wszystkich ważnych plików byłaby zawsze poza granicami tego VPS. Najlepiej, gdyby te pliki były
umieszczany na moim własnym komputerze, czy jakiejś stacji roboczej, która ma robić za taki
backup'owy serwer. Problem w tym, że ciężko jest zsynchronizować sobie poprawnie katalogi na
odległość, choć jest to możliwe przy pomocy `ssh` , `rsync` oraz `sudo` . Z tym, że mamy tutaj
szereg problemów związanych z uprawnieniami do plików. No i oczywiście trzeba także uwzględnić inny
port SSH. Trochę było z tym zamieszania ale ostatecznie udało się to zadanie rozwiązać.

<!--more-->
## Port SSH

Jako, że usługa SSH jest dość krytyczna, to zwykle nie zostawiamy jej na porcie 22. Gdybyśmy jednak
zostawili na moment ten standardowy port, to katalogi można by zsynchronizować w następujący sposób:

    $ rsync -avx --delete-excluded root@192.168.10.10:/ ./

My jednak nie będziemy wykorzystywać domyślnego portu. Niemniej jednak, ten nowo obrany port trzeba
zdefiniować. Są z grubsza dwa rozwiązania tego problemu. Pierwszym z nich jest przerobienie tego
powyższego polecenia do poniższej postaci:

    $ rsync -avx --delete-excluded -e "ssh -p 2222" root@192.168.10.10:/ ./

Drugim i do tego o wiele lepszym wyjściem z tej sytuacji jest skorzystanie z pliku `~/.ssh/config` .
W nim możemy podać konfigurację dla określonego hosta, z którym zamierzamy się łączyć zdalnie.
Możemy również określić inny port i ten port później zostanie podłapany przez `rsync`
automatycznie. Poniżej jest przykład sekcji z konfiguracją hosta:

    Host morfitronik.lh 192.168.10.10
        User root
        Port 2222

Określiliśmy nie tylko port ale także użytkownika. Od tego momentu, gdy tylko będziemy się łączyć z
tymi hostami określonymi w `Host` , te dane zostaną wykorzystane i nie ma potrzeby ich określania w
poleceniu `rsync` . Zatem nasza linijka upraszcza się nieco i teraz wygląda mniej więcej tak:

    $ rsync -avx --delete-excluded 192.168.10.10:/ ./

## Klucze SSH i hasło do konta root

To powyższe polecenie powinno już nam wykonać synchronizację katalogów, ale jako że szereg plików
wymaga dostępu root, to musimy korzystać z tego konta na VPS. Nie zaleca się jednak logowania przez
SSH na root'a. Chyba, że mu [skonfigurujemy klucze SSH][1] oraz ustawimy na serwerze poniższą opcję
w pliku `/etc/ssh/sshd_config` :

    PermitRootLogin without-password

W takim przypadku, użytkownik root będzie w stanie się zalogować przez SSH ale tylko z
wykorzystaniem swojego prywatnego klucza SSH. Mając klucze, odpada nam też potrzeba posługiwania się
hasłem, co czyni proces backup'u o wiele prostszy w implementacji.

## Prawa do plików/katalogów

Kolejny problem z jakim musimy się zmierzyć, to prawa do plików i katalogów. Nawet jeśli spróbujemy
zsynchronizować zdalny katalog przy pomocy jednego z tych dwóch przykładowych poleceń opisanych
powyżej, to i tak na drugim końcu połączenia wszystkim zsynchronizowanym plikom zostaną przepisane
uprawnienia. Poniżej przykład synchronizacji katalogu `/etc/` . Spójrzmy na plik `shadow` :

    $ ls -al shadow
    -rw-r----- 1 morfik morfik 966 2016-07-23 12:05:08 shadow

Standardowo ten plik ma UID `root` oraz GID `shadow` . Widzimy zatem, że jedynie prawa do zapisu
tego pliku zostały zachowane (0640). Natomiast zmienił się użytkownik i grupa. Niby wykonaliśmy
backup za pomocą `rsync` logując się po SSH na root ale uprawnienia nam się rozjechały. To
przepisywanie się użytkownika i grupy danego pliku jest winą faktu, że lokalny proces jest
uruchomiony jako zwykły użytkownik, a nie root. Trzeba zatem uruchomić `rsync` jako root.

## Wykorzystanie sudo

Na potrzeby skryptów najlepiej posłużyć się `sudo` . Można przy jego pomocy usunąć hasło przy
zmianie użytkownika, co nam jeszcze bardziej ułatwi proces backup'u. Musimy po prostu dodać
polecenia, przy pomocy których zamierzamy dokonać synchronizacji katalogów. Najprościej byłoby dodać
sam `rsync` , choć może to powodować zagrożenie bezpieczeństwa. Jeśli to nie problem, to wpisujemy w
terminalu `visudo` i dodajemy poniższe wpisy:

    Host_Alias HOSTY = localhost,morfikownia,morfikownia.mhouse
    morfik      HOSTY = (root) NOPASSWD: /usr/bin/rsync

Jeśli natomiast bezpieczeństwo lokalnego systemu ma dla nas znaczenie, to musimy odpowiednio
przerobić powyższe polecenie `/usr/bin/rsync` . Możemy, np. dać coś takiego:

    Host_Alias HOSTY = localhost,morfikownia,morfikownia.mhouse
    morfik      HOSTY = (root) NOPASSWD: /usr/bin/rsync -avx --delete-excluded root@192.168.10.10\:/etc/ /media/morfitronik/etc/

Trzeba tutaj wspomnieć o kilku rzeczach. Przede wszystkim, będziemy mogli wpisać polecenie dokładnie
w takiej formie jak zostało ono powyżej uwzględnione. Będziemy musieli pilnować kolejności
parametrów, jak i praktycznie każdego znaku, np. nie możemy zapomnieć dopisać na końcu ścieżki
`/` . Jako, że zamierzamy sobie zbudować skrypt, to ten problem nas raczej nie dotyczy.

Druga sprawa, to użytkownik w `root@192.168.10.10` . Musimy go podać w `sudo` i konsekwentnie
definiować w wydawanych poleceniach. Jeśli tego nie uczynimy, to dostaniemy poniższy błąd:

    receiving incremental file list
    rsync: change_dir "/var/lib/mysql" failed: Permission denied (13)

    sent 8 bytes  received 79 bytes  58.00 bytes/sec
    total size is 0  speedup is 0.00
    rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1655) [Receiver=3.1.1]
    rsync: [Receiver] write error: Broken pipe (32)

I ostatnia sprawa, to musimy postawić `\` przed `:` , który oddziela adres hosta od zdalnej ścieżki.

## Skrypt backup'u wykorzystujący ssh, rsync i sudo

Przydałoby się napisać prosty skrypt, który zrobi nam backup wszystkich ważnych danych trzymanych na
VPS'ie. Poniżej jest prosty szkielet, który można rozbudować o kolejne ścieżki. Pamiętajmy tylko, by
dodać stosowne wpisy w konfiguracji `sudo` :

    #!/bin/sh

    USER="root"
    HOST="192.168.10.10"
    DIR="/media/morfitronik"

    mv $DIR.tar.gz $DIR.backup-$(date +'%Y-%m-%d-%H-%M-%S').tar.gz
    sudo tar czpf $DIR.tar.gz $DIR/

    sudo rsync -avx --delete-excluded $USER@$HOST:/home/ $DIR/home/
    sudo rsync -avx --delete-excluded $USER@$HOST:/etc/ $DIR/etc/

I to w zasadzie cała filozofia robienia backup'u przy pomocy `rsync` , `ssh` oraz `sudo` . Skrypt
można odpalić jako zwykły użytkownik, a synchronizacja określonych katalogów dokona się
automatycznie bez naszej ingerencji.


[1]: /post/uwierzytelniajace-klucze-ssh/
