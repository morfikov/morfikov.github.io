---
author: Morfik
categories:
- Linux
date: "2016-02-09T03:04:34Z"
date_gmt: 2016-02-09 02:04:34 +0100
published: true
status: publish
tags:
- firefox
- skrypty
title: Aktualizacja Firefox'a i Thunderbird'a w debianie
---

W 2006 roku, Mozilla przyczepiła się do debiana o to, że ten [wykorzystuje ich znaki
towarowe](https://en.wikipedia.org/wiki/Mozilla_Corporation_software_rebranded_by_the_Debian_project).
Chodziło głównie o to, że debian wprowadzał swoje poprawki, które nie były zatwierdzone przez zespół
Mozilli. W efekcie czego, debian pozmieniał nazwy szeregu produktów Mozilli i tak zamiast normalnego
Firefox'a mamy Iceweasel, podobnie z Thunderbird'em i Icedove. Obecnie nie ma możliwości wgrania
aplikacji Mozilli wykorzystując repozytorium debiana. Trzeba się trochę wysilić i paczki pobierać
ręcznie z serwerów Mozilli. Takie rozwiązanie nie jest zbytnio praktyczne, bo przecie w linux'ie
aplikacji nie aktualizuje się za pomocą ich interfejsów graficznych. Jeśli tak by było, to
musielibyśmy uruchamiać przeglądarkę z uprawnieniami root w trybie graficznym, czego raczej nikt
rozsądny nie próbowałby robić. Można, co prawda, napisać skrypt i całą operację aktualizacji nieco
zautomatyzować. Problem w tym, że zarówno Firefox jak i Thunderbird ważą tak około 50 MiB każdy i
taka aktualizacja polegająca na pobraniu całej aplikacji i zainstalowaniu jej na nowo zjadłaby
trochę transferu. Istnieje jednak rozwiązanie, które zakłada wykorzystanie [plików
MAR](https://wiki.mozilla.org/Software_Update:Manually_Installing_a_MAR_file). Ważą one zaledwie
kilka MiB, bo zawierają jedynie aktualizację danej aplikacji. W tym wpisie spróbujemy się przyjrzeć
procesowi aktualizacji z wykorzystaniem tych właśnie plików.

Zgodnie z [informacją na tym blogu](https://glandium.org/blog/?p=3622), Firefox wraca do debiana i
zastępuje tym samym Iceweasel. Od tego momentu można już instalować pakiet `firefox` i cieszyć się
normalnym produktem Mozilli. Niniejszy artykuł w dalszym ciągu znajduje zastosowanie ale nie można
mieszać opisanego niżej sposobu aktualizacji Firefox'a z tym dostarczanym w ramach menadżera
pakietów `apt`/`aptitude` . Zatem albo instalujemy Firefox'a bezpośrednio z repozytorium debiana,
albo ściągamy pakiet z serwerów Mozilli.

<!--more-->
## Pozyskanie pliku MAR

Jak możemy wyczytać na wiki Mozilli, pliki MAR zawierają aktualizację, którą można zaaplikować na
produkt Mozilli, np. Firefox'a, bez potrzeby uruchamiania samej aplikacji. W takim przypadku odpada
nam uruchamianie jej jako administrator. Skąd zatem wziąć te pliki? Są one przechowywane na serwerze
Mozilli, a lokalizację możemy uzyskać wpisując w przeglądarce [ten link](http://download.cdn.mozilla.net/pub/firefox/releases/44.0.1/update/linux-x86_64/en-US/).

Niestety musimy się posiłkować numerkami wersji aplikacji, bo w katalogu `latest/` jest tylko [plik
README.txt](http://download-origin.cdn.mozilla.net/pub/thunderbird/releases/latest/README.txt),
który podaje instrukcje do pobrania całej paczki, a nas ona zbytnio nie interesuje. W tym powyższym
linku mamy szereg kluczowych elementów, które musimy uwzględnić przy pobieraniu pliku MAR. Musimy
znać nazwę aplikacji ( `firefox` ), jej wersję ( `44.0.1` ), system operacyjny ( `linux` ), jego
architekturę ( `x86_64` ) oraz język ( `en-US` ).

Po przejściu pod ten link, zostanie nam wyświetlony listing plików MAR. Podczas pisania tego
artykułu jest ich tam 5. W zależności od tego, którą mamy wersję Firefox'a, będziemy musieli wybrać
odpowiedni plik MAR. Rozmiary tych plików różnią się znacząco. Jeśli aktualizujemy oprogramowanie na
bieżąco, to prawdopodobnie interesować nas będzie plik `firefox-44.0-44.0.1.partial.mar` . Jest to
najnowsza aktualizacja tej przeglądarki i zajmuje ona oko 3 MiB. Zapisujemy zatem ten plik na dysku.

## Weryfikacja sygnatury i sum kontrolnych

Musimy teraz zweryfikować dane zawarte w tym pliku MAR. By tego dokonać, musimy pobrać dwa dodatkowe
pliki [SHA512SUMS](http://download.cdn.mozilla.net/pub/firefox/releases/44.0.1/SHA512SUMS) oraz
[SHA512SUMS.asc](http://download.cdn.mozilla.net/pub/firefox/releases/44.0.1/SHA512SUMS.asc), które
znajdują się nieco wyżej w drzewie katalogów. Pierwszy z tych plików zawiera sumy kontrolne
poszczególnych plików na serwerze. Drugi plik zawiera sygnaturę. Weryfikacja pliku MAR odbywa się w
dwóch etapach. Najpierw sprawdzamy podpis cyfrowy, a później sumę kontrolną pliku MAR.

By sprawdzić sygnaturę, wpisujemy w terminalu to poniższe polecenie:

    $ gpg --verify SHA512SUMS.asc
    ...
    gpg: Signature made Sat 06 Feb 2016 11:48:57 PM CET
    ...
    gpg:                using RSA key 0x1C69C4E55E9905DB
    gpg: Good signature from "Mozilla Software Releases <release@mozilla.com>" [unknown]
    ...

Wyżej widzimy, że sygnatura została złożona przez "Mozilla Software Releases" dnia 2016-02-06
identyfikująca się kluczem 0x1C69C4E55E9905DB . Informacje na temat pochodzenia tego klucza można
znaleźć [tutaj](http://hearsum.ca/blog/mozilla-software-release-gpg-key-transition.html). W
przypadku, gdy zostanie nam zwrócony błąd `Can't check signature: public key not found` , to
będziemy musieli pobrać klucz z serwera kluczy przy pomocy `gpg --search ID` . W miejscu `ID`
podajemy oczywiście ID klucza.

Jeśli chodzi zaś o sumę kontrolną, to możemy ją wygenerować za pomocą poniższego polecenia:

    $ sha512sum firefox-44.0-44.0.1.partial.mar
    a0d0ad90cf96c22b3aa096c2f44be57fb8e048d4c4c99fbeb8eba2bc7508d0382621f708fe0a25e4790e71cb1d2eb327b266dbdb93a2c300167085c7bc6dd566  firefox-44.0-44.0.1.partial.mar

Teraz musimy porównać otrzymaną sumę z tą, która jest pliku `SHA512SUMS` :

    $ cat SHA512SUMS | egrep firefox-44.0-44.0.1.partial.mar | egrep "linux-x86_64/en-US/"
    a0d0ad90cf96c22b3aa096c2f44be57fb8e048d4c4c99fbeb8eba2bc7508d0382621f708fe0a25e4790e71cb1d2eb327b266dbdb93a2c300167085c7bc6dd566  update/linux-x86_64/en-US/firefox-44.0-44.0.1.partial.mar

I już widzimy, że weryfikacja pliku MAR przebiegła pomyślnie, bo suma kontrolna w podpisanym cyfrowo
pliku zgadza się z tą, którą otrzymaliśmy. Możemy teraz przejść do wykonania aktualizacji.

## Aktualizacja z wykorzystaniem pliku MAR

Tworzymy katalog tymczasowy, `/tmp/firefox/out/` . Do tego katalogu musimy wgrać plik `updater` ,
który znajduje się w folderze z zainstalowaną już wcześniej wersją Firefox'a. Dodatkowo musimy do
tego katalogu tymczasowego przenieść plik z update. Na sam koniec, ustawiamy katalog roboczy w taki
sposób, by wskazywał na folder instalacyjny Firefox'a:

    # mkdir /tmp/firefox/out/
    # cp ./firefox-44.0-44.0.1.partial.mar /tmp/firefox/out/update.mar
    # cp /opt/firefox/updater /tmp/firefox/out/
    # cd /opt/firefox/

Będąc w katalogu głównym Firefox'a, uruchamiamy aktualizację tym poniższym poleceniem:

    # /tmp/firefox/out/updater /tmp/firefox/out/ /opt/firefox/ /opt/firefox/

Firefox powinien zostać po chwili zaktualizowany. Jeśli mieliśmy odpaloną przeglądarkę w trakcie
procesu aktualizacji, to musimy ją zrestartować, by aktualizacja zaczęła obowiązywać:

![]({{< baseurl >}}/img/2016/02/1.aktualizacja-mozilla-firefox-about.png)

## Problemy z plikiem libmozsqlite3.so

Na debianie mogą pojawić się problemy z plikiem `libmozsqlite3.so` , którego w tej dystrybucji
linux'a zwyczajnie nie ma. Ten plik jest jednak dostarczany wraz z Firefox'em. Znajduje się on zaraz
w głównym katalogu przeglądarki. Wszystko co musimy zrobić, to podlinkować ten plik do katalogu
`/usr/lib/` , przykładowo:

    # ln -s /opt/firefox/libmozsqlite3.so /usr/lib/

## Automatyzacja aktualizacji przy pomocy skryptu

To, co zostało opisane powyżej, to dość skomplikowany proces aktualizacji Firefox'a. Praktycznie
każdą aplikację Mozilli możemy w ten sposób aktualizować. Niemniej jednak, ta czynność jest jakby
nie patrzeć bardzo czasochłonna i niezbyt komfortowa. Możemy za to pokusić się o napisanie prostego
skryptu, który za sprawą jednego polecenia przeprowadzi za nas wszystkie te powyższe kroki. Skrypt
znajduje się jak zawsze [na moim
github'ie](https://github.com/morfikov/files/blob/master/scripts/ff-tb-updater.sh). Poniżej zaś jest
przykładowa aktualizacja Firefox'a i Thunderbird'a za pomocą tego skryptu:

![]({{< baseurl >}}/img/2016/02/2skrypt-aktualizacja-mozilla-firefox-thunderbird.png)
