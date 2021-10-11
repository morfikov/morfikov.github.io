---
author: Morfik
categories:
- Linux
date: "2015-11-20T14:01:17Z"
date_gmt: 2015-11-20 13:01:17 +0100
published: true
status: publish
tags:
- firefox
- dns
GHissueID: 259
title: Konfiguracja cache DNS w Firefox'ie
---

We wpisie poświęconym [systemowemu cache DNS w
linux'ie](/post/cache-dns-buforowania-zapytan/) mieliśmy okazję zobaczyć jak
wzrasta wydajność po zaimplementowaniu tego mechanizmu. W skrócie, to ponad drugie tyle zapytań było
rozwiązywanych lokalnie bez potrzeby odwoływania się do zdalnego serwera DNS, co zajmuje sporo czasu
(20-40ms). Przeglądarki internetowe, np. Firefox, mają swoje wynalazki, które potrafią wyeliminować
opóźnienia związane z surfowaniem po stronach www. Do nich zalicza się również cache DNS, z tym, że
w tym przypadku zaimplementowany jest on na poziomie przeglądarki, a nie globalnie w systemie.
Dzięki temu rozwiązaniu, nawet bez `dnsmasq` , Firefox jest nam w stanie zaoszczędzić sporo czasu
przy przeglądaniu internetu. Zajrzyjmy zatem Firefox'owi pod maskę i sprawdźmy, które parametry
dotyczące cache DNS wymagają dostosowania.

<!--more-->
## Parametry odpowiedzialne za cache DNS w about:config

Dostęp do wszystkich interesujących nas opcji możemy uzyskać wpisując w polu adresu Firefox'a
`about:config` . Tam z kolei wyszukujemy frazę `network.dns` . Powinno to wyglądać mniej więcej tak
jak na tym obrazku poniżej:

![](/img/2015/11/1.cache-dns-firefox-about-config.png#huge)

### network.dns.disablePrefetch

Pierwszym parametrem, którym się zajmiemy, jest
[network.dns.disablePrefetch](http://kb.mozillazine.org/Network.dns.disablePrefetch). Ma on za
zadanie poprawić czas ładowania się stron www przez wcześniejsze jednoczesne rozwiązywanie nazw
wszystkich odnośników do innych stron www, obrazków czy też styli CSS. Problem z tą opcją jest taki,
że strony www w dużej mierze są dynamiczne i najeżone dziesiątkami różnorakich linków. W efekcie
czego, Firefox wykonuje czasem setki niepotrzebnych zapytań DNS za każdym razem gdy odwiedzamy
pojedynczą stronę www, np. musi przeskanować te wszystkie linki, w które i tak nigdy nie klikniemy.
W przypadku gdy bardzo intensywnie biegamy po internecie, to ta opcja może być nawet użyteczna.
Natomiast w każdym innym przypadku można z tej opcji zrezygnować.

### network.dnsCacheEntries

Kolejnym parametrem, który warto sobie dostosować jest
[network.dnsCacheEntries](http://kb.mozillazine.org/Network.dnsCacheEntries). Odpowiada on za ilość
wpisów w cache DNS, które Firefox jest w stanie przechowywać. Im większa wartość tego parametru, tym
będzie ich więcej. Pociąga to także za sobą zwiększony apetyt Firefox'a na pamięć RAM. Nie będzie
tego raczej znowu aż tak dużo ale raczej nie powinniśmy ustawiać tutaj wartości idącej w dziesiątki
czy setki tysięcy.

### network.dnsCacheExpiration

Musimy także rzucić okiem na
[network.dnsCacheExpiration](http://kb.mozillazine.org/Network.dnsCacheExpiration). Ta opcja z kolei
odpowiada za czas (w sekundach) ważności wpisu w cache DNS Firefox'a. Im dłuższy czas ustawimy
tutaj, tym rzadziej Firefox będzie odpytywał upstream'owy serwer DNS. Oczywiście może się zdarzyć
tak, że adres IP powiązany z jakąś domeną ulegnie zmianie i w takim przypadku będziemy posiadać
nieaktualne wpisy w cache, przez co nie będziemy w stanie nawiązać połączenia z określonymi
domenami. Gdy ten parametr zostanie ustawiony na `0` , cache DNS zostanie wyłączony, a Firefox
będzie korzystał z tego udostępnianego przez system operacyjny. Zaletą ustawienia tutaj `0` jest
fakt, że możemy [posiadać więcej niż jeden profil w
Firefox'ie](/post/wiecej-niz-jeden-profil-w-firefoxie/), a każdy z nich będzie
korzystał z tego samego cache DNS oferowanego przez system operacyjny.

### network.dns.get-ttl

Dalej mamy [network.dns.get-ttl](https://support.mozilla.org/en-US/questions/1049324), który ma na
celu wymusić by Firefox dokonywał [asynchronicznych zapytań
DNS](http://unixwiz.net/tools/fastzolver.html#adns). W czasie czekania na odpowiedź od serwera DNS,
Firefox może przeprowadzać inne operacje. Jak możemy wyczytać w pierwszym linku, takie zachowanie
może prowadzić do problemów związanych z zaporami sieciowymi, bo w momencie odnalezienia
odpowiedzi, serwer DNS będzie się próbował połączyć z naszym komputerem w celu jej przekazania. Ten
parametr może także w pewnych przypadkach prowadzić do [znacznego spowolnienia działania
Firefox'a](https://support.mozilla.org/en-US/questions/1050161). Jeśli strony ładują nam się
dramatycznie wolno, to można spróbować przestawić ten parametr na `false` .

### network.dns.disableIPv6

Kolejny parametr to [network.dns.disableIPv6](http://kb.mozillazine.org/Network.dns.disableIPv6) i
odpowiada on za konfigurację zapytań DNS w protokole ipv6. W pewnych sytuacjach może on powodować
problemy z połączeniem objawiające się powolnym ładowaniem stron www. Jeśli nasz komputer nie ma
przydzielonej adresacji ipv6, to możemy pokusić się o przestawienie tej opcji na `true` .

### network.dns.ipv4OnlyDomains

Parametr [network.dns.ipv4OnlyDomains](http://kb.mozillazine.org/Network.dns.ipv4OnlyDomains)
zawiera ciągi domen, które mają być rozwiązywane jedynie za pomocą protokołu ipv4. Chodzi o to, że
nie wszystkie serwery w internecie poprawnie potrafią obsługiwać protokół ipv6 i mogą zwracać błędne
wyniki. Przez taki san rzeczy, z pewnymi domenami nie będziemy mogli się połączyć lub też połączenie
będzie trwało dość długo. Dlatego też takie domeny najlepiej jest dopisać tutaj. Trzeba tylko
pamiętać, że opcja `network.dns.disableIPv6` musi być jednocześnie ustawiona na `false` .

## Czyszczenie zawartości cache DNS w Firefox'ie

Jeśli z jakichś powodów nie możemy nawiązać połączenia z określoną domeną, a wszystko przy tym
wskazuje, że działa ona poprawnie, to prawdopodobnie jest jakiś problem z wpisami w cache DNS.
Zawartość tego cache możemy wyciągnąć wpisując w polu adresu Firefox'a `about:networking` , poniżej
przykład:

![](/img/2015/11/2.statystyki-cache-dns-firefox.png#huge)

W kolumnie `Expires` jest określony czas ważności wpisu. W przypadku gdy znajdują się w niej duże
wartości, np. 3600 (godzina), to przydałoby się ten cache opróżnić. Możemy to zrobić przez
przestawienie parametru `network.dnsCacheExpiration` tymczasowo na `0` . Choć na dobrą sprawę, to
możemy ten parametr przestawić na dowolną wartość, bo znaczenie ma jedynie jej zmiana, która
powoduje opróżnienie całego cache.
