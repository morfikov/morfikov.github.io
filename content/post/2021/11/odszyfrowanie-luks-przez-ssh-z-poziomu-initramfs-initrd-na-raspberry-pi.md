---
author: Morfik
categories:
- RaspberryPi
date:    2021-11-12 23:13:00 +0100
lastmod: 2021-11-13 13:25:00 +0100
published: true
status: publish
tags:
- luks
- raspberry-pi-4b
- raspios
- raspbian
- szyfrowanie
- ssh
- initramfs
- initrd
GHissueID: 581
title: Odszyfrowanie LUKS przez SSH z poziomu initramfs/initrd na Raspberry Pi
---

Od paru dni bawię się swoim Raspberry Pi w kontekście zaszyfrowania jego systemu RasPiOS/Raspbian.
O ile samo [zaszyfrowanie tego minikomputera przy pomocy mechanizmu LUKS][5] nie było jakoś
specjalnie trudne, to trzeba było pomyśleć nad rozwiązaniami mającymi ułatwić otworzenie takiego
zaszyfrowanego kontenera przy starcie systemu. Póki co udało się wypracować w miarę zadowalające
[rozwiązanie wykorzystujące dedykowane urządzenia USB w roli klucza][6], bez którego system się nie
uruchomi. Są jednak i inne rozwiązania, które mogą nam pomóc odszyfrować system RPI bez potrzeby
fatygowania się do pomieszczenia i wtykania w jej port USB jakiegoś pendrive. Mowa o zaprzęgnięciu
usługi SSH, która by została uruchomiona w fazie initramfs/initrd, chwilę przed wpisaniem hasła do
kontenera LUKS. Takie rozwiązanie wymaga jednak zainstalowania innego serwera SSH, tj. Dropbear,
pogodzenia go z serwerem OpenSSH oraz też trzeba odpowiednio przygotować sam obraz initramfs/initrd.
Po pomyślnym skonfigurowaniu systemu, będziemy się logować do Raspberry Pi przez SSH i wpisywać
hasło do kontenera LUKS, a jeśli to hasło będzie prawidłowe, to system zostanie odszyfrowany i
uruchomiony.

<!--more-->
## Konfiguracja usługi OpenSSH

Standardowo w każdym Raspberry Pi z zainstalowanym systemem RasPiOS/Raspbian mamy dostępny serwer
SSH bazujący na oprogramowaniu `openssh-server` . Ten serwer SSH nie będzie jednak potrafił
poradzić sobie z zadaniem, które tutaj próbujemy rozwiązać, ze względu na fakt braku integracji z
obrazem initramfs/initrd. Taką integrację zapewnia inny serwer SSH, tj. `dropbear` . Niemniej
jednak, te dwa serwery SSH bardzo wiele od siebie różni i Dropbear nie posiada całej masy
opcji, które są wspierane standardowo w OpenSSH. Trzeba zatem się zastanowić czy pozostawić w
systemie pakiet `openssh-server` i doinstalować drugi serwer SSH (i jakoś te dwie usługi
pogodzić ze sobą) albo też pozbyć się z systemu pakietu `openssh-server` całkowicie i w jego
miejsce wygrać `dropbear` .

Jako, że pakiet `openssh-server` jest domyślnie wgrany w Raspberry Pi, to postanowiłem doinstalować
`dropbear` i skonfigurować ten drugi serwer SSH w taki sposób, by był on uruchamiany jedynie w fazie
initramfs/initrd. Póki co jednak skupmy się na skonfigurowaniu usługi OpenSSH. Zwykle ta usługa
jest nieaktywna i wymaga ręcznego włączenia, co możemy zrobić w poniższy sposób:

    root@raspberrypi:/home/pi# systemctl enable ssh.service

### Brak możliwości zalogowania się na konto root w RPI

W systemie RasPiOS/Raspbian na konto administratora systemu root nie da się domyślnie zalogować.
Dlatego też trzeba korzystać z mechanizmu `sudo` . By nieco uprościć sobie życie, możemy pokusić
się o włączenie możliwości logowania się na konto root, co przyda nam się przy logowaniu via SSH.
Wszytko co musimy zrobić, by mieć możliwość zalogowania się na konto root, to ustawienie hasła temu
użytkownikowi przy pomocy `passwd` .

Logujemy się zatem na użytkownika root przez `sudo` :

    pi@raspberrypi:~ $ sudo su

Zmieniamy/ustawiamy hasło dla root:

    root@raspberrypi:/home/pi# passwd
    New password:
    Retype new password:
    passwd: password updated successfully

    root@raspberrypi:/home/pi# exit
    exit

