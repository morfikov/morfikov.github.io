---
author: Morfik
categories:
- Linux
date: "2015-06-21T15:55:30Z"
date_gmt: 2015-06-21 13:55:30 +0200
published: true
status: publish
tags:
- email
- szyfrowanie
- prywatność
title: Szyfrowanie poczty przy pomocy Enigmail
---

Szyfrowanie wiadomości email można uznać za paranoiczne podejście do kwestii wymiany informacji, bo
zwykły człowiek mówi sobie: "ja przecie za pomocą poczty nie wysyłam nic ważnego, nawet swoich
nagich fotek, a nawet jeśli już, to są one od pasa w dół. Poza tym, gmail jest szyfrowany, bo używa
SSL/TLS". SSL/TLS, co prawda, jest ale ogranicza się do szyfrowania tego co robimy na gmailu. Same
wiadomości natomiast są przesyłane między różnymi serwerami i niekoniecznie są szyfrowane. Poza tym
google kiedyś wspominał, że w przypadku gdy policja będzie żądała dostępu do naszej skrzynki
pocztowej, to nie dość, że on im to umożliwi, to jeszcze wyciągnie wszelkie maile jakie przez tę
skrzynkę zostały przepuszczone -- zarówno te odebrane jak i wysłane.

<!--more-->
## Szyfrować każdy może

