---
author: Morfik
categories:
- OpenWRT
date: "2016-04-24T01:14:41Z"
date_gmt: 2016-04-23 23:14:41 +0200
published: true
status: publish
tags:
- ssh
- szyfrowanie
- chaos-calmer
- router
GHissueID: 409
title: Klucze szyfrujące RSA w OpenWRT (ssh)
---

Klucze RSA w protokole SSH mogą być wykorzystane jako sposób identyfikacji danej osoby przy
logowaniu się do zdalnego serwera. Te klucze zawsze występują w parach. Jeden prywatny, drugi
publiczny. Pierwszy z nich jest znany tylko nam i powinien być trzymany w sekrecie i pilnie
strzeżony. Klucz publiczny z kolei zaś jest przesyłany na każdy serwer SSH, z którym chcemy się
połączyć. Gdy serwer jest w posiadaniu naszego klucza publicznego i widzi przy tym, że próbujemy
nawiązać połączenie, używa on tego klucza, by wysłać do nas zapytanie (challange). Jest ono
zakodowane i musi na nie zostać udzielona prawidłowa odpowiedź. Tej z kolei może udzielić ktoś, kto
jest w posiadaniu klucza prywatnego. Nie ma innej opcji, by rozkodować wiadomość. Dlatego też nikt
inny nie może udzielić na nią prawidłowej odpowiedzi. To rozwiązanie eliminuje wrażliwość na różne
formy podsłuchu. Ten kto nasłuchuje nie będzie w stanie przechwycić pakietów zawierających hasło, bo
ono nie jest nigdy transmitowane prze sieć. No i oczywiście jeśli chodzi o samo hasło, to odpadają
nam ataki bruteforce pod kątem jego złamania. W tym wpisie postaramy się zaimplementować na routerze
z OpenWrt system logowania oparty o klucze RSA.

<!--more-->
## Wybór i generowanie klucza

