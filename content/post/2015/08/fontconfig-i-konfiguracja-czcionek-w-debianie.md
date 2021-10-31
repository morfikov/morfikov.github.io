---
author: Morfik
categories:
- Linux
date: "2015-08-05T17:10:33Z"
date_gmt: 2015-08-05 15:10:33 +0200
published: true
status: publish
tags:
- debian
- czcionki
- monitor
- lcd
GHissueID: 280
title: Fontconfig i konfiguracja czcionek w Debianie
---

Od zawsze podobały mi się czcionki windosowskie ale po przejściu na linux'a okazało się, że tutaj
fonty wyglądają zupełnie inaczej i co mogło zdziwić, nie było w standardzie tych moich ulubionych,
tj. Arial, Times New Roman i Courier New. Przez szereg lat miałem obecną w systemie dość dziwną
konfigurację dla fontconfig'a, która działała na takiej zasadzie, że te czcionki aplikacji były w
prządku, natomiast te pobierane z serwisów www (np. w Firefox'ie) dość słabo się renderowały i bez
przeprowadzania kilku zabiegów były one zwyczajnie nieczytelne. Postanowiłem w końcu poczytać trochę
dokumentacji na temat tego jak wygląda konfiguracja czcionek w debianie i po kilku dniach udało mi
się osiągnąć dość zadowalające efekty wizualne.

<!--more-->
## Wielkość plamki

Jeśli spojrzymy na monitor LCD z odległości około pół metra, nasze oko niekoniecznie może dostrzec
pojedyncze piksele w momencie gdy ich gęstość nie przekracza 300 PPI (Pixels Per Inch ). Ile zatem
mają przeciętne wyświetlacze dołączone do naszych komputerów? Możemy to w łatwy sposób
[policzyć](https://en.wikipedia.org/wiki/Pixel_density#Calculation_of_monitor_PPI). Wszystko czego
nam trzeba to kilku informacji, tj. rozdzielczość poziomą, pionową monitora oraz długość przekątnej
ekranu. Dla przykładu weźmy monitor 21.5 cala o rozdzielczości 1920x1080. Poniżej jest wzór, do
którego podstawimy te liczby:

![wyliczanie-ppi-wzor](/img/2015/07/1.wyliczanie-ppi-wzor.png#small)

`a` to rozdzielczość pozioma w pikselach, `b` to rozdzielczość pionowa, również w pikselach, z kolei
`x` to długość przekątnej w calach. Zatem mamy 1080^2=1166400 oraz 1920^2=3686400, co po zsumowaniu
daje wartość 4852800. Wyciągamy pierwiastek i otrzymujemy liczbę liczbę około 2203, którą to trzeba
podzielić przez 21.5. W rezultacie PPI dla tego przykładowego monitora wynosi około 102. Jeśli ktoś
nie chce przeprowadzać obliczeń ręcznie, to zawsze może skorzystać z [kalkulatora
PPI](https://www.sven.de/dpi/).

PPI o wartości 102 jest niewystarczające by oko ludzkie nie zbuntowało się przy czytaniu tekstu na
tego typu monitorze. Dlatego też jesteśmy zmuszeni do stosowania pewnych trików by oszukać nasze
oczyska.

## Antyaliasing oraz hinting

Jest kilka terminów, z którymi musimy się zaznajomić by umiejętnie podejść do konfiguracji czcionek
w systemie. Na początek zajmijmy się antyaliasingiem (anti-aliasing) oraz hintingiem (hinting).
(Część z poniższych grafik pochodzi [z tego serwisu](https://www.grc.com/ct/ctwhat.htm)).

Zwykle gdy widzimy jakiś tekst na ekranie, to nie zastanawiamy się nad tym co tak naprawdę widzimy.
W przypadku tekstu, może to być kilka pikseli zbitych w coś, co my postrzegamy jako
![](/img/2015/07/2.litera-a.gif> . Jest to duża litera `A` zapisana
czcionką Times New Roman. Nie jest ona zbyt wyraźna. Jeśli powiększymy ją sobie parokrotnie, tak by
zobaczyć układ pikseli, to naszym oczom ukaże się taki obrazek:

![litera-a-piksele](/img/2015/07/3.litera-a-piksele.gif#small)

Większość ludzi raczej nie chciała by oglądać tekstu na swoich monitorach w takiej formie. Dlatego
też ludzkość stworzyła kilka technik, które miały uporać się z prezentacją czcionek na naszych
ekranach.

Pierwszą z nich jest antyaliasing, który ma za zadanie wygładzać krawędzie czcionek, tak by nie były
zbytnio poszarpane. Ta technika polega na zaprzęgnięciu do pracy odcieni szarości w nadziei, że oko
ludzkie, widząc dwa przyległe szare piksele, dostrzeże jedynie ten po środku. Poniżej jest fotka tej
samej litery `A` , którą widzieliśmy wyżej ale z nałożonym antyaliasingiem:

![litera-a-antyaliasing](/img/2015/07/4.litera-a-antyaliasing.gif#small)

Problem z tego typu rozwiązaniem można zaobserwować w przypadku małych czcionek, gdzie takie
rozlanie pikseli powoduje, że tekst jest zwyczajnie nieczytelny. I tu w grę wchodzi hinting. Na
poniższej fotce ([źródło](https://pl.wikipedia.org/wiki/Hinting)), dolne wiersze są z hintingiem i,
jak możemy zauważyć, są wyraźnie ostrzejsze:

![konfiguracja-czionek-hinting](/img/2015/07/5.konfiguracja-czionek-hinting.png#small)

## Piksele i wygładzanie podpikselowe w monitorach LCD

Obecnie chyba każdy korzysta z monitorów LCD, a w nich to, co my zwykliśmy nazywać pikselem nie do
końca znajduje zastosowanie. Każdy piksel bowiem składa się z trzech podpikseli (subpixels):
czerwonego (R), zielonego (G) oraz niebieskiego (B). W taki oto sposób nasze oko widzi piksel, np.
tak: </img/2015/07/6.piksel.gif> , a w rzeczywistości jeśli byśmy
go powiększyli, dostrzeżemy coś takiego:
![](/img/2015/07/7.piksel-lcd-podpiksele.gif> . W przypadku gdybyśmy
potraktowali te podpiksele osobno, rozdzielczości pozioma w panelu LCD potroi się i będzie mieć 3072
piksele -- 1024 czerwonych, 1024 zielonych i 1024 niebieskich. Zatem jeśli nasz monitor ma poniżej
100 PPI, to czcionki przy wygładzaniu podpikselowym będą się łapać akurat na pułap około 300 PPI,
czyli istnieje spore prawdopodobieństwo, że oko ludzkie uda się oszukać przy pomocy tej techniki.

W czym taki podział pikseli może nam pomóc? Przede wszystkim, w rysowaniu obiektów (np. czcionek),
które mają dość postrzępione krawędzie. Popatrzmy na te poniższe obrazki. Załóżmy, że jest to
kawałek jakiejś literki. Grafika z lewej przedstawia zastosowanie całych pikseli, natomiast te dwie
z prawej obrazują podział piksela na podpiksele:

![krawedz-lcd](/img/2015/07/8.krawedz-lcd.gif#medium)

Jak widzimy krawędź jest o wiele bardziej wygładzona w przypadku rozbicia piksela na trzy
podpiksele. I choć te kawałki pikseli tuż przy krawędzi nie są do końca białe (bo nie zawierają
poszczególnych podpikseli), to nasze oko i tak postrzega je w taki sposób, a dzieje się to za sprawą
tych przyległych podpikseli. Ta technologia znajduje zastosowanie jedynie w przypadku wyświetlaczy
LCD. Jeśli teraz spojrzymy jeszcze raz na tę literkę `A` , to będzie ona się prezentować mniej
więcej w poniższy sposób, oczywiście przy założeniu, że nasz monitor korzysta z układu poziomego
RGB:

![litera-a-podpiksele](/img/2015/07/9.litera-a-podpiksele.gif#small)

Jako, że wygładzanie podpikselowe zależy głównie od fizycznej bliskości przyległych podpikseli,
algorytm renderujący musi znać ich kolejność, tj. czy mamy do czynienia z RGB, BGR, VRGB czy VBGR.
Jeśli wskażemy złą sekwencję podpikseli, jakoś obrazu ulegnie znacznemu pogorszeniu:

![litera-a-zly-uklad-podpikseli](/img/2015/07/10.litera-a-zly-uklad-podpikseli.gif#small)

W przypadku, gdy nie wiemy jaki układ podpikseli ma nasz monitor, zawsze możemy [przeprowadzić
test](http://www.lagom.nl/lcd-test/subpixel.php), który pomoże nam to określić.

Większość ludzi jest zdania, że jakość czcionek ulega znacznej poprawie. Mnie jednak szlag trafia
gdy muszę patrzeć na tekst, który wygląda mniej więcej tak:

![konfiguracja-czcionek-wygladzanie-podpikselowe](/img/2015/07/11.konfiguracja-czcionek-wygladzanie-podpikselowe.png#small)

To jest oczywiście dość duże powiększenie ale ta czcionka mieni się wszystkimi kolorami za wyjątkiem
czerni i mi się strasznie wzrok męczy od czytania tekstu napisanego czymś takim.

## Przestarzała konfiguracja fontconfig'a

Za to jak wyglądają czcionki w systemie linux'owym odpowiada narzędzie `fontconfig` i jest dostępne
w pakiecie pod tą samą nazwą. Raczej nie musimy go instalować, bo standardowa instalacja Debiana
już go powinna zawierać. Jeśli chodzi zaś o całą konfigurację fontconfig'a, to jest ona trochę
przytłaczająca. Przede wszystkim, w systemie jest cała masa miejsc, w których można określić to jak
mają być wyświetlane czcionki. Na przestrzeni lat fontconfig zmieniał nieco swoje katalogi i sporo z
nich jest już albo nieaktualnych, albo zostanie usuniętych w niedalekiej przyszłości, dlatego też
warto wiedzieć gdzie ewentualnie definiować wpisy konfiguracyjne dla czcionek.

Szereg plików konfiguracyjnych jest trzymanych w katalogu `/usr/share/fontconfig/conf.avail/` i to
do nich będziemy zwykle się odwoływać przy zmianie konfiguracji czcionek, a tego możemy dokonać dla
wszystkich użytkowników naraz lub też dla każdego z nich osobno. Jeśli interesują nas ustawienia
globalne, to zmian dokonujemy w katalogu `/etc/fonts/` . Natomiast jeśli chcemy ograniczyć się
jedynie do konkretnego konta użytkownika, to pod uwagę bierzemy katalog `~/.config/fontconfig/` .

Schemat konfiguracyjny w obu powyższych przypadkach jest dokładnie taki sam, tj. wewnątrz tych dwóch
katalogów ma być obecny folder `conf.d/` , w którym to będą przechowywane dowiązania symboliczne do
plików w katalogu `/usr/share/fontconfig/conf.avail/` .

Dodatkowo, mamy do dyspozycji plik `fonts.conf` , który jest zlokalizowany bezpośrednio w
`/etc/fonts/` lub `~/.config/fontconfig/` . Jeśli brakuje któregoś z nich, to można go utworzyć.
Zwykle jednak nie będzie to konieczne.

Jest jeszcze jeden plik, o którym warto wspomnieć, mianowicie `/etc/fonts/local.conf`. W przypadku
gdy będzie nas interesować systemowa konfiguracja czcionek, to lepiej nie ruszać pliku
`/etc/fonts/fonts.conf` , bo zostanie on nadpisany przy aktualizacji pakietu `fontconfig` . Wszelkie
lokalne zmiany wprowadzamy do pliku `local.conf` , choć raczej też nie będziemy musieli sobie nim
głowy zawracać.

## Konfiguracja czcionek w systemie

Jeśli korzystamy ze środowiska graficznego, np. GNOME, to jego ustawienia mogą nadpisywać te
poniższe i nie bądźmy zaskoczeni gdy wprowadzane przez nas zmiany nie odnoszą większego efektu. W
każdym razie jeśli mamy na pokładzie jedynie jakiś menadżer okien i nie odpowiada nam wygląd
czcionek, to możemy lekko go dostosować przy pomocy dowiązań symbolicznych do plików w katalogu
`/usr/share/fontconfig/conf.avail/` . Dla przykładu, załóżmy, że chcemy włączyć wygładzanie
podpikselowe. W tym celu wystarczy wydać poniższe polecenie:

    # ln -s /usr/share/fontconfig/conf.avail/10-sub-pixel-rgb.conf /etc/fonts/conf.d/

Powyżej jest przykład konfiguracji globalnej i jeśli o nią chodzi to mamy możliwość skorzystania z
narzędzia `dpkg-reconfigure` :

    # dpkg-reconfigure fontconfig-config

Po wpisaniu powyższego polecenia, zostanie nam wyświetlonych szereg opcji, które wstępnie
skonfigurują czcionki. Zostanie w ten sposób utworzonych szereg dowiązań i nie trzeba będzie tego
robić ręcznie. Trzeba jednak pamiętać, że nie wszystkie opcje idzie skonfigurować przy pomocy
powyższej linijki.

### Konfiguracja czcionek za pośrednictwem pliku fonts.conf

Jeśli nie chcemy bawić się w dowiązania symboliczne, możemy napisać własny plik `fonts.conf` i
umieścić w nim całą konfigurację. Trzeba pamiętać przy tym, że kolejność wpisów ma znaczenie.
Niemniej jednak, konfiguracja tych podstawowych rzeczy nie powinna nam przysporzyć problemów. Trzeba
również pamiętać, że konfiguracja globalna ma być umieszczana w pliku `/etc/fonts/local.conf` .

#### Konfiguracja antyaliasingu, hintingu oraz wygładzania podpikselowego

Na sam początek ustawmy sobie antyaliasing, hinting oraz wygładzanie podpikselowe:


	<!-- true, false -->
	<match target="font">
	<edit name="autohint" mode="assign"><bool>true</bool></edit>
	</match>

	<!-- true, false -->
	<match target="font">
	<edit name="hinting" mode="assign"><bool>true</bool></edit>
	</match>

	<!-- hintnone, hintslight, hintmedium, hintfull -->
	<match target="font">
	<edit name="hintstyle" mode="assign"><const>hintslight</const></edit>
	</match>

	<!-- unknown , rgb , bgr , vrgb , vbgr , none -->
	<match target="font">
	<edit name="rgba" mode="assign"><const>none</const></edit>
	</match>

	<!-- true, false -->
	<match target="font">
	<edit name="antialias" mode="assign"><bool>true</bool></edit>
	</match>

	<!-- lcdnone , lcddefault , lcdlight , lcdlegacy -->
	<match target="pattern">
	<edit mode="append" name="lcdfilter"><const>lcddefault</const></edit>
	</match>

Tych opcji oczywiście jest więcej ale na dobrą sprawę nam wystarczą tylko te powyższe. Jeśli ktoś ma
ochotę, to może zapoznać się z [dokumentacją
fontconfig'a](https://www.freedesktop.org/software/fontconfig/fontconfig-user.html) .

#### Ustawianie domyślnych fontów

Czcionki dzielimy z grubsza na trzy rodzaje: `sans-serif` , `serif` i `monospace`. Można się także
spotkać z `sans` , choć jest on już przestarzały i został zastąpiony przez `sans-serif` . Dla
przykładu, Arial robi za sans/sans-serif, Times New Roman za serif, a Courier New za monospace.
Dzięki tym aliasom jesteśmy w stanie znacznie ułatwić sobie konfigurację czcionek, bo nie musimy w
każdym programie osobno definiować nazw czcionek. Aplikacje domyślnie mają określone te trzy
powyższe typy, poniżej przykład:

![konfiguracja-czcionek-domyslnych-geany](/img/2015/07/12.konfiguracja-czcionek-domyslnych-geany.png#big)

Jeśli w tym przypadku zmienimy domyślne czcionki, ta powyższa aplikacja przy takiej konfiguracji
automatycznie uwzględni zmiany. Jedyne czego potrzebujemy to odpowiednio ustawić aliasy w
konfiguracji fontconfig'a.

By zdefiniować domyślne czcionki musimy do pliku `local.conf` dodać poniższy blok kodu:

	<!-- Generic name assignment -->
	<alias>
	<family>Times New Roman</family>
	<default><family>serif</family></default>
	</alias>
	<alias>
	<family>Arial</family>
	<default><family>sans-serif</family></default>
	</alias>
	<alias>
	<family>Courier New</family>
	<default><family>monospace</family></default>
	</alias>

	<!-- Generic name aliasing -->
	<alias>
	<family>serif</family>
	<prefer><family>Times New Roman</family></prefer>
	</alias>
	<alias>
	<family>sans-serif</family>
	<prefer><family>Arial</family></prefer>
	</alias>
	<alias>
	<family>monospace</family>
	<prefer><family>Courier New</family></prefer>
	</alias>

Jedyne co musimy dostosować do nazwy czcionek, które chcemy ustawić jako domyślne w systemie.

#### Małe czcionki

Jak wspomniałem wyżej, małe czcionki nie będą dobrze renderowane przy tej powyższej konfiguracji.
Możemy jednak wymusić by była ona stosowana jedynie do określonego rozmiaru czcionek i możemy to
zrobić dopisując do pliku `/etc/fonts/local.conf` poniższy kod:

	<!-- small fonts -->
	<match target="font">
	<test name="family" compare="eq" ignore-blanks="true"><string>Courier New</string></test>
	<test name="pixelsize" compare="less"><double>20.1</double></test>
	<edit name="hinting" mode="assign"><bool>true</bool></edit>
	<edit name="hintstyle" mode="assign"><const>hintfull</const></edit>
	<edit name="autohint" mode="assign"><bool>false</bool></edit>
	<edit name="antialias" mode="assign"><bool>false</bool></edit>
	</match>

	<match target="font">
	<test name="family" compare="eq" ignore-blanks="true"><string>Arial</string></test>
	<test name="pixelsize" compare="less"><double>20.1</double></test>
	<edit name="hinting" mode="assign"><bool>true</bool></edit>
	<edit name="hintstyle" mode="assign"><const>hintfull</const></edit>
	<edit name="autohint" mode="assign"><bool>false</bool></edit>
	<edit name="antialias" mode="assign"><bool>false</bool></edit>
	</match>

	<match target="font">
	<test name="family" compare="eq" ignore-blanks="true"><string>Times New Roman</string></test>
	<test name="pixelsize" compare="less"><double>20.1</double></test>
	<edit name="hinting" mode="assign"><bool>true</bool></edit>
	<edit name="hintstyle" mode="assign"><const>hintfull</const></edit>
	<edit name="autohint" mode="assign"><bool>false</bool></edit>
	<edit name="antialias" mode="assign"><bool>false</bool></edit>
	</match>

Powyższa konfiguracja włączy maksymalny hinting i wyłączy antyaliasing czyniąc tym samym możliwie
wyostrzone czcionki, których rozmiar nie przekracza 20 pikseli.

### Uwagi końcowe

Jeszcze kilka uwag na sam koniec. Przede wszystkim unikajmy stosowania autohintingu i wygładzania
podpikselowego jednocześnie. Na necie można wyczytać, że te dwie opcje kompletnie nie współgrają ze
sobą. Jest także możliwość zabronienia użytkownikom zmiany konfiguracji czcionek przez usunięcie
dowiązania `/etc/fonts/conf.d/50-user.conf` .
