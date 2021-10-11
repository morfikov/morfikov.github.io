---
author: Morfik
categories:
- WordPress
date: "2015-06-03T19:14:23Z"
date_gmt: 2015-06-03 17:14:23 +0200
published: true
status: publish
tags:
- blog
GHissueID: 121
title: 'WordPress: Graficzne emotikonki'
---

Począwszy od wersji 4.2, WordPress wstawia pliki powiązane z
[Emoji](https://codex.wordpress.org/Emoji) bezpośrednio do nagłówka strony. Jest to porcja 1600
znaków, która ma służyć zamianie tekstowych emotikonek na ich graficzne odpowiedniki. Jeśli z nich
nie korzystamy albo wolimy zamiast nich kilka znaków tekstowych, to ten kod jest dla nas kompletnie
zbędny i zaśmieca tylko nagłówek, wobec czego powinniśmy się jak najszybciej pozbyć tego balastu.

<!--more-->
## Niepożądane emotikonki na blogu

WordPress zapewnia wsparcie dla emoji, które liczą sobie prawie 900 ikonek, a wszystko to za sprawą
Twitter, który zdecydował się na oddanie tych ikonek w ręce społeczności i otworzył ich kod, dzięki
czemu teraz każdy może z nich korzystać. Niektórym osobnikom mogą one nie przypaść do gustu ale na
wypadek gdyby ktoś jednak chciał sobie tę graficzne emotikonki włączyć, to warto wiedzieć, że opcja
odpowiadająca za ich zamianę widnieje w panelu administracyjnym WordPressa (Settings => Writing):

![](/img/2015/06/1.wordpress-opcje-emotikonki.png#big)

Nie ma znaczenia przy tym, czy zaznaczymy tę powyższą opcję czy też nie, to i tak w kodzie strony
będziemy mieć skrypt i style od tych emotikonek:

![](/img/2015/06/2.wordpress-kod-strony-emotikonki.png#huge)

Na szczęście bez większego trudu możemy posprzątać ten bałagan. Chodzi generalnie o dodanie do pliku
`functions.php` w swoim motywie tego poniższego kodu:

    remove_action( 'wp_head', 'print_emoji_detection_script', 7 );
    remove_action( 'admin_print_scripts', 'print_emoji_detection_script' );
    remove_action( 'admin_print_styles', 'print_emoji_styles' );
    remove_action( 'wp_print_styles', 'print_emoji_styles' );