I sprawdzamy jeszcze na koniec czy jesteśmy się w stanie zalogować w terminalu przez wpisanie w
nim polecenia `su -` podając wcześniej ustawione hasło:

    pi@raspberrypi:~ $ su -
    Password:
    root@raspberrypi:~#

### Zezwolenie na zdalne logowanie się na użytkownika root via SSH

Kolejnym krokiem jest zezwolenie na zdalne logowanie się na użytkownika root via SSH. W tym celu
trzeba poddać edycji plik `/etc/ssh/sshd_config` i dodać do niego [parametr PermitRootLogin][4]:

    PermitRootLogin yes

Zapisujemy plik i restartujemy usługę OpenSSH:

    root@raspberrypi:~# systemctl restart ssh.service

### Transfer publicznego klucza SSH klienta na Raspberry Pi

By móc się podłączyć do serwera SSH uruchomionego na Raspberry Pi i zalogować na konto
administratora root w sposób bezpieczny, musimy przesłać na to urządzenie swój publiczny klucz SSH.
Taki klucz możemy wygenerować przy pomocy `ssh-keygen` na maszynie klienta w poniższy sposób:

    $ ssh-keygen -t rsa -b 4096 -C "rpi-raspios-$(date -I)_rsa"

Klucz publiczny został zapisany w katalogu `~/.ssh/` pod nazwą `rpi-raspios_rsa.pub` . Ten klucz
trzeba przesłać na Raspberry Pi i możemy to zrobić na kilka sposób, choć najprostszym z nich jest
zaprzęgnięcie do pracy narzędzia `ssh-copy-id` . Przy pomocy flagi `-i` wskazujemy położenie klucza
publicznego na dysku:

    $ ssh-copy-id -i ~/.ssh/rpi-raspios_rsa.pub -p 2222 root@192.168.1.239
    /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/morfik/.ssh/rpi-raspios_rsa.pub"
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    root@192.168.1.239's password:

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh '192.168.1.239'"
    and check to make sure that only the key(s) you wanted were added.


Klucz został przesłany z powodzeniem. Możemy ten stan rzeczy zweryfikować zaglądając do pliku
`/root/.ssh/authorized_keys` na Raspberry Pi, w którym to powinna się znajdować poniższa linijka
(zawartość przycięta dla lepszej czytelności):

    ssh-rsa AAAAB...xnzQ== rpi-raspios-2021-11-11_rsa

### Włączenie logowania jako root jedynie za pomocą klucza SSH

Po przesłaniu klucza SSH do Raspberry Pi, możemy wyłączyć możliwość logowania się na konto root
przy pomocy hasła, bo takie rozwiązanie nie jest zbyt bezpieczne. Zamiast logować się z
wykorzystaniem hasła, włączymy sobie opcję logowania za pomocą klucza SSH, który dopiero co
przesłaliśmy do naszej maliny. Edytujemy zatem na RPI plik `/etc/ssh/sshd_config` zmieniając
wartość uprzednio dodanego parametru `PermitRootLogin` z `yes` na `prohibit-password` :

    # PermitRootLogin yes
    PermitRootLogin prohibit-password

Zapisujemy plik i restartujemy serwer SSH:

    root@raspberrypi:~# systemctl restart ssh.service

### Test połączenia SSH

By być pewnym, że wszystko działa jak należy, łączymy się testowo po SSH ze stacji klienckiej na
adres IP Raspberry Pi:

    $ ssh root@192.168.1.239 -p 2222
    Linux raspberrypi 5.10.63-v7l+ #1459 SMP Wed Oct 6 16:41:57 BST 2021 armv7l

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Fri Nov 12 01:01:39 2021 from 192.168.1.150
    root@raspberrypi:~#

Jak widać zostaliśmy zalogowani bez najmniejszego problemu i co najważniejsze bez pytania nas o
hasło, zatem klucz SSH działa prawidłowo.

### Konfiguracja połączenia SSH w pliku ~/.ssh/config

By nie wpisywać w poleceniu parametrów połączenia za każdym razem ilekroć chcemy się logować do
serwera SSH, dobrze jest w pliku `~/.ssh/config` na stacji klienckiej stworzyć zwrotkę podobną do
tej poniżej i odpowiednio uzupełnić stosowne parametry:

    Host 192.168.1.239
        User root
        IdentitiesOnly
        IdentityFile ~/.ssh/rpi-raspios_rsa
        CheckHostIP no
        Port 2222

W ten sposób w terminalu wystarczy wpisać `ssh 192.168.1.239` i zostaniemy zalogowani bez większego
problemu bez podawania tych pozostałych parametrów połączenia.

