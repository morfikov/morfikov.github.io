---
author: Morfik
categories:
- Linux
date: "2015-05-26T13:25:26Z"
date_gmt: 2015-05-26 12:25:26 +0200
published: true
status: publish
tags:
- bezpieczeństwo
- apache2
GHissueID: 220
title: Problemy wynikające z używania pliku .htaccess
---

Na stronie Apache jest [kawałek ciekawego
artykułu](http://httpd.apache.org/docs/2.0/howto/htaccess.html#when) na temat zagrożeń jakie niesie
ze sobą stosowanie pliku `.htaccess` w katalogach stron trzymanych na serwerze wwww. Oczywiście, w
pewnych sytuacjach, takich jak np. hostingi, nie będziemy mieć innego wyboru jak zdać się na ten
plik ale jeśli mamy bezpośredni dostęp do konfiguracji Apache, możemy przenieść wszystkie te wpisy z
pliku `.htaccess` i poprawić tym bezpieczeństwo witryny jak i po części wydajność samego serwera
www.

<!--more-->
## Zagrożenia wynikające z posiadania pliku .htaccess

Jeśli pozwalamy zwykłym użytkownikom na zmiany konfiguracji serwera, a to właśnie ma na celu plik
`.htaccess` , to będą oni w stanie wprowadzać zmiany, nad którymi nie będziemy mieli żadnej kontroli
jako administrator serwera www. Dlatego też jeśli zdecydowaliśmy się na nadanie użytkownikom
uprawnień dotyczących zmiany konfiguracji, to przynajmniej zastosujmy pewne restrykcje wynikające z
dyrektywy [AllowOverride](http://httpd.apache.org/docs/2.0/mod/core.html#allowoverride), bo nie
zawsze w każdym przypadku musimy stosować `All` .

Poza tym, jakby nie patrzeć, dostęp do pliku `.htaccess` powinien być bardzo restrykcyjny, bo mogą
znajdować się tam wrażliwe dane, a poza tym, figuruje on bezpośrednio w katalogu strony hostowanej
publicznie, przez co można uzyskać do niego dostęp i np. w wyniku błędnych uprawnień, zmienić go.
Dlatego też bezpieczne prawa do tego pliku powinny zostać ustawione jedynie na odczyt i to w taki
sposób by tylko Apache mógł go czytać. Można również się pokusić o dopisanie poniższej zwrotki do
konfiguracji wirtualnych hostów w Apache:

    <files "^\.ht">
        Order allow,deny
        Deny from all
    </files>

Choć standardowo powinna być ona już wpisana w głównym pliku konfiguracyjnym, bo dzięki takiemu
rozwiązaniu, próba odczytu pliku `.htaccess` (w sumie jakiegokolwiek pliki, którego nazwa zaczyna
się od `.ht` ) poprzez wywołanie jego adresu, np. w przeglądarce, nie będzie możliwa.

## Wydajność serwera www płynąca z braku pliku .htaccess

Jeśli chodzi o wydajność, to serwer Apache będzie szukał pliku `.htaccess` w każdym możliwym
folderze, a jeśli tych jest dużo, to niemal pewne jest, że wydajność na tym ucierpi. Poza tym, ten
plik jest wczytywany przy każdym zapytaniu, czyli przy odwiedzaniu kolejnych linków w naszej
witrynie, a to dodatkowe obciążenie. No i oczywiście, jeśli mamy szereg folderów, w których ten plik
występuje, to serwer musi przeanalizować wszystkie pliki `.htaccess` kolejno w nadrzędnych
katalogach, by zaaplikować wszystkie reguły.

Tak więc jeśli mamy przykładową strukturę folderów `/www/htdocs/example` , to żądanie o plik
`.htaccess` zostanie wysłane 4 razy:

    /.htaccess
    /www/.htaccess
    /www/htdocs/.htaccess
    /www/htdocs/example/.htaccess

## Przenoszenie zawartości pliku .htaccess

Dlatego też jeśli zależy nam na wydajności serwera oraz jego bezpieczeństwie i mamy przy tym dostęp
do konfiguracji swojego wirtualnego hosta, to przydałoby się przenieść całą zawartość pliku
`.htaccess` właśnie w to miejsce. Przykładowo, jeśli mamy blog WordPressa i stosujemy w nim
[permalinki](/post/wordpress-odnosniki-bezposrednie-permalinks/) , to będziemy
potrzebować poniższego kodu, by te linki zaczęły działać:

    <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /blog/
    RewriteRule ^index\.php$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /blog/index.php [L]
    <IfModule>

By przenieść ten kod, kopiujemy go "tak jak jest" i umieszczamy wewnątrz dyrektywy `<Directory>` w
pliku `/etc/apache2/apache2.conf` . Można również bezpośrednio w plikach wirtualnych hostów (katalog
`/etc/apache/sites-available/`), z tym, że jeśli nasz serwer ma zaimplementowaną obsługę SSL, to
musimy ten powyższy blok wpisać dwa razy.

Reasumując, konfiguracja permalinków w powyższym przykładzie powinna wyglądać mniej więcej tak:

    <Directory /apache/morfitronix>

        Options FollowSymLinks
        AllowOverride None
        Require all granted

    # Wordpress permalinks
        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteBase /
            RewriteRule ^index\.php$ - [L]
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteRule . /index.php [L]
        </IfModule>

    </Directory>

I w ten sposób, nie będziemy już musieli się martwić o treść pliku `.htaccess`, bo nie będzie miała
ona już dla nas żadnego znaczenia. Mało tego, serwer www nie będzie nawet brał pod uwagę tego pliku,
oczywiście w przypadku gdyby został przez kogoś utworzony.
