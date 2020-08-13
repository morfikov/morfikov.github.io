---
author: Morfik
categories:
- Linux
date:    2020-08-13 23:50:00 +0200
lastmod: 2020-08-13 23:50:00 +0200
published: true
status: publish
tags:
- debian
- blog
- hugo
- jekyll
title: Migracja bloga z Jekyll na Hugo i jego publikacja w GitHub Pages
---

Prawdopodobnie zauważyliście drobne zmiany w wyglądzie tego bloga oraz pewnie też cześć osób miała
w ostatnim czasie problemy z uzyskaniem do niego dostępu. Odpowiedzialne za zaistniałą sytuację są
[limity narzucone przez GitHub Pages][1]. Zakładają one maksymalny rozmiar repozytorium pod stronę
WWW w granicach 1 GiB oraz czas budowania takiego serwisu krótszy niż 10 minut. Limit miejsca na
pliki przekroczony został w zasadzie w chwili przeniesienia bloga WordPress na GitHub Pages.
Natomiast parę dni temu został przekroczony limit czasu budowania tego projektu. Do momentu
transformacji, niniejszy blog działał w oparciu o [Jekyll][2], który generował w zasadzie statyczny
kod HTML ze zwykłych plików tekstowych (np. artykuły pisane w [MarkDown][3]). [GitHub Pages
posiada wsparcie dla Jekyll][4], przez co taką stronę WWW można bardzo prosto wdrożyć. Niemniej
jednak, cały ten mechanizm generowania projektu w obrębie infrastruktury GitHub'a zajmuje bardzo
dużo czasu. Po rozmowie z supportem okazało się, że gdyby chodziło o miejsce na pliki, to mogli by
nagiąć reguły i nie było by problemu ale nie dadzą rady tego zrobić w przypadku przekroczenia czasu
generowania projektu. Dlatego też trzeba było zrezygnować z Jekyll'a i poszukać dla niego jakiejś
alternatywy. Padło na [Hugo][5], który w odróżnieniu od Jekyll'a nie jest jako tako wspierany przez
GitHub i by [stronę wygenerowaną przez Hugo podpiąć pod GitHub Pages][6] trzeba się trochę wysilić,
bo ten proces nie jest automatyczny i właśnie o tym jak dokonać migracji z Jekyll na Hugo w
kontekście GitHub Pages będzie ten poniższy artykuł.

<!--more-->
## Hugo i Debian

Jako, że projekt Hugo jest otwartoźródłowy, to stosowny pakiet do instalacji w systemie znajduje
się w głównym repozytorium Debiana. Możemy go zatem bez najmniejszego problemu zainstalować przy
pomocy menadżera pakietów `apt-get`/`aptitude` :

    # aptitude install hugo

## Importowanie projektu Jekyll do Hugo

By ułatwić nieco przejście z Jekyll na Hugo, istnieje możliwość zaimportowania naszego bloga, tak
by nie musieć przeprowadzać tego procesu ręcznie. Projekt będzie importowany do osobnego katalogu,
także nie ma co się bać ewentualnej utraty danych. By zaimportować nasz serwis WWW do Hugo,
wpisujemy w terminalu polecenie `hugo import jekyll` podając mu dwa argumenty w postaci ścieżki do
katalogu z projektem Jekyll oraz ścieżki do katalogu wynikowego:

    $ hugo import jekyll ~/morfikov.jekyll/ ~/morfikov.hugo/

Po chwili, nasz serwis powinien znaleźć się w miejscu docelowym. Przechodzimy do tego katalogu:

    $ cd ~/morfikov.hugo/

Od tego momentu, wszystkie polecenia w tym wpisie będą wydawane względem tego katalogu roboczego.

## Repozytorium git dla bloga

Strony hostowane na GitHub Pages są utrzymywane w systemie kontroli wersji git. W przypadku mojego
bloga niestety jego import (i późniejsze dostosowanie plików) wymagał bardzo dużej ilości zmian,
przez co pliki repozytorium by się dość znacznie rozrosły i zaczęłyby niepotrzebnie marnować
miejsce zarówno na dysku mojego laptopa, jak i w zdalnym repozytorium na GitHub. Dlatego też po
przygotowaniu całego projektu zdecydowałem się na zaoranie katalogu `.git/` oraz usunięcie projektu
GitHub Pages (zdalnego repo) i utworzenie na jego miejsce świeżego repozytorium.

    $ git init
    $ git add .
    $ git commit -S -m "move the blog from jekyll to hugo"
    $ git remote add origin git@github.com:morfikov/morfikov.github.io.git
    $ git push --set-upstream origin master

