---
author: Morfik
categories:
- Linux
date: "2015-10-19T22:32:41Z"
date_gmt: 2015-10-19 20:32:41 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- szyfrowanie
- swap
title: Zaszyfrowana przestrzeń wymiany SWAP
---

Opisując mechanizm szyfrowania katalogu domowego przy pomocy [narzędzia
encfs](/post/szyfrowanie-katalogu-home-przy-pomocy-encfs/) , wspomniałem o
problemie jaki powstaje przy jednoczesnym braku szyfrowania przestrzeni wymiany SWAP. Oczywiście,
jeśli posiadamy w systemie dużą ilość pamięci RAM, to raczej nie potrzebna nam jest przestrzeń
wymiany. Podobnie sprawa ma się w przypadku, gdy nie korzystamy z hibernacji. Natomiast, jeśli jedna
z naszych partycji jest sformatowana jako SWAP i aktywnie z niej korzystamy, to niepełne szyfrowanie
dysku, jakie zapewnia `encfs` może doprowadzić do skompromitowania zaszyfrowanych danych.

<!--more-->
## Problematyczna przestrzeń wymiany

Wrażliwe dane, takie jak klucze szyfrujące, czy też hasła do tych kluczy, są przechowywane w pamięci
operacyjnej RAM. W przypadku gdy ilość obecnej w systemie pamięci nie pokrywa w pełni jego potrzeb,
kernel broni się przed powieszeniem przez zrzucanie danych do SWAP. Te wszystkie newralgiczne dane
mogą być w ten sposób złapane przez SWAP i zapisane na dysku w formie niezaszyfrowanej. Podobnie
sprawa wygląda w przypadku hibernacji, gdzie system zrzuca całą pamięć na dysk.

## Tworzenie szyfrowanego kontenera pod przestrzeń wymiany

Oczywistym zatem jest, że jeśli przestrzeń wymiany jest obecna w systemie i wykorzystujemy jakiś
mechanizm szyfrujący, musimy zatroszczyć się ten SWAP i zaszyfrować go. W ten sposób dane, które do
niego trafią, będą również szyfrowane. Tworzymy zatem partycję, np. w `gparted` i robimy szyfrowany
kontener wydając w terminalu to poniższe
    polecenie:

    # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 5000 --use-random --verify-passphrase --verbose luksFormat /dev/sda3

Następnie otwieramy kontener:

    # cryptsetup luksOpen /dev/sda3 swap
    Enter passphrase for /dev/sda3:

## Tworzenie SWAP

Przestrzeń wymiany SWAP tworzymy korzystając ze standardowego narzędzia `mkswap` . Ważne jest by
operować na urządzeniach w katalogu `/dev/mapper/` :

    # mkswap /dev/mapper/swap

## Konfiguracja fstab i cryptab

Zwykle UUID tak utworzonego SWAP zostanie nam podane chwilę po wydaniu powyższego polecenia. Jeśli
jednak z jakichś powodów przeoczyliśmy ten numerek, możemy go odczytać z wyjścia `lsblk` :

    # lsblk -o name,mountpoint,uuid /dev/sda
    NAME            MOUNTPOINT UUID
    sda
    ...
    ├─sda3                     95f1f54d-a1d6-4e02-8ee6-7f79d7558ebd
    │ └─swap (dm-0)            22614608-5406-4562-8532-9e8203a7721d
    ...

Mając UUID, trzeba uzupełnić odpowiednie pliki.

Plik `/etc/fstab` :

    UUID=22614608-5406-4562-8532-9e8203a7721d            swap    swap              0       0

Plik `/etc/crypttab` :

    swap   UUID=95f1f54d-a1d6-4e02-8ee6-7f79d7558ebd none  luks

## Obsługa hibernacji

By hibernacja była możliwa, system musi wiedzieć, z jakiego miejsca ma zacząć wczytywać dane po
uruchomieniu komputera. Musimy zatem dodać poniższy wpis do pliku
`/etc/initramfs-tools/conf.d/resume` :

    RESUME=UUID=22614608-5406-4562-8532-9e8203a7721d

Oraz przeładować initramfs:

    # update-initramfs -u -k all

Te kilka powyższych kroków znacznie zwiększa bezpieczeństwo danych, które są przechowane w takim
systemie. Dlatego też zawsze pamiętajmy o tym by zaszyfrować SWAP.