## Instalacja i konfiguracja Dropbear na Raspberry Pi

Konfigurację OpenSSH mamy z głowy. Przyszła pora na instalację drugiego serwera SSH, tj. Dropbear.
Musimy zainstalować w zasadzie dwa pakiety: `dropbear` oraz `dropbear-initramfs` . Pakiet
`dropbear-initramfs` jest wymagany, by aktywować serwer SSH w fazie initramfs/initrd. By być w
stanie odszyfrować kontener LUKS przez SSH w fazie initramfs/initrd potrzebny nam będzie także
pakiet `cryptsetup-initramfs` ale ten powinniśmy już posiadać w systemie. Pozostałe pakiety takie
jak `busybox` , `initramfs-tools` oraz `lvm2` również powinniśmy mieć zainstalowane w systemie
skoro posiadamy zaszyfrowany kontener LUKS. Reasumując, na wypadek gdyby czegoś brakowało,
instalujemy te poniższe pakiety:

    root@raspberrypi:/home/pi# apt-get install \
                               dropbear dropbear-initramfs \
                               cryptsetup-initramfs \
                               busybox \
                               initramfs-tools \
                               lvm2

### Dwie różne usługi SSH na jednym hoście

Mając zainstalowane w Raspberry Pi dwa różne serwery SSH, trzeba zastanowić się nad kwestią kluczy
SSH. Każdy serwer SSH wymaga takich kluczy do swojego poprawnego działania, tj. by ruch pochodzący
od klientów SSH mógł zostać z powodzeniem zdeszyfrowany na serwerze. Przy instalacji serwera SSH
(czy to OpenSSH, czy też Dropbear), stosowny zestaw kluczy dla takiej usługi zostanie wygenerowany
automatycznie. Taka rozbieżność w kluczach sprawia, że nasz Raspberry Pi w fazie initramfs/initrd
przed otworzeniem zaszyfrowanego kontenera LUKS będzie się identyfikował innym kluczem SSH niż
będzie to miało miejsce po odszyfrowaniu kontenera LUKS i uruchomieniu się systemu operacyjnego.
Taka sytuacja sprawia, że będziemy mieli do zaakceptowania dwa różne fingerprint'y kluczy, którymi
te dwa serwery SSH się będą posługiwać. Niemniej jednak, gdy odcisk palca klucza ulega zmianie, to
podczas podłączania się do serwera, klient SSH wyrzuci nam taki oto niezbyt komfortowy komunikat:

    $ ssh 192.168.1.239
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
    Someone could be eavesdropping on you right now (man-in-the-middle attack)!
    It is also possible that a host key has just been changed.
    The fingerprint for the RSA key sent by the remote host is
    SHA256:DxolO3jZe+KzcPuDsDqL6+y+C0Igl6DyRqxjvRZ5mfk.
    Please contact your system administrator.
    Add correct host key in /home/morfik/.ssh/known_hosts to get rid of this message.
    Offending ECDSA key in /home/morfik/.ssh/known_hosts:24
      remove with:
      ssh-keygen -f "/home/morfik/.ssh/known_hosts" -R "[192.168.1.239]:2222"
    Host key for [192.168.1.239]:2222 has changed and you have requested strict checking.
    Host key verification failed.

Mamy tutaj wyraźne oświadczenie, że odcisk palca klucza serwera SSH (a więc i też i jego klucz)
uległ zmianie, co zwykle oznacza, że ktoś mógł manipulować przy serwerze SSH. By się podłączyć do
takiego serwera SSH ze zmienionym kluczem, trzeba będzie usunąć stary klucz z bazy klienta w pliku
`~/.ssh/known_hosts` przez wpisanie w terminalu poniższego polecenia:

    $ ssh-keygen -f "/home/morfik/.ssh/known_hosts" -R "[192.168.1.239]:2222"

Później, gdy system się już poprawnie zainicjuje, to przy logowaniu via SSH trzeba będzie to
powyższe polecenie wpisać jeszcze raz, bo klucz SSH na Raspberry Pi ponownie ulegnie zmianie i tak
w kółko z każdym restartem maszyny. Trzeba przyznać, że takie zachowanie nie jest zbyt pożądane i
trzeba ten zaistniały problem jakoś rozwiązać.

#### Ignorowanie pliku ~/.ssh/known_hosts

Jednym z rozwiązań jest pominięcie weryfikacji tożsamości serwera SSH za sprawą sprawdzania
odcisku palca klucza, którym taka maszyna się identyfikuje podczas nawiązywania z nią połączenia.
Taka weryfikacja jest jedynie opcjonalna i ma na celu poprawę bezpieczeństwa, tj. trudno ten
powyższy komunikat o zmianie fingerprint'a klucza przeoczyć. W przypadku naszej maliny wiemy, że
ten klucz będzie się zmieniał, przez co możemy rozluźnić nieco politykę weryfikacji tożsamości
serwera przez ignorowanie pliku `~/.ssh/known_hosts` .

