---
author: Morfik
categories:
- Blog
date:    2021-10-12 00:15:00 +0200
lastmod: 2021-10-25 11:30:00 +0200
published: true
status: publish
tags:
- hugo
- komentarze
- disqus
- github
- shell
- sed
- javascript
GHissueID: 574
title: Komentarze GitHub Issues na blogu Hugo korzystającym z GitHub Pages
---

Jakiś czas temu [Disqus wprowadził reklamy][1] w swoim systemie komentarzy i bez mojej świadomej
oraz dobrowolnej zgody serwował je użytkownikom odwiedzającym mojego bloga. W tamtym czasie nie
miałem za bardzo informacji jak te komentarze zastąpić na statycznej stronie renderowanej przy
pomocy Hugo. Jako, że sporo osób narzekało na to, że tutaj nie ma komentarzy, przez co liczba
maili, które do mnie docierały, była trochę ponad moje moce przerobowe. Postanowiłem zatem
zaimplementować system komentarzy oparty o GitHub Issues, czyli ten mechanizm, z którym
prawdopodobnie każdy z nas miał już styczność zgłaszając bug'a (czy też inne zapytanie) twórcom
swoich ulubionych linux'owych projektów OpenSource. Okazało się, że przy [pomocy API GitHub'a można
w dość prosty sposób te komentarze pokazać na blogu][2] ale problem pojawił się w przypadku stron
mających setki czy nawet tysiące wpisów już opublikowanych. Jak zatem na takich blogach wdrożyć
komentarze GitHub'a, tak by przy okazji nie wyskoczyć z okna o 5 nad ranem?

<!--more-->
## GitHub Issues i GitHub Pages

Mój blog [jest hostowany w infrastrukturze GitHub'a][3] za sprawą rozwiązania zwanego [GitHub
Pages][4]. W ten sposób nie trzeba opłacać hostingu, serwer jest przedniej klasy, a cały blog jest
utrzymywany w systemie kontroli wersji Git. To rozwiązanie ma jednak jedną wadę, tj. strona
serwowana przez GitHub Pages musi być w formie statycznej, czyli wyrenderowanego kodu HTML, który
jest wrzucany do repozytorium.

Taka statyczność strony rozwiązuje co prawda całą masę problemów natury bezpieczeństw czy to samej
witryny, czy też serwera WWW. Niemniej jednak, statyczność strony odbija się na jej funkcjonalności
i trzeba korzystać z innych mechanizmów, by tą utraconą funkcjonalność przywrócić. Tak jest w
przypadku komentarzy, które są dynamicznym kontentem zamieszczanym pod postami na blogu przez
różnych użytkowników.

Po usunięciu modułu Disqus'a, przez jakiś czas nie było komentarzy tutaj na blogu ale postanowiłem
ten stan rzeczy zmienić przy pomocy mechanizmu GitHub Issues, który można wykorzystać w roli systemu
komentarzy i te komentarze można wyświetlić pod konkretnymi postami na blogu.

### Konto na GitHub i potrzeba logowania

Komentarze oparte o GitHub Issues nie są pozbawione wad. Pierwszy problem dotyczy logowania. Disqus
oferował możliwość komentowania bez identyfikacji autorów komentarzy, przynajmniej patrząc z
perspektywy samego bloga. W przypadku GitHub Issues już takiej możliwości nie będzie. By być w
stanie skomentować jakiś post, trzeba posiadać konto w [serwisie GitHub][5]. Naturalnie, by
przeglądać komentarze nie trzeba się nigdzie logować czy zakładać konta.

### Limitowanie zapytań

Drugi problem z GitHub Issues to limit zapytań, które użytkownicy mogą wysłać do API GitHub'a bez
autoryzacji. Ta wartość to [60 zapytań na godzinę na konkretny adres IP][9]. Nie jest to jakoś
specjalnie dużo, zwłaszcza w przypadku adresów IPv4 i mechanizmu NAT. Niemniej jednak, przy dość
intensywnym przeglądaniu bloga, przydział 60 zapytań może się nam dość szybko wyczerpać i wtedy
przez godzinę nie będziemy widzieć żadnych komentarzy, przynajmniej bezpośrednio na blogu.

