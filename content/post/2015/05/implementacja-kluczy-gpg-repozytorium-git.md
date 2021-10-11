---
author: Morfik
categories:
- Linux
date: "2015-05-17T21:11:41Z"
date_gmt: 2015-05-17 19:11:41 +0200
published: true
status: publish
tags:
- git
- gpg
GHissueID: 228
title: Implementacja kluczy GPG w repozytorium GIT
---

W tym artykule zostanie przedstawiony sposób na wykorzystanie kluczy GPG w przypadku
przeprowadzanych działań w repozytorium GIT. Będziemy w ten sposób w stanie podpisać swoje commit'y
czy tagi, by było wiadomo, że zmiany, które zostały poczynione pochodzą naprawdę od konkretnego
użytkownika. Oczywiście to może się wydać przesadą dla wielu ludzi ale skoro mamy udostępnioną
możliwość wykorzystania kluczy GPG, to czemu z tej opcji nie skorzystać? Potrzebne będą nam tylko
[klucze GPG](/post/bezpieczny-klucz-gpg/). Sposób ich tworzenia jak i [konfiguracja
zdefiniowana w pliku gpg.conf](/post/konfiguracja-gpg-w-pliku-gpg-conf/) nie
zostaną tutaj opisane. Zamiast tego skupimy się jedynie na implementacji samych kluczy GPG.

<!--more-->
## Definiowanie klucza GPG w ~/.gitconfig

Plik konfiguracyjny narzędzia `git` ma sporo użytecznych opcji. Jedną z nich jest ta odpowiadająca
za sprecyzowanie klucza GPG, którym będziemy podpisywać pewne rzeczy. Rozchodzi się oczywiście o
parametr `user.signingkey` . By dodać odpowiedni wpis, listujemy pierw prywatne klucze szyfrujące,
które mamy w systemie:

    $ gpg --list-secret-keys
    /home/morfik/.gnupg/secring.gpg
    -------------------------------
    sec   4096R/0x72F3A416B820057A 2014-08-08
    Key fingerprint = 9F30 BD23 F4A2 3921 5E29  A3B6 72F3 A416 B820 057A
    uid                            Mikhail Morfikov <morfik@nsa.com>
    ssb   4096R/0x46DF79598289C782 2014-08-08

Teraz wpisujemy w terminal poniższą linijkę:

    $ git config --global user.signingkey 0x72F3A416B820057A

Spowoduje to dodanie do pliku konfiguracyjnego `~/.gitconfig` odpowiednich wpisów. Można oczywiście
ręcznie edytować ten plik i dodać poniższą linijkę:

    [user]
    ...
    signingkey = 0x72F3A416B820057A

Od od tego momentu jesteśmy w stanie podpisywać tagi oraz commit'y.

## Podpisywanie tagów w repozytorium GIT

Spróbujmy zatem coś podpisać. Na początek tagi:

    $ git tag -s v1.0 -m 'morfiprojekt'

    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov <morfik@nsa.com>"
    4096-bit RSA key, ID 0x72F3A416B820057A, created 2014-08-08