Zanim popchniemy zmiany do zdalnego repozytorium, upewnijmy się, że w katalogu projektu Hugo nie ma
żadnych wygenerowanych plików serwisu, tj. tych wygenerowanych przez Jekyll/Hugo. To co ma zostać
pchnięte do zdalnego zdalnego repozytorium, to tylko i wyłącznie pliki źródłowe bloga, z których
Hugo będzie budował w późniejszym czasie nasz serwis.

## Motyw dla Hugo

Pierwszą rzeczą, którą będziemy musieli zrobić po imporcie bloga, to poszukać jakiegoś motywu
(theme), który będzie odpowiedzialny za wyrenderowanie strony. Bez tego motywu, w przeglądarce
przywita nas jedynie biała strona. [Skórek dla Hugo jest cała masa][7] i z pewnością znajdziemy tę,
która nam będzie odpowiadać. Jeśli jednak będziemy mieli z tym problem, to będziemy musieli napisać
własny motyw. Ja zdecydowałem się na [Binario][8] jako, że jego kolorystyka bardzo przypomina mi
wygląd pulpitu mojego linux'a.

Motywy dla Hugo są przechowywane w systemie kontroli wersji git, podobnie jak nasz projekt GitHub
Pages. Wiążą się z tym pewne komplikacje, mianowicie projekt skórki trzeba będzie dołączyć do
projektu naszego serwisu WWW w [postaci modułu][9]. Zarówno nasz blog jak i jego skórka będą
posiadać dwa niezależne repozytoria git i nie mogą ze sobą w żaden sposób kolidować. By dodać moduł,
wpisujemy w terminal poniższe polecenie:

    $ git submodule add https://github.com/vimux/binario themes/binario

    $ git status
    On branch master

    No commits yet

    Changes to be committed:
      (use "git rm --cached <file>..." to unstage)
            new file:   .gitmodules
            new file:   themes/binario

Jak widać, wydanie tego polecenia wygenerowało plik `.gitmodules` oraz pojawił się też katalog
`themes/binario/` . W `themes/binario/` jest zawartość motywu Hugo, a w `.gitmodules` mamy poniższą
treść:

    [submodule "themes/binario"]
        path = themes/binario
        url = https://github.com/vimux/binario

Dodatkowo w pliku `.git/config` została dodana taka zwrotka:

    [submodule "themes/binario"]
        url = https://github.com/vimux/binario
        active = true

Potwierdzamy dodanie plików modułu (motywu Hugo) do głównego repozytorium:

    $ git commit -S -m "add the binario theme submodule"
    $ git push origin master

## Lokalny test Hugo