Każdy użytkownik może sobie podejrzeć aktualny stan limitów przy pomocy `curl` , choć trzeba
pamiętać, że każde takie zapytanie zmniejsza ten limit o 1. By podejrzeć ile nam jeszcze zostało
zapytań, trzeba w terminalu wpisać to poniższe polecenie:

    $ curl --silent -I https://api.github.com/users/octocat
    HTTP/2 200
    server: GitHub.com
    date: Mon, 11 Oct 2021 17:29:15 GMT
    content-type: application/json; charset=utf-8
    cache-control: public, max-age=60, s-maxage=60
    vary: Accept, Accept-Encoding, Accept, X-Requested-With
    etag: W/"1b19b9d45e66326cb555aba937fe333d28ac8927960cfd325afafb7ce517e29e"
    last-modified: Wed, 22 Sep 2021 14:27:38 GMT
    x-github-media-type: github.v3; format=json
    access-control-expose-headers: ETag, Link, Location, Retry-After, X-GitHub-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Used, X-RateLimit-Resource, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes, X-Poll-Interval, X-GitHub-Media-Type, Deprecation, Sunset
    access-control-allow-origin: *
    strict-transport-security: max-age=31536000; includeSubdomains; preload
    x-frame-options: deny
    x-content-type-options: nosniff
    x-xss-protection: 0
    referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
    content-security-policy: default-src 'none'
    x-ratelimit-limit: 60
    x-ratelimit-remaining: 53
    x-ratelimit-reset: 1633973759
    x-ratelimit-resource: core
    x-ratelimit-used: 7
    accept-ranges: bytes
    content-length: 1319
    x-github-request-id: 07C8:D794:A9BF8:C1C09:61647473

Mamy tam szereg nagłówków `x-ratelimit-*` , które informują nas o wielkości przydziału (60), o
licznie wykonanych zapytań (7) i o liczbie pozostałych zapytań (53), które będziemy mogli wykonać
do czasu określonego w `x-ratelimit-reset` . Widnieje tam co prawda niezbyt czytelna dla człowieka
wartość ale można ją przeliczyć na zrozumiały dla każdego czas:

    $ date -d @1633973759
    2021-10-11T19:35:59 CEST

Z chwilą, gdy zegar wybije tę konkretną godzinę, przydział zapytań zostanie zresetowany i znowu
przez godzinę będziemy w stanie wykonać ich 60.

## Jak wdrożyć komentarze GitHub Issues na blogu

