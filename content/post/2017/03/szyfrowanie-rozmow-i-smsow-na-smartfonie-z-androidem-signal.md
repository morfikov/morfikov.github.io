---
author: Morfik
categories:
- Android
date:    2017-03-26 18:35:14 +0200
lastmod: 2017-03-26 18:35:14 +0200
published: true
status: publish
tags:
- szyfrowanie
- prywatność
- smartfon
- aplikacje
GHissueID: 53
title: Szyfrowanie rozmów i SMS'ów na smartfonie z Androidem (Signal)
---

Każdy z nas ma już raczej w swoim posiadaniu telefon, czy jego nieco bardziej zaawansowaną wersję
określaną mianem smartfona. Te urządzenia to w zasadzie przenośne i do tego bardzo małe komputery,
które umożliwiają nam komunikowanie się z osobami na całym świecie. Wykonywanie połączeń głosowych,
przesyłanie SMS'ów/MMS'ów czy też korzystanie z Internetu w naszych komórkach od dawna jest już
standardem i ciężko byłoby nam się obejść bez tej technologii obecnie. Problem w tym, że nasza
komunikacja jest narażona na podsłuch. W przypadku Internetu większość połączeń jest już szyfrowana
na linii dwóch klientów (E2E, End To End). Natomiast jeśli chodzi o telefony, to tutaj sprawa kuleje
i to bardzo poważnie, bo w zasadzie nasze połączenia głosowe czy SMS'y są do wglądu dla każdych
służb, które z jakiegoś powodu uznają, że mogą naruszać naszą prywatność. Jedyna opcja, która jest
w stanie zabezpieczyć nas przed tego typu praktykami, to szyfrowanie rozmów. Tak się składa, że jest
kilka aplikacji na Androida, które są w stanie realizować tego typu przedsięwzięcie. Jedną z nich
jest darmowa i otwartoźródłowa aplikacja Signal, której się przyjrzymy nieco bliżej w tym artykule.

<!--more-->
## Czym jest aplikacja Signal

Aplikacją Signal zainteresowałem się głównie dlatego, że, [jak można wyczytać na stronie
producenta][1], ma ona poprawiać w znacznym stopniu prywatność naszej korespondencji, którą
prowadzimy za pośrednictwem telefonu dzięki zastosowaniu [Signal Protocol][2].

Jeśli zamierzamy korzystać z aplikacji Signal, to musimy ją zainstalować na swoim smartfonie z
[Android][3]/[iOS][4]. Tej aplikacji jesteśmy też w stanie używać [na zwykłym desktopie][5],
z tym, że tutaj musimy mieć zainstalowaną przeglądarkę Google Chrome. Każdy kontakt, z którym
zamierzamy się w jakiś sposób komunikować, również musi zainstalować Signal. W przeciwnym razie nie
damy rady zestawić zaszyfrowanego połączenia.

Posiadanie odpowiedniego urządzenia i zainstalowanie aplikacji Signal to jednak nie wszystko. Na
dobrą sprawę, Signal jedynie imituje standardową aplikację telefonu w naszych smartfonach i cały
ruch, jaki generujemy za pomocą tej aplikacji, jest przesyłany tak na prawdę przez Internet.
Wymagane zatem jest posiadanie połączenia sieciowego w komórce, np. pakiety danych 3G/LTE lub WiFi.
Gdy wyjdziemy poza zasięg sieci bezprzewodowej, to nie będziemy w stanie połączyć się z żadnym z
kontaktów, które mamy podpięte pod Signal. Oczywiście w dalszym ciągu będzie działało połączenie
tradycyjne przez sieć GSM, które możemy wykorzystać do powiadomienia osoby, by przełączyła się ona
na Signal.

Jako, że Signal bazuje na połączeniu z globalną siecią Internet, to nie tylko wiadomości tekstowe
jesteśmy w stanie przesyłać za jego pomocą. Za pośrednictwem tego komunikatora jesteśmy w stanie
przesłać każdą informację, którą można zapisać w postaci binarnej. Innymi słowy, możemy nawiązywać
połączenia głosowe, video i przesyłać pliki dowolnego formatu. W zasadzie Signal jest to taki Skype
ale bardziej ficzerzasty i, co najważniejsze, [wolny i otwartoźródłowy][6]. Signal również eliminuje
bariery dystansowe, przez co nie musimy się martwić o koszty roamingu i bez problemu możemy
rozmawiać z osobami na całym świecie, o ile one i my mamy dostęp do Internetu.