Jeśli chcemy tylko tymczasowo zignorować plik `~/.ssh/known_hosts` , to wystarczy w poleceniu `ssh`
podać opcję `-o StrictHostKeyChecking=no` . Jeśli chcemy dodać wyjątek na stałe, to edytujemy plik
`~/.ssh/config` i dodajemy do niego poniższy wpis przy bloku z konfiguracją połączenia SSH dla
Raspberry Pi:

    Host 192.168.1.239
        ...
        StrictHostKeyChecking no
        ...

Skorzystanie z tej powyższej opcji sprawi, że będziemy w stanie nawiązać połączenie z serwerem SSH,
ale gdy fingerprint ulegnie zmianie, to na ekranie pojawi się stosowny komunikat. Jeśli
chcielibyśmy się tego komunikatu również pozbyć, to trzeba by skorzystać z opcji
`-o UserKnownHostsFile=/dev/null` podczas połączenia lub też dodać ten parametr na stałe do pliku
`~/.ssh/config` :

    Host 192.168.1.239
        ...
        UserKnownHostsFile=/dev/null
        ...

#### Ujednolicenie kluczy SSH dla OpenSSH i Dropbear

Prawdę mówiąc, to pominięcie weryfikacji tożsamości serwera SSH nie jest zbyt dobrą praktyką i nie
korzystałbym z tego rozwiązania. Zamiast tego podejścia opisanego wyżej, lepszym rozwiązaniem jest
ujednolicenie kluczy SSH, tak by zarówno OpenSSH jak i Dropbear miały te same klucze prywatne,
przez co ta opisana wyżej sytuacja ze zmianą fingerprint'a nie będzie mieć miejsca.

By ujednolicić klucze SSH, nie musimy ich na nowo generować. Możemy skorzystać z kluczy, które
zostały już wygenerowane podczas procesu instalacji pakietu `openssh-server` lub `dropbear` .

Klucze prywatne dla serwera OpenSSH znajdują się w katalogu `/etc/ssh/` pod nazwami `ssh_host_*` :

    root@raspberrypi:/home/pi# ls -al /etc/ssh/
    total 604
    drwxr-xr-x   2 root root   4096 Nov 12 01:14 .
    drwxr-xr-x 126 root root   4096 Nov 12 00:53 ..
    -rw-r--r--   1 root root 565189 Mar 12  2021 moduli
    -rw-r--r--   1 root root   1580 Mar 12  2021 ssh_config
    -rw-r--r--   1 root root   3271 Nov 12 01:14 sshd_config
    -rw-------   1 root root   1385 May  7  2021 ssh_host_dsa_key
    -rw-r--r--   1 root root    606 May  7  2021 ssh_host_dsa_key.pub
    -rw-------   1 root root    505 May  7  2021 ssh_host_ecdsa_key
    -rw-r--r--   1 root root    178 May  7  2021 ssh_host_ecdsa_key.pub
    -rw-------   1 root root    411 May  7  2021 ssh_host_ed25519_key
    -rw-r--r--   1 root root     98 May  7  2021 ssh_host_ed25519_key.pub
    -rw-------   1 root root   1823 May  7  2021 ssh_host_rsa_key
    -rw-r--r--   1 root root    398 May  7  2021 ssh_host_rsa_key.pub
    -rw-r--r--   1 root root    338 May  7  2021 ssh_import_id

Natomiast klucze dla serwera Dropbear znajdują się w katalogu `/etc/dropbear/` pod nazwami
`dropbear_*` :

    root@raspberrypi:/home/pi# ls -al /etc/dropbear/
    total 28
    drwxr-xr-x   3 root root 4096 Nov 11 07:09 .
    drwxr-xr-x 126 root root 4096 Nov 12 00:53 ..
    -rw-------   1 root root  457 Nov 11 07:09 dropbear_dss_host_key
    -rw-------   1 root root  141 Nov 11 07:09 dropbear_ecdsa_host_key
    -rw-------   1 root root  805 Nov 11 07:09 dropbear_rsa_host_key
    drwxr-xr-x   2 root root 4096 Nov 11 07:08 log
    -rwxr-xr-x   1 root root  100 Feb 12  2019 run

##### Różne formaty kluczy SSH