Jak widać, zostaliśmy poproszeni o hasło do prywatnego klucza GPG. Popatrzmy jak zatem wygląda sam
tag po podpisaniu:

    $ git show v1.0
    tag v1.0
    Tagger: Mikhail Morfikov <morfik@nsa.com>
    Date:   Mon Feb 16 17:48:13 2015 +0100

    morfiprojekt
    -----BEGIN PGP SIGNATURE-----

    iQIcBAABCgAGBQJU4h9NAAoJEM0EaBB3G2UgsuMQAKtAx9m/bG4zCtsrSH5Mq832
    +P6YPkZMqbnXCITVRy7HqVigFDunYtGXu5Y6ZfUYV+kjRlAXMnMQHYFghwVej7cu
    Os3WEu2OFs1sbyzSgUkpVlVcHsjO4TjoKoZum7Wgk3IIzONdreyv8nrYsNjiPb5F
    UTdu5ERb/AHExzS2wS6MXdBl/PtAY8b+o2vdWahhuZ5nIHC2F5GK6m05Dhkr6LvE
    2ikO4pKZY+/toDV9Q1xalUDIloaeB5pcB+2VOuLBQYKT8OaVjfSt9vH6ZrJbe7SE
    oJ7cSIMRhPbqz5kCqEoYesOoOA6J75FaeWiEx0DNHlegnVg6CqkSwPyeeccVrY4H
    hslNYEf3lEGWV4HakwuOZFq1Rx+R6D/vnMCoFED/S56W2GLIv7HfPveb3AW+1Pap
    rFbZ0SGIarSePLUF+qkvD53qeQ5HoV05laCKIUp9D93ybryjA6YiMib8sylh7DOr
    GwWuLi06hpzQw4QN1Vj8Cmw900J8MB5GpW7NoxN00NBz6sWNpnBeDmBpS77tcyOy
    KGyrGO5MzCfIKA79jJaBBSUB9h3G94haNDoYPbMrv3x6Sx8CIsDds8cQ+YSDlnKS
    94Kf7yJ/9QNiX15dmo8j21+oIjQlaoq9aX2TtvxND4ELJhIE0ulIobU5xZy1Bc0d
    Gc7txjpF4eQsOQh+l4Ka
    =7UmQ
    -----END PGP SIGNATURE-----

Widzimy, że została dodana do niego sygnatura. Zweryfikujmy ją zatem. Do tego celu potrzebny będzie
klucz publiczny tej osoby co ten tag podpisała, a sam proces weryfikacji wygląda tak:

    $ git tag -v v1.0
    object 75f7bdc3ef9f6846371cf24604a53f5d9d9e51ad
    type commit
    tag v1.0
    tagger Mikhail Morfikov <morfik@nsa.com> 1424105293 +0100

    morfiprojekt
    gpg: Signature made Mon 16 Feb 2015 05:48:13 PM CET
    gpg:                using RSA key 0x72F3A416B820057A
    gpg: Good signature from "Mikhail Morfikov <morfik@nsa.com>" [ultimate]
    Primary key fingerprint: 9F30 BD23 F4A2 3921 5E29  A3B6 72F3 A416 B820 057A

Sygnatura została poprawnie zweryfikowana.

## Podpisywanie commit'ów w repozytorium GIT

No to teraz spróbujmy podpisać jakiegoś commit'a:

    $ git status
    On branch master
    Your branch is up-to-date with 'origin/master'.
    Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

    new file:   jakis_tam_plik_testowy

    $ git commit -a -S -m 'test podpisu commita'
    You need a passphrase to unlock the secret key for
    user: "Mikhail Morfikov <morfik@nsa.com>"
    4096-bit RSA key, ID 0x72F3A416B820057A, created 2014-08-08

    [master 2332031] test podpisu commita
    1 file changed, 1 insertion(+)
    create mode 100644 jakis_tam_plik_testowy

    $ git status
    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
    (use "git push" to publish your local commits)
    nothing to commit, working directory clean

I teraz już tylko zostało zweryfikowanie podpisu:

    $ git log --show-signature -1
    commit 23320312843f5631dd15851eb61483ef3a4491a4
    gpg: Signature made Mon 16 Feb 2015 06:15:29 PM CET
    gpg:                using RSA key 0x72F3A416B820057A
    gpg: Good signature from "Mikhail Morfikov <morfik@nsa.com>" [ultimate]
    Primary key fingerprint: 9F30 BD23 F4A2 3921 5E29  A3B6 72F3 A416 B820 057A
    Author: Mikhail Morfikov <morfik@nsa.com>
    Date:   Mon Feb 16 18:15:29 2015 +0100

    test podpisu commita

I tutaj również wszystko przebiegło pomyślnie.

Więcej informacji na temat wykorzystania kluczy GPG można znaleźć
[tutaj](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work#_signing).
