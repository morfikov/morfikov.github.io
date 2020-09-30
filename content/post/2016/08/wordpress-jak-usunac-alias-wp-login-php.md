---
author: Morfik
categories:
- WordPress
date: "2016-08-14T20:05:10Z"
date_gmt: 2016-08-14 18:05:10 +0200
published: true
status: publish
tags:
- blog
title: 'WordPress: Jak usunąć alias "login" dla wp-login.php'
---

Przeglądając sobie ostatnio logi mojego serwera Apache2, zauważyłem tam dziwną aktywność. Jakieś
boty czy też inne ustrojstwa próbują od czasu do czasu uzyskać dostęp do formularza logowania tego
bloga. Jako, że tutaj mamy do czynienia z WordPress'em oraz [ochroną pliku wp-login.php oraz
katalogu wp-admin/ za pomocą certyfikatów][1], to taki bot nigdy nie uzyska dostępu do tych zasobów.
Niemniej jednak, te automaty generują zapytania do serwera o zasób `login` , który z kolei
przekierowuje je pod `wp-login.php` za pomocą kodu 301 lub 302. Moglibyśmy uniknąć tego typu
przekierowań zwracając im kod 404 i przy tym odciążając nieco serwer www. W tym krótkim artykule
zobaczymy jak tego typu zabieg przeprowadzić.

<!--more-->
## Zapytania o wp-login.php i login

WordPress standardowo tworzy alias na plik `wp-login.php` w postaci `login` . Nie trzeba zatem
wpisywać w pasku adresu nazwy pliku, by uzyskać formularz logowania, bo wystarczy wpisać samo
`login` . W ten sposób serwer zwykle zwraca kod 301, który informuje przeglądarkę o przeniesieniu
zasobu pod inny adres (tzw. Permanent Redirect). Czasami też może zostać zwrócony kod 302, który
informuję klienta o odnalezieniu zasobu. W obu tych przypadkach zostanie zwrócony nowy adres zasobu,
pod który przeglądarka powinna się udać. Poniżej przykład:

![](/img/2016/08/1.wp-login-php-wordpress-ukrycie.png#huge)

Wyżej w nagłówku widzimy pole `Location` , a w nim `wp-login.php` . Zatem niby wpisujemy w
przeglądarce `login` , a i tak lądujemy docelowo na `wp-login.php` i dopiero wtedy zwracany jest
nam formularz logowania. Problem w tym, że te wszystkie boty szukające formularza logowania nie do
końca mogą być świadome naszej konfiguracji WordPress'a. W efekcie wysyłają dwa czy trzy zapytania,
by sprawdzić, czy dany serwer www hostuje jakąś stronę opartą o tego CMS'a. Moglibyśmy im ulżyć w
cierpieniach i zwrócić od razu kod 404 informujący je o tym, że takiego zasobu nie ma na serwerze.

## Jak usunąć alias "login"

By usunąć alias `login` w WordPress'ie, musimy usunąć [akcję wp_redirect_admin_locations][2].
Możemy to zrobić za pomocą pliku `functions.php` , który jest zlokalizowany w katalogu
wykorzystywanego motywu. W tym pliku dodajemy tę oto poniższą linijkę:

    remove_action( 'template_redirect', 'wp_redirect_admin_locations', 1000 );

Jeśli teraz spróbujemy odwiedzić `login` , to powinien zostać zwrócony kod 404:

![](/img/2016/08/2.wp-login-php-wordpress-ukrycie.png#huge)

Usunięcie tej powyższej akcji ma także wpływ na aliasy `admin` oraz `dashboard` .

[1]: /post/certyfikat-chroniacy-wp-login-php-wp-admin/
[2]: https://developer.wordpress.org/reference/functions/wp_redirect_admin_locations/
