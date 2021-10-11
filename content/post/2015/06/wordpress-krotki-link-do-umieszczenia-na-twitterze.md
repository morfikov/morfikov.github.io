---
author: Morfik
categories:
- Blog
date: "2015-06-03T10:13:38Z"
date_gmt: 2015-06-03 08:13:38 +0200
published: true
status: publish
tags:
- wordpress
- twitter
GHissueID: 141
title: 'WordPress: Krótki link do umieszczenia na Twitterze'
---

W chwili gdy zabierałem się za rozpracowywanie szeregu zagadnień związanych z oczyszczaniem nagłówka
generowanego przez WordPress, nie miałem pojęcia, że niektóre opcje nie są dostępne w standardowej
instalacji, którą każdy może sobie pobrać z ich strony i wgrać do siebie na serwer. Chodzi
oczywiście o [krótkie odnośniki](https://en.wikipedia.org/wiki/URL_shortening) (shortlinks). Ich
głównym celem jest skracanie linków do paru znaków, tak by można je umieścić np. na Twitterze
gdzie długość wiadomości jest znacznie ograniczona i raczej wklejanie tam linku, który by zjadł
wszystkie dostępne znaki nie jest zbytnio pożądane.

<!--more-->
## Co to są krótkie linki?

Zapewne spotkaliśmy się już z serwisami takimi jak <http://tinyurl.com/> czy <https://bitly.com/> ,
które umożliwiają skracanie linków. WordPress ma swój własny mechanizm generujący linki w domenie
`wp.me` . Jeśli mamy bloga na [wordpress.com](https://wordpress.com/), wtedy możliwość korzystania z
krótkich odnośników typu `http://wp.me/8cz78asd` będzie nam zapewniona i do każdego opublikowanego
przez nas postu będziemy w stanie wyciągnąć taki linki za pomocą formularza edycji:

![](/img/2015/06/1.wordpress-krotki-link.png#big)

Natomiast jeśli nasz blog jest hostowany poza stroną WordPress'a, to nie będziemy mogli skorzystać z
tych krótkich odnośników. Nie wszystko jednak stracone, bowiem jeśli spróbujemy wygenerować krótki
link dla jakiegoś postu, zostanie nam on podany w formie `http://domena.com/?p=10` , czyli jest to
standardowy link do treści w WordPress'ie przed przepisaniem go do postaci
[permalinku](/post/wordpress-odnosniki-bezposrednie-permalinks/):

![](/img/2015/06/2.wordpress-krotki-link-permalink.png#small)

Tak czy inaczej, oba linki będą działać bez żadnych dodatkowych kroków z naszej strony, bo jeśli
korzystamy z permalinków, to te krótsze ich wersje zostaną przekierowane automatycznie przy pomocy
kodu błędu `301` .

## Czyszczenie kodu

WordPress ma jednak specjalną opcję dotyczącą tych krótkich odnośników i skoro z niej nie będziemy
korzystać, to przydałoby się ją odznaczyć. Wprawdzie nie zauważymy żadnej różnicy w działaniu bloga,
bo w kodzie nawet nie są generowane linijki z `rel="shortlink"` ale dobrze jest czasem zajrzeć
WordPresowi pod maskę i sprawdzić jak on działa.

Za generowanie tych skróconych odnośników odpowiedzialny jest plik
`wp-includes/default-filters.php` , a konkretnie poniższy kod:

    add_action( 'wp_head', 'wp_shortlink_wp_head', 10, 0 );

By wyłączyć ten ficzer zupełnie, musimy w pliku `functions.php` używanego przez nasz serwis motywu
usunąć tę akcję, przykładowo:

    remove_action( 'wp_head', 'wp_shortlink_wp_head', 10, 0 );

I po bólu. Oczywiście to działanie miało by więcej sensu gdybyśmy hostowali bloga bezpośrednio na
stronie WordPress'a ale kto wie, może kiedyś przyjdzie nam się tam przenieść i dobrze jest wiedzieć
zawczasu, że tego typu opcje są do dyspozycji i gdzie ich szukać.
