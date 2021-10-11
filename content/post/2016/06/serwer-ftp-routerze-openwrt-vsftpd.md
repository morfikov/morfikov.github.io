---
author: Morfik
categories:
- OpenWRT
date: "2016-06-23T14:30:39Z"
date_gmt: 2016-06-23 12:30:39 +0200
published: true
status: publish
tags:
- chaos-calmer
- ftp
- router
GHissueID: 351
title: Serwer FTP na routerze z OpenWRT (vsftpd)
---

[Usługa FTP](https://pl.wikipedia.org/wiki/File_Transfer_Protocol) jest w miarę wygodnym
rozwiązaniem w przypadku, gdy chcemy korzystać z zasobów udostępnianych przez router zarówno na
linux'ach jak i na windowsach. Jedyne czego potrzebujemy to kawałek przeglądarki albo jakiegoś
klienta FTP. W tym protokole nie ma też znaczenia system plików, w którym znajdują się udostępniane
pliki. Jedyne co nas interesuje, to postawienie serwera na routerze i podanie klientom namiarów na
niego. W OpenWRT możemy do tego celu zaprzęgnąć `vsftpd` . W tym wpisie pokażę jedynie jak tego typu
usługę uruchomić na domowym routerze WiFi i jak ją wstępnie skonfigurować. Wszelkie kwestie
techniczne związane z działaniem samego serwera FTP jak i jego wszystkich parametrów już opisywałem
przy okazji [wdrażania vsftpd w dystrybucji
debian](/post/konfiguracja-vsftpd-w-debianie/). Zachęcam zatem do zapoznania się
również z tym podlinkowanym wpisem.

<!--more-->
## Który serwer FTP zainstalować

W repozytorium OpenWRT mamy do wyboru szereg serwerów. Jako, że ja głównie miałem do czynienia z
serwerem `vsftpd` , to opiszę akurat to konkretne oprogramowanie. Mamy tutaj do wyboru dwa pakiety:
`vsftpd` oraz `vsftpd-tls` . To jest dokładnie ten sam serwer, z tą różnicą, że ten drugi pakiet ma
zaimplementowaną obsługę szyfrowania SSL/TLS. W warunkach domowych, gdzie serwer FTP będzie
udostępniany jedynie w sieci LAN, nie potrzebujemy sobie zawracać głowy pakietem `vsftpd-tls` .
Oczywiście nic nie stoi na przeszkodzie, by go zainstalować. Trzeba mieć jednak na uwadze, że
przeciętne routery WiFi nie zaoferują nam wysokiego transferu przy szyfrowaniu ruchu. Do tego
jeszcze dochodzą zależności, które swoje ważą (w sumie około 1 MiB). Dlatego też lepiej ograniczyć
się do podstawowego pakietu `vsftpd` . Logujemy się zatem na router i w terminalu wpisujemy te
poniższe polecenia:

    # opkg update
    # opkg install vsftpd

## Konfiguracja serwera FTP

Jako, że mamy zamiar korzystać z serwera FTP w sieci domowej, to możemy skonfigurować na nim jedynie
użytkownika anonimowego i nadać mu prawa do odczytu i zapisu całej zawartości udostępnianego
katalogu. Konfiguracja demona `vsftpd` jest trzymana w pliku `/etc/vsftpd.conf` . Domyślnie mamy już
w nim określonych szereg opcji. Czyścimy ten plik i uzupełniamy go poniższą zawartością:

    background=YES
    listen=YES
    listen_address=192.168.2.1
    anonymous_enable=YES
    no_anon_password=NO
    anon_root=/mnt/serwer/
    write_enable=YES
    anon_upload_enable=YES
    anon_mkdir_write_enable=YES
    anon_other_write_enable=YES
    anon_world_readable_only=NO
    check_shell=NO
    ftpd_banner=Welcome to OpenWRT
    session_support=NO

Opcje dotyczące szyfrowania SSL/TLS możemy zwyczajnie usunąć, bo ten pakiet, który zainstalowaliśmy,
nie ma przecie zaimplementowanej obsługi szyfrowania. W przypadku problemów ze startem serwera FTP,
dobrze jest także dopisać, te dwie poniższe opcje:

    syslog_enable=YES
    log_ftp_protocol=YES

## Udostępnienie katalogów na serwerze

W pliku `/etc/vsftpd.conf` określiliśmy katalog `/mnt/serwer/` jako miejsce docelowe naszego serwera
FTP. Ten folder musimy sobie utworzyć:

    # mkdir /mnt/serwer/

Następnie w pliku `/etc/passwd` trzeba dostosować katalog domowy dla użytkownika `ftp` :

    ftp:*:55:55:ftp:/mnt/serwer:/bin/false

Teraz już możemy zrestartować `vsftpd` :

    # /etc/init.d/vsftpd restart

Trzeba pamiętać, że właścicielem głównego katalogu serwera FTP nie może być użytkownik `ftp` . W ten
sposób, w głównym katalogu nie damy rady zapisać plików. Dlatego też musimy wewnątrz katalogu
`/mnt/serwer/` utworzyć podkatalog i to tam umieszczać pliki czy montować dodatkowe zasoby.
Pamiętajmy przy tym, by zmienić prawa właściciela do tego podkatalogu na `ftp` .

## Test konfiguracji serwera FTP

Teraz już potrzebujemy jedynie odpalić klienta FTP i spróbować podłączyć się do serwera w celu
sprawdzenia czy konfiguracja umożliwi nam na odczyt i zapis plików. W tym przypadku korzystamy z
programu [Filezilla](https://filezilla-project.org/). Gdy w grę wchodzi użytkownik anonimowy, to nie
musimy praktycznie nic zmieniać w ustawieniach połączenia:

![](/img/2016/06/1.ftp-openwrt-filezilla-vsftpd.png#big)

Poniżej zaś jest test przesyłania plików na serwer:

![](/img/2016/06/2.ftp-openwrt-filezilla-vsftpd.png#huge)

Nasz serwer powinien być także widoczny w przeglądarce po przejściu na adres `ftp://192.168.2.1/` :

![](/img/2016/06/3.ftp-openwrt-firefox-przegladarka-vsftpd.png#huge)