Nie możemy jednak sobie od tak skopiować kluczy SSH z jednego katalogu i wrzucić ich do drugiego
katalogu pod zmienionymi nazwami. Klucze OpenSSH są w formacie ASCII, a klucze Dropbear w formacie
binarnym. Trzeba zatem jedne z tych kluczy przekonwertować. Najprościej jest przerobić klucze
OpenSSH na ten format, z którego korzysta Dropbear, bo mamy do tego celu dedykowane narzędzie
`dropbearconvert` znajdujące się w katalogu `/usr/lib/dropbear/` .

##### Error: Unrecognised key type

Problem jednak w tym, że te klucze OpenSSH, które zostały domyślnie wygenerowane na moim Raspberry
Pi, są najwyraźniej dość egzotyczne i przy próbie skorzystania z `dropbearconvert` generowany jest
poniższy błąd:

    root@raspberrypi:/home/pi# /usr/lib/dropbear/dropbearconvert openssh dropbear /etc/ssh/ssh_host_rsa_key /etc/dropbear/dropbear_rsa_host_key
    Error: Unrecognised key type
    Error reading key from '/etc/ssh/ssh_host_rsa_key'

Szukając informacji na temat tego problemu znalazłem taki oto [post na forum Archlinux][1], w
którym jest mowa o tym, że klucze OpenSSH są formacie RFC4716 i trzeba je przerobić do PEM zanim
będzie można skorzystać z narzędzia `dropbearconvert` .

Klucze OpenSSH do formatu PEM możemy przerobić za pomocą `ssh-keygen` w poniższy sposób (hasło
podajemy puste):

    root@raspberrypi:/etc/ssh# ssh-keygen -m PEM -p -f ssh_host_rsa_key
    Key has comment 'root@raspberrypi'
    Enter new passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved with the new passphrase.

    root@raspberrypi:/etc/ssh# ssh-keygen -m PEM -p -f ssh_host_ed25519_key
    Key has comment 'root@raspberrypi'
    Enter new passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved with the new passphrase.

    root@raspberrypi:/etc/ssh# ssh-keygen -m PEM -p -f ssh_host_dsa_key
    Key has comment 'root@raspberrypi'
    Enter new passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved with the new passphrase.

    root@raspberrypi:/etc/ssh# ssh-keygen -m PEM -p -f ssh_host_ecdsa_key
    Key has comment 'root@raspberrypi'
    Enter new passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved with the new passphrase.

Warto w tym miejscu wspomnieć, że serwer OpenSSH bez problemu działa z kluczami w formacie PEM.

##### Zmiana formatu kluczy OpenSSH via dropbearconvert

Po przerobieniu kluczy OpenSSH z RFC4716 na PEM, nie powinniśmy mieć już problemów by
przekonwertować klucze OpenSSH na format wykorzystywany przez Dropbear. Korzystamy zatem z narzędzia
`dropbearconvert` podając mu cztery argumenty: format klucza wejściowego, format klucza wyjściowego,
ścieżkę do pliku wejściowego oraz ścieżkę do pliku wyjściowego:

    root@raspberrypi:/home/pi# /usr/lib/dropbear/dropbearconvert openssh dropbear /etc/ssh/ssh_host_rsa_key /etc/dropbear/dropbear_rsa_host_key
    Key is a ssh-rsa key
    Wrote key to '/etc/dropbear/dropbear_rsa_host_key'

    root@raspberrypi:/home/pi# /usr/lib/dropbear/dropbearconvert openssh dropbear /etc/ssh/ssh_host_ecdsa_key /etc/dropbear/dropbear_ecdsa_host_key
    Key is a ecdsa-sha2-nistp256 key
    Wrote key to '/etc/dropbear/dropbear_ecdsa_host_key'

    root@raspberrypi:/home/pi# /usr/lib/dropbear/dropbearconvert openssh dropbear /etc/ssh/ssh_host_dsa_key /etc/dropbear/dropbear_dss_host_key
    Key is a ssh-dss key
    Wrote key to '/etc/dropbear/dropbear_dss_host_key'

### Podmiana kluczy w katalogu /etc/dropbear-initramfs/

Klucze SSH dla Dropbear, które wyżej przerobiliśmy, byłyby wykorzystywane w przypadku, gdybyśmy
korzystali z usługi `dropbear` w miejscu `sshd` . Domyślnie jednak ta usługa jest nieaktywna (za
sprawą pliku `/etc/default/dropbear` ), bo w przeciwnym przypadku doszłoby do konfliktu między tymi
dwoma usługami, które chciałyby jednocześnie nasłuchiwać na tym samym porcie 22. Niemniej jednak,
usługa `dropbear` , którą zamierzamy uruchomić w fazie initramfs/initrd nie będzie miała dostępu do
kluczy w katalogu `/etc/dropbear/` , bo ten katalog rezyduje w obrębie głównego systemu plików,
który na tym etapie rozruchu systemu będzie jeszcze zaszyfrowany. Dlatego te klucze trzeba
skopiować do katalogu `/etc/dropbear-initramfs/` :

    root@raspberrypi:/home/pi# cp /etc/dropbear/dropbear_* /etc/dropbear-initramfs/

