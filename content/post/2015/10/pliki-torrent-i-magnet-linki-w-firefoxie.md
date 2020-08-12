---
author: Morfik
categories:
- Linux
date: "2015-10-27T20:02:45Z"
date_gmt: 2015-10-27 18:02:45 +0100
published: true
status: publish
tags:
- firefox
title: Pliki .torrent i magnet linki w Firefox'ie
---

Przeglądarki mają to do siebie, że każda z nich korzysta z własnych ustawień dotyczących [typów MIME
(mime type)](https://pl.wikipedia.org/wiki/Typ_MIME). Do tego dochodzi jeszcze fakt, że często te
ustawienia są inne od tych, które mamy w systemie. Może to nie jest jakiś wielki problem, bo w
opcjach Firefox'a możemy bez trudu szereg rzeczy poprzestawiać. Natomiast jest jeden problem,
którego w prosty sposób się obejść nie da i trzeba się trochę na nim pochylić. Chodzi o dodawanie
nowych typów MIME, które nie są pokazane na liście obsługiwanych typów w Preferences -\>
Applications. Wiąże się z tym tak skonfigurowanie przeglądarki, by automatycznie otworzyła ona jakiś
program ilekroć dany typ pliku będzie pobierany.

<!--more-->
## Domyślne typy MIME w Firefox'ie

Co się stanie, gdy odwiedzimy jakiś link, a ten będzie zawierał plik o określonym rozszerzeniu?
Jako, że Firefox ma zdefiniowanych szereg typów MIME, to zostaną uruchomione określone w nich
aplikacje, które można bez problemu sobie dostosować. Co się jednak stanie, gdy taki plik będzie
zawierał rozszerzenie, które nie występuje na liście typów MIME w Firefox'ie? Zostanie wyrzucone
okienko z informacją na temat tego gdzie zapisać dany plik. W sporej części przypadków, ten
mechanizm może chronić przed wirusami i innym tego typu robactwem, no bo jakby nie patrzeć, np. plik
`.pdf` nie zostanie automatycznie odpalony. Są też przypadki, gdzie taka polityka bardziej wnerwia
człowieka niż go zabezpiecza.

Poniżej zostaną przedstawione dwa przykłady. Jeden dla linków MAGNET, a drugi dla plików `.torrent`
. Domyślnie, Firefox nie ma pojęcia o MAGNET linkach, no i nie wie w czym otworzyć plik `.torrent` .
Postaramy się go zatem tego nauczyć.

## Plik mimeTypes.rdf

Dokładnie nie wiem jak Firefox buduje swoją bazę danych MIME, za to wiem, że jest ona trzymana w
pliku [mimeTypes.rdf](http://kb.mozillazine.org/MimeTypes.rdf) ulokowanym w katalogu
`~/.mozilla/firefox/profil.default/` . To jest zwykły plik tekstowy i można go podejrzeć w każdym
edytorze tekstu. Wiem też, że Firefox tworzy ten plik, gdy ten zostanie w jakiś sposób skasowany czy
uszkodzony. Standardowo jest w nim niewiele wpisów ale ten fakt zmienia się z czasem jak
konfigurujemy typy MIME w opcjach Firefox'a. Zatem wszelkie niestandardowe ustawienia, tj. zmiany,
które wprowadziliśmy, są zapisywane w tym pliku.

Plik `mimeTypes.rdf` ma kilka kontenerów, tj. bloków ujętych w znaczniki XML. Nas będą interesować
[te
poniższe](https://askubuntu.com/questions/384375/how-can-i-get-firefox-to-open-torrent-files-with-transmission):

    <RDF:RDF ... ...>
    ...
          <RDF:Seq RDF:about="urn:mimetypes:root">
          ...#1
          </RDF:Seq>
    ...
          <RDF:Seq RDF:about="urn:schemes:root">
          ...#2
          </RDF:Seq>
    ...#3
    </RDF:RDF>

W miejsca oznaczone numerkami `#1` , `#2` i `#3` musimy wstawić odpowiedni kod.

### Firefox i pliki .torrent

Na sam początek zajmijmy się implementacją plików `.torrent` . W przypadków plików o konkretnym
rozszerzeniu, trzeba ustalić pierw jak dany plik jest widziany w systemie, tj. jaki ma przypisany
typ MIME. Poniżej jest fotka obrazująca właściwości pliku `.torrent` w menadżerze plików SpaceFM.

![]({{< baseurl >}}/img/2015/10/1.spacefm-mimetypes.rdf_.png)

Widzimy, że plik `.torrent` ma typ MIME `application/x-bittorrent` , zatem w `mimeTypes.rdf` w
miejscu `#1` na powyższym schemacie dodajemy ten poniższy kawałek kodu:

    <RDF:li RDF:resource="urn:mimetype:application/x-bittorrent"/>

Za to tam gdzie `#3` , wrzucamy ten poniższy:

    <RDF:Description RDF:about="urn:mimetype:handler:application/x-bittorrent"
          NC:useSystemDefault="true"
          NC:alwaysAsk="false">
          <NC:possibleApplication RDF:resource="urn:handler:local:/usr/bin/qbittorrent"/>
          <NC:externalApplication RDF:resource="urn:mimetype:externalApplication:application/x-bittorrent"/>
    </RDF:Description>

    <RDF:Description RDF:about="urn:mimetype:externalApplication:application/x-bittorrent"
          NC:prettyName="qbittorrent"
          NC:path="/usr/bin/qbittorrent" />

    <RDF:Description RDF:about="urn:handler:local:/usr/bin/qbittorrent"
          NC:prettyName="qbittorrent"
          NC:path="/usr/bin/qbittorrent" />

    <RDF:Description RDF:about="urn:mimetype:application/x-bittorrent"
          NC:fileExtensions="torrent"
          NC:description="BitTorrent seed file"
          NC:value="application/x-bittorrent"
          NC:editable="true">
          <NC:handlerProp RDF:resource="urn:mimetype:handler:application/x-bittorrent"/>
    </RDF:Description>

### Firefox i MAGNET linki

W przypadku MAGNET linków, sprawa ma się nieco inaczej, bo tutaj nie mamy do czynienia z żadnym
plikiem, a jedynie z ciągiem znaków zaczynającym się od `magnet:` , po którym następuje [szereg
parametrów](https://en.wikipedia.org/wiki/Magnet_URI_scheme). Nie ma zatem jak ustalić tutaj typu
MIME dla tego linku. Niemniej jednak, obsługę MAGNET linków w dalszym ciągu jesteśmy w stanie
zaimplementować w Firefox'ie.

W tym przypadku, musimy dodać do pliku `mimeTypes.rdf` ten poniższy kawałek w miejscu `#2` :

    <RDF:li RDF:resource="urn:scheme:magnet"/>

Oraz ten w miejsce `#3` :

    <RDF:Description RDF:about="urn:scheme:handler:magnet"
          NC:useSystemDefault="true"
          NC:alwaysAsk="false" />

    <RDF:Description RDF:about="urn:scheme:magnet"
          NC:value="magnet">
          <NC:handlerProp RDF:resource="urn:scheme:handler:magnet"/>
    </RDF:Description>

### Inne typy MIME

Za pośrednictwem pliku `mimeTypes.rdf` jesteśmy w stanie określić dowolną konfigurację dla typów
MIME. Edycja tego pliku jest nieco ryzykowna i dobrze jest się zaznajomić z jego składnią. Tak czy
inaczej, po dokonaniu zmian, musimy jeszcze ponownie uruchomić przeglądarkę. Jeśli wszystko dobrze
zrobiliśmy, powinniśmy zobaczyć dodatkowe pozycje w menu Firefox'a:

![]({{< baseurl >}}/img/2015/10/2.firefox-mimetypes.rdf_.png)
