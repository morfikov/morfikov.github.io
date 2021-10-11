---
author: Morfik
categories:
- Linux
date: "2015-11-07T10:12:15Z"
date_gmt: 2015-11-07 09:12:15 +0100
published: true
status: publish
tags:
- systemd
GHissueID: 276
title: Skrót Ctrl-Alt-Del w systemd
---

Po przesiadce z sysvinit na systemd okazało się, że system inaczej się zachowuje po przyciśnięciu
kombinacji klawiszy Ctrl-Alt-Del . Niby za wiele nie zmieniałem w konfiguracji systemu ale w żaden
sposób przy pomocy plików konfiguracyjnych nie szło zmienić zachowania tego powyższego skrótu.
Okazuje się bowiem, że w systemd, akcję pod ten skrót przypisuje się w nieco innym miejscu niż to
było robione w sysvinit. W tym wpisie postaramy się zmienić domyślne zachowanie tego skrótu, tak by
po jego przyciśnięciu wyłączyć komputer.

<!--more-->
## Plik /etc/inittab

Za czasów sysvinit konfiguracja skrótu Ctrl-Alt-Del była określana w pliku `/etc/inittab` .
Odpowiadała za niego ta poniższa linijka:

    # What to do when CTRL-ALT-DEL is pressed.
    ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now

Zatem bez większego problemu byliśmy w stanie zmienić to polecenie na takie, które uważaliśmy za
odpowiednie. Jeśli już coś chcieliśmy tutaj przestawić, to zwykle opcję `-r` podmienialiśmy na
`-h` , co powodowało, że po przyciśnięciu klawiszy Ctrl-Alt-Del , system się wyłączał. Dodatkowo, po
skorzystaniu z opcji `-a` mieliśmy możliwość określenia w pliku `/etc/shutdown.allow` użytkowników,
którzy byli w stanie używać tego skrótu. [Systemd nie korzysta z pliku /etc/inittab][1], dlatego
też te powyższe ustawienia nie mają większego sensu.

## Target ctrl-alt-del.target

W systemd do obsługi skrótu Ctrl-Alt-Del używa się `ctrl-alt-del.target` . W zależności od tego
gdzie zostanie on podlinkowany, to taka akcja zostanie wykonana. Domyślnie ten target wskazuje na
`reboot.target` :

    # ls -al /lib/systemd/system/ctrl-alt-del.target
    lrwxrwxrwx 1 root root 13 2015-10-09 13:58:24 /lib/systemd/system/ctrl-alt-del.target -> reboot.target

I to właśnie dlatego po przyciśnięciu Ctrl-Alt-Del , komputer zostaje zresetowany. W przypadku
gdybyśmy chcieli zmienić tę akcję i zamiast resetowania wyłączyć maszynę, to `ctrl-alt-del.target`
musimy podlinkować do `poweroff.target` :

    # ln -s /lib/systemd/system/poweroff.target /etc/systemd/system/ctrl-alt-del.target
    # systemctl daemon-reload


[1]: https://lists.debian.org/debian-user/2014/02/msg00826.html
