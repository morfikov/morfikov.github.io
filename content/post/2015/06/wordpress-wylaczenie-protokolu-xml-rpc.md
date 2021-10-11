---
author: Morfik
categories:
- WordPress
date: "2015-06-03T13:55:33Z"
date_gmt: 2015-06-03 11:55:33 +0200
published: true
status: publish
tags:
- blog
- bezpieczeństwo
GHissueID: 110
title: 'WordPress: Wyłączenie protokołu XML-RPC'
---

Jeśli publikujemy posty na swoim blogu, to pierw musimy je gdzieś sporządzić. Możemy to robić w
zwykłym notatniku lub też bezpośrednio w formularzu WordPressa dostępnego przy edycji postu.
Istnieje też inna możliwość, mianowicie korzystanie ze specjalnie przeznaczonego do tego celu
oprogramowania -- klientów blogowych działających w oparciu o protokół
[XML-RPC](https://codex.wordpress.org/XML-RPC_Support). Jest to mechanizm podobny tego znanego
choćby z poczty internetowej, czyli mamy konto email, np. na gmailu ale do tworzenia i zarządzania
wiadomościami wykorzystujemy np. Thunderbirda. Podobnie możemy postępować z treścią pojawiającą się
na blogu i nawet nie musimy być w tym czasie online. Jednym z bardziej znanych klientów, przy pomocy
którego możemy wrzucać posty na bloga, to [Windows Live Writer
(WLW)](https://en.wordpress.com/windows-live-writer/). Poza tym, protokół XML-RPC wykorzystywany
jest także przez część serwisów internetowych, np.
[Flickr](https://www.flickr.com/services/api/request.xmlrpc.html) , co umożliwia im zamieszczanie
postów w naszej witrynie.

<!--more-->
## Problemy z protokołem XML-RPC

Protokół XML-RPC we wcześniejszych wersjach WordPressa powodował dość spore problemy z
bezpieczeństwem, w efekcie czego został domyślnie wyłączony ale nie zrezygnowano z niego zupełnie.
Po wielu latach ten protokół powrócił do łask i deweloperzy WordPressa postanowili go domyślnie
włączyć.

To czy dana strona jest zdolna do komunikacji za pośrednictwem protokołu XML-RPC można bardzo łatwo
ustalić. Wystarczy przejrzeć źródło witryny i odszukać tam dwie linijki zawierające `xmlrpc.php`
oraz `wlwmanifest.xml` ,
    przykładowo:

    <link rel="EditURI" type="application/rsd+xml" title="RSD" href="https://morfitronik.lh/xmlrpc.php?rsd" />
    <link rel="wlwmanifest" type="application/wlwmanifest+xml" href="https://morfitronik.lh/wp-includes/wlwmanifest.xml" />

Plik `xmlrpc.php` jest odpowiedzialny za możliwość publikowania treści na stronie za pośrednictwem
[różnych klientów blogowych](https://codex.wordpress.org/Weblog_Client), klientów email oraz
otrzymywanie pingbacków i trackbacków od innych blogów. Natomiast plik `wlwmanifest.xml` jest
potrzebny jedynie klientowi Microsoftu. Nie każdy jednak korzysta z tego typu funkcjonalności, a
jako że mogą one stwarzać spore zagrożenie, to przydałoby się je wyłączyć.

## Czyszczenie nagłówka

Wyłączenie obsługi zbędnych rzeczy w WowrdPressie oprócz poprawy bezpieczeństwa ma jeszcze jedną
zaletę, mianowicie oczyszcza kod, dzięki czemu wyszukiwarki lepiej traktują w rankingach takie
artykuły. Dlatego też jeśli nie publikujemy postów zdalnie przy pomocy email czy tych klientów
blogowych i nie korzystamy z pingbacków/trackbaków, które też potrafią sporo namieszać, to
zwyczajnie dla świętego spokoju wyłączmy sobie obsługę protokołu XML-RPC. Najprościej możemy to
zrobić przez dopisanie do pliku `functions.php` w używanym motywie poniższych linijek:

    remove_action( 'wp_head', 'wlwmanifest_link' );
    remove_action( 'wp_head', 'rsd_link' );
    add_filter('xmlrpc_enabled', '__return_false');

Dwie pierwsze linijki usuną zbędny kod z nagłówka bloga, natomiast ostatnia wyłącza kompletnie
obsługę protokołu XML-RPC i od tego momentu przy próbie, np. wysłania zdalnej wiadomości, zostanie
zwrócony komunikat z błędem. Niemniej jednak, te dwa pliki dalej będą obecne na serwerze pod
adresami `http://domena.com/xmlrpc.php` oraz `http://domena.com/wp-includes/wlwmanifest.xml` i
przydałoby się o nie jakoś zatroszczyć. Unikajmy jednak stosowania kodu błędu `404` . Zamiast niego
jest lepiej skorzystać z kodu `403` , który akurat jest generowany przy okazji stosowania modułu
`Files` obecnego na serwerze Apache. By zabronić publice wglądu w te dwa powyższe pliki, dopisujemy
do `.htaccess` poniższe kod:

    <Files "xmlrpc.php">
        Require all denied
    </Files>

    <Files "wlwmanifest.xml">
        Require all denied
    </Files>

## Pingbacki i Trackbaki

Trzeba także pamiętać, że jeśli wyłączamy całkowicie protokół XML-RPC, to również musimy wyłączyć
pingbacki i trackbaki w opcjach WordPressa (Settings => Discussion):

![](/img/2015/06/1.wordpress-xml-rpc-trackback.png#big)
