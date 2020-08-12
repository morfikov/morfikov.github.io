---
author: Morfik
categories:
- Linux
date: "2015-11-02T20:20:09Z"
date_gmt: 2015-11-02 18:20:09 +0100
published: true
status: publish
tags:
- debian
- apt/aptitude
title: Konfiguracja multiarch w dystrybucji Debian
---

Posiadając nowszej klasy procesor, jesteśmy w stanie korzystać z 64 bitowego systemu operacyjnego. W
przypadku windowsów uruchamianie aplikacji 32 czy 64 bitowych nie stanowi większego problemu. W na
debianie sprawa wygląda nieco inaczej. Gdy mamy wgranego 64 bitowego debiana, aplikacje 32 bitowe
nie będą chciały się nam odpalić. Wszystkiemu winne są biblioteki 32 bitowe, które są wykorzystywane
przez dany program, a bez nich on zwyczajnie nie może działać. Jednym z rozwiązań tego problemu może
być [kontener LXC]({{< baseurl >}}/post/konfiguracja-kontenerow-lxc/), gdzie jesteśmy w stanie
zainstalować 32 bitowy system wewnątrz środowiska 64 bitowego i to z tego systemu możemy uruchamiać
32 bitowe aplikacje. Skonfigurowanie takiego kontenera może być nieco skomplikowane, dlatego też
dużo lepszym rozwiązaniem jest przerobienie naszego 64 bitowego systemu na
[muliarch](https://wiki.debian.org/Multiarch), czyli taki, który jest w stanie obsługiwać wiele
architektur (multiarch).

<!--more-->
## Problem tyczące się multiarch

W debianie multiarch mamy możliwość zainstalowania w tym samym czasie bibliotek różnych architektur,
przykładowo, możemy mieć obok siebie `libssl1.0.2:amd64` i `libssl1.0.2:i386` . Niemniej jednak, ten
sam manewr nie jest jeszcze możliwy w przypadku plików binarnych. Dlatego też jeśli o nie chodzi, to
możemy posiadać w systemie albo 32 bitowego Firefox'a, albo jego 64 bitowy odpowiednik. Oczywiście
to obostrzenie dotyczy jedynie aplikacji instalowanych za pomocą menadżerów pakietów, a to z tego
względu, że podczas instalacji pliki wędrują w to samo miejsce w strukturze katalogów.

## Sprawdzenie architektury systemu

Jeśli z jakichś powodów nie jesteśmy pewni jaką architekturę ma nasz (czy też jakikolwiek inny)
system, to w bardzo prosty sposób jesteśmy w stanie to sprawdzić. Odpalamy terminal i wpisujemy w
nim to poniższe polecenie:

    $ dpkg --print-architecture
    amd64

W powyższym przypadku, `amd64` wskazuje jednoznacznie, że jest to 64 bitowy system.

## Dodawanie i usuwanie architektur

Dodawanie architektur nie powinno stanowić większego problemu. Zatem by dodać do naszego 64 bitowego
systemu architekturę 32 bitową, musimy w terminalu jako root wpisać to poniższe polecenie:

    # dpkg --add-architecture i386

Po tym kroku, nasz system powinien stać się multiarch. Zweryfikować to możemy przez sprawdzenie czy
mamy w nim jakieś zewnętrzne architektury, przykładowo:

    $ dpkg --print-foreign-architectures
    i386

W przypadku gdy nasz system nie byłby multiarch, to powyższe polecenie nie zwróciło by żadnego
wyniku.

Architektury możemy również usuwać z systemu. Jedyne ograniczenie jest takie, że nie możemy wywalić
tej bazowej. Wszystkie inne, które zostają zwrócone przy `--print-foreign-architecture` możemy bez
problemu usunąć jeśli zajdzie taka potrzeba:

    # dpkg --remove-architecture i386

## Repozytoria dla multiarch w pliku sources.list

Po dodaniu nowej architektury musimy zaktualizować listy pakietów w repozytoriach przy pomocy
`apt-get update` . Dopiero po tym kroku będziemy w stanie zainstalować pakiety innych architektur.
Jeśli się przyjrzymy nieco bliżej temu co `apt-get` wypisuje na ekranie, powinniśmy tam dostrzec
wpisy dwóch architektur, przykładowo:

    ...
    Get:5 http://ftp.de.debian.org sid/non-free amd64 Packages/DiffIndex [4,093 B]
    ...
    Get:8 http://ftp.de.debian.org sid/non-free i386 Packages/DiffIndex [4,231 B]
    ...

W taki sposób, listy pakietów wszystkich repozytoriów, które mamy wpisane do pliku
`/etc/apt/sources.list` zostaną pobrane dla tych dwóch architektur. Nie zawsze jednak tego typu
zachowanie jest pożądane. Są przecie repozytoria, które mogą udostępniać pakiety jedynie dla
wybranej architektury. Możemy też zwyczajnie nie chcieć pobierać list pakietów pewnych architektur z
określonych repozytoriów. W takim przypadku musimy poinformować `apt-get` by pobierał jedynie te
listy, których architektury mu wskażemy. W tym celu edytujemy plik `sources.list` i zmieniamy w nim
pożądane repozytoria dopisując architektury przed ich adresami:

    deb [arch=amd64,i386] http://ftp.de.debian.org/debian/ sid main non-free contrib

W przypadku źródeł pakietów ( `deb-src` ) nie ma potrzeby określać architektur.

## Instalowanie i usuwanie pakietów w multiarch

Mając na pokładzie dwie architektury musimy jeszcze nauczyć się jak instalować pakiety. Domyślnie po
wydaniu polecenia `aptitude install` pakiety będą pobierane dla bazowej architektury naszego
systemu. Jeśli chcemy zaś zainstalować aplikację czy bibliotekę 32 bitową, to musimy to wyraźnie
określić przy instalacji dodając `:i386` do nazwy pakietu, przykładowo:

    # aptitude install libgnome-keyring0:i386

Podobnie sprawa ma się przy usuwaniu pakietów. Należy jednak pamiętać, że przed usunięciem
architektury z systemu multiarch, trzeba usunąć wszystkie pakiety, które się do niej odnoszą. W
przeciwnym wypadku dostaniemy ten oto poniższy błąd:

    # dpkg --remove-architecture i386
    dpkg: error: cannot remove architecture 'i386' currently in use by the database

W przypadku, gdy mamy sporo pakietów z tej architektury i386 i nie chce nam się ich wyrzucać jeden
po drugim, to zawsze możemy zrobić sobie listę tych pakietów via `dpkg` i podać ją do `apt-get` ,
przykładowo:

    # apt-get purge `dpkg --get-selections | grep i386 | awk '{print $1}'`

Część pakietów może mieć przy tym niespełnione zależności ale wszystkie z nich odnosić się będą w
jakiś sposób do usuwanej z systemu architektury i `apt-get` powinien sobie z tym zadaniem bez
najmniejszego problemu poradzić.