Do wdrożenia komentarzy GitHub Issues na blogu Hugo hostowanym za sprawą mechanizmu GitHub Pages
będzie nam potrzebny [kawałek skryptu JavaScript][2]. Trzeba będzie również przerobić nieco [skórkę
Hugo][6] ale nie będzie to jakiś bardzo inwazyjny proces. Poniżej znajduje się skrypt, który trzeba
zapis w `blog/layouts/partials/comments.html` :

    <script>
      var id = {{ .Params.GHissueID }};

      if (id)
      {
        let url = "https://github.com/morfikov/morfitronik-comments/issues/".concat(id);
        let api_url = "https://api.github.com/repos/morfikov/morfitronik-comments/issues/".concat(id, "/comments");

        var commentsDiv = document.getElementById("comments");

        let xhr = new XMLHttpRequest();
        xhr.responseType = "json";
        xhr.open("GET", api_url);
        xhr.setRequestHeader("Accept", "application/vnd.github.v3.html+json");
        xhr.send();

        xhr.onload = function()
        {
          if (xhr.status != 200)
          {
            let errorText = document.createElement("p");
            errorText.innerHTML = "<i>Komentarze dla tego postu nie zostały jeszcze otworzone (albo skrypty GitHub'a są wyłączone). Być może też skończył ci się przydział 60/godzinę (limit GitHub'a).</i>";
            commentsDiv.appendChild(errorText);
          }
          else
          {
            let comments = xhr.response;

            let mainHeader = document.createElement("h3");
            mainHeader.innerHTML = "Komentarze: ".concat(comments.length);
            commentsDiv.appendChild(mainHeader);

            let issueLink = document.createElement("p");
            issueLink.innerHTML = "<i>Możesz zostawić komentarz korzystając z <a href='".concat(url, "'>GitHub issue</a>.</i>");
            commentsDiv.appendChild(issueLink);

            comments.forEach(function(comment)
            {
                const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', timeZone: 'Europe/Warsaw' };
                options.timeZoneName = 'short';
                let commentContent = document.createElement("div");
                commentContent.setAttribute('class', 'gh-comment')
                commentContent.innerHTML = "".concat(
                    "<div class='gh-header'>",
                      "<img src='", comment.user.avatar_url, "' />",
                      "<div style='margin:auto 0;'>",
                        "<b><a class='gh-username' href='", comment.user.html_url, "'>", comment.user.login, "</a></b>",
                        " | <em>", (new Date(comment.created_at)).toLocaleTimeString('pl-PL', options), "</em>",
                      "</div>",
                    "</div>",
                    "<div class='gh-body'>",
                      comment.body_html,
                    "</div>"
                );
                commentsDiv.appendChild(commentContent);
            });
          }
        };

        xhr.onerror = function()
        {
          let errorText = document.createElement("p");
          errorText.innerHTML = "<i>Wystąpił jakiś błąd przy ładowaniu komentarzy.</i>";
          commentsDiv.appendChild(errorText);
        };
      }
    </script>

Trzeba w tym skrypcie dostosować parametry `url` oraz `api_url` , tak by wskazywały na konkretne
repozytorium w obrębie infrastruktury GitHub'a. Ja dodatkowo zdecydowałem się utworzyć całkiem
osobne repozytorium dla komentarzy i trzymać je niezależnie od repozytorium mojego bloga.

### Przerobienie skórki bloga

Druga sprawa, to przerobienie samej skórki Hugo. Nie będziemy tego robić w katalogu
`blog/themes/binario/` , a jedynie skopiujemy sobie z niego plik `layouts/_default/single.html` i
umieścimy go pod `blog/layouts/_default/single.html` . W tym pliku musimy dodać/zmienić wpisy
odpowiedzialne za renderowanie komentarzy, tj. trzeba dodać ten poniższy kod:

    <div id="comments">
        {{ partial "comments.html" $ }}
    </div>

Dla większej czytelności, [tutaj jest stosowny commit][7], który te dwie powyższe czynności wdraża
na moim blogu.

### Stylizowanie komentarzy

By komentarze wyglądały jakoś bardziej przyzwoicie, trzeba je trochę wystylizować przy pomocy
stylów CSS. Wrzucamy zatem do pliku  `blog/content/css/custom.css` poniższą zawartość:

    /* Comments */
    #comments {
        margin: .625rem .3125rem;
        padding: .875rem;
        background-color: #2a2a2a;
    }

    .gh-comment {
      border: 1px solid #515151;
      margin-top: 15px;
    }

    .gh-header {
      display: flex;
      color: #586069;
      background-color: #111;
      border-bottom: 1px solid #515151;
      padding: .875rem;
      font-size: .850rem;
    }

    .gh-username {
      color: #f8ae00;
    }

    .gh-body {
        padding: .875rem;
    }

    .gh-header > img {
      width: 24px;
      height: 24px;
      margin-right: 0.500rem;
      vertical-align: middle;
    }

## Tworzenie wątków pod konkretne posty w GitHub Issues

