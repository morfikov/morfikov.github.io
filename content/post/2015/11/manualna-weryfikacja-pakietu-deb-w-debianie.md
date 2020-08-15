---
author: Morfik
categories:
- Linux
date: "2015-11-02T00:16:03Z"
date_gmt: 2015-11-01 22:16:03 +0100
published: true
status: publish
tags:
- gpg
- debian
- apt
- aptitude
title: Manualna weryfikacja pakietu deb w debianie
---

W dobie całego tego świata informatycznego zwykliśmy polegać na osobach, których nigdy w życiu na
oczy nie wiedzieliśmy, nie wspominając o jakimkolwiek kontakcie fizycznym. Zaufanie to obecnie chyba
najbardziej krytyczna luka bezpieczeństwa jeśli chodzi o oprogramowanie, z którego korzystamy na co
dzień. My, którzy używamy debiana w swojej pracy, polegamy na mechanizmach jakie oferuje nam `apt`
czy `aptitude` przy [weryfikacji pakietów przed ich instalacją](https://wiki.debian.org/SecureApt) w
systemie. Co się jednak by stało gdyby w tych menadżerach pojawił się błąd, który by uniemożliwiał
poprawną weryfikację pakietów? Skąd wiemy czy te mechanizmy zabezpieczające w ogóle działają? Może
one nam dają jedynie fałszywe poczucie bezpieczeństwa, a tak naprawdę przez niczym nas nie chronią?
W tym wpisie postaramy się odpowiedzieć na te powyższe pytania i sprawdzimy czy manualna weryfikacja
pakietu jest w ogóle możliwa

<!--more-->
## Dlaczego weryfikacja pakietu jest możliwa

Każde repozytorium, z którego możemy pobierać pakiety zwykle jest podpisane. Wszystkie repozytoria,
które nie są podpisane, powinniśmy omijać szerokim łukiem. Podobnie sprawa ma się w przypadku tych
źródeł pakietów, gdzie nie możemy zweryfikować właścicieli klucza, którym podpisane jest takie
repozytorium. W tym wpisie skupimy się jedynie na tych repozytoriach, które są podpisane, a możemy
to poznać po obecności plików `Release` , `Release.gpg` lub `InRelease` . Gdy dodajemy adres nowego
repozytorium do pliku `/etc/apt/sources.list` i aktualizujemy listy pakietów via `apt-get update` ,
te pliki powinny zostać pobrane do katalogu `/var/lib/apt/lists/` .

## Pliki Release, Release.gpg oraz InRelease

Pierwszym krokiem będzie zatem zweryfikowanie danych zawartych w pliku `Release` przy pomocy
sygnatury przechowywanej w pliku `Release.gpg` . Plik `InRelease` różni się od tych dwóch wyżej
tylko tym, że zarówno dane jak i sygnatura są przechowywane w jednym pliku, a nie dwóch. W
zależności od tego z jakim repozytorium mamy do czynienia, weryfikację danych możemy przeprowadzić
w poniższy sposób:

    # cd /var/lib/apt/lists/

    # gpg --verify ftp.de.debian.org_debian_dists_sid_InRelease
    # gpg --verify dl.google.com_linux_chrome_deb_dists_stable_Release.gpg dl.google.com_linux_chrome_deb_dists_stable_Release

Pamiętajmy, że by zweryfikować podpis złożony w pliku, potrzebujemy klucza publicznego osoby, która
podpisała plik. Jeżeli przy weryfikacji otrzymamy informację, że `gpg: Good signature from` ,
oznacza to, że proces weryfikacji danych w powyższych plikach zakończył się powodzeniem.

## Suma kontrolna pliku Packages

Plik `Packages` zawiera opisy poszczególnych pakietów, które są dostępne w repozytorium. Są też tam
ich sumy kontrolne. Musimy zatem zweryfikować czy plik `Packages` nie został w żaden sposób
zmieniony. Wyciągamy zatem jego sumy kontrolne (md5, sha1, sha256):

    # sed -n "s,main/binary-amd64/Packages$,,p" ftp.de.debian.org_debian_dists_sid_InRelease
     585d2dfeeae78e0c6065a6d489c8e36a 38576950
     cc0e8217c2c133f8baff38790c0d8b44d5248cd7 38576950
     896abb4c2c12db3c18c7b87a87efaeecc9b8cbeb541dae650468549e443a1b5b 38576950

Tworzymy sumy kontrolne pliku `Packages` , który mamy w cache i porównujemy je z tymi zapisanymi w
pliku, w tym przypadku, `InRelease` :

    # sha256sum ftp.de.debian.org_debian_dists_sid_main_binary-amd64_Packages
    896abb4c2c12db3c18c7b87a87efaeecc9b8cbeb541dae650468549e443a1b5b  ftp.de.debian.org_debian_dists_sid_main_binary-amd64_Packages

Nie musimy sprawdzać wszystkich sum kontrolnych, wystarczy jedna i jak widzimy wyżej, plik
`Packages` nie został zmieniony.

## Suma kontrolna pakietu

Pozostało nam jeszcze zweryfikowanie sumy kontrolnej samego pakietu `.deb` . Weźmy dla przykładu
pakiet gajim. Wyciągamy pierw sumę kontrolną tego pakietu z opisu przy pomocy `apt-cache` :

    # apt-cache show gajim | sed -n "s/SHA256: //p"
    823a3d7b034a6cb4531f8ff790cbc9af18ae66b65be67b9c4e0a2c203142da89

Musimy teraz jeszcze uzyskać sumę kontrolną paczki, która trafiła do cache APT:

    # sha256sum /var/cache/apt/archives/gajim_0.16-1_all.deb
    823a3d7b034a6cb4531f8ff790cbc9af18ae66b65be67b9c4e0a2c203142da89  /media/Kabi/backup_packages/apt/archives/gajim_0.16-1_all.deb

Sumy się zgadzają zatem możemy mieć pewność, że ten plik trafił do nas w takiej formie w jakiej
został wgrany do repozytorium debiana. Mając na uwadze powyższe informacje, weryfikacja pakietu
zakończyła się sukcesem.