Gdyby ludzie wiedzieli jak prosto można zabezpieczyć komunikację między dwoma punktami, nie
mielibyśmy problemów z wdrażaniem różnych technik szyfrowania danych. Jak to się jednak dzieje, że
po mimo faktu prostoty implementacji oprogramowania szyfrującego, cała rzesza osób w dalszym ciągu
nie potrafi zabezpieczyć swojej poczty? Obecnie cała filozofia szyfrowania sprowadza się do
zainstalowanie jednego pakietu czy wtyczki i wyklikania odpowiednich opcji. W tym wpisie postaram
się prześledzić szyfrowanie poczty na przykładzie [klienta mailowego
Thunderbird](https://www.mozilla.org/pl/thunderbird/) i jednego z jego dodatków
[enigmail](https://www.enigmail.net/index.php/en/).

W debianie nie ma takiej aplikacji jak Thunderbird ale jest za to
[Icedove](https://wiki.debian.org/Icedove), który z grubsza ma tylko zmienioną nazwę, także
wszystkie kroki przeprowadzone pod kątem Thunderbirda stosują się także do Icedove. Jeśli jednak
zamierzamy korzystać z Icedove, to mamy także w systemie dostępny pakiet `enigmail` , który możemy
doinstalować. W przeciwnym wypadku, trzeba go będzie pobrać ze strony mozilli i ręcznie
zainstalować.

Zakładam, że mamy już dodane konto pocztowe, oraz, że orientujemy się w kwestiach związanych z
kluczami gpg. Jeśli nie, to polecam pierw zapoznać się z artykułami na temat [pliku konfiguracyjnego
gpg.conf]({{< baseurl >}}/post/konfiguracja-gpg-w-pliku-gpg-conf/), jak i również ze sposobem
[tworzenia samych kluczy gpg]({{< baseurl >}}/post/bezpieczny-klucz-gpg/).

## Konfiguracja dodatku enigmail

Po zainstalowaniu dodatku enigmail w Thunderbirdzie, w jego menu powinna się nam pojawić dodatkowa
pozycja:

![]({{< baseurl >}}/img/2015/06/1.enigmail-menu-thunderbird.png#medium)

Mamy tam do dyspozycji szereg opcji, dzięki którym jesteśmy w stanie tworzyć, usuwać klucze,
przypisywać je do określonych kont pocztowych. Możemy również szyfrować, deszyfrować i podpisywać
wiadomości przy pomocy tych kluczy:

![]({{< baseurl >}}/img/2015/06/2.enigmail-menu.png#small)

Jak wspomniałem wyżej, klucze możemy tworzyć za pomocą narzędzia `gpg` . Jeśli jednak chcemy tego
dokonać za pośrednictwem dodatku enigmail, to wybieramy z menu Enigmail => Key Managment i tam już
dajemy Generate => New Key Pair . Powinno nam się pokazać to poniższe okienko:

![]({{< baseurl >}}/img/2015/06/3.generowanie-klucza-enigmail.png#big)

Zgodnie z informacją widoczną wyżej, generowanie klucza może zająć nawet kilka minut. By
przyśpieszyć ten proces i poprawić siłę samego klucza, dobrze jest zmobilizować komputer do
generowania losowego szumu. Możemy to zrobić przez wciskanie przycisków na klawiaturze, chaotyczne
poruszanie myszą, czy też przez odczyt/zapis danych na dysku.

Po wygenerowaniu klucza, zostaniemy także poproszeni o stworzenie certyfikatu unieważniającego
klucz. To na wypadek gdyby klucz szyfrujący wpadł w niepowołane ręce:

![]({{< baseurl >}}/img/2015/06/4.generowanie-certyfikatu-enigmail.png#huge)

Klucz powinien już widnieć na liście kluczy gpg:

![]({{< baseurl >}}/img/2015/06/5.lista-kluczy-enigmail.png#big)

Pamiętajmy też by wysłać klucz na jeden z serwerów kluczy.

Jeśli z jakichś względów byśmy zmieniali adres email, lub też klucz nie został by przypisany do
żadnego z istniejących kont, to by go wykorzystać musimy go powiązać z jakimś kontem. W tym celu
przechodzimy w menu Thunderbirda do Enigmail => Edit Per Recipient Rules i definiujemy reguły dla
kluczy:

![]({{< baseurl >}}/img/2015/06/6.reguly-kluczy-enigmail.png#big)

Po określeniu dopasowania, nasz klucz powinien pojawić się na liście reguł:

![]({{< baseurl >}}/img/2015/06/7.lista-regul-enigmail.png#huge)

Możemy naturalnie dostosować sobie poszczególne opcje. Ja domyślnie podpisuje wszystkie swoje
wiadomości, dlatego też zaznaczyłem jedynie `sign` .

## Szyfrowanie poczty

Mając już przygotowany klucz, przejdźmy do zaszyfrowania testowej wiadomości. Zaszyfrowanego maila
tworzymy dokładnie w taki sam sposób, co w przypadku jego zwykłego odpowiednika:

![]({{< baseurl >}}/img/2015/06/10.szyfrowanie-wiadomosci-enigmail.png#huge)

Jeśli kłódka jest wciśnięta, wiadomość będzie szyfrowana. Podobnie sprawa ma się w przypadku ołówka,
z tym, że on odpowiada za podpisywanie wiadomości. Jeśli nie wysyłaliśmy klucza publicznego na jeden
z serwerów kluczy, to możemy dołączyć klucz publiczny do samej wiadomości. W ten sposób odbiorca
będzie w stanie zweryfikować sygnaturę, która zostanie przez nas złożona.

Jeśli to my jesteśmy odbiorcą wiadomości i w przypadku gdy ona została zaszyfrowana, to raczej nie
będziemy w stanie jej przeczytać bez odpowiedniego klucza prywatnego:

![]({{< baseurl >}}/img/2015/06/8.zaszyfrowania-wiadomosc-enigmail.png#huge)

Jeśli jednak ktoś zaszyfrował tę wiadomość przy pomocy naszego klucza publicznego, to przy
założeniu, że mamy odpowiadający mu klucz prywatny, jesteśmy w stanie odszyfrować tę wiadomość. W
tym celu z menu wybieramy Enigmail => Decrypt/Verify. Przy czym, nie musimy tego robić ręcznie za
każdym razem gdy tylko otrzymamy zaszyfrowaną wiadomość.Możemy zaznaczyć opcję pod Enigmail =>
Automatically Decrypt/Verify Messaged, by wiadomości były automatycznie deszyfrowane i weryfikowane
w przypadku gdy posiadamy odpowiednie klucze. Tak czy inaczej wiadomość powinna zostać odszyfrowana:

![]({{< baseurl >}}/img/2015/06/9.odszyfrowana-wiadomosc-enigmail.png#huge)

Jak widzimy wyżej, wszystko przebiegło bez problemów. Sygnatura także jest w porządku. Od tej chwili
szyfrowanie/deszyfrowanie wiadomości odbywać się będzie automatycznie. W przypadku gdy ktoś prześle
nam zaszyfrowaną wiadomość naszym kluczem publicznym, to by uzyskać do niej dostęp będziemy musieli
podać hasło do klucza prywatnego. W taki oto prosty sposób mamy pewność, że tylko my i nasz rozmówca
znamy treść wiadomości.