W ten sposób, klucze SSH będą automatycznie podbierane ilekroć tylko będziemy generować obraz
initramfs/initrd.

### Plik /etc/dropbear-initramfs/authorized_keys

Po tej całej zabawie z przerabianiem kluczy SSH, musimy jeszcze dodać publiczny klucz SSH maszyny, z
której zamierzamy logować się w fazie initramfs/initrd w celu otworzenia kontenera LUKS. Ten klucz
trzeba dodać do pliku `/etc/dropbear-initramfs/authorized_keys` , który standardowo nie istnieje i
trzeba go utworzyć. Możemy naturalnie skopiować wcześniej utworzony (podczas konfiguracji OpenSSH)
plik `/root/.ssh/authorized_keys` , bo zawiera on już potrzebny nam publiczny klucz SSH:

    root@raspberrypi:/home/pi# cp /root/.ssh/authorized_keys /etc/dropbear-initramfs/authorized_keys

### Plik /etc/dropbear-initramfs/config

Kolejną rzeczą, którą musimy ogarnąć, to sposób w jaki usługa `dropbear` będzie uruchamiana w fazie
initramfs/initrd. Konkretnie chodzi o flagi dla demona, które musimy ustawić w pliku
`/etc/dropbear-initramfs/config` . Potrzebujemy czegoś na wzór poniższej linijki:

    DROPBEAR_OPTIONS="-Fsjk -p 2222 -c /bin/cryptroot-unlock"

Poniżej znajduje się [wyjaśnienie tych bardziej użytecznych opcji][2], które przydałoby się
określić w  `DROPBEAR_OPTIONS` , choć niekoniecznie ich wszystkich będziemy potrzebować:

- `-F` -- uruchamia serwer SSH na pierwszym planie zamiast w tle.
- `-E` -- włącza logowanie błędów na standardowe wyjście, tj. logi pojawią się w terminalu zamiast
          przekierowania ich do logu systemowego. Ta opcja może nam się przydać w fazie testów, by
          sprawdzić czy serwer SSH w ogóle łapie zapytania od klientów i czy je poprawnie
          przetwarza.
- `-s` -- wyłącza uwierzytelnianie za pomocą hasła. Tylko przy pomocy klucza SSH będzie można się
          zalogować na serwer.
- `-j` oraz `-k` -- wyłączają port forwarding.
- `-p` -- określa port, na którym usługa `dropbear` będzie nasłuchiwać zapytań z sieci.
- `-c` -- po pomyślnym zalogowaniu się na serwer, polecenie w tym parametrze zostanie wykonane. Ta
          opcja zarazem sprawia, że użytkownik nie będzie w stanie wykonać żadnego innego polecenia
          interaktywnie po zalogowaniu się na serwer SSH w fazie initramfs/initrd.
- `-R` -- ma na celu generować z każdym rozruchem systemu nowy prywatny klucz SSH. Nie trzeba
          dodawać tej opcji, bo przydaje się ona jedynie na wypadek, gdyby serwer miał problemy z
          brakującym kluczem.

## Adresacja IP dla maszyny w fazie initramfs/initrd

Problem przed jakim teraz stoimy to odpowiedź na pytanie w jaki sposób maszyna mająca uruchomioną
usługę SSH w fazie initramfs/initrd może być w stanie odebrać jakiekolwiek połączenie sieciowe?
Standardowo przecież na tym etapie startu systemu nie ma jeszcze podniesionych żadnych interfejsów
sieciowych, ani też żadne usługi sieciowe nie działają. Jak zatem skonfigurować adresację IP dla
maszyny w fazie initramfs/initrd? By to zadanie zrealizować trzeba dopisać do pliku
`/etc/initramfs-tools/initramfs.conf` [parametr IP][3], którego składnia znajduje się poniżej:

    IP=<client-ip>:<server-ip>:<gw-ip>:<netmask>:<hostname>:<device>:<autoconf>:<dns0-ip>:<dns1-ip>:<ntp0-ip>

Nie wszystkie te pola są wymagane, a na nasze potrzeby możemy podać tylko te poniższe (dla
statycznej adresacji IP):

    IP="192.168.1.239::192.168.1.1:255.255.255.0:rpi:eth0:none"

