---
author: Morfik
categories:
- Linux
date: "2015-11-05T20:52:20Z"
date_gmt: 2015-11-05 19:52:20 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- iptables
title: Fwknop z obsługą kuczy GPG
---

Ostatnio opisywałem jak zaimplementować na swoim serwerze [mechanizm port
knocking'u](/post/port-knocking-i-single-packet-authorization/) , który oparty był
o [Single Packet Authorization](http://www.cipherdyne.org/fwknop/docs/fwknop-tutorial.html). Tamten
wpis dotyczył głównie wykorzystania szyfrów symetrycznych ale istnieje też możliwość skorzystania z
kluczy GPG. W ten sposób uwierzytelnianie oraz szyfrowanie pakietów odbywałoby się przy ich pomocy.
W tym wpisie postaramy się tak skonfigurować narzędzie `fwknop` , tak by było ono w stanie
przepuszczać jedynie tych klientów, którzy posługują się kluczami GPG.

<!--more-->
## Klucze GPG

By móc skorzystać z opcji kluczy szyfrujących w przypadku mechanizmu Single Packet Authorization,
musimy posiadać [klucz GPG](/post/bezpieczny-klucz-gpg/). Choć na dobrą sprawę
potrzebujemy dwóch, po jednym dla klienta oraz dla serwera. W przypadku wielu klientów, dla każdego
z nich można wygenerować osobny klucz. Można również korzystać z tego samego klucza w przypadku
wszystkich klientów. Jeśli chodzi jeszcze o maszyny klienckie, to można skorzystać z normalnego
klucza GPG wykorzystywanego, np. do szyfrowania poczty. Dla serwera trzeba stworzyć nowy klucz, a to
ze względu na fakt, że hasło do tego klucza musi zostać zapisane w pliku `/etc/fwknop/access.conf` .
`fwknop` może także wykorzystać klucze bez hasła. Tak czy inaczej żadne powyższe rozwiązanie nie
jest w stanie tego klucza należycie zabezpieczyć.

Korzystanie z kluczy GPG w przypadku `fwknop` ma jedno ograniczenie. Jako, że zaszyfrowana wiadomość
musi się zawrzeć w jednym pakiecie IP, to długość klucza GPG w przypadku serwera [nie może
przekraczać 2048 bitów](http://www.cipherdyne.org/fwknop/docs/gpghowto.html).

Jak wspomniałem wyżej, potrzebujemy dwóch kluczy. Zatem to poniższe polecenie wykonujemy dwukrotnie,
raz na kliencie i raz na serwerze. Zrezygnujmy także z ustawiania tym kluczom hasła:

    $ gpg --gen-key

Klucze publiczne trzeba wyeksportować do pliku (format ASCII):

    # gpg -a --export 0x72F3A416B820057A > server.asc
    $ gpg -a --export 0xFFE5312387B46932 > client.asc

Pliki `server.asc` oraz `client.asc` przesyłamy w bezpieczny sposób na drugą maszynę. Najlepiej to
zrobić posługując się narzędziem `scp` :

    $ scp ./client.asc root@192.168.1.150:/root/
    $ scp root@192.168.1.150:/root/server.asc ./

W tej chwili powinniśmy mieć kopie kluczy publicznych zarówno na serwerze jak i na kliencie. Na
każdej z tych maszyn importujemy klucze do keyring'a GPG i podpisujemy je:

    # gpg --import ./client.asc
    # gpg -u 0x72F3A416B820057A --edit-key 0xFFE5312387B46932
    gpg> sign

    $ gpg --import server.asc
    $ gpg -u 0xFFE5312387B46932 --edit-key 0x72F3A416B820057A
    gpg> sign

## Konfiguracja fwknop na serwerze

Musimy jeszcze skonfigurować dostęp do serwera. Edytujemy zatem plik `/etc/fwknop/access.conf` i
wpisujemy w nim ten poniższy blok kodu:

    SOURCE: ANY;
    OPEN_PORTS: tcp/22;
    #DATA_COLLECT_MODE: PCAP;
    GPG_REMOTE_ID: 0x72F3A416B820057A;
    GPG_DECRYPT_ID: 0xFFE5312387B46932;
    #GPG_DECRYPT_PW: xxxxx;
    GPG_ALLOW_NO_PW: Y;
    GPG_HOME_DIR: /root/.gnupg;
    FW_ACCESS_TIMEOUT: 20;

## Konfiguracja fwknop na kliencie

Musimy także wygenerować plik `~/.fwknoprc` na maszynie klienckiej. Robimy to w poniższy
    sposób:

    $ fwknop -A tcp/22 --gpg-recipient-key 0x72F3A416B820057A  --gpg-signer-key 0xFFE5312387B46932 -a 192.160.10.10 -D 192.168.1.150 --gpg-no-signing-pw --save-rc-stanza

Po tej operacji w pliku `~/.fwknoprc` powinniśmy mieć tę poniższą zwrotkę:

    [192.168.1.150]
    ALLOW_IP: 192.168.10.10;
    USE_GPG: Y;
    GPG_RECIPIENT: 0x72F3A416B820057A;
    GPG_SIGNER: 0xFFE5312387B46932;
    GPG_NO_SIGNING_PW: Y;
    ACCESS: tcp/22;
    SPA_SERVER: 192.168.1.150;

Od tej pory, by wysłać ciasteczko do serwera i otworzyć tym samym port, wpisujemy:

    $ fwknop -n 192.168.1.150
