---
author: Morfik
categories:
- Blog
date: "2016-08-08T09:44:17Z"
date_gmt: 2016-08-08 07:44:17 +0200
published: true
status: publish
tags:
- seo
- wordpress
GHissueID: 561
title: 'WordPress: Jak edytować plik robots.txt'
---

Co jakiś czas zaglądam sobie do [panelu Google Search
Console](https://www.google.com/webmasters/tools/) w poszukiwaniu błędów na tym blogu. Do tej pory
trafiały mi się pojedyncze przypadki błędnego przekierowania adresu URL. Nic wielkiego, bo wystarczy
odszukać dany post i poprawić w nim konkretny link. Niemniej jednak, Google zaczyna mnie drażnić
"błędami" odnośnie plików `xmlrpc.php` oraz `wp-includes/wlwmanifest.xml` . Oba te pliki są
zablokowane na poziomie serwera Apache2 z powodu [zagrożeń jakie niesie protokół
XML-RPC](https://niebezpiecznik.pl/post/trwaja-ataki-ddos-wykorzystujace-wordpressa-sprawdz-czy-twoj-blog-zostal-uzyty-w-ataku/).
Dlaczego zatem mechanizmy Google zwracają błędy? Winna jest tutaj błędna konfiguracja instrukcji dla
wyszukiwarek, tj. plik `robots.txt` . Jakby tego było mało, to w przypadku WordPress'a ten plik jest
generowany w locie i fizycznie nie istnieje na serwerze. Jak zatem poddać go edycji? Na to pytanie
postaramy się znaleźć odpowiedź w tym wpisie.

<!--more-->
## Do czego WordPress wykorzystuje pliki xmlrpc.php oraz wlwmanifest.xml

Pliki `xmlrpc.php` oraz `wlwmanifest.xml` są częścią implementacji protokołu XML-RPC w WordPress'ie,
który umożliwia, np. publikowanie postów za pomocą różnych klientów blogowych. Gdy obsługę tego
protokołu mamy włączoną, to niekoniecznie musimy pisać posty przez panel administracyjny
WordPress'a. W przypadku, gdy kogoś ta funkcjonalność zbytnio nie interesuje, tak jak mnie tutaj, to
ze strony wygenerowanej przez skrypt WordPress'a można [usunąć odwołania do plików xmlrpc.php i
wlwmanifest.xml](/post/wordpress-wylaczenie-protokolu-xml-rpc/). Niemniej jednak, w
pewnych specyficznych przypadkach wyszukiwarka Google ciągle będzie próbowała te zasoby odwiedzić. W
jakich?

W tym wyżej podlinkowanym artykule były uwzględnione linki zawierające te dwa w/w pliki oraz domena,
która wskazywała na ten blog. Nie były to wprawdzie klikalne linki, tylko ujęte w znaczniki `pre` i
`code` , ale wyszukiwarka potraktowała taki tekst jako odnośnik i stąd błąd w panelu Google Search
Console:

![](/img/2016/08/1.search-console-google-bledy.png#huge)

Rozwiązania tego problemu są dwa. Pierwszym z nich jest edycja wszystkich postów i poprawienie
domeny, by wskazywała na jakąś poza obszarem naszego serwisu. Wtedy Google nie będzie nas winił, że
zasoby nie są dostępne. To rozwiązanie znalazło zastosowanie właśnie w tym przypadku. Innym sposobem
jest edycja pliku `robots.txt` , z tym, że WordPress generuje dynamicznie ten plik, przez co nie
istnieje on w strukturze naszego bloga. Zatem jak moglibyśmy go poddać edycji?

## Edycja pliku robots.txt w WordPress'ie

Jest kilka dedykowanych narzędzi, które są w stanie poddać edycji plik `robots.txt` . Można to
zrobić, np. korzystając z [wtyczki Yoast
SEO](https://kb.yoast.com/kb/how-to-edit-robots-txt-through-yoast-seo/). Problem w tym, że nie da
się edytować tego pliku w przypadku, gdy mamy [wyłączoną edycję plików przez panel
WordPress'a](/post/wordpress-edycja-i-modyfikacja-plikow-dodatkow/).

Możemy jednak nadpisać plik `robots.txt` , który jest generowany przez skrypt WordPress'a. Musimy
tylko w głównym katalogu strony (określony dyrektywą `DocumentRoot` w Apache2) stworzyć taki plik i
dodać do niego stosowną zawartość. Standardowo WordPress daje do swojego pliku te poniższe wpisy:

    User-agent: *
    Disallow: /wp-admin/
    Allow: /wp-admin/admin-ajax.php

My dodatkowo musimy do niego dopisać jeszcze te dwa poniższe:

    Disallow: /xmlrpc.php
    Disallow: /wp-includes/wlwmanifest.xml

Teraz można poprosić boty Google za sprawą penelu Search Console, by zaktualizowały sobie ten plik.
Po chwili te dwa dodatkowe wpisy powinny zostać uwzględnione:

![](/img/2016/08/2.search-console-google-robots-txt.png#huge)