## Przygotowanie aplikacji Signal do pracy

Wiemy zatem, że do zestawienia kanału komunikacyjnego między dwoma kontaktami potrzebne nam są dwa
urządzenia i zainstalowana aplikacja Signal na każdym z nich. Ja dysponuję jedynie smartfonami
Neffos z system Android 5.1 (Lollipop) i 6.0 (Marshmallow) i na tych dwóch systemach ta aplikacja
działa bez większego problemu:

![szyfrowanie-rozmow-sms-smartfon-signal-instalacja-aplikacja](/img/2017/03/001.szyfrowanie-rozmow-sms-smartfon-signal-instalacja-aplikacja.png#huge)

Warto w tym miejscu zaznaczyć, że [aplikacja Signal może zostać zainstalowana spoza sklepu Google
Play][7] i działać nawet, gdy usługi Google w naszym smartfonie są wyłączone lub w ogóle nie są
zainstalowane. Można zatem używać Signal bez Google, co bardzo cieszy.

Po zainstalowaniu aplikacji Signal musimy z nią powiązać nasz numer telefonu, tj. zarejestrować go
ale ta rejestracja nie jest rejestracją rozumianą w znaczeniu potocznym. Po prostu numer telefonu
unikalnie identyfikuje dany kontakt i można ten fakt wykorzystać. Za każdym razem, gdy odinstalujemy
aplikację Signal, krok rejestracji numeru (nawet tego samego) trzeba będzie powtórzyć. Dzieje się
tak dlatego, że przy rejestracji numeru są tworzone klucze szyfrujące wykorzystywane do
zabezpieczenia komunikacji. Niektóre z kluczy wykorzystywanych w Singal są generowane czasowo, np.
do zaszyfrowania pojedynczej wiadomości, którą komuś przesyłamy. Natomiast jest też generowany jeden
klucz, który jest przypisany do naszego urządzenia. Jeśli teraz reinstalujemy aplikację Signal, to
ten klucz jest usuwany i trzeba go wygenerować jeszcze raz.

![szyfrowanie-rozmow-sms-smartfon-signal-rejestrowanie-numer](/img/2017/03/002.szyfrowanie-rozmow-sms-smartfon-signal-rejestrowanie-numer.png#huge)

Po zarejestrowaniu numeru, dobrze jest rzucić jeszcze okiem na opcje w ustawieniach. W szczególności
zwróćmy uwagę na pozycję dotyczącą domyślnej aplikacji SMS, ochrony ekranu oraz potwierdzeń
zmienionych kluczy naszych kontaktów.

![szyfrowanie-rozmow-sms-smartfon-signal-opcje](/img/2017/03/003.szyfrowanie-rozmow-sms-smartfon-signal-opcje.png#medium)

![szyfrowanie-rozmow-sms-smartfon-signal-opcje](/img/2017/03/004.szyfrowanie-rozmow-sms-smartfon-signal-opcje.png#huge)

![szyfrowanie-rozmow-sms-smartfon-signal-opcje](/img/2017/03/005.szyfrowanie-rozmow-sms-smartfon-signal-opcje.png#huge)

## Czy Signal obsługuje smartfony bez numeru

Numer telefonu, który podamy podczas rejestracji, będzie przypisany do aplikacji Signal i ten numer
musi być prawidłowy z możliwością odbierania połączeń. Na niego dostaniemy wiadomość SMS z kodem
potwierdzającym. Tylko jeden numer może być obsługiwany przez aplikację Signal w danym czasie, nawet
jeśli nasz telefon posiada dual czy triple SIM.

Może i do rejestracji jest wykorzystywany działający numer telefonu ale testując kilka ciekawych
rzeczy udało mi się ustalić, że karta SIM z tym numerem nie musi być obecna w telefonie po
zarejestrowaniu go w aplikacji. Możemy zatem korzystać z dwóch fizycznych smartfonów i przez jeden z
nich będziemy się komunikować za pomocą aplikacji Signal, a drugi telefon posłuży nam do
tradycyjnych rozmów i SMS'ów.

Takie rozdzielenie ról może poprawić naszą prywatność, bo operator GSM w przypadku braku karty SIM
może nie być w stanie ustalić położenia telefonu. Trzeba tylko pamiętać by wyłączyć moduł GSM, bo
ten nawet w przypadku braku karty SIM w telefonie i tak umożliwia dzwonienie na numery alarmowe. Z
kolei by móc je wykonywać, to z jakimś operatorem GSM jesteśmy zawsze połączeni. Moduł GSM można
wyłączyć aktywując [Tryb Samolotowy][8].

## Aplikacja Signal i Aero2

Posiadacze kart SIM od Aero2 są w stanie wykorzystać transfer oferowany w usłudze darmowego
Internetu tego operatora. Może transfer nie przekracza 512 kbit/s (64 KiB/s) ale z tego co
zauważyłem, to nawet taki słaby transfer nie stanowi większego wyzwania dla aplikacji Signal.
Jesteśmy w stanie bez problemu rozmawiać głosowo oraz też przesyłać obraz video.

Szacunkowo, transfer jaki jest zjadany przez aplikację Signal przy połączeniu video w czasie jednej
minuty, to około 1,5 MiB do 2 MiB. Jest to zatem około 300 kbit/s (nieco ponad połowa prędkości
oferowanej przez Aero2.

Jako, że na kartach SIM od Aero2 nie mamy możliwości odbierania SMS'ów, to nie damy rady numeru tej
karty zarejestrować w aplikacji Signal. Nie powinno to stanowić problemu, bo zawsze możemy sobie
zakupić jakiegoś prepaida, by tylko taki numer otrzymać, a dalej możemy już jechać na Aero2.

Jedyny problem przy korzystaniu z aplikacji Signal w przypadku Aero2, to naturalnie kod CAPTCHA,
który trzeba wpisywać co godzinę. Nawet jeśli nie będziemy tego robić regularnie, to i tak zawsze
możemy ten kod wpisać tuż przed nawiązaniem połączenia. Problematyczne za to może być odbieranie
SMS'ów i rozmów, bo gdy ponownie trzeba będzie ten kod CAPTCHA wpisać, to do momentu aż to uczynimy
będziemy offline i nikt z nami nie będzie w stanie się skontaktować. Zawsze też możemy wykupić sobie
pakiet 3 GiB za 5 zł na miesiąc i zapomnieć o CAPTCHA.

## Weryfikacja kontaktów w aplikacji Signal

Wysyłając w aplikacji Signal jakąś wiadomość tekstową, ta zostanie uprzednio zaszyfrowana i
przesłana do drugiej osoby. Skąd jednak wiemy, że ta druga strona w ogóle otrzymała daną wiadomość
oraz, że tylko ona ją odebrała? Każdy przecież może taką rozmowę przechwycić i podszyć się pod nasz
kontakt. Signal ma naturalnie wbudowany mechanizm weryfikacji kontaktu, tj. czy numer, który my
widzimy na ekranie naszego telefonu pasuje do tego, którym dysponuje nasz kontakt. Ten numer możemy
zweryfikować ręcznie lub przez zeskanowanie kodu QR, który zostanie wyświetlony na ekranie telefonu
tej drugiej osoby. Przykładowa weryfikacja numeru bezpieczeństwa wygląda mniej więcej tak:

![szyfrowanie-rozmow-sms-smartfon-signal-odcisk-palca-klucza](/img/2017/03/006.szyfrowanie-rozmow-sms-smartfon-signal-odcisk-palca-klucza.png#huge)

![szyfrowanie-rozmow-sms-smartfon-signal-skanowanie-kod-qr](/img/2017/03/007.szyfrowanie-rozmow-sms-smartfon-signal-skanowanie-kod-qr.png#huge)

By przeprowadzić proces weryfikacji, trzeba te wszystkie wyżej wygenerowane cyferki sprawdzić pod
kątem poprawności na drugim urządzeniu. Skanowanie kodu QR jest automatyczne i o wiele szybsze i
bardziej dokładne. Jeśli weryfikacja zakończy się powodzeniem, to znaczy, że komunikacja jest
należycie zabezpieczona i nikt nie jest w stanie jej podsłuchać.

Jeśli nasz rozmówca w późniejszym czasie przeinstaluje aplikację Signal, to nam zostanie wyświetlony
komunikat, że ten kontakt dysponuje innym kluczem szyfrującym, bo ten przecież został na nowo
wygenerowany podczas ponownej instalacji programu. Tutaj trzeba jednak uważać, bo informacja o
zmianie klucza może również oznaczać, że ktoś między nami i naszym kontaktem próbuje przechwycić
rozmowę:

![szyfrowanie-rozmow-sms-smartfon-signal-zmiana-klucza](/img/2017/03/008.szyfrowanie-rozmow-sms-smartfon-signal-zmiana-klucza.png#huge)

W takim przypadku, niezbędna będzie weryfikacja numeru bezpieczeństwa w sposób opisany powyżej. Bez
weryfikacji nie będziemy w stanie wymieniać wiadomości. Można oczywiście tę weryfikację pominąć ale
wtedy możemy narazić się na podsłuch. Jak tylko zweryfikujemy numer, to przy wiadomości pojawi się
znaczek podwójnego ptaszka oznaczającego, że wiadomość dodarła do odbiorcy oraz, że została ona z
powodzeniem odczytana.

Nie zawsze jednak pojawią się dwa ptaszki. Może też pojawić się tylko jeden. Jeden ptaszek oznacza,
że wiadomość została poprawnie zaszyfrowana i wysłana do odbiorcy ale z racji, że nasz kontakt może
być offline, to może się zdarzyć tak, że wiadomość nie została do niego dostarczona natychmiast. Jak
tylko nasz kontakt podłączy się do Internetu, to ta wiadomość zostanie mu przekazana przez serwer.
Po jej dostarczeniu i odczytaniu przez odbiorce, w logu rozmowy powinien pojawić się nam drugi
ptaszek.

![szyfrowanie-rozmow-sms-smartfon-signal-potwierdzenie-wiadomosc](/img/2017/03/009.szyfrowanie-rozmow-sms-smartfon-signal-potwierdzenie-wiadomosc.png#huge)

## Wykonywanie szyfrowanych rozmów głosowych

Osoby, które chciałyby wykonać szyfrowaną rozmowę głosową, mogą ją zainicjować bez większego trudu.
Po przejściu do historii wiadomości danego kontaktu, w prawym górnym rogu jest ikonka słuchawki z
kłódką. Po jej przyciśnięciu rozpocznie się dzwonienie do tej osoby:

![szyfrowanie-rozmow-sms-smartfon-signal-polaczenie-glosowe](/img/2017/03/010.szyfrowanie-rozmow-sms-smartfon-signal-polaczenie-glosowe.png#huge)

Jeśli kontakt będzie offline, to cały czas na ekranie będzie widniał zapis "WYBIERANIE". Po jakiejś
minucie od rozpoczęcia procesu dzwonienia i nie odebraniu połączenia przez drugą ze stron, proces
ten zostanie automatycznie przerwany. W przypadku, gdy kontakt będzie online, to "WYBIERANIE"
przejdzie w stan "DZWONI". A tak wygląda odbieranie rozmowy po drugiej stronie:

![szyfrowanie-rozmow-sms-smartfon-signal-polaczenie-glosowe](/img/2017/03/011.szyfrowanie-rozmow-sms-smartfon-signal-polaczenie-glosowe.png#huge)

Standardowo mamy aktywną tylko funkcję rozmowy głosowej ale nic nie stoi na przeszkodzie by włączyć
kamerę selfie i zacząć przesyłać obraz i dźwięk jednocześnie.

## Szyfrowane archiwum wiadomości

Wiadomości oraz wszystkie załączniki, które w nich zamieścimy, np. zdjęcia czy filmy, są
przechowywane w lokalnej bazie danych na obu końcach połączenia, tj. u nas i u naszego kontaktu.
Nawet jeśli nasz telefon oferuje szyfrowanie partycji `/data/` , to i tak przydałoby się zaszyfrować
niezależnie wszystkie konwersacje w aplikacji Signal.

By tego typu szyfrowanie aktywować, musimy ustawić hasło w aplikacji. To hasło trzeba będzie
wpisywać za każdym razem, gdy będziemy chcieli uzyskać dostęp do listy kontaktów. Naturalnie można
ustawić czas przechowywania hasła w pamięci RAM, co sprawi, że przez pewien czas hasło nie będzie
wymagane.

Zaszyfrowaniu ulegnie też klucz użytkownika wykorzystywany do deszyfrowania wiadomości. Niemniej
jednak, część informacji pozostanie niezaszyfrowana, np. lista kontaktów oraz czas przesyłanych
wiadomości.

![szyfrowanie-rozmow-sms-smartfon-signal-haslo](/img/2017/03/012.szyfrowanie-rozmow-sms-smartfon-signal-haslo.png#medium)

Bezpieczeństwo bazy danych będzie zależeć od stanu skomplikowania hasła, które sobie wybierzemy.
Dlatego też jeśli decydujemy się na zaszyfrowanie bazy, to podejdźmy do tego zagadnienia należycie i
stosujmy długie hasła.

Gdy baza zostanie zaszyfrowana, to w dalszym ciągu jesteśmy w stanie odbierać wiadomości ale nie
możemy ich odczytać do momentu podania hasła. Rozmowy głosowe natomiast można bez problemu
odbierać.

## Automatyczne kasowanie wiadomości

Przechowywanie wiadomości przez zbyt długi okres czasu może nie być zalecane, zwłaszcza jeśli nie
powinien ich nikt zobaczyć. Zwykle bardzo rzadko kasujemy wiadomości ręcznie i dlatego w aplikacji
Signal możemy ustawić, by wiadomości z bazy były usuwane automatycznie, gdy licznik przekroczy,
np. 100. Tego typu kasowanie wiadomości dotyczy jednak tylko lokalnej bazy danych. Przesłane
wiadomości będą przechowywane zgodnie z polityką, którą wybrał nasz kontakt. Może się zatem zdarzyć
tak, że znaszego archiwum dana wiadomość zniknęła ale w archiwum naszego kontaktu w dalszym ciągu
ona figuruje.

Mając na uwadze powyższe, w aplikacji Signal został zaimplementowany ciekawy mechanizm ustawiania
czasu ważności dla wiadomości. W taki sposób jedna ze stron może ustawić, np. czas na 5 minut i po
upłynięciu tego okresu, ta wiadomość zostanie automatycznie skasowana zarówno u tej osoby jak i u
jej rozmówcy. Trzeba tylko zdawać sobie sprawę, że ten czas jest liczony od momentu wyświetlenia się
wiadomości na ekranie. U nas ta wiadomość się pojawia w zasadzie natychmiast ale u naszego kontaktu
może się ona nie wyświetlić przez dni czy tygodnie, jeśli użytkownik był offline przez dłuższy czas.

W każdym razie, ten timeout można ustawić wybierając opcję "Znikające wiadomości" z menu kontaktu.
Po wybraniu okresu czasu, obok numeru pojawi się zegar ze stosowną informacją:

![szyfrowanie-rozmow-sms-smartfon-signal-timeout](/img/2017/03/013.szyfrowanie-rozmow-sms-smartfon-signal-timeout.png#huge)

Wszystkie wiadomości które zostaną napisane przy widocznym zegarze znikną automatycznie po czasie,
który wskazuje zegar. Po wysyłaniu wiadomości, każda z nich będzie miała ikonkę animowanej
klepsydry, która jest w stanie wizualnie odliczać pozostały czas.

Ten czas zostanie również ustawiony automatycznie na drugim końcu połączenia.

![szyfrowanie-rozmow-sms-smartfon-signal-timeout](/img/2017/03/014.szyfrowanie-rozmow-sms-smartfon-signal-timeout.png#medium)

## Konfiguracja diody powiadomień

Ciekawą opcją jest też konfiguracja diody powiadomień. W przypadku standardowych SMS'ów czy
nieodebranych połączeń nie mamy zwykle możliwości konfiguracji tej diody, a konkretnie chodzi o
sposób (częstotliwość) jej migania. Dodatkowo jeśli nasz smartfon posiada wielokolorową diodę, to
możemy także wybrać sobie i kolor.

![szyfrowanie-rozmow-sms-smartfon-signal-dioda](/img/2017/03/015.szyfrowanie-rozmow-sms-smartfon-signal-dioda.png#huge)

## Ochrona ekranu przed robieniem zrzutów

Co ciekawe, gdy aplikacja Signal jest na pierwszym planie, nie działa możliwość robienia zrzutów
ekranu. W sumie jest to ciekawa opcja. Jakby nie patrzeć może i nasze wiadomości są przesyłane i
przechowywane w formie zaszyfrowanej ale można by spróbować je wydobyć robiąc właśnie zrzut ekranu.
Tę opcję ochrony można naturalnie wyłączyć w ustawieniach samej aplikacji ale nie zalecałbym tego
robić. Mając włączoną ochronę ekranu, żadna aplikacja ani nawet skrót przycisków Power + VolDown nie
będą w stanie zrobić zrzutu ekranu. Oczywiście po zminimalizowaniu czy zamknięciu aplikacji Signal,
skrin będzie można już wykonać bez większego problemu.

## Dla kogo jest przeznaczona aplikacja Signal

Gdy pierwszy raz usłyszałem o aplikacji Signal, to od razu wiedziałem, że jest to jeden z tych
programików, które zagoszczą w moim smartfonie na stałe. Nie chodzi tutaj o to, że moje rozmowy ze
Snowden'em czy szefem NSA są jakoś ściśle tajne i nie powinny ujrzeć światła dziennego ale moim
zdaniem sieć GSM to przeżytek.

Popatrzmy jak obecnie wygląda komunikacja przez Internet. Możemy rozmawiać w bezpieczny sposób
szyfrując połączenie między dwoma punktami i praktycznie nie ponosimy z tego tytułu żadnych opłat,
tj. ani nie płacimy za szyfrowanie, ani też za czas połączenia bez znaczenia przy tym gdzie znajduje
się nasz rozmówca. A jak wygląda sprawa w przypadku sieci GSM (pomińmy już kwestię braku
przyzwoitych zabezpieczeń E2E)? Spróbujmy pogadać przez telefon z kimś z US czy Rosji. Ile nas
wyniesie rachunek za tego typu konwersację? Nawet te paru minutowe rozmowy mogą kosztować fortunę.

Teraz, gdy upowszechnieniu uległy sieci bezprzewodowe WiFi (min. darmowe hotspoty) i LTE (choć ten
zależy od operatora GSM), to nasze komórki mogą być praktycznie non stop podłączone do Internetu.
Nawet jeśli nie mamy w okolicy żadnego punktu WiFi czy BTS'a z LTE, to w dalszym ciągu możemy
korzystać z 3G w postaci darmowego połączenia od Aero2 (gdyby tylko zlikwidowali ten wredny kod
CAPTCHA) i koszt telefonu (rozmowy głosowe, SMS i MMS) zostaje zredukowany praktycznie do 0 i to
przy jednoczesnym wzroście prywatności użytkowników. Jeśli zależy nam na prywatności, to już dziś
powinniśmy sobie tę aplikację Signal zainstalować i zacząć namawiać do tego swoich znajomych.


[1]: https://whispersystems.org/
[2]: https://en.wikipedia.org/wiki/Signal_Protocol
[3]: https://play.google.com/store/apps/details?id=org.thoughtcrime.securesms
[4]: https://itunes.apple.com/us/app/signal-private-messenger/id874139669
[5]: https://chrome.google.com/webstore/detail/signal-private-messenger/bikioccmkafdpakkkcpdbppfkghcmihk
[6]: https://github.com/WhisperSystems/Signal-Android/
[7]: https://signal.org/android/apk/
[8]: https://pl.wikipedia.org/wiki/Tryb_samolotowy