W tym momencie mamy co prawda zaimplementowane już komentarze na blogu Hugo ale każdy z postów,
który ma taki system komentarzy wyświetlić, musi mieć przypisany konkretny numerek ID z wątkiem
otworzonym w GitHub Issues. Gdy rozpoczynamy przygodę z blogowaniem, to bez większego problemu
będziemy w stanie przejść sobie na stronę GitHub'a do naszego repozytorium i wcisnąć przycisk
stworzenia nowego wątku. Taki zabieg wygeneruje jakiś numerek ID, zwykle `#1` . Ten numerek trzeba
będzie w poście bloga uwzględnić w poniższy sposób (we [front matter][8]):

    ---
    ...
    GHissueID: 1
    ...
    ---

Niemniej jednak, mając blisko 600 wpisów, ręczne tworzenie wątku dla każdego z nich na GitHub'ie, a
potem jeszcze uzupełnianie w plikach tekstowych odpowiednich numerów, przekracza moje zdolności
usiedzenia w jednym miejscu bez opcji wyrzucenia komputera przez okno. Dlatego też musiałem to
zadanie rozwiązać w nieco inny sposób.

### Automatyzacja tworzenia wątków

Najpierw zautomatyzowałem tworzenie wątków w GitHub Issues przy pomocy poniższego skryptu
shell'owego:

    #!/bin/sh

    titles=$(egrep -hr ^title\: /media/debuilder/hugo/blog/content/post | sed 's/title\:\ //g' | sed "s/'//g" | sed "s/\"//g")

    echo "${titles}" | while IFS= read -r title
    do
        curl \
        --silent  \
        -X POST https://api.github.com/repos/morfikov/morfitronik-comments/issues \
        -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:58.0) Gecko/20100101 Firefox/58.0" \
        -H "Content-Type: text/plain; charset=utf-8" \
        -H "Authorization: token  tutaj_numerek" \
        -H "Accept: application/vnd.github.v3+json" \
        -d '{"title":"'"$title"'"}' >> output.txt 2>&1

        echo ${title}

        sleep 5
    done

    exit 0

Ten skrypt ma na celu stworzenie listy wpisów, które są przechowywane w katalogu
`/media/debuilder/hugo/blog/content/post/` . Każdy taki plik ma we front matter linijkę z `title:` .
Po jej wydobyciu za sprawą `egrep` uzyskamy ścieżkę do pliku, którą trzeba będzie potem podać w
`sed` . Przy pomocy `sed` trzeba z tytułów postów pousuwać wszystkie problematyczne znaki, takie
jak np. `"` czy `'` . Jeśli nie pozbędziemy się takich znaków, to podczas wykonywania zapytań do
API GitHub'a pojawią się błędy. Później w pętli `while` podajemy listę tytułów, w oparciu o które
`curl` utworzy konkretne wątki w repozytorium GitHub'a. Wszystko jest logowane do pliku
`output.txt` , tak by na zakończenie całego procesu sprawdzić czy nie było jakichś błędów przy
wykonywaniu zapytań.

Za samo tworzenie wątków w GitHub Issues odpowiada `-d '{"title":"'"$title"'"}'` , który w polu
tytułu wątku wstawi tytuł posta. Można naturalnie [dodać więcej pól][11] ale w zasadzie jedynie
tytuł jest wymagany.

#### Token uwierzytelniający zapytania do API

Problem z automatycznym tworzeniem wątków jest taki, że bardzo szybko można uderzyć w przydzielony
nam limit 60 zapytań na godzinę. W takim tempie tworzenie tych wszystkich potrzebnych wątków
zajęłoby dosłownie pół dnia. Dlatego też trzeba było ten proces ździebko przyśpieszyć za sprawą
uwierzytelnienia zapytań do API GitHub'a. By te zapytania uwierzytelnić, potrzebny nam jest
odpowiedni token. Token uwierzytelniający możemy wygenerować sobie w ustawieniach konta GitHub'a
(avatar w prawym górnym rogu > Settings):

