---
author: Morfik
categories:
- Linux
date: "2016-02-08T00:45:07Z"
date_gmt: 2016-02-07 23:45:07 +0100
published: true
status: publish
tags:
- hdd
- ssd
- sieć
title: Backup dysku przez sieć przy pomocy dd i netcat
---

Dyski talerzowe mają to do siebie, że zawierają elementy mechaniczne, np. ramię głowicy czy też sam
napęd dysku. Te ruchome elementy się zużywają podczas eksploatacji dysku i trzeba mieć na uwadze, że
prędzej czy później taki dysk ulegnie awarii. Statystycznie rzecz biorąc, około 5% dysków rocznie
zdycha. Oczywiście to tylko statystyka i w sporej części przypadków dyski twarde ulegają awarii
znacznie wcześniej. Niekoniecznie musimy mieć tutaj do czynienia z [planowanym postarzaniem
sprzętu](https://pl.wikipedia.org/wiki/Planowane_starzenie) i zwyczajnie możemy trafić na trefny
model, którego wada fabryczna wyjdzie po 2-3 miesiącach użytkowania. Poza tym, producenci dysków
implementują w nich te energooszczędne rozwiązania, które znacznie skracają żywotność nośników.
Można o tym przekonać się analizując [193 parametr SMART (Load/Unload Cycle) odpowiadający za
parkowanie głowicy](/post/parkowanie-glowicy-w-dyskach-wstern-digital/) w dyskach
firmy Western Digital. Także na dobrą sprawę nie możemy być pewni kiedy nam ten dysk zwyczajnie
odmówi posłuszeństwa. Dlatego też powinniśmy się zabezpieczyć na taką ewentualność robiąc kopię
bezpieczeństwa (backup) danych zawartych na dysku. W tym wpisie postaramy się zrobić kompletny obraz
dysku laptopa przy pomocy narzędzi `dd` i `nc` (netcat). Nie będziemy przy tym rozkręcać urządzenia
czy też podłączać do portu USB zewnętrznego nośnika. Dane prześlemy zwyczajnie przez sieć.

<!--more-->
## Dlaczego backup będziemy robić za pomocą dd i nc

By zrobić backup sporej ilości danych przez sieć, niekoniecznie musimy korzystać z pomocy `dd` i
`nc`. Moglibyśmy do tego celu zaprzęgnąć SSHFS albo też `rsync` za pośrednictwem SSH, NFS czy SMB.
Moglibyśmy też skorzystać z protokołu FTP,
[SFTP](https://pl.wikipedia.org/wiki/SSH_File_Transfer_Protocol), lub
[FTPS](https://pl.wikipedia.org/wiki/FTPS). Na dobrą sprawę, to nie ma jednego rozwiązania, a to,
którą opcję wybierzemy zależeć będzie od konkretnego przypadku.

Przede wszystkim, jeśli robimy backup w zaufanej sieci, to prawdopodobnie nie potrzebujemy nawet
szyfrować danych, których kopię chcemy wykonać. Jeśli w takim przypadku zaprzęgniemy do roboty SSH,
to dane przed wysłaniem będzie trzeba zaszyfrować na jednej maszynie, no i też trzeba będzie je
odszyfrować na drugim komputerze. To pociąga za sobą znaczą utylizację procesora.

Możemy naturalnie skorzystać z `rsync` . Trzeba jednak mieć na uwadze fakt, że wykonanie backup'u za
jego pomocą może być dramatycznie wolne w przypadku NFS. Chodzi o to, że tutaj `rsync` musi [wykonać
szereg wywołań systemowych](http://cialug.org/pipermail/cialug/2010-August/017416.html). Na każde
takie wywołanie musi być udzielona odpowiedź od zdalnego hosta. Te zapytania nie są wysyłane
zbiorczo i odpowiedź na każde z nich musi przyjść osobno. To z kolei zajmuje cenny czas równy RTT
(droga pakietu w obie strony). Wyobraźmy sobie teraz, że przesyłamy 100K plików o średnim rozmiarze
paru KiB. Nie wygląda to zachęcająco, prawda? Trochę lepiej wygląda sprawa, gdy zrezygnujemy z NFS
na rzecz SSH. W tym przypadku wywołania systemowe są dokonywane lokalnie na każdej z maszyn, a przez
sieć wędrują jedyne dane pliku. Odpada nam, co prawda, czas potrzebny na przesłanie odpowiedzi na
wywołania systemowe ale z kolei dochodzi overhead związany z szyfrowaniem.

Przesyłanie plików za pomocą protokołu FTP nie pociąga za sobą problemu wynikającego z szyfrowania
danych. Jako, że dane można przechwycić, to nie powinniśmy używać tej metody do przesyłania plików
przez internet. Podstawowy problem z protokołem FTP jest taki, że nie respektuje on uprawnień do
plików i zwyczajnie przepisuje je. Nie nadaje się więc on do robienia backup'u plików systemowych.

Problematyczne może być także pełne szyfrowanie dysku. Tutaj sytuacja wygląda mniej więcej tak samo
jak w przypadku robienia kopi przez SSH. No chyba, że ślemy ten backup przez internet przez SSH.
Wtedy dane są deszyfrowane w celu ich odczytania z dysku. Następnie szyfrowane przed przesłaniem ich
przez sieć. Na drugim końcu połączenia dane są deszyfrowane po odebraniu, no i szyfrowane przed
zapisem na dysk. Jest to bardzo nieefektywne zachowanie i sporą cześć zasobów zjadają operacje
szyfrowania i deszyfrowania danych.

W przypadku, gdy robimy binarną kopię dysku, to narzędzie `dd` pomija zupełnie system plików i nie
interesuje go to czy dane znajdujące się na tym nośniku trzeba odszyfrować po odczycie albo
zaszyfrować przed zapisem. Bit to bit i dlatego właśnie będziemy korzystać z `dd` i `nc` . To
rozwiązanie oferuje niewielkie zużycie procesora, a ogranicza nas zwykle przepustowość sieci.

## Tworzenie i przesyłanie backup'u przez sieć

Procedura backup'u nie jest zbytnio skomplikowana. Niemniej jednak, musimy zdać sobie sprawę, że
nieodpowiednie posługiwanie się takimi narzędziami jak `dd` czy `nc` może nam przynieść tylko więcej
szkody niż pożytku. Prawdopodobnie będziemy musieli doinstalować na obu maszynach netcat'a. Jest on
dostępny standardowo w repozytorium debiana w pakiecie `netcat-traditional` .

Jako, że będziemy tworzyć kopię binarną dysku znajdującego się w laptopie, to mamy do wyboru z
grubsza trzy opcje. Możemy skopiować cały dysk w jednym podejściu i zapisać go w pliku na dysku NAS.
Zamiast całego dysku, możemy też skopiować poszczególne partycje i również je zapisać w osobnych
plikach na dysku NAS. Możemy także na tym dysku NAS stworzyć partycję, której wielkość odpowiada
rozmiarowi backup'owanej partycji i to tam bezpośrednio wgrać kopiowany obraz. My zdecydujemy się na
skorzystanie z tej ostatniej opcji. Głównie dlatego, że do danych tak utworzonego backup'u będziemy
mieli prostszy dostęp.

Na docelowej maszynie, tj. tam gdzie obraz dysku ma zostać zapisany, uruchamiamy demona `nc` :

    # nc -l -s 192.168.1.166 -p 19000 | dd bs=32M of=/dev/sda5

Opcja `-l` odpowiada za nasłuch na porcie określonym przez `-p` . Z kolei parametr `-s` to adres, na
którym netcat ma nasłuchiwać. Dalej mamy pipe ( `|` ), za pomocą którego wszystkie dane otrzymane
przez netcat zostaną przesłane do `dd` , a ten zacznie zapisywać piątą partycję głównego dysku.

Mając uruchomionego demona `nc` , na laptopie musimy jeszcze rozpocząć transfer danych. Robimy to
wpisując do terminala to poniższe polecenie:

    # dd bs=32M if=/dev/sda6 | pv -s 142G | nc 192.168.1.166 19000

Tutaj z kolei rozpoczynamy od `dd` , który czyta szóstą partycję dysku. Następie strumień danych
przesyła do `pv` , dzięki któremu będziemy widzieć postęp przesyłu. Następnie dane trafiają już do
`nc` , który przesyła je na określony adres i port, gdzie powinien nasłuchiwać demon `nc` . Po
wklepaniu tego polecenia, transfer danych zostanie rozpoczęty.

W przypadku dysków sporych rozmiarów, ten powyższy sposób nie jest zbytnio praktyczny. Raczej nikomu
nie chciałoby się przesyłać kilku TiB danych za każdym razem, gdy ten ktoś chciałby dokonać
backup'u. Dlatego też dobrze jest rozważyć dokonać takiej wstępnej synchronizacji partycji wyżej
opisanym sposobem, a później już tylko jechać na `rsync` .
