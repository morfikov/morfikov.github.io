---
author: Morfik
categories:
- Linux
date: "2019-02-16T05:29:25Z"
published: true
status: publish
tags:
- debian
- git
GHissueID: 310
title: Jak sklonować duże repozytorium git na niestabilnym łączu sieciowym
---

Połączenie z internetem, z którego ostatnio przyszło mi korzystać, nie należało zbytnio do tych
najbardziej wydajnych pod względem prędkości przesyłu danych. O ile szybkość transferu można by
jeszcze przemilczeć, to owe łącze nie należało też do tych najstabilniejszych i czasem kontakt
ze światem był zwyczajnie zrywany. Krótko rzecz ujmując, ten net nadawał się chyba jedynie do
przeglądania stron WWW. Pech jednak chciał, że potrzebowałem zassać na takim
połączeniu [repozytorium git'a z patch'ami Debiana](https://salsa.debian.org/kernel-team/linux)
nakładanymi na kernel linux'a. To repozytorium nie należy do największych ale też do małych ono
się nie zalicza -- waży około 2 GiB. Oczywiście można by zrobić jedynie płytki klon takiego repo za
sprawą `git clone --depth=1` , co skutkowałoby pobraniem plików w wersji z ostatniego commit'a, co
z kolei zredukowałoby w znacznym stopniu wielkość pobieranych danych (parę-paręnaście MiB). Co
jednak zrobić w przypadku, gdy musimy sklonować sporych rozmiarów repozytorium git, a dysponujemy
przy tym dość wolnym połączeniem sieciowym? Czy jest to w ogóle możliwe, bo przecież git nie ma
zaimplementowanej opcji wznawiania przerwanej synchronizacji i każde zakłócenie tego procesu
skutkować będzie tym, że całe repozytorium trzeba będzie pobrać jeszcze raz i to bez względu na to,
czy proces się zawiesi zaraz na początku, czy też pod koniec operacji.

<!--more-->
## Płytki klon repozytorium git

W większości przypadków nie ma potrzeby zasysać całego repozytorium git, np. gdy nie uczestniczymy
w rozwoju jakiegoś projektu, a chcemy go sobie jedynie lokalnie zbudować i używać. Przykładem
takiego większego projektu może być Android. W jego przypadku pobieranie całego repozytorium,
które waży chyba już koło 100 GiB nie jest najlepszym wyjściem dla użytkowników, którzy by chcieli
sobie jedynie zbudować nowszą wersję ROM'u i wgrać ją na swój smartfon. Przez stworzenie płytkiego
klonu repozytorium możemy zaoszczędzić całą masę zasobów sieciowych jak i również nasz cenny czas.

Poniżej jest przykład jak tego typu płytki klon dowolnego repozytorium utworzyć:

    $ git clone --depth=1 https://salsa.debian.org/kernel-team/linux.git/
    Cloning into 'linux'...
    remote: Enumerating objects: 1089, done.
    remote: Counting objects: 100% (1089/1089), done.
    remote: Compressing objects: 100% (738/738), done.
    remote: Total 1089 (delta 48), reused 651 (delta 31)
    Receiving objects: 100% (1089/1089), 1.49 MiB | 221.00 KiB/s, done.
    Resolving deltas: 100% (48/48), done.

Jak widać wyżej, zostało pobranych niecałe 1,5 MiB z tych ponad 2,2 GiB, zatem różnica jest dość
znaczna.

Jeśli zajrzymy sobie w log, to zobaczymy tam tylko i wyłącznie jeden commit:

    $ git log --all --oneline

    7d94791 (grafted, HEAD -> refs/heads/master, refs/remotes/origin/master, refs/remotes/origin/HEAD) Update to 4.20.7

## Pogłębianie płytkiego klonu repozytorium git

Jeśli z jakichś przyczyn taki płytki klon repozytorium nas nie satysfakcjonuje, to możemy go
pogłębić. Głębokość repozytorium jest mierzona w commit'ach. To wyżej utworzone lokalne
repozytorium git ma jeden commit, więcej jego głębokość to 1.

Do pogłębiania klonu repozytorium git używa się polecenia `git fetch` , z tym, że mamy możliwość
przeprowadzenia tego procesu na dwa sposoby. Pierwszy z nich wykorzystuje parametr `--depth=` ,
który określa ilość commit'ów, które trzeba będzie dociągnąć licząc od samej góry, czyli jeśli
byśmy dali `--depth=100` , to pobranych by zostało 99 commit'ów, bo jeden już mamy u siebie (w
sumie mielibyśmy ich 100). Drugi ze sposobów wykorzystuje opcję `--deepen` , która umożliwia
pobranie dodatkowych commit'ów licząc od miejsca granicy klonu. W tym przypadku jeśli byśmy
określili `--deepen=100` , to pobranych zostałoby 100 commit'ów, przez co byśmy mieli ich 101.

Jeśli chcemy zatem pobrać 10000 commit'ów na słabej jakości łączu internetowym, to możemy je
pobierać w porcjach, np. po 1000.

    $ git fetch --depth=1000
    $ git fetch --depth=2000
    ...
    $ git fetch --depth=6000
    ...
    $ git fetch --depth=10000