Są różne typy kluczy SSH. Zwykle wykorzystuje się [klucze
RSA](https://pl.wikipedia.org/wiki/RSA_%28kryptografia%29) 1024/2048/4096 bitowe. Innym typem klucza
jest ECDSA, który to opiera się o [krzywe
eliptyczne](https://pl.wikipedia.org/wiki/Kryptografia_krzywych_eliptycznych) zapewniając ten sam
poziom bezpieczeństwa co klucze RSA, z tym, że sam klucz w przypadku ECDSA jest o wiele krótszy.
Istnieje jeszcze [klucz ED25519](https://en.wikipedia.org/wiki/EdDSA), który to jest zmodyfikowaną
wersją ECDSA i powstał w odpowiedzi na podejrzenia pod kątem NSA. Czy prawdziwe, tego nie wiem. Na
necie natomiast można się natknąć na informację, że te krzywe wykorzystywane przy ECDSA nie są chyba
aż tak idealnie krzywe czy coś i mogą stanowić zagrożenie bezpieczeństwa. W każdym razie, poniżej
jest tabelka porównująca długość poszczególnych kluczy:

    Symmetric  |  ECC2N  |  ECP  |  DH/DSA/RSA
           80  |   163   |  192  |     1024
          128  |   283   |  256  |     3072
          192  |   409   |  384  |     7680
          256  |   571   |  521  |    15360

Jak widzimy, zaledwie 521 bitów wystarczy, by zapewnić poziom bezpieczeństwa, który można uzyskać
stosując klucze RSA 15360 bitów. Problem w tym, że domyślnie `dropbear` nie obsługuje kluczy SSH
innych niż RSA i by mieć obsługę tych dwóch dodatkowych typów, trzeba rekompilować źródła.
Zostaniemy zatem przy kluczach RSA. Poniżej jest komenda, za pomocą której to wygenerujemy parę
kluczy RSA.

    $ ssh-keygen -t rsa -b 4096 -C "$(whoami)@$(hostname)-$(date -I)"

Poniżej zaś jest fotka obrazująca cały proces generowania klucza RSA:

![generowanie-klucza-rsa-openwrt-ssh-router](/img/2016/04/1.generowanie-klucza-rsa-openwrt-ssh-router.png#huge)

Jako, że klucze RSA zawierają wrażliwe informacje, można je zaszyfrować, tak by każdorazowe ich
wykorzystanie wymagało podania hasła. Jednak wtedy przy logowaniu się do routera trzeba podać hasło
do klucza zamiast do konta. Nie jest to zbytnio wygodne w przypadku ciągłego wywoływania polecenia
`ssh` lub `scp` . Niemniej jednak, na linux'ach mamy do dyspozycji narzędzia, np. `gpg-agent` ,
które pozwalają na dodanie i przechowywanie szeregu kluczy w systemowym keyring'u. W takim
przypadku hasło podajemy tylko raz i możemy korzystać z kluczy do woli, oczywiście w ramach pewnego
interwału czasowego.

## Przesyłanie klucza RSA na router

Powstały w ten sposób klucz jest przechowywany na linux'ie w katalogu `~/.ssh/` . Każdy może go
podejrzeć, dlatego warto też zadbać o odpowiednie prawa dostępu do plików. Trzeba przy tym pamiętać
o jednej istotniej rzeczy. Użytkownik root może i tak obejść te ograniczenia dostępu i jeśli nie
ufamy adminowi, lepiej nie trzymać w jego systemie kluczy prywatnych. W każdym razie, jeśli to my
jesteśmy administratorem systemu i przy tym mamy w nim wielu użytkowników, powinniśmy zabezpieczyć
swoje klucze przed nieuprawnionym dostępem.

Przy tworzeniu pary kluczy, zostały stworzone dwa pliku: `router-tp-link_rsa` oraz
`router-tp-link_rsa.pub` . Pierwszy z nich to klucz prywatny i go zostawiamy w spokoju. Z kolei ten
drugi trzeba przesłać na router. Robimy to przy pomocy `scp` w poniższy sposób:

    $ scp ~/.ssh/router-tp-link_rsa.pub root@192.168.1.1:

Po adresie hosta jest dodany znak `:` . Wskazuje on katalog domowy użytkownika, w tym przypadku
root. Nie trzeba podawać pełnej ścieżki typu `/root/` .

Logujemy się teraz na router za pomocą `ssh` i dodajemy zawartość przesłanego pliku do pliku
`/etc/dropbear/authorized_keys` :

    # cat /root/router-tp-link_rsa.pub >> /etc/dropbear/authorized_keys
    # rm /root/router-tp-link_rsa.pub

Po ponownym zalogowaniu, OpenWrt powinien uwierzytelnić nas już za pomocą klucza, który przed chwilą
mu przesłaliśmy. Jeśli nie podaliśmy hasła przy tworzeniu pary kluczy, to zostaniemy zalogowani bez
pytania o hasło.

## Konfiguracja połączenia z routerem

Jeśli nuży nas trochę ciągłe wpisywanie nazwy użytkownika przy nawiązywaniu połączenia z routerem,
to możemy zrezygnować z frazy `root@` , którą zwykle się podaje przed adresem IP. W tym celu musimy
utworzyć w pliku `~/.ssh/config` stosowny wpis, przykładowo:

    Host 192.168.1.1
          User root
          IdentitiesOnly yes
          IdentityFile ~/.ssh/router-tp-link_rsa
          CheckHostIP yes
          Port 22

By dodatkowo zwiększyć bezpieczeństwo routera, przydałoby się wyłączyć możliwość logowania przy
pomocy hasła. W przypadku, gdy ktoś trzeci będzie się chciał podłączyć do routera i nie będzie przy
tym posiadał klucza prywatnego, to w dalszym ciągu będzie mógł próbować odgadnąć hasło do konta
root. Trzeba jednak pamiętać, że w przypadku utraty klucza, dostęp do routera stanie się niemożliwy
i trzeba będzie sięgać do [trybu
failsafe](/post/tryb-ratunkowy-failsafe-w-openwrt/). Jeśli jednak chcemy
zrezygnować z uwierzytelniania opartego o hasło, edytujemy na routerze plik `/etc/config/dropbear`
i przepisujemy go do poniższej postaci:

    config dropbear
          option PasswordAuth 'off'
          option RootPasswordAuth 'off'
          option RootLogin '1'
          option Port         '22'
    #     option BannerFile   '/etc/banner'
          option Interface 'br-lan'
          option IdleTimeout '300'
