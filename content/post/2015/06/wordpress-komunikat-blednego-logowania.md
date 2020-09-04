---
author: Morfik
categories:
- WordPress
date: "2015-06-06T13:05:41Z"
date_gmt: 2015-06-06 11:05:41 +0200
published: true
status: publish
tags:
- blog
title: 'WordPress: Komunikat błędnego logowania'
---

Jest kilka rzeczy, które mogą zdradzić dane logowania do serwisu WordPressa. Nie będę tutaj opisywał
tej najbardziej oczywistej, czyli [posługiwania się loginem bezpośrednio na
blogu]({{< baseurl >}}/post/wordpress-administrator-bloga/), tylko skupię się na nieco bardziej
chytrej opcji, która zakłada odpytywanie formularza logowania w celu ustalenia, które loginy są
dostępne w bazie danych. To tak jak grać w statki i strzelać na oślep, aż w końcu się trafi w
któryś z nich, a skąd wiemy, że trafiliśmy? Po zwracanej odpowiedzi od drugiego gracza. I podobnie
jest w przypadku formularza logowania WordPressa, z tym, że jeśli chodzi o strony www, to mamy nieco
zawężone pole strzału, bo administrator/użytkownik zwykle wybiera niezbyt skomplikowany login często
kojarzący się z pewnymi elementami jego życia, a gdy potencjalny atakujący ustali taki login, to
będzie w stanie dokonać ataku Brute Force na hasło do tego konta.

<!--more-->
## Wiadomości przy logowaniu

By przekonać się jak działa powyżej opisany schemat, spróbujmy zalogować się na nieistniejące konto
na naszym blogu. Powinniśmy dostać poniższy komunikat:

![]({{< baseurl >}}/img/2015/06/2.wordpress-bledny-komunikat.png#small)

Z kolei, jeśli spróbujemy zalogować się na istniejące konto ale bez znajomości hasła do niego, to
dostaniemy nieco inny monit:

![]({{< baseurl >}}/img/2015/06/1.wordpress-bledny-komunikat.png#small)

Komunikaty są bardzo oczywiste i na ich podstawie możemy bez problemu stwierdzić, które konto
istnieje w bazie, a które nie. Jeśli prowadzimy blog samotnie i mamy do tego zaprzęgnięty system
komentarzy, np. z disqusa, to powinniśmy zastanowić się jak zabezpieczyć się przed takim
próbkowaniem loginów. Oczywiście [możemy zabronić
dostępu]({{< baseurl >}}/post/wordpress-ukrycie-wp-login-php-oraz-wp-admin/) do pliku
`wp-login.php` ale to tylko pod warunkiem, że administratorzy lub/i autorzy wpisów mają stały adres
IP. W przeciwnym wypadku musimy pomyśleć o innym rozwiązaniu.

## Niejednoznaczny komunikat

Istnieje możliwość [ujednolicenia tych dwóch powyższych
komunikatów](http://www.sutanaryan.com/how-to-filter-or-change-wordpress-admin-error-message/), tak
by nie rezygnować z samej informacji o wprowadzeniu błędnych danych logowania ale też by nie
zdradzać czy dane konto faktycznie istnieje w systemie.

Jeśli interesuje nas taki mechanizm, to do pliku `functions.php` w motywie bloga dodajemy poniższy
kod:

    // Filter Wordpress admin error message
    function rs_custom_login_error( $message ) {
        global $errors;

        if (isset($errors->errors['invalid_username']) || isset($errors->errors['incorrect_password'])) :
            $message = __('ERROR: Invalid username or password combination.', 'morfik') . ' ' .
            sprintf(('%3$s?'),
            site_url('wp-login.php?action=lostpassword', 'morfik'),
            __('Password Lost and Found', 'morfik'),
            __('Lost Password', 'morfik'));
        endif;

        return $message;
    }
    add_filter( 'login_errors', 'rs_custom_login_error' );

I teraz już nie będzie miało znaczenia czy wprowadzimy zły login czy hasło, zawsze zostanie nam
wyrzucony ten sam błąd:

![]({{< baseurl >}}/img/2015/06/3.wordpress-bledny-komunikat.png#small)