lub też:

    $ git fetch --deepen=1000
    $ git fetch --deepen=1000
    $ git fetch --deepen=1000
    ...
    $ git fetch --deepen=1000


Gdy osiągniemy granicę repozytorium, czyli zejdziemy na maksymalną głębokość, to `git` nam zwróci
poniższy komunikat:

    $ git fetch --depth=10000
    remote: Total 0 (delta 0), reused 0 (delta 0)

Jeśli ujrzymy taką informację po wydaniu polecenia `git fetch` , to możemy być pewni, że pobraliśmy
wszystkie możliwe commit'y.

## Opcja unshallow

Innym ze sposobów pobrania całego repozytorium git mając utworzony płytki klon jest skorzystanie z
opcji `--unshallow` w poleceniu `git fetch` . Ta procedura w skutkach będzie dokładnie taka sama
jak opisany wyżej proces pogłębiania klonu repozytorium. Jedyna różnica jest taka, że `--unshallow`
pobierze nam wszystkie brakujące commit'y naraz. Jeśli dojdzie do przerwania tego procesu, to
naturalnie trzeba będzie go ponowić. Z opcji `--unshallow` możemy skorzystać na dowolnej głębokości
płytkiego klonu.

W przypadku, gdy nie jesteśmy pewni, czy w procesie pogłębiania klonu repozytorium nie
zapomnieliśmy o czymś, to zawsze można wydać to poniższe polecenie:

    $ git fetch --unshallow
    fatal: --unshallow on a complete repository does not make sense

Mamy tutaj informację, która oznajmia nam, że to polecenie nie ma sensu, bo całe repozytorium mamy
już pobrane na dysk.

## Problemy z gałęziami

W zasadzie to można by ten artykuł skończy już, gdyby nie fakt, że coś się liczby nie zgadzają. W
statystykach repozytorium można było wyczytać coś na wzór: **13,746 Commits** , 29 Branches ,
1,418 , Tags , **2.2 GB** Files. Jednak przy głębokości 10000 mamy niby już całe repozytorium. Waga
plików również się nie zgadzała.

Okazuje się, że wykonując płytki klon repozytorium git, klonujemy jedynie główną gałąź. A jak widać
wyżej, tych gałęzi w tym repozytorium było 29. Gdzie zatem podziało się pozostałe 28 gałęzi i jak
je odtworzyć?

Najprościej byłoby dodać parametr `--no-single-branch` do polecenia `git clone --depth=1` . W takim
przypadku nie zostanie stworzony minimalny klon repozytorium, choć i tak zostanie pobranych sporo
miej danych niż przy pełnej synchronizacji.

Poniżej jest przykład polecenia:

    $ git clone --depth=1 --no-single-branch https://salsa.debian.org/kernel-team/linux.git/
    Cloning into 'linux'...
    remote: Enumerating objects: 1003994, done.
    remote: Counting objects: 100% (1003994/1003994), done.
    remote: Compressing objects: 100% (395010/395010), done.
    Rceiving objects:  10% (107017/1003994), 89.49 MiB | 2.13 MiB/s

No jak widać wyżej, z 1,5 MiB, zrobiło się prawie 90 MiB, a to tylko 10% , bo proces pobierania
ciągle jest toku. Po ilości obiektów można stwierdzić, że pobieranych jest około 1/7 faktycznej
objętości repozytorium, co niekoniecznie musi się przekładać na takie same proporcje w MiB.
Niemniej jednak, mając słabe łącze możemy nie być w stanie dociągnąć takiego wielkiego kawałka.

### Określenie gałęzi w pliku .git/config

Jeżeli powyższe rozwiązanie nie wchodzi w grę, to zawsze możemy pokusić się o ręczną edycję pliku
`.git/config` , który jest ulokowany w głównym katalogu repozytorium (po wykonaniu płytkiego klonu).
Ten plik zawiera sekcję `[remote "origin"]` , która prezentuje się w poniższy sposób:

    [remote "origin"]
        url = https://salsa.debian.org/kernel-team/linux.git/
        fetch = +refs/heads/master:refs/remotes/origin/master

Widzimy, że mamy tylko jeden wpis z `+refs/heads/...` , który odnosi się do gałęzi `master` . Tego
typu linijek możemy dodać tyle ile w danym repozytorium jest gałęzi. Dla przykładu, by dodać
kolejną interesującą nas gałąź, przepisujemy powyższy kawałek do poniższej postaci:

    [remote "origin"]
        url = https://salsa.debian.org/kernel-team/linux.git/
        fetch = +refs/heads/master:refs/remotes/origin/master
        fetch = +refs/heads/sid:refs/remotes/origin/sid
        ...