Poniżej jest zaś wersja dla konfiguracji via DHCP:

    IP="::::rpi:eth0:dhcp"

Bez względu na to z której opcji skorzystamy, nasz Raspberry Pi w fazie initramfs/initrd będzie
miał skonfigurowaną adresację IP i będzie w stanie odbierać zapytania SSH pochodzące z sieci.

### Ograniczenia związane z adresacją IP (problem z WiFi)

Problem z tak skonfigurowaną adresacją IP jest jeden. Mianowicie, działa ona jedynie na przewodowym
interfejsie Raspberry Pi, tj. `eth0` . Jeśli nasza malina łączy się z siecią bezprzewodowo przez
interfejs WiFi ( `wlan0` ), to w taki sposób nie damy rady uzyskać połączenia po SSH.

Brak wsparcia dla WiFi w tym przypadku bierze się w zasadzie z faktu, że w obrazie initramfs/initrd
brakuje oprogramowania odpowiedzialnego za zestawienie połączenia WiFi, np. `wpa_supplicant` czy
`wpa_cli` , nie wspominając o samych sterownikach do karty WiFi. Brakuje też konfiguracji sieci
WiFi trzymanej w pliku `/etc/wpa_supplicant.conf` .

Z ogólniejszego przejrzenia internetów wynika jednak, że takie [wsparcie dla WiFi w fazie
initramfs/initrd][7] można dorobić ale wymaga to trochę pracy. Jeśli ktoś jest zainteresowany tym
tematem, to stosowne instrukcje zostały zamieszczone w osobnym artykule.

## Plik /etc/crypttab

Jeśli zaś chodzi o zawartość pliku `/etc/crypttab` , to jest ona w zasadzie standardowa, tj. taka
sama co w przypadku zwykłego otwierania kontenera LUKS przy pomocy manualnie wpisywanego hasła z
klawiatury podłączonej do Raspberry Pi. Poniżej znajduje się przykładowy wpis w tym pliku, tak by
nie było żadnych niedopowiedzeń i niejasności:

    rpi_crypt  UUID=0b9b66eb-d5ec-4371-80e3-f3a6ae92e0be   none  luks,initramfs,keyslot=0

## Regeneracja obrazu initramfs/initrd

Po dostosowaniu tych wszystkich wyżej wymienionych rzeczy trzeba będzie wygenerować nowy obraz
initramfs/initrd. Robimy to tradycyjnie przy wykorzystaniu polecenia `update-initramfs` :

    root@raspberrypi:/home/pi# update-initramfs -u -k $(uname -r)
    ln: failed to create hard link '/boot/initrd.img-5.10.63-v7l+.dpkg-bak' => '/boot/initrd.img-5.10.63-v7l+': Operation not permitted
    update-initramfs: Generating /boot/initrd.img-5.10.63-v7l+
    Building v7l+ image, updating top initrd

Warto tutaj zaznaczyć, że podczas instalowania pakietu `dropbear` , w terminalu można było dostrzec
ten poniższy komunikat:

    dropbear: WARNING: Invalid authorized_keys file, remote unlocking of cryptroot via SSH won't work!

Po poprawnym skonfigurowaniu serwera SSH, to ostrzeżenie powinno zniknąć.

## Próba odszyfrowania kontenera LUKS przez SSH

Skrypt shell'owy `cryptroot-unlock` , który został podany w parametrze `-c` (w pliku
`/etc/dropbear-initramfs/config` ) ma za zadanie odblokować zaszyfrowany kontener. W ten sposób, od
razu po zalogowaniu się via SSH zobaczymy tekst `Please unlock disk rpi_crypt:` i jedyne co
będziemy mogli w takiej sytuacji zrobić, to wpisać hasło do zaszyfrowanego kontenera LUKS, co
wygląda mniej więcej tak:

    $ ssh 192.168.1.239
    Please unlock disk rpi_crypt:

Po wpisaniu hasła, po chwili pojawi się taki oto błąd i połączenie z serwerem SSH zostanie
zamknięte:

    Error: Timeout reached while waiting for PID 215.
    Connection to 192.168.1.239 closed.

Póki co nie wiem z czego wynika ten błąd, niemniej jednak, jeśli wpisaliśmy prawidłowe hasło, to
zaszyfrowany kontener LUKS zostanie otworzony i system się uruchomi w dokładnie taki sam sposób jak
byśmy to hasło wpisali ręcznie z klawiatury podłączonej do Raspberry Pi.