Powinniśmy w tej chwili mieć już zaimportowany stary serwis Jekyll'a oraz jeden motyw do
wyrenderowania strony WWW. W głównym katalogu repozytorium powinien znajdować się plik
`config.toml` , w którym to określamy szereg opcji konfiguracyjnych dla Hugo. Poniżej znajduje się
cały mój `config.toml` :

    baseurl          = "https://morfikov.github.io"
    title            = "Morfitronik"
    languageCode     = "pl-PL"
    paginate         = "10" # Number of elements per page in pagination
    theme            = "binario"
    disqusShortname  = "morfitronik" # Enable comments by entering your Disqus shortname
    googleAnalytics  = "UA-119125303-1" # Enable Google Analytics by entering your tracking id
    canonifyURLs     = true
    #summarylength   = 100
    rssLimit         = 20
    defaultContentLanguage = "pl"
    #publishDir      = "docs"

    [taxonomies]
      category = "categories"
      tag = "tags"

    [permalinks]
      post = "/post/:filename/"

    [sitemap]
      changefreq = "weekly"
      filename = "sitemap.xml"
      priority = 1.0

    [Author] # Used in authorbox
      name   = "Mikhail Morfikov"
      bio    = "Po ponad 10 latach spędzonych z różnej maści linux'ami (Debian/Ubuntu, OpenWRT,
      Android) mogę śmiało powiedzieć, że nie ma rzeczy niemożliwych i problemów, których nie da
      się rozwiązać. Jedną umiejętność, którą ludzki umysł musi posiąść, by wybrnąć nawet z tej
      najbardziej nieprzyjemniej sytuacji, to zdolność logicznego rozumowania."
      avatar = "img/avatar.png"

    [Params]
      description        = "Blog o bezpieczeństwie, prywatności oraz systemach linux (Debian/Ubuntu, OpenWRT/LEDE i Android)" # Site Description. Used in meta description
      copyright          = "Mikhail Morfikov" # Copyright holder, otherwise will use .Site.Title
      opengraph          = true # Enable OpenGraph if true
      schema             = true # Enable Schema
      twitter_cards      = true # Enable Twitter Cards if true
      columns            = 2 # Set the number of cards columns. Possible values: 1, 2, 3
      mainSections       = ["post"] # Set main page sections
      dateFormat         = "02/01/2006" # Change the format of dates
      colorTheme         = "" # dark-green, dark-blue, dark-red, dark-violet
      customCSS          = ["css/custom.css"] # Include custom CSS files
      customJS           = ["js/custom.js"] # Include custom JS files
      mainMenuAlignment  = "right" # Align main menu (desktop version) to the right side
      authorbox          = true # Show authorbox at bottom of single pages if true
      comments           = true # Enable comments for all site pages
      related            = true # Enable Related content for single pages
      relatedMax         = 5 # Set the maximum number of elements that can be displayed in related block. Optional
      mathjax            = true # Enable MathJax for all site pages
      mathjaxPath        = "https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.6/MathJax.js" # Specify MathJax path. Optional
      mathjaxConfig      = "TeX-AMS-MML_HTMLorMML" # Specify MathJax config. Optional
      hideNoPostsWarning = false # Don't show no posts empty state warning in main page, if true

    [Params.Entry]
      meta    = ["date", "categories"] # Enable meta fields in given order
      toc     = true # Enable Table of Contents
      tocOpen = false # Open Table of Contents block. Optional

    [Params.Featured]
      previewOnly = false # Show only preview featured image

    [Params.Breadcrumb]
      enable   = true # Enable breadcrumb block globally
      homeText = "Home" # Home node text

    [Params.Social]
      email         = "morfik@nsa.com"
    # facebook      = "username"
      twitter       = "mikhailmorfikov"
      telegram      = "morfikov"
    # instagram     = "username"
    # pinterest     = "username"
    # vk            = "username"
    # linkedin      = "username"
      github        = "morfikov"
      gitlab        = "morfikov"
      stackoverflow = "3015317"
    # mastodon      = "https://some.instance/@username"
    # medium        = "username"

    [Params.Share] # Entry Share block
      facebook  = true
      twitter   = true
      reddit    = true
      telegram  = true
      linkedin  = true
      vk        = true
      pocket    = true
      pinterest = true

    # Web App Manifest settings
    # https://www.w3.org/TR/appmanifest/
    # https://developers.google.com/web/fundamentals/web-app-manifest/
    [Params.Manifest]
      name            = "Morfitronik"
      shortName       = "Morfitronik"
      display         = "browser"
      startUrl        = "/"
      backgroundColor = "#2a2a2a"
      themeColor      = "#1b1b1b"
      description     = "Blog o bezpieczeństwie, prywatności oraz systemach linux (Debian/Ubuntu, OpenWRT/LEDE i Android)"
      orientation     = "portrait"
      scope           = "/"

    [outputFormats]
      [outputFormats.MANIFEST]
        mediaType      = "application/json"
        baseName       = "manifest"
        isPlainText    = true
        notAlternative = true
      [outputFormats.RSS]
        mediatype      = "application/rss"
        baseName       = "feed"

    [mediaTypes]
      [mediaTypes."application/rss"]
        suffixes = ["xml"]

    [outputs]
      home = ["HTML", "RSS", "MANIFEST"]

    [menu]
    #  [[menu.main]]
    #    identifier = "archives"
    #    name       = "Archiwum"
    #    url        = "/archives/"
    #    weight     = 10

      [[menu.main]]
        identifier = "categories"
        name       = "Kategorie"
        url        = "/categories/"
        weight     = 20

      [[menu.main]]
        identifier = "tags"
        name       = "Tagi"
        url        = "/tags/"
        weight     = 30

    [markup]
      defaultMarkdownHandler = "goldmark"
      [markup.asciidocExt]
        backend = "html5"
        docType = "article"
        extensions = []
        failureLevel = "fatal"
        noHeaderOrFooter = true
        safeMode = "unsafe"
        sectionNumbers = false
        trace = false
        verbose = true
        workingFolderCurrent = false
        [markup.asciidocExt.attributes]
      [markup.blackFriday]
        angledQuotes = false
        footnoteAnchorPrefix = ""
        footnoteReturnLinkContents = ""
        fractions = true
        hrefTargetBlank = false
        latexDashes = true
        nofollowLinks = false
        noreferrerLinks = false
        plainIDAnchors = true
        skipHTML = false
        smartDashes = false
        smartypants = false
        smartypantsQuotesNBSP = false
        taskLists = true
      [markup.goldmark]
        [markup.goldmark.extensions]
          definitionList = true
          footnote = true
          linkify = true
          strikethrough = true
          table = true
          taskList = true
          typographer = false
        [markup.goldmark.parser]
          attribute = true
          autoHeadingID = true
          autoHeadingIDType = "github"
        [markup.goldmark.renderer]
          hardWraps = false
          unsafe = false
          xhtml = false
      [markup.highlight]
        codeFences = true
        guessSyntax = false
        hl_Lines = ""
        lineNoStart = 1
        lineNos = false
        lineNumbersInTable = true
        noClasses = true
        style = "monokai"
        tabWidth = 4
      [markup.tableOfContents]
        endLevel = 5
        ordered = true
        startLevel = 2