Listę gałęzi można uzyskać posługując się poleceniem `git ls-remote` :

    $ git ls-remote --heads
    From https://salsa.debian.org/kernel-team/linux.git/
    af210b506d24cf7585629e346c921aed90e07133        refs/heads/carnil/import-4.14.14
    a56ffb69de38383f26e77b740b9270ee0faf834c        refs/heads/carnil/jessie-security/CVE-2017-7533
    7c1804aa712bae44319bd481523c0500df16733b        refs/heads/carnil/stretch-security/CVE-2017-7533
    0205ec9ec76ff2d993241a55628e3bd196ecec44        refs/heads/dgit-master
    0839599a02f9cbbcb1933e4a60bcbbcf5bcacc11        refs/heads/etch
    46f3979bd0f6a567b563f0c99f95c592067e94bb        refs/heads/etch-security
    a0fe8cfa8617c895bf6d42383e1975199f342c5b        refs/heads/etchnhalf
    66235807ed13884dc8986d9238f83c54baa26ace        refs/heads/etchnhalf-security
    7b995d73957608b43a4b2dfacfa9e17bdc37e711        refs/heads/jessie
    1d63dcd6296b9ad495d1a7d70c63fa5db68b77b7        refs/heads/jessie-backports
    5009a129295a70d8017974c9821ecdd12d7d9675        refs/heads/jessie-security
    c7b190d1b45dd7a7df71eb0651b677754972f7fd        refs/heads/jessie-security-4.9
    13bc212fa183d61162a1708df0ee9382c0ac66f0        refs/heads/jessie-updates
    cd2e2790cfeebeec461ce28530f23b8bf0e24e7b        refs/heads/lenny
    14865a3daf8e808072ebca063297815e0967b129        refs/heads/lenny-security
    7d94791b95d147a13f2eae4fa9a5e852c95330c9        refs/heads/master
    ce61976b4810abb7e88f0bf08dc8efc7e6a19dc5        refs/heads/rosh/armel_mini
    26e9f62e399a67e454ff40deca2eef531c279139        refs/heads/sid
    9e549e30eb6e12e97f3ff29497821aa41564ee93        refs/heads/squeeze
    f8d465883cbbf23432db1d0139f0b43e570ebc9d        refs/heads/squeeze-backports
    03cf6ed02e5128674ac954b345cdea2a3c6b9ddd        refs/heads/squeeze-security
    155c325ba50c3cdd60d9fcdfdd961a4a636a6367        refs/heads/stretch
    5357a00fc4892878f8e532862540a575df621e2c        refs/heads/stretch-backports
    0a50b0efbc995f6c5710f4d52e321ef6102ee298        refs/heads/stretch-security
    1d69dac66c315a290fb61c5f400e056e8d01fe50        refs/heads/ukleinek/jessie
    beed93421656d6290a4fe3952e6a29696a6cbaf7        refs/heads/waldi/cloud-arm64
    15ca73ce51f27d275a1e67e570cfe3a13ade3368        refs/heads/wheezy
    b479e71dc9fd2215f08b3ea795e0fe9264cc39c3        refs/heads/wheezy-backports

Gdy teraz zaktualizujemy informacje o zdalnym repozytorium, to zostaną pobrane brakujące gałęzie.
Nie musimy przy tym pobierać ich wszystkich na raz ale jeśli chcemy mieć ich lokalną kopię, to
stosowne wpisy trzeba będzie ręcznie pododawać do pliku `.git/config` . Sprawdzamy teraz jak
wygląda proces dodania nowej zdalnej gałęzi:

    $ git remote update && git status
        Fetching origin
        remote: Enumerating objects: 209, done.
        remote: Counting objects: 100% (209/209), done.
        remote: Compressing objects: 100% (72/72), done.
        remote: Total 209 (delta 92), reused 199 (delta 89)
        Receiving objects: 100% (209/209), 260.34 KiB | 136.00 KiB/s, done.
        Resolving deltas: 100% (92/92), done.
        From https://salsa.debian.org/kernel-team/linux
         * [new branch]          sid        -> origin/sid
        On branch master
        Your branch is up to date with 'origin/master'.

        nothing to commit, working tree clean

    $ git branch -a

    * master
      remotes/origin/HEAD -> origin/master
      remotes/origin/master
      remotes/origin/sid

Jeśli chcielibyśmy pobrać wszystkie gałęzie naraz, to zamiast precyzować osobne linijki dla każdej
z gałęzi w pliku `.git/config` , to możemy dodać tam poniższy wpis:

    [remote "origin"]
        url = https://salsa.debian.org/kernel-team/linux.git/
        fetch = +refs/heads/*:refs/remotes/origin/*

Jeśli interesuje nas inna gałąź niż ta domyślna i chcemy wykonać jedynie jej płytki klon, to
korzystamy z poniższego polecenia:

    $ git clone --depth=1 https://salsa.debian.org/kernel-team/linux.git/ -b sid

## Czyszczenie lokalnego klonu repozytorium

Gdy pobierzemy już całe zdalne repozytorium na dysk, to może się okazać, że ta nasza kopia lokalna
zajmuje więcej miejsca niż niż jej zdalny odpowiednik. Można nieco zoptymalizować rozmiar tak
utworzonego repozytorium przez wydanie poniższych poleceń:

    $ git gc --prune=now
    $ git remote prune origin

Przebudują nam one lokalne repozytorium i pousuwają zbędne rzeczy, przez co zoptymalizują ilość
zajmowanego przez repo miejsca na dysku.
