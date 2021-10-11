---
author: Morfik
categories:
- Linux
date: "2016-07-03T15:30:22Z"
date_gmt: 2016-07-03 13:30:22 +0200
published: true
status: publish
tags:
- git
- gpg
- github
GHissueID: 383
title: Implementacja kluczy GPG na github'ie
---

Github to serwis, w którym użytkownicy mogą pracować wspólnie nad różnymi projektami utrzymywanymi w
systemie kontroli wersji GIT. Konto w w/w serwisie może być łakomym kąskiem dla przestępców,
zwłaszcza gdy uczestniczymy w dość rozbudowanych projektach i przyczyniamy się do ich tworzenia w
dużym stopniu. Z tego powodu github implementuje rozwiązania, które mają na celu poprawić
bezpieczeństwo naszej pracy. Mamy już uwierzytelnianie dwuetapowe, [obsługę kluczy
SSH](/post/github-z-obsluga-kluczy-ssh/) ale nadal brakowało odpornego systemu,
który by uwierzytelnił osoby współdziałające z nami. Chodzi o to, że wszelkie zmiany w repozytorium
GIT muszą być przez kogoś poczynione. Każdy commit ma zatem swojego właściciela ale my nigdy nie
mamy pewności co do tego, kto tak naprawdę tej zmiany dokonał. Dlatego też [github umożliwił
ostatnio podpisywanie tagów i commit'ów przy pomocy kluczy
GPG](https://github.com/blog/2144-gpg-signature-verification). To właśnie temu tematowi będzie
poświęcony niniejszy wpis.

<!--more-->
## GIT, GPG i github

GIT już od dość dawna umożliwia podpisywanie tagów i commit'ów przy pomocy kluczy GPG. By wdrożyć
takie klucze w serwisie github, musimy naturalnie posiadać same klucze szyfrujące oraz odpowiednio
skonfigurować sobie lokalne repozytorium. Proces [implementacji kluczy GPG w repozytorium
GIT](/post/implementacja-kluczy-gpg-repozytorium-git/) został opisany w osobnym
artykule i nie będę go tutaj poruszał na nowo. Podobnie sprawa ma się w przypadku [tworzenia kluczy
GPG](/post/bezpieczny-klucz-gpg/) oraz [konfiguracji narzędzia gpg w pliku
~/.gnupg/gpg.conf](/post/konfiguracja-gpg-w-pliku-gpg-conf/). Jeśli nie mieliśmy do
tej pory styczności z kluczami GPG, to zapraszam do zapoznania się z w/w artykułami.

## Jak dodać klucze GPG w github'ie

Mając już odpowiednie klucze oraz przygotowane repozytorium GIT, możemy przejść do implementacji
kluczy GPG na github'ie. W tym celu logujemy się w tym serwisie, przechodzimy do opcji swojego
profilu i z menu po lewej stronie wybieramy `SSH and GPG keys` :

![](/img/2016/07/1.github-git-klucze-gpg.png#small)

Po prawej stronie, pod kluczami SSH, powinien znajdować się formularz dodania klucza GPG:

![](/img/2016/07/2.github-git-klucze-gpg.png#huge)

Odpalamy teraz terminal i eksportujemy publiczną część klucza GPG w poniższy sposób:

    $ gpg --armor --export 0xCD046810771B6520
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    ...
    -----END PGP PUBLIC KEY BLOCK-----

Teraz całą zwróconą zawartość trzeba przekopiować i wrzucić do formularza na stronie:

![](/img/2016/07/3.github-git-klucze-gpg.png#huge)

Po kliknięciu w przycisk `Add GPG key` powinniśmy zostać poproszeniu o podanie hasła do konta. Tylko
po jego poprawnym podaniu klucz zostanie dodany:

![](/img/2016/07/4.github-git-klucze-gpg.png#huge)

Jeśli teraz przejdziemy na stronę naszego projektu w celu przejrzenia ostatnich commit'ów, to
zobaczymy, że szereg z nich ma ikonkę `Verified` :

![](/img/2016/07/5.github-git-klucze-gpg.png#huge)

Ta ikonka informuje nas o fakcie zweryfikowania podpisu osoby, która przesłała commit'a i
identyfikuje się znanym nam kluczem GPG.
