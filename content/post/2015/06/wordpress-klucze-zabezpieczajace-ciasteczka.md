---
author: Morfik
categories:
- WordPress
date: "2015-06-06T06:36:59Z"
date_gmt: 2015-06-06 04:36:59 +0200
published: true
status: publish
tags:
- cookies
- blog
title: 'WordPress: Klucze zabezpieczające ciasteczka'
---

WordPress stara się dbać o bezpieczeństwo swoich użytkowników najlepiej jak potrafi. Dowodem na to
mogą być unikalne klucze i sole uwierzytelniające (Authentication Unique Keys and Salts) wprowadzone
w wersji 2.5 oraz nieco rozbudowane w wersji 2.7. Ten mechanizm nie wykorzystuje sesji PHP do
śledzenia np. statusu zalogowania, tylko zaprzęga do tego celu ciasteczka, by utrudnić (albo raczej
uniemożliwić) potencjalnemu atakującemu dostęp do witryny. Bez tych kluczy i soli jesteśmy w stanie
przewidzieć wszystkie składowe hasha danego ciasteczka, dlatego też powinniśmy pokusić się o ich
implementację na swojej stronie. Niemniej jednak, na necie jest bardzo dużo niskiej jakości
informacji na temat działania mechanizmu walidacji ciasteczek, a artykuły, które
[potrafią](https://boren.blog/2008/07/14/ssl-and-cookies-in-wordpress-26/)
[go](http://www.securitysift.com/understanding-wordpress-auth-cookies/)
[opisać](http://codeseekah.com/2012/04/09/why-wordpress-authentication-unique-keys-and-salts-are-important/)
w szerszym stopniu, są już lekko nieaktualne, bo zmienia się on praktycznie z każdą nową wersją
WordPressa. Zanim jednak przejdziemy do samych kluczy/soli, powinniśmy się zaznajomić ze sposobem w
jaki WordPress generuje i sprawdza wspomniane ciasteczka, na podstawie których to autoryzowane są
działania użytkowników naszego bloga.

<!--more-->
## Co wiemy o danych ciasteczka?

Gdy użytkownik odwiedza nasz serwis i uzyskuje dostęp do określonych jego zasobów, np. dashboard,
przesyłane są przy pomocy ciasteczka dane uwierzytelniające, które są następnie sprawdzane przez
funkcję `wp_validate_auth_cookie()` . Poniżej jest wyciągnięte ciasteczko przy pomocy snifera
wireshark:

![](/img/2015/05/1.wordpress-cookie.png#huge)

Jeśli komuś udałoby się podrobić te zaznaczoną pozycję, mógłby on uzyskać dostęp do konta na
stronie. Poniżej sprawdzimy czy wykucie takiego ciasteczka i tym samym obejście zabezpieczeń
WordPressa jest w ogóle możliwe.

Przede wszystkim, poszczególne parametry ciasteczka są od siebie oddzielone przy pomocy `|` ale
wyżej w kodzie reprezentacją tego znaku jest `%7C` . Całe ciasteczko można więc rozbić na poniższe
wartości:

    wordpress_d6ea62b26612d05d8092cb54fc07b67d
    morfik
    1433146289
    o9F7AdaHdVVM8peu4XzAHFgXSo0QGiXmUpe47K5PPtP
    cea82a90a7cfb6d4f1aa676c52de77795eb3fbfaa630c6a9d031523224be46b8

Pierwsza linijka to nazwa ciasteczka. Następnie mamy nazwę użytkownika, który wysłał ciasteczko do
sprawdzenia. Dalej zaś jest linuxowy timestamp , a później mamy token oraz hash. By podrobić
ciasteczko, potrzebujemy wszystkich tych danych.

Jako, że Wordpress to projekt opensource, możemy skorzystać z tego przywileju i zajrzeć w jego kod.
Interesować nas będą głównie dwa pliki: `wp-includes/default-constants.php` oraz
`wp-includes/pluggable.php` . To na ich podstawie możemy ocenić czy mechanizm uwierzytelniający,
który WordPress ma zaimplementowany, jest bezpieczny.

### Nazwa ciasteczka

Za nazwę, jaką przybierze ciasteczko, odpowiada plik `default-constants.php` , a konkretnie poniższy
jego kod:

    if ( !defined( 'COOKIEHASH' ) ) {
          $siteurl = get_site_option( 'siteurl' );
          if ( $siteurl )
                define( 'COOKIEHASH', md5( $siteurl ) );
          else
                define( 'COOKIEHASH', '' );
    }

    ...

    if ( !defined('AUTH_COOKIE') )
          define('AUTH_COOKIE', 'wordpress_' . COOKIEHASH);

`COOKIEHASH` to hash md5 adresu bloga. Trzeba jednak pamiętać, że ten hash będzie się różnił w
zależności od tego czy w grę wchodzi protokół SSL, który przepisuje adres z http na https. Zatem
nazwa ciasteczka, zdefiniowana wyżej jako `AUTH_COOKIE` , składa się z dwóch połączonych ze sobą
wartości: `wordpress_` oraz tego hasha md5.

Jeśli ktoś chciałby uzyskać hash md5 jakiejś witryny, może to zrobić przy pomocy poniższego
polecenia:

    $ echo -n http://blog.lh | md5sum
    1d8e95c9e916e9c049bad8285a33990a  -

Tutaj mała uwaga -- nie możemy dodawać do adresu `/` na jego końcu.

### Nazwa użytkownika

Nazwa użytkownika zwykle jest znana, przynajmniej [jeśli nie
zadbamy](/post/wordpress-administrator-bloga/) o nią odpowiednio. Jeśli jednak
podjęliśmy ku temu stosowne kroki, to może być tutaj pewna rozbieżność.

### Czas ważności ciasteczka

Ciasteczka generowane przez WordPress mają pewien określony termin ważności, po którym to takie
ciastko staje się zwyczajnie bezużyteczne. W standardzie są to `2` dni, liczone od utworzenia
ciasteczka. Sama forma tego parametru, to linuxowy timestamp. Zrzućmy zatem okiem na plik
`pluggable.php` , bo ten czas jest tam dokładnie
    określony:

    $expiration = time() + apply_filters( 'auth_cookie_expiration', 2 * DAY_IN_SECONDS, $user_id, $remember );

Jeśli `$remember` zostałby ustawiony, wtedy ważność ciasteczka zostanie przeciągnięta do `14` dni.
Natomiast WordPress zaakceptuje każde ciasteczko pod warunkiem, że jego czas będzie ważny, czyli
będzie miał mniej niż `2` lub `14` dni. Ważność ciasteczka możemy za to sprawdzić przy pomocy
poniższego polecenia:

    $ date -d "@1433146289"
    Mon Jun  1 10:11:29 CEST 2015

### Token

Bez tokenów nie jesteśmy w stanie opublikować postu/strony ani też nawet zaktualizować skryptu
WordPressa czy jego wtyczek. Okazuje się bowiem, że w przypadku gdy [wyraźnie wylogujemy się z
systemu](https://core.trac.wordpress.org/ticket/20276) (przez wciśnięcie wyloguj), to ciasteczka w
dalszym ciągu mogą być używane. Token ma na celu poprawienie tego problemu. Nie wpływa on jednak na
losowość ciastka, a ma jedynie za zadanie przywiązać ciasteczka do sesji użytkownika, która może
wygasnąć:

    if ( ! $token ) {
          $manager = WP_Session_Tokens::get_instance( $user_id );
          $token = $manager->create( $expiration );
    }

### Hash

Rodzaj hasha jaki zostanie użyty zależy od właściwości serwera, a konkretnie od konfiguracji PHP,
tj. czy będą do dyspozycji dodatkowe algorytmy. Taki warunek widnieje w pliku `pluggable.php` :

    $algo = function_exists( 'hash' ) ? 'sha256' : 'sha1';

Krótko mówiąc, jeśli istnieje jest funkcja `hash` to zostanie użyty `sha256` , w przeciwnym wypadku
`sha1` . Chodzi o to, że funkcja PHP `hash_hmac()` nie obsługuje `sha256` . Jeśli nie wiemy, który
hash zostanie użyty na konkretnym serwerze, zawsze możemy to sprawdzić via `<php phpinfo() ?>` :

![](/img/2015/05/2.php-hash.png#huge)

Dalej w pliku mamy kod odpowiadający za generowanie samego hasha:

    $hash = hash_hmac( $algo, $user->user_login . '|' . $expiration . '|' . $token, $key );

W zależności od użytego algorytmu, hash będzie miał, albo 64 (sha256) znaki, albo 40(sha1). W tym
przypadku został wykorzystany algorytm `sha256` . Natomiast w skład ciągu, który zostanie
zahashowany wchodzi nazwa użytkownika, linuxowy timestamp oraz token. Funkcja `hash_hmac()`
wykorzystuje także tajny klucz
    współdzielony:

    $key = wp_hash( $user->user_login . '|' . $pass_frag . '|' . $expiration . '|' . $token, $scheme );

Jest to hash md5 nazwy użytkownika połączonej z kawałkiem hasha jego hasła, linuxowym timestampem
oraz tokenem i scheme.

Zmienna `$pass_frag` to `4` znaki hasha hasła od pozycji `8`:

    $pass_frag = substr($user->user_pass, 8, 4);

Zatem jeśli hasło w bazie danych ma postać `$P$B5TB6BnD0szKav74fMjSP77UAr3LoI/` , to wyciągnięte w
ten sposób znaki odpowiadają `BnD0` . Ten hash hasła składa się z dużych i małych liter, cyfr i
dwóch znaków specjalnych `.` oraz `/` , zatem ilość kombinacji nie powala i jest to 16,777,216,
przy czym, nie ma tutaj znaczenia czy użytkownik wybrał sobie hasło 4 czy 40 znakowe.

Jeśli chodzi zaś o zmienną `scheme` , to przybiera ona jedną z 3 wartości `auth`, `secure_auth` oraz
`logged_in` . Domyślnie jest to `auth` .

## Klucze i sole zdefiniowane w wp-config.php

Wszystkie z powyższych parametrów można przewidzieć z większym lub mniejszym prawdopodobieństwem i
tu właśnie do gry wchodzi plik `wp-config.php` i zdefiniowane w nim klucze oraz sole. Samym plikiem
`wp-config.php` zajmiemy się później, teraz zaś rzućmy okiem jeszcze na plik `pluggable.php` . Mamy
tam poniższy kod:

    function wp_hash($data, $scheme = 'auth') {
          $salt = wp_salt($scheme);

          return hash_hmac('md5', $data, $salt);
    }

I jak widzimy, zaprzęgnięta jest do roboty funkcja `wp_salt()` , która generuje sól z podanych w
`wp-config.php` wartości (akurat w tym przypadku) `AUTH_KEY` oraz `AUTH_SALT` :

    $cached_salts[ $scheme ] = $values['key'] . $values['salt'];

W ten sposób zasolonych ciasteczek nie da się przewidzieć.

## Zabezpieczamy ciasteczka

Do dyspozycji mamy 4 pary klucz-sól: `AUTH_KEY/AUTH_SALT` , `SECURE_AUTH_KEY/SECURE_AUTH_SALT` ,
`LOGGED_IN_KEY/LOGGED_IN_SALT` oraz `NONCE_KEY/NONCE_SALT` . Pierwsze trzy pary zabezpieczeają
ciasteczka, ostatnia para zaś jest odpowiedzialna za [ochronę danych
przesyłanych](https://codex.wordpress.org/WordPress_Nonces) w URLach oraz formularzach przed
pewnymi rodzajami nadużyć.

Ciasteczka, nazwijmy je roboczo `AUTH` oraz `SECURE_AUTH` , są przesyłane w przypadku korzystania z
protokołów `http` oraz `https` i odpowiadają jedynie za panel admina dając tym samym możliwość
wprowadzania zmian na blogu. W zależności od tego czy podczas logowania korzystamy z protokołu SSL,
zostanie wysłane tylko jedno z tych ciasteczek. Natomiast ciasteczko `LOGGED_IN` jest odpowiedzialne
za status zalogowania na blogu i jest przesyłane zawsze bez znaczenia czy przeglądamy stronę po http
czy https. Nie uzyskamy za jego pomocą dostępu do panelu administracyjnego i nie wprowadzimy żadnych
zmian w serwisie.

Te klucze są bardzo pożądane z punktu widzenia bezpieczeństwa witryny. WordPress ułatwia nawet ich
generowanie przy pomocy [tego adresu](https://api.wordpress.org/secret-key/1.1/salt/) . Możemy
oczywiście sami sobie wygenerować te 64 znakowe ciągi, z tym, że nie wiem czy są jakieś ograniczenia
co do ich długości. Na stronie WordPressa jest jedynie informacja, że te klucze powinny być długie i
możliwie losowe, no i oczywiście, każdy z nich inny. Jako, że takie ciasteczko jest proste do
przechwycenia, powinniśmy [rozważyć wymuszenie korzystania z protokołu
SSL](/post/wordpress-szyfrowanie-ssltls/) przynajmniej w przypadku logowania i
korzystania z panelu administracyjnego.

Jeśli nie podamy tych kluczy w pliku `wp-config.php` , zostaną one automatycznie wygenerowane przez
WordPressa i umieszczone w bazie danych w tabeli `wp_options` . Trzeba się liczyć z faktem, że jest
to zagrożenie bezpieczeństwa i nie powinniśmy do takiej sytuacji doprowadzać. Choć znowu bez
przesady w drugą stronę, bo potencjalny atakujący musiałby uzyskać dostęp do bazy danych. Niemniej
jednak, jeśli klucze i sole trzymamy osobno w pliku, ten ktoś musiałby również uzyskać dostęp do
systemu plików serwera tak by i ten plik pozyskać. Dlatego trzeba pamiętać, że [prawa dostępu do
pliku](/post/wordpress-ograniczone-prawa-dostepu/) `wp-config.php` powinny być
bardzo restrykcyjne.

Nie musimy się tymi kluczami za to zbytnio przejmować w środowisku produkcyjnym, tj. nic się nie
stanie jeśli je zgubimy i będziemy musieli wygenerować nowe klucze. Niemniej jednak, gdy to robimy,
ciasteczka wszystkich użytkowników, których mamy w swojej bazie danych, z automatu przestaną
działać. Ale bez obaw, taki użytkownik będzie mógł się bez problemu zalogować i zostaną mu
przesłane nowe ciasteczka z hashem wygenerowanym przy pomocy nowych kluczy. Raz na jakiś czas
powinniśmy wygenerować nowe klucze/sole tak by unieważnić w ten sposób wszelkie potencjalnie
skompromitowane ciasteczka.

### Stała COOKIEHASH

Domyślnie stała `COOKIEHASH` jest ustawiona na hash md5 URLu strony bloga (bez końcowego `/`) i by
utrudnić potencjalnemu atakującemu zadanie, przydałoby się przepisać ten parametr, tak by nie był
znowu aż tak przewidywalny. Możemy to zrobić przez plik `wp-config.php` dodając w nim poniższy kod:

    define('COOKIEHASH', md5($siteurl . 'rozowe-misie'));

Nie zmieniliśmy dużo, jedynie dodaliśmy do adresu URL strony jakiś ciąg znaków i teraz hash md5
zostanie wygenerowany w oparciu o `http://moj-blog.comrozowe-misie` , co powinno nieco utrudnić
odgadnięcie wartości stałej `COOKIEHASH` .