Trochę tych opcji nazbierałem w drodze konfiguracji swojego bloga ale niekoniecznie potrzebujemy
ich wszystkich. Te najważniejsze to `baseurl` , który ma wskazywać na stronę naszego serwisu (w tym
przypadku, blog jest dostępny pod `https://morfikov.github.io` ). Dalej mamy `theme` , który
określa skórkę użytą przy renderowaniu strony. I to są w zasadzie kluczowe elementy, które musimy
skonfigurować, by Hugo nam wygenerował stronę.

Możemy teraz odpalić testowo Hugo, by sprawdzić czy nasz blog się bez problemu zbuduje:

    $ hugo server

Polecenie `hugo server` generuje lokalną wersję strony WWW, tj. blog będzie serwowany z adresu
pętli zwrotnej ( `127.0.0.1:1313` ) przez wbudowany w Hugo serwer WWW. To rozwiązanie jest
przeznaczone głównie do testów, choć też może być wykorzystywane na produkcji. Niemniej jednak,
zaleca się wykorzystywanie pełnowymiarowego serwera WWW do tego celu. W przypadku GitHub Pages,
taki serwer WWW będzie zapewniony i by wygenerować w późniejszym czasie stronę, trzeba będzie
ominąć `server` z powyższego polecenia i użyć jedynie `hugo` . Standardowo też projekt będzie
budowany w pamięci RAM. Jeśli mamy mało pamięci operacyjnej albo też chcemy by pliki były zapisywane
na dysku, to musimy określić opcję `--renderToDisk` . Dobrze jest także włączyć tryb rozmowny
`--verbose` oraz ostrzeżenia związane z translacją strony WWW `--i18n-warnings` . Reasumując, do
lokalnych testów bloga Hugo będziemy używać poniższego polecenia:

    $ hugo server --i18n-warnings --renderToDisk --verbose

