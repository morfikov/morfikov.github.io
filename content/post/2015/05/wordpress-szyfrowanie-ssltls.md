---
author: Morfik
categories:
- WordPress
date: "2015-05-24T19:54:15Z"
date_gmt: 2015-05-24 17:54:15 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- blog
- ssl
- tls
GHissueID: 225
title: 'WordPress: Szyfrowanie SSL/TLS'
---

W dzisiejszych czasach bez szyfrowania ani rusz i na dobrą sprawę by prowadzić jakikolwiek serwis
www, trzeba wliczyć w koszty overhead jaki niesie ze sobą zaszyfrowanie zawartości takiej witryny.
Oczywiście, niektórzy minimalizują straty ograniczając się jedynie do zaszyfrowania formularza
rejestracji/logowania lub też i ewentualnego panelu administracyjnego. Tak czy inaczej, [WordPress
domyślnie nie szyfruje niczego](https://codex.wordpress.org/Administration_Over_SSL) i przydałoby
się zmienić nieco jego konfigurację.

Poniższe instrukcje zadziałają jedynie w przypadku gdy pod serwer mamy już podpięty certyfikat. Nie
będę tutaj opisywał jak poprawnie skonfigurować serwer Apache by włączyć w nim obsługę szyfrowania
SSL/TLS. Jak znajdę trochę czasu, to na pewno wstawię tutaj link do przystępnego opisu.

<!--more-->
## Konfiguracja adresu

Na sam początek przydałoby się odpowiednio dostosować adres bloga, bo ten standardowo zaczyna się od
**http** . Nie jest to nic trudnego i można to zrobić z poziomu panelu WordPress'a (Settings =>
General):

![](/img/2015/05/1.przepisanie-adresu-bloga-na-https.png#big)

## Formularz logowania/rejestracji

Jeśli interesuje nas ochrona danych wprowadzanych w formularzu logowania czy rejestracji na blogu,
wystarczy, że dopiszemy poniższą linijkę do pliku `wp-config.php` zlokalizowanym w głównym katalogu
bloga:

    define( 'FORCE_SSL_LOGIN', true );

## Panel administracyjny

Za szyfrowanie panelu administracyjnego odpowiada poniższy kod:

    define( 'FORCE_SSL_ADMIN', true );

Choć nie musimy go dorzucać do pliku `wp-config.php` , bo ustawienie `FORCE_SSL_LOGIN` automatycznie
implikuje również i opcję.

## Zawartość bloga

Na dobrą sprawę zawartości bloga nie musimy szyfrować, bo jak to ludzie mówią: "przecie tam jest
tylko zwykły tekst" ale przecie za tym pozornie niegroźnym blokiem tekstu idzie także informacja na
temat zainteresowań osoby przeglądającej bloga i można na tej podstawie zbierać dane na temat
czyichś upodobaniach. Dlatego też ja wyznaję bardzo prostą zasadę -- zaszyfrować wszystko lub nie
szyfrować nic. Trzeba jednak przy tym pamiętać, że szyfrowanie całej zawartości strony może także
nieco spowolnić działanie naszego bloga. Dlatego też jeśli mamy słaby hosting, to lepiej jest
odpuścić sobie ten krok.

Nie znalazłem wprawdzie WordPress'owego sposobu, który by umożliwiał wymuszenie szyfrowania całego
bloga ale za to natknąłem się na przepisywanie adresów via plik `.htaccess` . Jeśli korzystamy z
[permalinków](/post/wordpress-odnosniki-bezposrednie-permalinks/) (a powinniśmy),
to zwrotka dla WordPress'a w tym pliku powinna teraz wyglądać mniej więcej tak:

    # Force SSL, see also apache virtual hosts
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /
        RewriteCond %{HTTPS} !=on
        RewriteRule ^(.*) https://%{HTTP_HOST}/$1 [R,L]
    </IfModule>

    # Wordpress permalinks
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /blog/
        RewriteRule ^index\.php$ - [L]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /blog/index.php [L]
    </IfModule>

Kolejność obu bloków ma znaczenie -- ten od SSL musi być przed przepisaniem permalinków. Po tym
zabiegu, cała zawartość bloga powinna zostać zaszyfrowana i nawet próba odwiedzenia strony po
**http** skończy się załadowaniem jej po **https** .

## Blokowanie nieszyfrowanej treści

Jeśli już zdecydowaliśmy się na szyfrowanie bloga (niekoniecznie w pełni), w pewnych sytuacjach
spotkamy się z komunikatem na pasku adresu w przeglądarce, który będzie nas informował, że część
elementów strony została zablokowana:

![](/img/2015/05/2.blokowanie-kontentu-http.png#big)

Zwykle jest to wina niezabezpieczonych adresów URL i pod terminem "niezabezpieczone" rozumiem
zawartość na stronie pochodzącą z linków zaczynających się od **http** . [Na jednym z blogów
mozilli](https://blog.mozilla.org/tanvi/2013/04/10/mixed-content-blocking-enabled-in-firefox-23/)
jest artykuł na ten temat i z niego możemy dowiedzieć się iż generalnie rzecz biorąc tego typu
sytuacja nie powinna mieć miejsca, bo może skompromitować całą zaszyfrowaną komunikację, wobec czego
zawartość serwowana po **http** jest zwykle blokowana. Poza tym, możemy też wyczytać, że z reguły
większość stron działa poprawnie pomimo tej blokady i nie trzeba nic robić by połączenie było
zabezpieczone.

Tak czy inaczej, dobrze jest chociaż pokusić się o sprawdzenie tego jaki kontent jest blokowany i
[poprawić](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content/How_to_fix_website_with_mixed_content)
to co możemy. Najczęściej będą to linki do grafiki, które będą się pojawiać na naszej stronie.
