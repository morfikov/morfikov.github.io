---
author: Morfik
categories:
- WordPress
date: "2015-05-31T11:38:55Z"
date_gmt: 2015-05-31 09:38:55 +0200
published: true
status: publish
tags:
- blog
GHissueID: 219
title: 'WordPress: Wersja obecna w kodzie źródłowym'
---

Standardowo WordPress dorzuca nieco informacji na swój temat bezpośrednio do kodu wygenerowanej
witryny. Jednym z bardziej niebezpiecznych takich dodatkowych wpisów jest wskazówka na temat
zainstalowanej wersji skryptu. Oczywiście, że powinniśmy aktualizować WordPressa na bieżąco i
możliwie od razu jak tylko zostanie wypuszczona jego nowa wersja ale nie zawsze możemy nadążyć za
tymi wszystkimi zmianami i wyrobić się w terminach. Ja jestem też zdania, że kluczowe oprogramowanie
(zwłaszcza w internecie) powinno posiadać wersję nie do ustalenia, albo chociaż jakiś mylący ciąg
znaków, tak by automatyczne ataki opierające się o wersję danej aplikacji, nie tyle zostawiły nas w
spokoju, co zaczęły popełniać błędy, które będą widoczne w logach. Przynajmniej takie jest
optymistyczne założenie.

<!--more-->
## Wersja WordPress'a

Wersję WordPress'a możemy odczytać z nagłówka strony, którą załadujemy w przeglądarce:

    <head>
    ...
    <meta name="generator" content="WordPress 4.2.2" />
    ...
    </head>

Za dodanie powyższego kodu odpowiada plik `wp-includes/default-filters.php` , a konkretnie poniższa
linijka:

    add_action( 'wp_head', 'wp_generator' );

## Jak usunąć wersję z kodu

By ukryć wersję WordPress'a w kodzie, musimy usunąć tę powyższą akcję przy pomocy pliku
`functions.php` , który znajduje się w wykorzystywanym przez naszą stronę motywie. Przechodzimy
zatem do edycji tego pliku i dodajemy tam poniższy kod:

    remove_action('wp_head', 'wp_generator');

## Podmiana wersji

Usuwanie wersji nie zawsze jest najlepszym wyjściem. Potencjalny atakujący nie ma, co prawda,
informacji na temat wersji w kodzie źródłowym strony ale może spróbować poszukać cech
charakterystycznych, które są widoczne tylko w pewnych określonych wersjach danego oprogramowania,
np. dodatkowy kod wynikający z rozbudowy aplikacji. W starszych wersjach, tego kodu zwyczajnie nie
będzie.

Dlatego też mając na uwadze powyższe, możemy ustawić wersję WordPress'a na jedną z tych starszych,
powiedzmy 2.7.1 , tylko znowu nie przeginajmy w drugą stronę, bo ta wersja powinna być faktycznie
wydana. W przeciwnym razie, atakujący może poznać nasz podstęp i cały ten misterny plan się nie
powiedzie. Jeśli chcemy się bawić w tego typu podchody, to dopisujemy także poniższy kod do pliku
`functions.php` :

    function wpset_custom_generator_meta_tag() {
       echo '<meta name="generator" content="WordPress 2.7.1" />' . "\n";
    }
    add_action( 'wp_head', 'wpset_custom_generator_meta_tag' );

## Wersja w dołączanych plikach .css/.js

Te powyższe czynności nie usuną jednak wszystkich informacji na temat wersji zainstalowanego skryptu
jakie zdradza WordPress. W każdym dołączanym pliku `.css` i `.js` znajduje się fraza podobna do tej:
`?ver=4.5.3'` . Zatem i tak wiadomo, której wersji CMS'a używamy. Niemnie jednak, tego kawałka
tekstu również możemy się pozbyć z kodu. Tym razem w pliku `functions.php` dopisujemy poniższą
zwrotkę:

    add_filter( 'style_loader_src', 'sdt_remove_ver_css_js', 9999 );
    add_filter( 'script_loader_src', 'sdt_remove_ver_css_js', 9999 );
    function sdt_remove_ver_css_js( $src ) {
        if ( strpos( $src, 'ver=' ) )
            $src = remove_query_arg( 'ver', $src );
        return $src;
    }
