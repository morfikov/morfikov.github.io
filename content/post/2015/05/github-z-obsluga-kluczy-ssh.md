---
author: Morfik
categories:
- Linux
date: "2015-05-17T21:06:05Z"
date_gmt: 2015-05-17 19:06:05 +0200
published: true
status: publish
tags:
- ssh
- git
title: GitHub z obsługą kluczy SSH
---

W końcu przyszedł czas na eksperymenty z serwisem GitHub. Jakby nie patrzeć, do tej pory jedyne co
potrafiłem zrobić w przypadku samego gita, to wydać jedno polecenie, którym było `git clone` .
Wszelkie inne rzeczy, choć nie było ich wcale tak dużo, robiłem via panel www, co trochę było
upierdliwe. Postanowiłem nauczyć się obsługi gita i nieco uprościć sobie życie. Jeśli chodzi o samą
naukę, to tutaj jest dostępny [dość obszerny pdf](https://git-scm.com/book/en/v2) (prawie 500
stron). Nie będę tutaj przerabiał wyżej podlinkowanej książki, bo w sumie jeszcze jej nie
przeczytałem, tylko zajmę się ciekawym tematem jakim jest implementacja kluczy SSH, tak by operować
na gicie bez zbędnych haseł.

<!--more-->
## Implementacja kluczy SSH w serwisie GitHub

Przede wszystkim potrzebne będą nam [klucze
SSH]({{< baseurl >}}/post/uwierzytelniajace-klucze-ssh/) ale cały proces ich tworzenia nie
zostanie uwzględniony w tym artykule. Dalsza część zakłada, że mamy już do dyspozycji jakiś klucz
SSH oraz, że jesteśmy w posiadaniu konta na [githabie](https://github.com/).

Mając klucz SSH i konto, możemy przejść do powiązania tych dwóch rzeczy ze sobą. W tym celu włazimy
w opcje konta i po lewej stronie klikamy w `SSH keys` :

![]({{< baseurl >}}/img/2015/06/1.ssh-github-profil.png#medium)

Teraz klikamy na `ADD SSH key` (prawy róg) i uzupełniamy pola od nazwy i klucza. Powinno to wyglądać
mniej więcej tak:

![]({{< baseurl >}}/img/2015/06/2.dodawanie-kluczy-ssh-github.png#huge)

Po pomyślnym dodaniu klucza powinniśmy zobaczyć go na liście:

![]({{< baseurl >}}/img/2015/06/3.dodawanie-kluczy-ssh-github-2.png#huge)

I to w sumie z grubsza tyle jeśli chodzi o konfigurację GitHub'a. Natomiast by mieć pewność, że
wszystkie powyższe kroki przeprowadziliśmy należycie, możemy przetestować konfigurację:

    $ ssh -T git@github.com
    Hi morfikov! You've successfully authenticated, but GitHub does not provide shell access.

## Konfiguracja klienta

Teraz wystarczy założyć jakieś repozytorium i możemy przejść do konfiguracji klienta. Tu oczywiście
musimy sobie zainstalować paczkę `git` . Jeśli komuś nie odpowiada operowanie na konsoli, może
skorzystać z graficznego narzędzia co się zwie `git-cola` .

Mając już swoje własne repozytorium oraz odpowiednie pakiety w systemie, odpalamy terminal i
klonujemy to repozytorium:

    $ git clone https://github.com/morfikov/conffiles
    Cloning into 'conffiles'...
    remote: Counting objects: 61, done.
    remote: Compressing objects: 100% (29/29), done.
    remote: Total 61 (delta 29), reused 58 (delta 29)
    Unpacking objects: 100% (61/61), done.
    Checking connectivity... done.

By komunikować się z tym repozytorium, potrzebne są odpowiednie adresy, a te można podejrzeć przez:

    $ cd conffiles
    $ git remote -v
    origin  https://github.com/morfikov/conffiles (fetch)
    origin  https://github.com/morfikov/conffiles (push)

I jak widać, jest tutaj link zaczynający się od `https` i jeśli spróbowalibyśmy zrobić, np. `push`,
zostanie wyrzucone zapytanie o użytkownika oraz hasło:

    $ git push
    Username for 'https://github.com': morfikov
    Password for 'https://morfikov@github.com':

I tego właśnie chcemy uniknąć. Raz, że przesyłanie hasła przez sieć wyszło z mody, a dwa, to lepiej
nie utrudniać sobie zanadto życia pamiętaniem zbędnych rzeczy. By pozbyć się hasła, musimy [zmienić
te dwa powyższe adresy](https://help.github.com/articles/changing-a-remote-s-url/). Nie jest to
jakoś specjalnie trudne, jedyne co musimy zrobić to dostosować poniższą linijkę:

    $ git remote set-url origin git@github.com:morfikov/conffiles.git

W moim przypadku nazwa użytkownika to `morfikov`, zaś nazwa repozytorium to `conffiles` (adres:
https://github.com/morfikov/conffiles). Po wydaniu powyższego polecenia, adresy powinny ulec
zmianie:

    $ git remote -v
    origin  git@github.com:morfikov/conffiles.git (fetch)
    origin  git@github.com:morfikov/conffiles.git (push)

Teraz `push` powinien przejść bez problemu:

    $ git push
    Everything up-to-date

Jeśli nie chce nam się przechodzić przez proces zmiany linków, możemy sklonować repozytorium w
poniższy sposób:

    $ git clone git@github.com:morfikov/conffiles.git

Linki zostaną automatycznie ustawione na ssh.
