---
author: Morfik
categories:
- WordPress
date: "2015-05-24T13:33:53Z"
date_gmt: 2015-05-24 11:33:53 +0200
published: true
status: publish
tags:
- seo
- blog
GHissueID: 221
title: 'WordPress: Odnośniki bezpośrednie (permalinks)'
---

Permalinks, czyli [odnośniki bezpośrednie](https://codex.wordpress.org/Using_Permalinks), stosowane
są głównie w celach estetycznych oraz by poprawić nieco
[SEO](https://pl.wikipedia.org/wiki/Optymalizacja_dla_wyszukiwarek_internetowych) danego artykułu
czy postu. Jeśli chodzi o efekty wizualne, to raczej wszyscy się zgodzimy, że link w postaci
`morfitronik.pl/wordpress-administrator-bloga/` jest o wiele prostszy do odszyfrowania dla człowieka
niż link typu `morfitronik.pl/?p=1` . Natomiast w przypadku SEO mamy kluczowe słowa czy fazy
bezpośrednio w linku, który jest indeksowany i wyświetlany przez wyszukiwarki, np. google. W tym
wpisie postaramy skonfigurować sobie właśnie te odnośniki.

<!--more-->
## Permalinks w WordPress'ie

Przede wszystkim, odnośniki bezpośrednie możemy ustawić z poziomu panelu administracyjnego
(Settings => Permalinks):

![](/img/2015/05/1.wordpress-permalinks.png#huge)

I jak widzimy wyżej, mamy do dyspozycji kilka opcji ale nie wszystkie są dobrym rozwiązaniem.
Generalnie, z punktu widzenia SEO, powinniśmy uwzględnić w linku jedynie nazwę postu, pomijając przy
tym zupełnie kategorie czy daty (choć moim zdaniem daty są jak najbardziej racjonalne, przynajmniej
sam rok), a już na pewno nie powinniśmy używać `?p=1` .

Jeśli przyjrzymy się uważniej, dostrzeżemy, że w proponowanych linkach mamy pozycję odnoszącą się do
skryptu WordPress'a: `/index.php/` . Przydałoby się obciąć ten kawałek i już nie tylko ze względu
estetyki czy SEO ale również z powodu bezpieczeństwa, bo niekoniecznie musimy się wszystkim chwalić,
który skrypt został wywołany. Choć w sumie, czy to ma znaczenie w WordPress'ie?

Tak czy inaczej, przy próbie wycięcia `/index.php/` z permalinków, nasze posty zwyczajnie się nie
będą chciały załadować. Teoretycznie adres jest poprawny ale po przejściu do niego jest zwracany
komunikat: "**Not Found The requested URL /blog/hello-world/ was not found on this server.**", a to
z kilku powodów.

## Prawa dostępu do pliku .htaccess

W zależności od typu instalacji WordPress'a, dostęp do pewnych plików może być utrudniony i pewnych
przypadkach WordPress nie będzie w stanie stworzyć sobie pliku `.htaccess` , o czym nas powinien
poinformować poniższym komunikatem:

![](/img/2015/05/3.wordpress-info-o-potrzebie-aktualizacji-htaccess.png#small)

Niżej tam na stronie po aktualizacji permalinków mamy instrukcję co dokładnie musimy zrobić aby nowe
adresy zaczęły obowiązywać. Konkretnie chodzi o dodanie poniższego bloku kodu do pliku `.htaccess` ,
który powinien być zlokalizowany w głównym katalogu WordPress'a:

    <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /blog/
    RewriteRule ^index\.php$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /blog/index.php [L]
    <IfModule>

Powyższy kod nie jest uniwersalny i trzeba go nieco przerobić -- w miejscach gdzie jest `blog`
wystarczy wstawić nazwę katalogu WordPress'a.

Oczywiście, ta powyższa zwrotka nie zadziała pod kątem wszystkich serwerów www i jest ona
przeznaczona raczej tylko dla Apache i to tylko tych z włączonym modułem `mod_rewrite` .

## Konfiguracja katalogu WordPress'a na serwerze

Jeśli stawiamy WordPress'a na swoim własnym serwerze www, to raczej natkniemy się na taki problem,
że ten powyższy kod zwyczajnie nie zadziała i nie będzie miało to znaczenia czy WordPress sam ten
kod sobie dopisze, czy też my zrobimy to za niego ręcznie. Jak można przeczytać
[tutaj](https://codex.wordpress.org/Using_Permalinks#Using_.22Pretty.22_permalinks), potrzebny jest
dodatkowy zabieg, który umożliwi zaaplikowanie reguł użytkownika, czyli tych wpisanych do pliku
`.htaccess` . Domyślnie, ze względów bezpieczeństwa, serwer Apache nie zezwala na tego typu operacje
i trzeba ręcznie wskazać katalogi, w których można stosować ten plik.

Edytujemy zatem plik konfiguracyjny Apache, na debianie jest to `/etc/apache2/apache2.conf` i
wrzucamy do niego poniższy kod:

    <Directory /apache/blog>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
    </Directory>

Chodzi generalnie o dodanie
[dyrektywy](https://httpd.apache.org/docs/2.2/mod/core.html#allowoverride) `AllowOverride` , która
to mając wartość `All` umożliwi nadpisywanie wszystkich opcji w tym konkretnym katalogu za
pośrednictwem pliku `.htaccess` .