Jeśli zaś rzucimy okiem na monitor podłączony bezpośrednio do naszej maliny, to przed wpisaniem
hasła będziemy mieli na nim takie oto poniższe informacje:

![raspberry-pi-rpi-ssh-dropbear-initramfs-initrd-luks-ip-address](/img/2021/11/001.raspberry-pi-rpi-ssh-dropbear-initramfs-initrd-luks-ip-address.jpg#huge)

Jak widać, mamy tutaj wypisaną dokładną konfigurację adresacji IP, którą sobie określiliśmy w pliku
`/etc/initramfs-tools/initramfs.conf` za sprawą parametru `IP` . Włączenie możliwości wpisania
hasła do zaszyfrowanego kontenera LUKS przez SSH nie odbiera nam opcji wpisania tego hasła w
tradycyjny sposób, co możemy poznać po prompt, który na powyższej fotce oczekuje od nas wpisania
hasła. Po wpisaniu hasła przez SSH, start systemu przebiega już normalnie:

![raspberry-pi-rpi-ssh-dropbear-initramfs-initrd-luks-unlock-system](/img/2021/11/002.raspberry-pi-rpi-ssh-dropbear-initramfs-initrd-luks-unlock-system.jpg#huge)

## Problemy związane z bezpieczeństwem kluczy prywatnych

To przedstawione wyżej rozwiązanie z ujednoliceniem kluczy serwerów OpenSSH i Dropbear oraz
zsynchronizowanie tych kluczy z tymi obecnymi w fazie initramfs/initrd nie jest pozbawione wad.
Trzeba sobie zdawać sprawę, że obraz initramfs/initrd obecny na partycji `/boot/` w zasadzie nie
jest chroniony w żaden sposób. Każda osoba mająca fizyczny dostęp do Raspberry Pi potencjalnie może
uzyskać również dostęp do tych kluczy. Jeśli ktoś wejdzie w ich posiadanie, to będzie w stanie
przechwycić całą komunikację po SSH, w tym też hasło do kontenera LUKS.

Może się wydawać, że lepszym rozwiązaniem byłoby rozdzielenie kluczy SSH, tj. osobne klucze SSH dla
serwera OpenSSH działającego po odszyfrowaniu systemu i osobne klucze SSH dla Dropbear
działającego w fazie initramfs/initrd. Niemniej jednak, takie rozdzielenie kluczy niewiele nam da,
bo w dalszym ciągu obraz initramfs/initrd pozostaje niechroniony, przez co potencjalny złoczyńca
jest w stanie podsłuchać komunikację z serwerem SSH, którą prowadzimy w fazie initramfs/initrd, a
przecie to w tej fazie podajemy hasło do kontenera LUKS. Jeśli atakujący jest w stanie przechwycić
hasło do kontenera LUKS, to będzie miał dostęp do naszego systemu i bez znaczenia jest to jakie
zabezpieczenia w tym systemie sobie wdrożymy.

## Podsumowanie

Problem z odszyfrowaniem kontenera LUKS przez SSH w fazie initramfs/initrd jest w zasadzie jeden,
tj. niechroniony obraz initramfs/initrd, który może skompromitować bezpieczeństwo systemu w sposób,
którego na pierwszy rzut oka się nawet nie da wykryć. Podobnie sprawa wyglądała w przypadku
wykorzystywania pendrive jako klucz do kontenera LUKS. Te dwa rozwiązania łączy niezabezpieczony
obraz initramfs/initrd czyniąc je mało użytecznymi. Korzystanie z kluczy SSH ma jednak tę przewagę
nad kluczem w postaci pendrive, że hasło zawsze pozostaje w naszej głowie, a nie na nośniku, który
ktoś potencjalnie może znaleźć. Jeśli się zorientujemy w porę, że coś może być nie tak, to jest
jakaś szansa, że uda nam się zaszyfrować system Raspberry Pi (np. odcinając od niego zasilanie) i
pozostanie on w takim stanie na już zawsze.


[1]: https://bbs.archlinux.org/viewtopic.php?id=250512
[2]: https://manpages.debian.org/unstable/dropbear-bin/dropbear.8.en.html
[3]: https://www.kernel.org/doc/html/latest/admin-guide/nfs/nfsroot.html
[4]: https://manpages.debian.org/unstable/openssh-server/sshd_config.5.en.html
[5]: /post/jak-zaszyfrowac-raspberry-pi-raspios-raspbian-luks/
[6]: /post/wykorzystanie-nosnika-usb-jako-klucz-do-odszyfrowania-raspberry-pi/
[7]: /post/wsparcie-dla-wifi-w-initramfs-initrd-by-odszyfrowac-luks-przez-ssh-bezprzewodowo/
