---
author: Morfik
categories:
- Linux
date: "2015-10-19T21:01:03Z"
date_gmt: 2015-10-19 19:01:03 +0200
published: true
status: publish
tags:
- pendrive
- bezpieczeństwo
- moduły-pam
title: Zabezpieczenie konta root przy pomocy pam-usb
---

Jakiś czas temu natknąłem się na moduł `pam-usb` , który to w dość ciekawy sposób zabezpiecza dostęp
do konta użytkownika root. [Cały mechanizm opiera się o pendrive](https://wiki.debian.org/pamusb),
którego to unikalne cechy są brane pod uwagę przy uwierzytelnianiu podczas logowania się na konto
super użytkownika, czyli min. gdy wydajemy polecenie `su` albo `sudo` . Jest to o tyle ciekawa
rzecz, że konto użytkownika root możne stać się niewrażliwe na próby złamania hasła w przypadku
połączenia sieciowego. Jak by nie patrzeć, atakujący, który łączy się zdalnie, nie jest w stanie
podłączyć do naszego komputera żadnego fizycznego urządzenia, w wyniku czego nigdy nie uzyska
dostępu do konta administratora.

Pakiet `libpam-usb` wyleciał z debiana jakiś czas temu. [Powodem
były](https://tracker.debian.org/news/686153) zależności, które wskazywały na przestarzały już
pakiet `udisks` . Poza tym, nikt nie zajmował się tym pakietem. Obecnie jest on dostępny jedynie w
starszych wydaniach debiana.

<!--more-->
## Aktywowanie modułu pam-usb

Moduł `pam-usb` daje nam dwie możliwości, z których pierwsza zakłada wykorzystanie pendrive'a jako
dodatkowego zabezpieczenia, czyli potrzebne będzie hasło do konta root oraz wymagana będzie obecność
pendrive'a w porcie usb. Drugą opcją jest zrezygnowanie z hasła na rzecz obecności pendrive'a w
porcie usb komputera. Ten drugi sposób może ułatwić trochę życie ale trzeba wziąć pod uwagę, że w
przypadku braku pendirve'a, zalogowanie na konto root będzie możliwe po podaniu hasła, czyli tak jak
w przypadku standardowego logowania się na konto administratora systemu.

By aktywować to zabezpieczenie, musimy mieć w systemie te dwa pakiety `libpam-usb` oraz
`pamusb-tool` . Po ich instalacji, odpalamy `pam-auth-update` i zaznaczamy pozycję `USB
authentication` :

    ┌──────────────────┤ PAM configuration ├───────────────────┐
    │ PAM profiles to enable:                                  │
    │                                                          │
    │    [*] USB authentication                                │
    │    [ ] encfs encrypted home directories                  │
    │    [*] Unix authentication                               │
    │    [ ] Mount volumes for user                            │
    │    [*] GNOME Keyring Daemon - Login keyring management   │
    │    [*] ConsoleKit Session Management                     │
    │    [*] Inheritable Capabilities Management               │
    │              <Ok>                  <Cancel>              │
    │                                                          │
    └──────────────────────────────────────────────────────────┘

## Konfiguracja modułu pam-usb

Moduł `pam-usb` konfigurujemy przy pomocy narzędzia `pamusb-conf` . Wkładamy zatem pendrive do portu
usb i wydajemy poniższe polecenie:

    # pamusb-conf --add-device=pendrak
    Please select the device you wish to add.
    * Using "Kingston DT 101 G2 (001CB0EC34A2BC318709104B)" (only option)

    Which volume would you like to use for storing data ?
    0) /dev/sdb5 (UUID: 9365d879-1715-4346-8fc0-7674684765e7)
    1) /dev/sdb6 (UUID: <UNDEFINED>)
    2) /dev/sdb4 (UUID: <UNDEFINED>)
    3) /dev/sdb1 (UUID: 6bf4d915-2b62-444e-a2c8-16307769b5c2)
    4) /dev/sdb2 (UUID: 90ec6f73-8fdb-4c8d-aebd-cadd0f51b412)
    5) /dev/sdb3 (UUID: 0dd5e51c-c133-492a-a6ca-d14e0c7d1e39)

    [0-5]: 5

    Name            : pendrak
    Vendor          : Kingston
    Model           : DT 101 G2
    Serial          : 001CB0EC34A2BC318709104B
    UUID            : 0dd5e51c-c133-492a-a6ca-d14e0c7d1e39

    Save to /etc/pamusb.conf ?
    [Y/n] y
    Done.

Można oczywiście ręcznie edytować plik `/etc/pamusb.conf` ale lepiej jest korzystać z przeznaczonych
do tego celu narzędzi, bo uchroni nas to przez popełnieniem literówek i innych drobnych błędów.

Definiujemy teraz użytkownika, który będzie podlegał pod moduł `pam-usb` . W tym przypadku jest to
`root` :

    # pamusb-conf --add-user=root
    Which device would you like to use for authentication ?
    0) bohun
    1) pendrak

    [0-1]: 1

    User            : root
    Device          : pendrak

    Save to /etc/pamusb.conf ?
    [Y/n] y
    Done.

Plik konfiguracyjny powinien zostać uzupełniony o odpowiednie wpisy, zawsze można go podejrzeć i
sprawdzić czy aby wszystko jest tak jak być powinno.

## Próba logowania na konto root

Jeśli w tej chwili spróbowalibyśmy się zalogować na konto użytkownika root, powinniśmy zobaczyć
poniższy log:

    $ su
    * pam_usb v0.5.0
    * Authentication request for user "root" (su)
    * Device "pendrak" is connected (good).
    * Performing one time pad verification...
    * Regenerating new pads...
    * Access granted.
    root:~#

Jak widzimy wyżej, dostęp został przyznany bez proszenia o hasło, bo pendrive jest podpięty. Jeśli
odłączymy pendrive i ponownie spróbujemy się zalogować, powinniśmy zostać poproszeni o hasło:

    morfik:~$ su
    * pam_usb v0.5.0
    * Authentication request for user "root" (su)
    * Device "pendrak" is not connected.
    * Access denied.
    Password:
    root:~#

## Dwuskładnikowe uwierzytelnianie

Jeśli chcemy z modułu `pam-usb` uczynić dodatkowe zabezpieczenie działające na zasadzie
dwuskładnikowego uwierzytelniania, musimy wyedytować plik `/etc/pam.d/common-auth` i zmienić w nim
linijkę z:

    auth   sufficient      pam_usb.so

na:

    auth   required        pam_usb.so

Po zapisaniu zmian, nie da rady zalogować się już na konto root bez uprzedniego podpięcia pendrive'a
do portu usb i to nawet po poprawnym podaniu hasła:

    morfik:~$ su
    * pam_usb v0.5.0
    * Authentication request for user "root" (su)
    * Device "pendrak" is not connected.
    * Access denied.
    Password:
    su: Authentication failure

Opcja dwuskładnikowego uwierzytelniania skutecznie może nas ochronić przed wszystkimi atakami
zdalnymi zakładającymi wykorzystanie hasła podczas uzyskiwania dostępu do konta root. Trzeba jednak
uważać na jedną rzecz. Jednym z parametrów pendrive branym pod uwagę w procesie uwierzytelniania
jest UUID wskazanej partycji. Jeśli ten numerek ulegnie zmianie, np. wskutek tworzenia nowego
systemu plików, stracimy dostęp do konta root i trzeba będzie ratować się systemem live.