![hugo-github-issues-disqus-comments-token](/img/2021/10/001.hugo-github-issues-disqus-comments-token.png#huge)

Z menu po lewej stronie wybieramy `Developer settings` :

![hugo-github-issues-disqus-comments-token](/img/2021/10/002.hugo-github-issues-disqus-comments-token.png#huge)

Tutaj najbardziej nas interesuje `Personal access tokens` . Tworzymy nowy token klikając w przycisk
`Generate new token` :

![hugo-github-issues-disqus-comments-token](/img/2021/10/003.hugo-github-issues-disqus-comments-token.png#huge)

Dodajemy jakąś notkę i ustawiamy czas ważności. Określamy uprawnienia w `Select scopes` i
zaznaczamy tutaj jedynie `Access public repositories` . Po wygenerowaniu tokenu powinniśmy uzyskać
jego numerek:

![hugo-github-issues-disqus-comments-token](/img/2021/10/004.hugo-github-issues-disqus-comments-token.png#huge)

Tak uzyskany token trzeba podać `curl` w nagłówku `-H "Authorization: token  tutaj_numerek"` . W
ten sposób będziemy mieli do dyspozycji nie 60, a 6000 zapytań na godzinę. Wciąż jednak należy
pamiętać, by tych zapytań nie wykonywać za często, bo uderzymy w [inny limit][10], co z kolei może
nam bardzo namieszać przy tworzeniu wątków. Dlatego też tworzenie każdego kolejnego wątku odbywa
się co 5 sekund.

### Uzupełnianie postów numerkami ID

Mając utworzone już stosowne wątki w repozytorium GitHub'a, trzeba numery tych wątków wpisać do
postów na blogu. Każdy wątek na GitHub'ie ma określony tytuł i numerek. Trzeba zatem w oparciu o
tytuł uzupełnić numerek w tekstowych plikach postów bloga. Do tego celu również potrzebny nam
będzie skrypt shell'owy:

    #!/bin/sh

    list=$(egrep -r ^title\: /media/debuilder/hugo/blog/content/post | cut -d: -f1)

    issue_number="5"

    echo "${list}" | while IFS= read -r title
    do
        sed -i "s@title:@GHissueID:\ $issue_number\ntitle:@g" ${title}
        issue_number=$((issue_number+1))
    done

    exit 0

Podobnie jak w przypadku tworzenia wątków w repozytorium GitHub'a, tutaj też musimy pozyskać listę
plików z postami, którym zamierzamy dopisać numerki. Wpisy na tej liście muszą być w dokładnie tej
samej kolejności, w której tworzyliśmy wątki w GitHub Issues. Przy pomocy `cut` wydobędziemy sobie
ścieżkę do pliku o określonym tytule. Tę ścieżkę trzeba będzie poddać w `sed` , by dodał on frazę
`GHissueID:` wraz z odpowiednim numerkiem.

Zmienna `$issue_number` wskazuje na numerek początkowy, tj. ten od którego trzeba zacząć liczyć. Z
racji, że trochę testów musiałem przeprowadzić, to parę wątków trzeba było utworzyć, po czym
zostały one skasowane. Niemniej jednak, numerki są ciągle zwiększane. Zatem pierwszy wątek
utworzony w GitHub Issues miał u mnie numerek 5. Mając odpowiednią listę plików, dodajemy do nich
numerki zwiększając o jeden z każdym przejściem pętli `while` . W ten sposób numerki w plikach
postów będą pasować do tych, które są w wątkach w GitHub Issues.

## Test instalacji systemu komentarzy

Jeśli wszystkie powyższe kroki przeprowadziliśmy prawidłowo, to system komentarzy oparty o GitHub
Issues powinien nam działać jak należy. Możemy ten stan rzeczy zweryfikować tworząc w pierwszym
lepszym wątku jakieś komentarze. Poniżej przykład:

![hugo-github-issues-disqus-comments-test](/img/2021/10/005.hugo-github-issues-disqus-comments-test.png#huge)

Teraz przechodzimy na bloga i szukamy tego konkretnego wpisu. Pod nim powinniśmy mieć blok
komentarzy mający dokładnie tę samą zawartość co w GitHub Issue:

![hugo-github-issues-disqus-comments-test](/img/2021/10/006.hugo-github-issues-disqus-comments-test.png#huge)

## Dodanie linku do posta na blogu w wątku na GitHub Issues

Wyżej zostało wspomniane, że do utworzenia nowego wątku w GitHub Issues wymagany jest w zasadzie
jedynie sam tytuł wątku. Można jednak pokusić się jeszcze o dodatnie linku do postu bloga wewnątrz
tak stworzonego wątku w GitHub Issues. W ten sposób ktoś, kto znajdzie wątek komentarzy, będzie
wiedział gdzie znajduje się komentowany wpis.

Jako, że ja wcześniej już utworzyłem wszystkie wątki, to muszę nieco inaczej sobie z tym zdaniem
dodania linków poradzić. Potrzebny będzie kolejny skrypt:

    #!/bin/sh

    posts=$(egrep -r ^GHissueID /media/debuilder/hugo/blog/content/post)

    echo "${posts}" | while IFS= read -r post
    do

    id=$(echo $post | cut -d" " -f 2)
    url=$(echo $post | cut -d":" -f 1 | cut -d"/" -f 10 | sed 's@.md@@g')

    body="Komentarze dla postu: https://morfikov.github.io/post/$url/"

     curl \
        --silent  \
        -X PATCH https://api.github.com/repos/morfikov/morfitronik-comments/issues/$id \
        -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:58.0) Gecko/20100101 Firefox/58.0" \
        -H "Content-Type: text/plain; charset=utf-8" \
        -H "Authorization: token  tutaj_numerek" \
        -H "Accept: application/vnd.github.v3+json" \
        -d '{"body":"'"$body"'"}' >> output.txt 2>&1

    echo "$body : $id"

    sleep 5

    done

U mnie na blogu po frazie `https://morfikov.github.io/post/` jest podawana nazwa pliku
sformatowanego językiem Markdown. Ten powyższy skrypt tworzy listę postów w oparciu o `GHissueID`
(nadany wcześniej) . Później w zmiennej `$id` znajdzie się numerek wątku w GitHub Issues, a w
zmiennej `$url` nazwa pliku bez końcówki `.md` . Za sprawą `-d '{"body":"'"$body"'"}'` oraz
wcześniej skonstruowanej zmiennej `$body` odpowiednio uzupełnimy sobie treść pierwszego komentarza
(tego już utworzonego, który był pusty). Cokolwiek było w tym pierwszy komentarzu, zostanie
przepisane frazą, którą tutaj podamy.

## Podsumowanie

W taki oto prosty sposób przy pomocy mechanizmu GitHub Issues udało się nam zaimplementować system
komentarzy na naszym blogu Hugo. Jednocześnie możemy się pozbyć Disqus'a i jego wrednego podejścia
w kwestii serwowania ludziom reklam. Ilekroć ktoś tylko doda jakiś post w stosownym wątku w GitHub
Issues, to pojawi się on także pod wpisem u nas na blogu. Ten nowy system komentarzy jest też
sprzęgnięty ze wszystkimi udogodnieniami, które zapewnia GitHub. W ten sposób użytkownicy
komentujący wpisy na blogu mogą być powiadamiani o nowych odpowiedziach via mail/rss, jeśli
naturalnie w opcjach wątku zaznaczyli sobie stosowne opcje.


[1]: /post/disqus-wprowadza-reklamy-disqus-musi-odejsc/
[2]: https://decovar.dev/blog/2019/04/19/github-comments-hugo/
[3]: https://github.com/morfikov/morfikov.github.io/tree/gh-pages
[4]: https://pages.github.com/
[5]: https://github.com/
[6]: https://github.com/Vimux/Binario
[7]: https://github.com/morfikov/morfikov.github.io/commit/9ac43466b3a4c3bfac639e09f1bbd97cbc854e62
[8]: https://gohugo.io/content-management/front-matter/
[9]: https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting
[10]: https://docs.github.com/en/rest/overview/resources-in-the-rest-api#secondary-rate-limits
[11]: https://docs.github.com/en/rest/reference/issues