Po wydaniu tego polecenia, nasz serwis powinien zostać zbudowany w katalogu `public/` i powinniśmy
być w stanie go wyświetlić w przeglądarce odwiedzając adres [http://127.0.0.1:1313][10] :

    $ hugo server --i18n-warnings --renderToDisk --verbose
    INFO 2020/08/13 21:30:03 Using config file:
    Building sites … INFO 2020/08/13 21:30:03 syncing static files to /home/morfik/morfikov.hugo/public/

                       |  PL
    -------------------+--------
      Pages            |  1049
      Paginator pages  |   273
      Non-page files   |     1
      Static files     | 10497
      Processed images |     0
      Aliases          |   254
      Sitemaps         |     1
      Cleaned          |     0

    Built in 69544 ms
    Watching for changes in /home/morfik/morfikov.hugo/{archetypes,content,data,i18n,layouts,static,themes}
    Watching for config changes in /home/morfik/morfikov.hugo/config.toml
    Environment: "development"
    Serving pages from /home/morfik/morfikov.hugo/public
    Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
    Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
    Press Ctrl+C to stop

## Projekt Hugo na GitHub Pages

Migrację bloga z Jekyll na Hugo mamy za sobą. Niemniej jednak, z tych plików projektu nie da rady
zbudować strony WWW i udostępnić jej za sprawą GitHub Pages. By strona wygenerowana przez Hugo mogła
zostać wyświetlona przez GitHub Pages musimy poczynić kilka dodatkowych kroków.

Podejścia są dwa. Pierwsze z nich zakłada zmianę nazwy budowanego katalogu z `public/` na `docs/`
jako, że GitHub Pages jest w stanie budować projekty z plików w katalogu `docs/` . Jeśli interesuje
nas to rozwiązanie, to wystarczy w konfiguracji Hugo (plik `config.toml` głównym katalogu
repozytorium git) określić parametr `publishDir` i podać mu wartość `docs` . My jednak stworzymy
nieco bardziej zaawansowaną konfigurację, która zakłada utworzenie nowej gałęzi dla wygenerowanych
plików. W taki sposób zmiany w kodzie źródłowym bloga będą utrzymywane osobno, w stosunku do zmian
wygenerowanej strony. Później w ustawieniach GitHub Pages wskażemy, że chcemy by projekt był
budowany z tej osobnej gałęzi. Zatem do dzieła.

Na początek tworzymy plik `.gitignore` w głównym katalogu repozytorium i dodajemy do niego nazwę
katalogu, w którym będą przechowywane wygenerowane przez Hugo pliki. W tym przypadku wykorzystywany
jest standardowy katalog, tj. `public/` :

    $ echo "public" >> .gitignore

Potwierdzamy zmiany w git:

    $ commit -S -m "add .gitignore"

Następnie musimy przygotować gałąź `gh-pages` i ją zainicjować:

    $ git checkout --orphan gh-pages
    $ git reset --hard
    $ git commit --allow-empty -m "Initializing gh-pages branch"
    $ git push origin gh-pages
    $ git checkout master

Po wydaniu polecenia `git reset --hard` prawdopodobnie nie będzie można skasować katalogu z
motywami Hugo. Pod żadnym pozorem nie kasujmy go ręcznie. Pozostawmy go w takiej formie jakiej jest
i zwyczajnie wydajmy następne polecenie, tj. `git commit ...` .

Musimy teraz usunąć cały katalog `public/` i [utworzyć go za pomocą git-worktree][11]:

    $ rm -rf public/
    $ git worktree add -B gh-pages public origin/gh-pages

    $ git worktree list
    /home/morfik/morfikov.hugo/         ebca1ad [master]
    /home/morfik/morfikov.hugo/public   52b3630 [gh-pages]

Ma to na celu stworzenie niezależnych drzew roboczych w osobnych katalogach i podpięcie ich pod
osobne gałęzie. W taki sposób pliki źródłowe naszego bloga (+ moduł motywu Hugo) będą w gałęzi
`master` , a wygenerowana zawartość strony w gałęzi `gh-pages` .

Przy pomocy polecenia `hugo` generujemy nasz blog na produkcję:

    $ hugo

                       |  PL
    -------------------+--------
      Pages            |  1044
      Paginator pages  |   272
      Non-page files   |     1
      Static files     | 10497
      Processed images |     0
      Aliases          |   252
      Sitemaps         |     1
      Cleaned          |     0

    Total in 41934 ms

Po wygenerowaniu, przechodzimy do katalogu `public/` , dodajemy wygenerowane pliki i zatwierdzamy
zmiany w git:

    $ cd public && git add --all && git commit -m "Publishing to gh-pages" && cd ..

Za każdym razem jak będziemy zmieniać zawartość strony, czy to poprawiać jej kod czy dodawać nowe
artykuły na blogu, te dwa powyższe polecenia trzeba będzie wydawać, by wygenerować nową stronę do
wdrożenia na produkcji.

Teraz już pozostaje nam pchniecie zmian do zdalnego repozytorium na GitHub:

    $ git push origin master
    $ git push origin gh-pages

Pierwsze z powyższych poleceń przesyła zmiany kodu źródłowego strony do zdalnej gałęzi `master` ,
drugie zaś wygenerowany kod strony do zdalnej gałęzi `gh-pages` . W przypadku wprowadzania zmian w
kodzie strony, tylko różnice będą przesyłane, choć rozmiary commit'ów  mogą czasem przytłoczyć.

### Kwestia opróżniania katalogu public/

Być może w którymś momencie w przyszłości zajdzie potrzeba opróżnienia zawartości katalogu
`public/` . Hugo w zasadzie nie operuje na tych plikach i wykorzystuje je jedynie do serwowania
strony przez wbudowany serwer WWW. Jak plików brakuje, to zostaną na nowo wygenerowane. Problemy
mogą się zacząć, gdy w grę wchodzi git. Trzeba pamiętać, że w katalogu `public/` utworzyliśmy
drzewo robocze, co z kolei skutkuje utworzeniem pliku `public/.git` . Jest to zwykły i do tego mały
plik tekstowy zawierający w zasadzie jedną linijkę:

	gitdir: /home/morfik/morfikov.hugo/.git/worktrees/public

Jeśli ten plik zostanie skasowany, to system przestanie widzieć zmiany w katalogu `public/` .
Dlatego też jeśli będziemy chcieli opróżnić zawartość katalogu `public/` , to nie ruszajmy tego
pliku -- wszystkie pozostałe pliki z tego katalogu możemy jak najbardziej skasować.

### GitHub Pages a gałąź do budowania projektu

Ostatnim krokiem jest wskazanie GitHub Pages gałęzi do budowania projektu. Teoretycznie GitHub
Pages powinien automatycznie podebrać gałąź `gh-pages` i z niej budować stronę WWW. Jeśli jednak
tak się nie dzieje, to wchodzimy w ustawienia zdalnego repozytorium:

![]({{< baseurl >}}/img/2020/08/001-github-pages-jekyll-hugo-settings.png#huge)

I przechodzimy do sekcji `GitHub Pages` :

![]({{< baseurl >}}/img/2020/08/002-github-pages-jekyll-hugo-settings-branch.png#huge)

Wybieramy gałąź `gh-pages` i zapisujemy ustawienia. Po paru minutach strona powinna zostać
wygenerowana. Proces generowania możemy śledzić przechodząc do gałęzi `gh-pages` :

![]({{< baseurl >}}/img/2020/08/003-github-pages-jekyll-hugo-build-check.png#huge)

![]({{< baseurl >}}/img/2020/08/004-github-pages-jekyll-hugo-build-check.png#huge)

I jak widać z powyższych fotek, blog zbudował się bez problemów. Co ciekawe, w porównaniu do 11-12
minut potrzebnych na zbudowanie projektu Jekyll, ten sam projekt Hugo buduje się na GitHub Pages w
około 2 minuty. oczywiście lokalnie pierw te pliki zbudowaliśmy ale i tak nawet dodając tę 1 minutę
lokalnego generowania strony, to jest około 3 minut. Zatem jeśli zdarzy nam się trafić na podobny
problem z uderzeniem w limit czasu generowania strony na GitHub Pages i korzystamy przy tym z
Jekyll'a, to pomyślmy nad przejściem na Hugo.


[1]: https://docs.github.com/en/github/working-with-github-pages/about-github-pages#usage-limits
[2]: https://jekyllrb.com/
[3]: https://www.markdownguide.org/
[4]: https://pages.github.com/
[5]: https://gohugo.io/
[6]: https://gohugo.io/hosting-and-deployment/hosting-on-github/
[7]: https://themes.gohugo.io/
[8]: https://github.com/vimux/binario/
[9]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[10]: http://127.0.0.1:1313
[11]: https://git-scm.com/docs/git-worktree
