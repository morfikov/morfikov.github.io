---
author: Morfik
categories:
- OpenWRT
date: "2016-06-26T19:15:01Z"
date_gmt: 2016-06-26 17:15:01 +0200
published: true
status: publish
tags:
- chaos-calmer
- sieć
- nfs
- router
GHissueID: 349
title: Sieciowy system plików w OpenWRT (NFS)
---

[Network File System][1]) to sieciowy system plików, za pomocą którego maszyny mające na pokładzie
system operacyjny linux, w tym tez OpenWRT, są w stanie udostępniać pliki w sieci. Zatem NFS to
głównie domena linux'ów. W przypadku windowsów można korzystać z protokołu SMB ([samba][2]). Sposób
udostępniania zasobów przy pomocy tego sieciowego systemu plików jest bardzo podobny do tego, który
jest realizowany w przypadku protokołu SSHFS. Zasadniczą różnicą między NFS i SSHFS jest brak
szyfrowania komunikacji. W warunkach domowej sieci, ta cecha raczej nie stanowi większego problemu.
Poza tym, trzeba też brać pod uwagę fakt, że szyfrowanie znacznie obciążyłoby router, co
przełożyłoby się na spadek prędkości transferu. W tym wpisie zobaczymy jak na routerze z OpenWRT
zaimplementować protokół NFS i udostępnić za jego pomocą zasoby w sieci lokalnej.

<!--more-->
## Instalacja pakietów na potrzeby NFS

Praktycznie każda funkcjonalność, którą zamierzamy wdrożyć na routerze, wymaga od nas doinstalowania
kilku pakietów. W tym przypadku również musimy wyskrobać nieco pamięci na flash routera. Generalnie
interesuje nas pakiet `nfs-kernel-server` . Pociąga on jednak za sobą szereg zależności i całość
waży około 500-600 KiB. Jeśli mamy wymaganą ilość miejsca, to instalujemy wspomniany pakiet:

    # opkg update
    # opkg install nfs-kernel-server

Na końcu procesu instalacyjnego zostanie wyrzuconych kilka
    komunikatów:

    exportfs: /etc/exports [1]: Neither 'subtree_check' or 'no_subtree_check' specified for export "*:/mnt".
      Assuming default behaviour ('no_subtree_check').
      NOTE: this default has changed since nfs-utils version 1.0.x

    exportfs: /mnt does not support NFS export

Na razie nie musimy się nimi zbytnio przejmować ale w drodze konfiguracji, te powyższe wiadomości
zostaną zlikwidowane.

NFS wykorzystuje do swojego działania narzędzie `portmap` (jest pociągany w zależnościach
automatycznie). Oba demony ( `nfsd` oraz `portmap` ) powinny być dodane już do autostartu. Dla
pewności możemy zajrzeć do katalogu `/etc/rc.d/` . Dodatkowo, te w/w demony powinny także być już
odpalone i nasłuchiwać zapytań, co możemy sprawdzić przy pomocy `ps` . W tej chwili mamy działający
serwer NFS na routerze ale wymaga on jeszcze konfiguracji zasobów, które chcemy udostępnić.

## Konfiguracja zasobów NFS udostępnianych w sieci

Zasoby udostępniane w sieci za sprawą protokołu NFS trzeba wyraźnie określić w pliku
`/etc/exports` . Podajemy tam zwyczajnie ścieżki do katalogów, które chcemy udostępnić. Pamiętajmy,
że w miejscu konkretnego katalogu możemy zamontować partycję dysku twardego. Wpisy w pliku
`/etc/exports` określamy w poniższy sposób:

    /mnt/sda3   192.168.1.0/24(rw,no_subtree_check,all_squash,anongid=1000,anonuid=1000)

Na samym początku mamy ścieżkę do zasobu. W tym przypadku jest to katalog `/mnt/sda3` . Pamiętajmy,
że katalog musi istnieć. Dalej mamy adres IP, a właściwie całą sieć `192.168.1.0/24` , z której
maszyny będą w stanie uzyskać dostęp do tego katalogu. Wewnątrz nawiasu z kolei mamy opcje, w
oparciu o które udostępniany jest dany katalog.

Opcja `rw` oznacza, że dany katalog po zamontowaniu na kliencie może być zapisywany. Zamiast `rw`
można określić `ro` , za sprawą którego zasób zostanie udostępniony w trybie tylko do odczytu.
Można także sprecyzować opcję `sync` , z tym, że czasem transfer może znacznie na tym ucierpieć.
Dalej mamy `no_subtree_check` , który wyłącza mechanizm `subtree_check` w przypadku eksportowania
jedynie jakiegoś podkatalogu konkretnego systemu plików. Generalnie rzecz biorąc, zasoby
przeznaczone głównie do odczytu powinny ten mechanizm mieć włączony. W przypadku innych zasobów
lepiej jest go wyłączyć, bo powoduje on spore problemy przy przepisywaniu nazw plików. Z kolei opcje
`all_squash` , `anongid=1000` oraz `anonuid=1000` odpowiadają za określenie numerków UID/GID
użytkownika anonimowego. Jeśli system plików został zamontowany z uwzględnieniem tych 3 opcji,
wszystkie nowo utworzone przez klienta pliki/katalogi będą miały odpowiednio dostosowany UID/GID na
routerze. W systemach linux, ID=1000 oznacza pierwszego zwykłego użytkownika, który został utworzony
przy instalacji systemu operacyjnego. Zatem jeśli pliki na routerze zostaną przemapowane na numerki
1000/1000, użytkownik na kliencie będzie miał do nich swobodny dostęp.

Zmiany w pliku `/etc/exports` nie są natychmiastowe. Po edycji tego pliku, ręcznie trzeba
wyeksportować nowe zasoby. Robimy to przy pomocy narzędzia `exportfs` :

    # exportfs -ar

To powyższe polecenie przepisze plik `/var/lib/nfs/etab` , w oparciu o który tak naprawdę są
podejmowane decyzje na temat obsługi udostępnianych w sieci zasobów.

## Montowanie zasobów NFS na kliencie

By zamontować zasób NFS na kliencie, potrzebne są standardowo prawa użytkownika root. Możemy
naturalnie korzystać z całej masy mechanizmów, które upraszczają montowanie zasobów sieciowych.
Przykładem może być [udevil][3]. Niżej są dwa polecenia, które zamontują nam zasoby routera na
stacji klienckiej:

    # mount -t nfs 192.168.1.1:/mnt/sda3 /media/sda3/
    $ udevil mount -t nfs 192.168.1.1:/mnt/sda3 /media/sda3/


[1]: https://pl.wikipedia.org/wiki/Network_File_System_(protok%C3%B3%C5%82)
[2]: https://pl.wikipedia.org/wiki/Samba_(program)
[3]: https://ignorantguru.github.io/udevil/
