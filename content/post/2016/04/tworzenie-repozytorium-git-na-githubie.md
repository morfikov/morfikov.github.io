---
author: Morfik
categories:
- Linux
date: "2016-04-19T17:08:26Z"
date_gmt: 2016-04-19 15:08:26 +0200
published: true
status: publish
tags:
- git
title: Tworzenie repozytorium git na GitHub
---

Po eksperymentah z repozytorium git na github'ie postanowiłem zrobić drobną ściągawkę na temat
obsługi tego mechanizmu z poziomu linii poleceń. Oczywiście, tutaj będą przedstawione jedynie
podstawy, tj. jak zacząć tworzyć takie repozytorium. Bardziej zaawansowane rzeczy, np.
[implementacja kluczy SSH na github'ie]({{< baseurl >}}/post/github-z-obsluga-kluczy-ssh/), czy
też [podpisywanie swojej pracy przy pomocy kluczy
GPG]({{< baseurl >}}/post/implementacja-kluczy-gpg-repozytorium-git/) zostały opisane w osobnych
wpisach, bo lekko wykraczają poza tematykę tego artykułu. Oczywiście te dwie rzeczy nie są wymagane
do pracy z git'em ale trzeba rozważyć ich wdrożenie jeśli faktycznie zamierzamy zacząć korzystać z
tego VCS'a. W każdym razie, tutaj zajmę się podstawowymi operacjami, które człowiek zwykle
przeprowadza, gdy przychodzi mu korzystać z git'a. Zdaję sobie sprawę, że są różne nakładki
graficzne na narzędzie `git` ale, jak się okaże po przeczytaniu tego wpisu, operowanie na tekstowym
`git` nie jest takie straszne, choć sama dokumentacja może przerazić, bo jest tego dość sporo i
raczej wszystkiego od razu nie ogarniemy.

<!--more-->
## Instalacja i wstępna konfiguracja git'a

Na sam początek instalujemy potrzebne narzędzia. W repozytoriach debiana mamy pakiet `git` i to w
nim znajdziemy wszystko co jest nam niezbędne do pracy, by tworzyć repozytoria i dokonywać w nich
zmian. Proces instalacji powinien być raczej bezproblemowy, zatem przejdźmy od razu do konfiguracji
git'a.

Konfiguracji można dokonywać na dwa sposoby. Pierwszy z nich zakłada wykorzystanie do tego celu
polecenia `git config` . Natomiast druga opcja polega na ręcznym uzupełnianiu plików
konfiguracyjnych. Pliki są czytane z określonych lokalizacji i w określonej kolejności. Pierwszym z
nich jest plik systemowy (dodanie `--system` do `git config`) i ten jest zlokalizowany w
`/etc/gitconfig` . Zwykle nie jest on obecny w systemie. Drugi w kolejności jest plik użytkownika
(dodane `--global` do `git config`) i jest to plik `~/.gitconfig` (lub `~/.config/git/config` ,
jeśli istnieje). To zwykle na nim będziemy operować dodając nowe opcje lub zmieniając stare.
Ostatnie miejsce gdzie `git` zagląda w poszukiwaniu konfiguracji, to lokalizacja repozytorium. Po
wydaniu `git clone` lub `git init` , plik konfiguracyjny będzie dostępny w `repo/.git/config` . To
jest lokalna konfiguracja, z której będzie korzystało określone repozytorium i jeśli mamy jakieś
niestandardowe opcje, które chcemy zaaplikować tylko do tego danego repozytorium, to właśnie w tym
pliku musimy je umieścić. Można także określić swój własny plik przy pomocy opcji `-f` dopisanej do
`git config` . Nam jednak wystarczy plik użytkownika. Parametry w tych plikach mogą się dublować. W
takim przypadku konfiguracja repozytorium ma pierwszeństwo nad konfiguracją lokalną użytkownika, a
ta z kolei ma pierwszeństwo nad konfiguracją systemową. Dlatego też, by wstępnie skonfigurować sobie
git'a, trzeba dopisać do `git config` opcję `--global` . [Tutaj jest obszerny
opis](https://www.kernel.org/pub/software/scm/git/docs/git-config.html) poszczególnych opcji, które
mogą zostać wykorzystane w procesie konfiguracji. Dodatkowe informacje można znaleźć również
[tutaj](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration).

Na początek skonfigurujmy sobie kilka podstawowych parametrów:

    $ git config --global user.name "Mikhail Morfikov"
    $ git config --global user.email morfik@nsa.com
    $ git config --global gpg.program gpg
    $ git config --global user.signingkey 0x72F3A416B820057A
    $ git config --global push.default simple
    $ git config --global color.ui always
    $ git config --global core.whitespace blank-at-eol,blank-at-eof,space-before-tab,tabwidth=4
    $ git config --global merge.tool meld
    $ git config --global diff.tool meld
    $ git config --global core.editor vim
    $ git config --global core.pager less
    $ git config --global core.excludesfile ~/.config/git/.gitignore_global
    $ git config --global core.ignorecase false
    $ git config --global core.eol lf
    $ git config --global core.autocrlf input
    $ git config --global web.browser firefox
    $ git config --global browser.firefox.path /opt/firefox/firefox
    $ git config --global browser.firefox.cmd 'firefox -P default'
    $ git config --global log.date iso8601
    $ git config --global log.decorate full

Większość opcji jest raczej zrozumiała. Jeśli nie korzystamy z kluczy GPG, to omińmy parametry
`gpg.program` oraz `user.signingkey` . Te powyższe polecenia dodadzą stosowne wpisy do pliku
`~/.config/git/config` .

## Tworzenie repozytorium

Mamy z głowy zatem wstępną konfigurację git'a. Przydałoby się teraz poćwiczyć umiejętność operowania
na repozytorium. Pierw musimy jakieś sobie stworzyć. Na lokalnym dysku wybieramy pusty katalog i
przechodzimy do niego. Następnie za pomocą `git init` tworzymy puste repozytorium git.

    $ mkdir repozytoria/practice-makes-perfect/
    $ cd repozytoria/practice-makes-perfect/

    $ git init

Po wydaniu ostatniego polecenia, w katalogu roboczym zostanie utworzony katalog `.git/` . To w nim
znajdują się wszystkie informacje na temat przeprowadzanych zmian w repozytorium. Struktura tego
katalogu wygląda mniej więcej tak:

    $ tree .git/
    .git/
    ├── HEAD
    ├── branches
    ├── config
    ├── description
    ├── hooks
    │   ├── applypatch-msg.sample
    │   ├── commit-msg.sample
    │   ├── post-update.sample
    │   ├── pre-applypatch.sample
    │   ├── pre-commit.sample
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample
    │   ├── prepare-commit-msg.sample
    │   └── update.sample
    ├── info
    │   └── exclude
    ├── objects
    │   ├── info
    │   └── pack
    └── refs
    ├── heads
    └── tags

    9 directories, 13 files

Jak widać, mamy tam kilka przykładowych skryptów (pliki `.sample` ) oraz to co nas bardziej
interesuje, czyli pliki `HEAD` , `description` i `config` . W pliku `description` trzymany jest opis
repozytorium. Dobrze jest edytować ten plik i dodać krótki opis. Plik `config` już po części
omówiliśmy. By połapać się jak wygląda konfiguracja dla jakiegoś określonego repozytorium,
przechodzimy do jego głównego katalogu i wydajemy poniższe polecenie:

    $ git config -l

W pliku `HEAD` są przechowywane informacje na temat gałęzi, w której aktualnie się znajdujemy.
Więcej o tym pliku będzie w dalszej części artykułu. Plik `HEAD` standardowo wygląda tak:

    $ cat .git/HEAD
    ref: refs/heads/master

## Plik licencji i README

Pliki w repozytorium [powinny być objęte
licencją](https://help.github.com/articles/open-source-licensing/). Jeśli nie zdefiniujemy
licencji, wtedy wszystkie udostępniane pliki będą na domyślnej licencji ustalonej prawnie, czyli
brak kopiowania/zmieniania plików. W debianie pliki z licencjami są zlokalizowane w katalogu
`/usr/share/common-licenses/` . Mamy tam do wyboru min. różne wersje GPL, Apache i BSD. Pliki w tym
repo będą na licencji GPL-2. Kopiujemy zatem odpowiedni plik z folderu powyżej do głównego katalogu
repozytorium:

    $ cp /usr/share/common-licenses/GPL-2 ./LICENSE

Przydałoby się także stworzyć plik `README.md` , który to będzie wyświetlany na głównej stronie
repozytorium na github'ie.

    $ touch README.md
    $ echo "This is a testing repository for learning purposes" > README.md

## Wprowadzanie zmian w repozytorium

Repozytorium mamy przygotowane. Musimy jeszcze uwzględnić te zmiany, tak by `git` je zapamiętał.
Najpierw sprawdzamy co tak naprawdę `git` zarejestrował:

    $ git status
    On branch master

    Initial commit

    Untracked files:
    (use "git add <file>..." to include in what will be committed)

    LICENSE
    README.md

    nothing added to commit but untracked files present (use "git add" to track)

Oba pliki zostały wykryte ale jeszcze ich stan nie podlega monitorowaniu. Dodajmy zatem wszystkie
utworzone przez nas pliki:

    $ git add \*

    $ git status
    On branch master

    Initial commit

    Changes to be committed:
    (use "git rm --cached <file>..." to unstage)

    new file:   LICENSE
    new file:   README.md

Teraz już tylko wystarczy zaaplikować zmiany:

    $ git commit -a -S -m 'Repository setup'

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov <morfik@nsa.com>"
    4096-bit RSA key, ID 0x72F3A416B820057A, created 2013-11-13

    [master (root-commit) 4de0d7f] Repository setup
    2 files changed, 340 insertions(+)
    create mode 100644 LICENSE
    create mode 100644 README.md

Wyżej zostało wypisane to jakie pliki zostały zmienione wraz z usuniętymi i dodanymi linijkami. W
tym przypadku tworzyliśmy pliki, zatem zostały dodane nowe linijki (+).

Jeśli nie korzystamy z kluczy GPG, to nie podajemy opcji `-S` . Z kolei parametr `-a` odpowiada za
uwzględnienie wszystkich plików, które zostały zmodyfikowane lub usunięte, za wyjątkiem tych nowych,
których póki co nie chcemy włączać do repozytorium. Opcja `-m` odpowiada za treść wiadomości, która
zostanie uwzględniona w logu.

Zmienił się także nieco wygląd drzewa katalogów repozytorium:

    $ tree -a
    .
    ├── .git
    │   ├── COMMIT_EDITMSG
    │   ├── HEAD
    │   ├── branches
    │   ├── config
    │   ├── description
    │   ├── hooks
    │   │   ├── applypatch-msg.sample
    │   │   ├── commit-msg.sample
    │   │   ├── post-update.sample
    │   │   ├── pre-applypatch.sample
    │   │   ├── pre-commit.sample
    │   │   ├── pre-push.sample
    │   │   ├── pre-rebase.sample
    │   │   ├── prepare-commit-msg.sample
    │   │   └── update.sample
    │   ├── index
    │   ├── info
    │   │   └── exclude
    │   ├── logs
    │   │   ├── HEAD
    │   │   └── refs
    │   │       └── heads
    │   │           └── master
    │   ├── objects
    │   │   ├── 22
    │   │   │   └── fb3a8ab3ec2e800d1e22255204ec8d735d3d43
    │   │   ├── 4d
    │   │   │   └── e0d7f07def79578ed908669c9bc0da4383a3b7
    │   │   ├── d1
    │   │   │   └── 59169d1050894d3ea3b98e1c965c4058208fe1
    │   │   ├── e4
    │   │   │   └── 67acb33a14fd0f651500057a6364403c1ebd38
    │   │   ├── info
    │   │   └── pack
    │   └── refs
    │       ├── heads
    │       │   └── master
    │       └── tags
    ├── LICENSE
    └── README.md

    17 directories, 24 files

Zostało dodanych kilka obiektów, oraz został utworzony log.

## Log

Sam log można przeglądać przy pomocy polecenia `git log` , przykładowo:

![]({{< baseurl >}}/img/2016/04/1.git-log.png#huge)

Log dokumentuje wszystkie operacje, które zostały przeprowadzone na repozytorium. W przypadku, gdy
wprowadzamy wiele zmian, powyższe polecenie może nam zwrócić sporo informacji. Oczywiście, możemy
podać [całe mnóstwo opcji](https://www.kernel.org/pub/software/scm/git/docs/git-log.html), które
odpowiednio sformatują wyjście logów. Te częściej używane są przystępnie opisane
[tutaj](https://git-scm.com/book/it/v2/Git-Basics-Viewing-the-Commit-History). Poniżej znajduje się
kilka przykładów.

Uwzględnianie zmian dokonanych w poszczególnych w commit'ach ( `git log -p` ):

![]({{< baseurl >}}/img/2016/04/2.git-log-diff.png#huge)

Zwracanie kilka ostatnich pozycji w logu ( `git log -2` ):

![]({{< baseurl >}}/img/2016/04/3.git-log-ostatnie-zmiany.png#huge)

Zwracanie wybranego commit'a ( `git log 4de0d7f` ). Poniżej został zwrócony commit w oparciu o kilka
pierwszych znaków hasha. Przydatne jeśli chcemy wyświetlić informacje, a dysponujemy jedynie tym
ciągiem.

![]({{< baseurl >}}/img/2016/04/4.git-log-commit.png#huge)

Zwracanie commit'ów z określonego przedziału czasowego ( `git log --since=14.months --until=now` ) .
Można także używać dat w postaci `git log --until="2015-02-18 12:22:00"` :

![]({{< baseurl >}}/img/2016/04/5.git-log-time.png#huge)

Wyświetlanie commit'ów określonego autora `git log --author=morfik` :

![]({{< baseurl >}}/img/2016/04/6.git-log-autor.png#huge)

Co ciekawe, `morfik` zwraca szereg wpisów, mimo, że nie ma tam takiego autora. W manie z kolei można
wyczytać, że w `--author=` podaje się jedynie wzór wyszukiwania autora, a nie jego dokładną nazwę.
Zatem wszystkie commit'y zawierające w polu `Author` frazę `morfik` zostaną zwrócone.

Statystyki commit'ów ( `git log --stat` ):

![]({{< baseurl >}}/img/2016/04/7.git-log-statystyki.png#huge)

Dostosowywanie wyglądu loga ( `git log --graph --decorate --oneline` ) :

![]({{< baseurl >}}/img/2016/04/8.git-log-pretty-format.png#huge)

Powyższe opcje możemy łączyć. Dostaniemy w ten sposób bardziej przefiltrowany log.

## Zdalna kopia repozytorium na github'ie

Pora przesłać kopię repozytorium na github'a. Pierw jednak musimy odwiedzić [stronę
github'a](https://github.com) i utworzyć tam repozytorium. Nie da rady inaczej stworzyć repo na
github'ie, zwłaszcza jeśli ma się włączone uwierzytelnianie dwuskładnikowe.

![]({{< baseurl >}}/img/2016/04/10.tworzenie-repozytorium-git-github.png#huge)

Jako, że wcześniej stworzyliśmy plik `README.md` i `LICENCE` , to nie musimy ich tutaj definiować,
bo zostaną przesłane przy synchronizacji. Po pomyślnym utworzeniu repozytorium, powinno zostać
wyświetlone podsumowanie:

![]({{< baseurl >}}/img/2016/04/11.tworzenie-repozytorium-git-github.png#huge)

Jak widać, mamy tam między innymi linki potrzebne do synchronizacji. Jest tam zarówno https jak i
ssh. Dodatkowo, jest też instrukcja jak stworzyć lokalne repozytorium (to już zrobiliśmy) i jak
dokonać synchronizacji.

Wracamy zatem do terminala i [dodajemy zdalne
ścieżki](https://help.github.com/articles/adding-a-remote/):

    $ git remote add origin git@github.com:morfikov/practice-makes-perfect.git

    $ git remote -v
    origin  git@github.com:morfikov/practice-makes-perfect.git (fetch)
    origin  git@github.com:morfikov/practice-makes-perfect.git (push)

Jeśli w tej chwili spróbowalibyśmy dać `push` , dostaniemy poniższy komunikat:

    $ git push
    fatal: The current branch master has no upstream branch.
    To push the current branch and set the remote as upstream, use

    git push --set-upstream origin master

Zgodnie z instrukcją wydajemy poniższe polecenie:

    $ git push --set-upstream origin master
    Counting objects: 4, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (4/4), 7.57 KiB | 0 bytes/s, done.
    Total 4 (delta 0), reused 0 (delta 0)
    To git@github.com:morfikov/practice-makes-perfect.git
    * [new branch]      master -> master
    Branch master set up to track remote branch master from origin.

W tej chwili odwiedzając repozytorium na github'ie powinniśmy ujrzeć już nasze pliki:

![]({{< baseurl >}}/img/2016/04/12.tworzenie-repozytorium-git-github.png#huge)

## Cofanie zmian w repozytorium git

Musimy także wiedzieć jak cofać/odrzucać zmiany wprowadzane do repozytorium. Mamy z grubsza dwie
kategorie zmian. Pierwsza z nich to zmiany przed dokonaniem commit'a, druga zaś po dokonaniu.

Stwórzmy zatem nowy plik w repozytorium:

    $ touch test_file
    $ echo "some random text" > test_file

    $ git add .

    $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

    new file:   test_file

Nie możemy teraz sobie od tak tego pliku usunąć, tj. przy pomocy `rm` lub wciskając klawisz Delete
na klawiaturze. Będzie to traktowane jako kolejna zmiana:

    $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

    new file:   test_file

    Changes not staged for commit:
    (use "git add/rm <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

    deleted:    test_file

Zgodnie z wypisanymi instrukcjami, by cofnąć zmiany, wpisujemy:

    $ git reset HEAD test_file

Jeśli teraz sprawdzimy status git'a, zobaczymy, że plik aktualnie nie jest śledzony:

    $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Untracked files:
    (use "git add <file>..." to include in what will be committed)

    test_file

    nothing added to commit but untracked files present (use "git add" to track)

Teraz już można spokojnie usunąć plik z repozytorium git'a:

    $ rm test_file

    $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    nothing to commit, working directory clean

A jak sprawa wygląda w przypadku, gdy dokonamy commit'a? Stwórzmy zatem jeszcze jeden plik testowy:

    $ touch test_file
    $ echo "some random text" > test_file

    $ git add .

    $ git commit -S -a -m "mistake"

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov <morfik@nsa.com>"
    4096-bit RSA key, ID 0x72F3A416B820057A, created 2013-11-13

    [master e175de8] mistake
    1 file changed, 1 insertion(+)
    create mode 100644 test_file

Możemy pójść nawet o krok dalej i przesłać zmiany na github'a:

    $ git push
    Counting objects: 3, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (3/3), 964 bytes | 0 bytes/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    To git@github.com:morfikov/practice-makes-perfect.git
    4de0d7f..e175de8  master -> master

Sprawdzając log możemy dostrzec nowy commit:

    $ git log
    commit e175de8b05df16c5f2b7d2c2c23a932a1c9efeaa (HEAD, refs/remotes/origin/master, refs/heads/master)
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   2015-02-17 16:59:49 +0100

    mistake

    commit 4de0d7f07def79578ed908669c9bc0da4383a3b7
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   2015-02-17 12:21:27 +0100

    Repository setup

Spróbujmy się tego jakoś pozbyć. Najpierw musimy ustalić gdzie jest HEAD. Zwykle będzie to ostatnie
commit ale zakładając, że nie wiemy gdzie dokładnie jest HEAD, to musimy to ustalić:

    $ cat .git/HEAD
    ref: refs/heads/master

    $ cat .git/refs/heads/master
    e175de8b05df16c5f2b7d2c2c23a932a1c9efeaa

    $ git show e175de8b05df16c5f2b7d2c2c23a932a1c9efeaa
    commit e175de8b05df16c5f2b7d2c2c23a932a1c9efeaa (HEAD, refs/remotes/origin/master, refs/heads/master)
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   2015-02-17 16:59:49 +0100

    mistake

    diff --git a/test_file b/test_file
    new file mode 100644
    index 0000000..4ab08b3
    --- /dev/null
    +++ b/test_file
    @@ -0,0 +1 @@
    +some random text

Czyli tak jak podejrzewaliśmy, HEAD wskazuje na ostatni commit. Jeśli chcemy cofnąć zmiany tylko z
tego ostatniego commit'a, to musimy zmienić położenie HEAD na przedostatni commit:

    $ git log
    commit e175de8b05df16c5f2b7d2c2c23a932a1c9efeaa (HEAD, refs/remotes/origin/master, refs/heads/master)
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   2015-02-17 16:59:49 +0100

    mistake

    commit 4de0d7f07def79578ed908669c9bc0da4383a3b7
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   2015-02-17 12:21:27 +0100

    Repository setup

    $ git reset HEAD~1

    morfik:~/repozytoria/practice-makes-perfect$ git log
    commit 4de0d7f07def79578ed908669c9bc0da4383a3b7 (HEAD, refs/heads/master)
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   2015-02-17 12:21:27 +0100

    Repository setup

Jeśli zmian byłoby więcej i nie chciałoby nam się liczyć ile commit'ów trzeba wrócić, aby
doprowadzić repozytorium do stanu używalności, to możemy skorzystać z hashów i zresetować repo w
oparciu o nie, przykładowo:

    $ git log --format=oneline
    e175de8b05df16c5f2b7d2c2c23a932a1c9efeaa (HEAD, refs/remotes/origin/master, refs/heads/master) mistake
    4de0d7f07def79578ed908669c9bc0da4383a3b7 Repository setup

    $ git reset 4de0d7f07def79578ed908669c9bc0da4383a3b7

    $ git log --format=oneline
    4de0d7f07def79578ed908669c9bc0da4383a3b7 (HEAD, refs/heads/master) Repository setup

Zatem lokalnie cofnęliśmy zmiany ostatniego commit'a:

    $ git status
    On branch master
    Your branch is behind 'origin/master' by 1 commit, and can be fast-forwarded.
    (use "git pull" to update your local branch)
    Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

    new file:   test_file

Jak widzimy wyżej, git uważa, że nasze lokalne repozytorium jest ździebko nieaktualne i według niego
musimy zrobić `git pull` by dokonać aktualizacji. Problem w tym, że właśnie chcemy czegoś
odwrotnego, czyli cofnąć zmiany wprowadzone z zdalnym repo github'a.

Co się stanie jeśli damy `git push` w tym momencie? Sprawdźmy:

    $ git push
    To git@github.com:morfikov/practice-makes-perfect.git
    ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'git@github.com:morfikov/practice-makes-perfect.git'
    hint: Updates were rejected because the tip of your current branch is behind
    hint: its remote counterpart. Integrate the remote changes (e.g.
    hint: 'git pull ...') before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.

Można się było tego spodziewać. `git` odrzucił proponowane zmiany, bo nasze repo jest "nieaktualne".
Musimy skorzystać z opcji `-f` (force):

    $ git push -f
    Total 0 (delta 0), reused 0 (delta 0)
    To git@github.com:morfikov/practice-makes-perfect.git
    + e175de8...4de0d7f master -> master (forced update)

## Przywrócenie stanu

To powyższe działanie wymazuje kompletnie informacje na temat wprowadzonych zmian, co nie zawsze
jest pożądane. Musimy jeszcze wymyślić jak zapisać w logu taką informację.

Tworzymy zatem kolejny plik testowy, robimy commit'a i przesyłamy zmiany na github'a:

    $ touch test_file
    $ echo "some random text" > test_file

    $ git add .

    $ git commit -S -a -m "mistake"
    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov <morfik@nsa.com>"
    4096-bit RSA key, ID 0x72F3A416B820057A, created 2013-11-13

    [master 49f8b65] mistake
    1 file changed, 1 insertion(+)
    create mode 100644 test_file

    $ git push
    Counting objects: 3, done.
    Delta compression using up to 2 threads.
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (3/3), 964 bytes | 0 bytes/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    To git@github.com:morfikov/practice-makes-perfect.git
    4de0d7f..49f8b65  master -> master

    $ git log --format=oneline
    49f8b65ed0d2ae0c7f17fd0003af832f61c7885a (HEAD, refs/remotes/origin/master, refs/heads/master) mistake
    4de0d7f07def79578ed908669c9bc0da4383a3b7 Repository setup

By cofnąć zmiany i zachować informacje o tej o tej czynności, wydajemy poniższe polecenie:

    $ git revert 49f8b65ed0d2ae0c7f17fd0003af832f61c7885a
    [master 172d0fd] Revert "mistake"
    1 file changed, 1 deletion(-)
    delete mode 100644 test_file

Sprawdźmy log:

    $ git log --format=oneline
    172d0fd7634ff0dd1140ca50f0f37a9211223251 (HEAD, refs/heads/master) Revert "mistake"
    49f8b65ed0d2ae0c7f17fd0003af832f61c7885a (refs/remotes/origin/master) mistake
    4de0d7f07def79578ed908669c9bc0da4383a3b7 Repository setup

Został utworzony nowy commit, który cofnął zmiany tego wskazanego przy pomocy hasha. Teraz tylko
wystarczy zsynchronizować repozytorium na github'ie przy pomocy `git push` . Sprawdźmy co
zarejestrował github:

![]({{< baseurl >}}/img/2016/04/13.repozytorium-git-github-revert-commit.png#huge)

## Odpowiednie narzędzia

W pliku konfiguracyjnym `~/.config/git/config` zdefiniowaliśmy trzy opcje, które pomogą nam w
pracach z git'em. Mowa o zewnętrznych narzędziach: `merge.tool meld` , `diff.tool meld` oraz
`core.editor vim` . To z jakich narzędzi możemy korzystać w przypadku `merge.tool` i `diff.tool`
można ustalić wydając to poniższe polecenie:

    $ git difftool --tool-help
    'git difftool --tool=<tool>' may be set to one of the following:
          araxis
          meld
          vimdiff
          vimdiff2
          vimdiff3

    The following tools are valid, but not currently available:
          bc3
          codecompare
          deltawalker
          diffmerge
          diffuse
          ecmerge
          emerge
          gvimdiff
          gvimdiff2
          gvimdiff3
          kdiff3
          kompare
          opendiff
          p4merge
          tkdiff
          xxdiff

    Some of the tools listed above only work in a windowed
    environment. If run in a terminal-only session, they will fail.

Jeśli chodzi o edytor tekstu, czy w ogóle o tekstowe narzędzia, to w ich przypadku, będziemy mieli
do dyspozycji kolorowanie wyjścia. Tak wygląda `vim` podczas robienia commit'a (via `git commit` ):

![]({{< baseurl >}}/img/2016/04/14.git-commit-vim-color.png#huge)

A tu przykład diff'a ( `git diff` ):

![]({{< baseurl >}}/img/2016/04/15.git-diff-color.png#huge)

Jeśli chodzi o narzędzia do przeglądania zmian, to ja jednak wolę GUI. W tym przypadku mamy do
dyspozycji, np. `meld` ( `git difftool`) :

![]({{< baseurl >}}/img/2016/04/16.git-gui-diff-meld.png#huge)

Jest jeszcze `git merge` i `git mergetool`, odpowiednio tekstowy i GUI ale póki co jeszcze nie było
dane mi z nich skorzystać, także nic na temat tej funkcjonalności nie napiszę, może kiedyś.

No i to w sumie by było na tyle. Pozostaje nam tworzyć nowe pliki, aktualizować/usuwać stare, robić
commit'y i push'ować wszystko na github'a. Więcej informacji zawsze można znaleźć w [helpie
github'a](https://help.github.com/), lokalnie w manach lub też na [stronie
kernela](https://www.kernel.org/pub/software/scm/git/docs/).
