---
author: Morfik
categories:
- Linux
date: "2015-10-28T21:21:51Z"
date_gmt: 2015-10-28 19:21:51 +0100
published: true
status: publish
tags:
- bezpieczeństwo
- prywatność
- flash
- debian
GHissueID: 202
title: Konfiguracja wtyczki flash na linux'ie (mms.cfg)
---

W obecnych czasach powoli się odchodzi od stosowania technologi flash w przeglądarkach. Na dobra
sprawę, po tym jak google w youtube przesiadł się na html5, to korzystanie z flash player'a nie ma
już większego sensu. Są jednak serwisy, które nie nadążają za zmieniającą się rzeczywistością i w
ich przypadku przejście z flash'a na html5 może jeszcze zająć kilka lat. Zatem nawet jeśli nie
korzystamy z flash'a na co dzień, to i tak większość z nas będzie chciała go mieć w systemie, tak na
wszelki wypadek, by nie być pozbawionym możliwości oglądania materiałów video na tych drugorzędnych
serwisach. Jako, że wtyczka flash jest bardzo dziurawa, przydałoby się ją nieco skonfigurować i w
tym wpisie zostanie przedstawionych szereg opcji, które można umieścić w pliku `mms.cfg` .

<!--more-->
## Opcje wtyczki flash

Jeżeli w naszym debianie mamy zainstalowany pakiet `flashplugin-nonfree` oznacza to, że przeglądarki
są w stanie korzystać z wtyczki flash. W opcjach Firefox'a mamy jedynie możliwość
wyłączenia/włączenia wtyczek lub też ustawienia ich tak by były ładowanie na żądanie. Nie damy
jednak rady skonfigurować opcji samych wtyczek. Te opcje, o których mowa, można konfigurować osobno
jedynie dla każdej strony www o ile taka korzysta z flash player'a. Te opcje można podejrzeć i by to
zrobić, wystarczy kliknąć prawym przyciskiem myszy na okienku z filmem i wybrać pozycję `Settings` ,
przykładowo:

![](/img/2015/10/1.www-flash-ustawienia-mms.cfg_.png#big)

Po jej wybraniu, powinno nam pokazać się okienko z szeregiem opcji:

![](/img/2015/10/2.flash-mms.cfg-opcje-linux-www.png#huge)

Jak widzimy wyżej, mamy możliwość ustawienia różnych rzeczy, wliczając w to dostęp do mikrofonu czy
też kamery. Każda strona www może te urządzenia wykorzystywać, chyba, że w tych opcjach wyraźnie
ustawimy, by tego nie robiła. Problem w tym, że domyślnie każda witryna korzystająca z flash'a ma
dostęp do tych urządzeń, zatem możemy być podsłuchiwani i nagrywani bez naszej wiedzy ilekroć tylko
odwiedzamy taki serwis, a tego raczej chyba byśmy nie chcieli. Manualne konfigurowanie wszystkich
możliwych stron www nie wchodzi w grę. Zatem jak podejść do konfiguracji wtyczki flash?

## Pliki /etc/adobe/mms.cfg oraz ~/mm.cfg

Plugin flash ma swój plik konfiguracyjny, w którym można skonfigurować te wyżej widoczne opcje.
Można również skonfigurować nieco więcej ale o tym za moment. Problem z konfigurowaniem flash'a
przez przeglądarkę jest taki, że te ustawienia można zmienić bez problemu. Nieco trwalsze ustawienia
zapewnia plik `mm.cfg` , który można wgrać do katalogu domowego. Standardowo on nie istnieje i
trzeba go sobie utworzyć. Niemniej jednak, ustawienia w tym pliku również mogą zostać zmienione bez
większych przeszkód. Alternatywą jest umieszczenie całej konfiguracji flash'a w pliku `mms.cfg` w
katalogu `/etc/adobe/` , gdzie możliwość wprowadzania zmian będzie miał jedynie administrator
systemu.

Na stronie Adobe widnieje informacja, że [wersja flash'a 11.2 będzie
ostatnią](https://get.adobe.com/flashplayer/) jaka zostanie wypuszczona na linux'a. Dlatego też
konfiguracja pliku `mms.cfg` będzie się opierać o tę wersję, a nie o jakąś nowszą. Wszystkie poniżej
wypisane opcje zostały zaczerpnięte z [manuala dostępnego na stronie
Adobe](http://www.adobe.com/devnet/flashplayer/articles/flash_player_admin_guide.html). Wszystkie
opcje, których domyślne wartości zmienimy za pomocą pliku `mms.cfg` , będą honorowane globalnie w
całym systemie i użytkownik, czy też sama przeglądarka, nie będą mogły ich zmienić. Trzeba o tym
pamiętać na wypadek późniejszych problemów.

## Parametry w pliku mms.cfg

Poniżej jest lista wraz z krótkim opisem tych parametrów, które ja umieściłem w swoim pliku
`mms.cfg` .

  - `AVHardwareDisable` -- gdy ten parametr zostanie ustawiony na `1` , flash nie będzie miał
    dostępu do mikrofonu i kamery.
  - `DisableDeviceFontEnumeration` -- w przypadku ustawienia na `1` , flash nie będzie w stanie
    wydobyć listy czcionek zainstalowanych w systemie.
  - `FullScreenDisable` -- ten parametr określa czy pliki SWF mogą być odtwarzane w trybie
    pełnoekranowym.
  - `LocalFileReadDisable` -- jeśli ten parametr zostanie ustawiony na `1` , lokalne pliki SWF nie
    będą w stanie odczytywać plików z dysku twardego, co oczywiście uniemożliwi ich uruchomienie.
    Dodatkowo, zdalne pliki SWF nie będą w stanie wysyłać i pobierać plików.
  - `FileDownloadDisable` -- po ustawieniu `1` w tym parametrze, pobieranie plików za pomocą flash'a
    nie będzie możliwe.
  - `FileUploadDisable` -- po ustawieniu `1` w tym parametrze, wysyłanie plików za pomocą flash'a
    nie będzie możliwe.
  - `LocalStorageLimit` -- sztywny limit dla danych przechowywanych lokalnie przez konkretną domenę.
    Możliwe do określania `1` = brak, `2` = 10 KB, `3` = 100 KB, `4` = 1 MB, `5` = 10 MB, `6` =
    użytkownik określa limit.
  - `ThirdPartyStorage` -- w przypadku ustawienia tego parametru na `0` , strony trzecie nie będą w
    stanie czytać i zapisywać lokalnie przechowywanych obiektów.
  - `AssetCacheSize` -- sztywny limit dla komponentów flash'a wykorzystywany przez flash player'a.
    Można, co prawda, ustawić tutaj `1` ale nie ma się wpływu na rozmiar tego cache, domyślnie
    wynosi on 20MB.
  - `AutoUpdateDisable` -- w przypadku linux'ów, funkcja auto update nie ma większego sensu, dlatego
    też można ją spokojnie wyłączyć.
  - `DisableProductDownload` -- w przypadku ustawienia tego parametru na `1` nie będzie możliwe
    automatyczne instalowanie przez flash'a pewnych aplikacji, które zostały podpisane cyfrowo przez
    Adobe.
  - `LocalFileLegacyAction` -- kontroluje to w jaki sposób flash player wykonuje lokalnie pewne
    pliki SWF, które zostały stworzone przez wcześniejsze wersje flash player'a. Dobrze jest tutaj
    ustawić `0` , czyli najbardziej restrykcyjny tryb.
  - `AllowUserLocalTrust` -- w przypadku ustawienia tego parametru na `0` , użytkownik nie może
    umieszczać plików w lokalnie zaufanym sandboxie.
  - `DisableSockets` -- ustawienie tutaj `0` wyłączy możliwość nawiązywania połączeń sieciowych z
    serwerami.
  - `OverrideGPUValidation` -- ten parametr może powodować problemy przy renderowaniu obrazu. Dobrze
    jest ustawić tutaj `0` .
  - `RTMFPP2PDisable` -- ustawienie tutaj `1` wyłączy możliwość udostępniania naszego łącza
    sieciowego na potrzeby połączeń p2p w strumieniach video serwowanych przez flash player podczas
    oglądania filmu.

Cały plik `mms.cfg` wraz z komentarzami (po angielsku) i innymi dodatkowymi opcjami, które nie
zostały domyślnie włączone zamieszczam poniżej:

    #
    # /etc/adobe/mms.cfg: Adobe Flash (v. 11.2) privacy and security settings
    #
    # For more details, please visit:
    # http://www.adobe.com/devnet/flashplayer/articles/flash_player_admin_guide.html
    #

    # If this value is set to 1, SWF files cannot access webcams or microphones. If this value is 0,
    # the Settings Manager or Settings tabs let the user specify settings for access to webcams and
    # microphones. If this value is set to 1, the privacy pop-up dialog never appears. However, the
    # user can still access the Privacy tab and the Settings Manager, as well as tabs to let them
    # designate which camera or microphone an application can use. These settings appear functional,
    # but any choices the user makes are ignored. Also the recording level meter on the Microphone tab
    # is disabled, and the Camera tab does not bring up a thumbnail of what the camera is seeing.
    #
    AVHardwareDisable = 1

    # If the AVHardwareDisable value is set to 1, it prohibits SWF files from accessing webcams or
    # microphones. The AVHardwareEnabledDomain settings provide exceptions to that rule. They create a
    # "white list" of approved domain names or IP addresses to which data can be transmitted using a
    # webcam or microphone. If the active security context is in the list of domains and IP addresses
    # then camera and microphone access will be allowed. Otherwise it will default to the behavior
    # specified by the AVHardwareDisable setting. This value must be set to a string containing a full
    # domain name or IP address. The string value must exactly match the domain name or IP address to
    # be enabled. Strings with wildcards such as *.adobe.com or 10.1.1.* are not supported. The mms.cfg
    # file can contain multiple AVHardwareEnabledDomain settings to allow access to multiple domains
    # and IP addresses. For example the following settings only allow access to cameras or microphones
    # when connected to servers with the domain name test.mydomain.com or the IP address 10.1.1.10:
    #
    #AVHardwareEnabledDomain = test.mydomain.com
    #AVHardwareEnabledDomain = 10.1.1.10

    # This setting controls whether the Font.enumerateFonts() method in ActionScript 3.0 and the
    # TextField.getFontList() method in ActionScript 1.0 and 2.0 return the list of fonts installed on
    # a user’s system. If this value is 1, information on installed fonts cannot be returned. If this
    # value is 0 (the default), information on installed fonts can be returned.
    #
    DisableDeviceFontEnumeration = 1

    # This setting controls whether a SWF file playing via a browser plug-in can be displayed in
    # full-screen mode; that is, taking up the entire screen and thus obscuring all application windows
    # and system controls. If you set this value to 1, SWF files that attempt to play in full-screen
    # mode fail silently. The default value is 0. Full-screen mode is implemented with a number of
    # security options already built in, so you might choose to disable it only in specific
    # circumstances.
    #
    FullScreenDisable = 0

    # Setting this option to 1 prevents local SWF files from having read access to files on local
    # hard drives; that is, local SWF files can’t even run. In addition, remote SWF files are unable
    # to upload or download files. The default value is 0. If this value is set to 1, ActionScript
    # cannot read any files referenced by a path (including the first SWF file that Flash Player opens)
    # on the user’s hard disk. Any ActionScript API that loads files from the local file system is
    # blocked. File upload/download via methods of the FileReference and FileReferenceList ActionScript
    # APIs are also blocked if this flag is set. In addition, any values set for FileDownloadDisable
    # and FileUploadDisable are ignored. It is important to remember that, except for uploading and
    # downloading files, the only SWF files that can read local files are SWF files that are themselves
    # local. Therefore, you do not need to use this option to prevent remote SWFs from reading local
    # data; that is always prevented anyway. If this option is disabled, the ActionScript methods
    # FileReference.browse() and FileReferenceList.browse() are also disabled.
    #
    LocalFileReadDisable = 0

    # If this value is set to 1, the ActionScript FileReference.download() method is disabled; the
    # user is not prompted to allow a download, and no downloads using the FileReference API are
    # allowed. If this value is set to 0 (the default), Flash Player allows the ActionScript
    # FileReference.download() method to ask the user where a file can be downloaded to, and then Flash
    # Player downloads the file after the user approves the file save location. Files are never
    # downloaded without user approval.
    #
    FileDownloadDisable = 0

    # If the FileDownloadDisable value is set to 1, it prevents SWF files from downloading files
    # using the FileReference API. The FileDownloadEnabledDomain settings provide exceptions to that
    # rule. They create a "white list" of approved domain names or IP addresses from which files can be
    # downloaded. If the active security context is in the list of domains and IP addresses then file
    # downloads will be allowed. Otherwise it will default to the behavior specified by the
    # FileDownloadDisable setting. This value must be set to a string containing a full domain name or
    # IP address. The string value must exactly match the domain name or IP address to be enabled.
    # Strings with wildcards such as *.adobe.com or 10.1.1.* are not supported. The mms.cfg file can
    # contain multiple FileDownloadEnabledDomain settings to allow downloading from multiple domains
    # and IP addresses.
    #
    #FileDownloadEnabledDomain = test.mydomain.com
    #FileDownloadEnabledDomain = 10.1.1.10

    # If this value is set to 1, all FileReference.upload(), FileReference.browse(), and
    # FileReferenceList.browse() activity is disabled; the user is not prompted to upload files, and no
    # uploads using the FileReference API are allowed. If this value is set to 0 (the default), Flash
    # Player allows files to be uploaded using the FileReference API. The user is prompted to select a
    # file to upload and to approve the selection. Files are never uploaded without user approval.
    #
    FileUploadDisable = 0

    # If the FileUploadDisable value is set to 1, it prevents SWF files from uploading files using
    # the FileReference API. The FileUploadEnabledDomain settings provide exceptions to that rule. They
    # create a "white list" of approved domain names or IP addresses to which files can be uploaded. If
    # the active security context is in the list of domains and IP addresses then file uploads will be
    # allowed. Otherwise it will default to the behavior specified by the FileUploadDisable setting.
    # This value must be set to a string containing a full domain name or IP address. The string value
    # must exactly match the domain name or IP address to be enabled. Strings with wildcards such as
    # *.adobe.com or 10.1.1.* are not supported. The mms.cfg file can contain multiple
    # FileDownloadEnabledDomain settings to allow uploading to multiple domains and IP addresses.
    #
    #FileDownloadEnabledDomain=test.mydomain.com
    #FileDownloadEnabledDomain=10.1.1.10

    # This value specifies a hard limit on the amount of local storage that Flash Player uses (per
    # domain) for persistent shared objects. The user can use the Settings Manager or Local Storage
    # Settings dialog box to specify local storage limits. If no value is set here and the user
    # doesn’t specify storage limits, the default limit is 100 KB per domain. If this value is set to
    # 6 (the default), the user specifies the storage limits for each domain. If LocalStorageLimit is
    # set, the Local Storage tab shows the limit specified, and the user can use this tab as if the
    # limit does not exist. If the user sets more restrictive settings than the value set by
    # LocalStorageLimit, they are honored (and displayed the next time the Settings dialog box is
    # loaded). However, if the user selects settings higher than the limit set by LocalStorageLimit,
    # the user’s settings are ignored. The local file storage limit is best obtained from the
    # Settings dialog box, because this security setting is just a maximum value, and the user may have
    # set a lower limit. The possible values are: 1 = no storage, 2 = 10 KB, 3 = 100 KB, 4 = 1 MB, 5 =
    # 10 MB, 6 = user specifies upper limit
    #
    LocalStorageLimit = 1

    # Third party refers to SWF files that are executing within a browser and have an originating
    # domain that does not match the URL displayed in the browser window. If this value is set to 1,
    # third-party SWF files can read and write locally persistent shared objects. If this value is set
    # to 0, third-party SWF files cannot read or write locally persistent shared objects. This setting
    # does not have a default value. If it is not included in the mms.cfg file, the Settings Manager or
    # Local Storage Settings dialog box lets the user specify whether to permit locally persistent
    # shared objects. If the user doesn’t make any changes, the default is to permit shared objects.
    #
    ThirdPartyStorage = 0

    # This value specifies a hard limit, in MB, on the amount of local storage that Flash Player uses
    # for the storage of common Flash components. If this option is not included in the mms.cfg file,
    # the Settings Manager lets the user specify whether to permit component storage. However, the user
    # can’t specify how much local storage space to use. The default limit is 20 MB. Setting this
    # value to 0 disables component storage, and any components that have already been downloaded are
    # purged the next time Flash Player runs.
    #
    AssetCacheSize = 0

    # If this value is set to 0 (the default), Flash Player lets a user with admin rights enable or
    # disable notification updates for all accounts on the machine in the Settings Manager. Standard
    # users, meaning users without admin rights, cannot change this setting for all accounts on the
    # machine. Standard users can enable or disable notification update for their individual account.
    # That is, you cannot use AutoUpdateDisable = 0 to prevent the user from disabling notification
    # updates for their individual account. If this value is set to 1, Flash Player disables
    # notification updates and the AutoUpdateInterval, DisableProductDownload, and ProductDisabled
    # options are ignored. However, you can still use the SilentAutoUpdateEnable and
    # SilentAutoUpdateVerboseLogging options because a standard user cannot disable background updates.
    #
    AutoUpdateDisable = 1

    # If this is a negative value (the default), Flash Player uses the notification update interval
    # value specified in the Settings Manager. (If users don't make any changes with the Settings
    # Manager, the default is every 7 days.) If this value is set to 0, Flash Player checks for an
    # update every time it starts. If this is a positive value, the value specifies the minimum number
    # of days between update checks.
    #
    #AutoUpdateInterval = 14

    # If this value is set to 0 (the default), Flash Player can install native code applications that
    # are digitally signed and delivered by Adobe. Adobe uses this capability to deliver Flash Player
    # updates through the developer-initiated Express Install process, and to deliver the Adobe Acrobat
    # Connect screen-sharing functionality. If this value is set to 1, these capabilities are disabled.
    # However, if you want to enable some but not all product downloads, set this value to 0 (or omit
    # it) and then use the ProductDisabled option to specify which product downloads are not permitted.
    #
    DisableProductDownload = 1

    # This option is effective only when DisableProductDownload has a value of 0 or is not present in
    # the mms.cfg file; it creates a list of ProductManager applications that users are not permitted
    # to install or launch. Unlike most other mms.cfg options, you can use this option as many times as
    # is appropriate for your environment.
    #
    #ProductDisabled = "application name"

    # This setting controls whether to allow a SWF file produced for Flash Player 6 and earlier to
    # execute an operation that has been restricted in a newer version of Flash Player. Flash Player 6
    # made security sandbox distinctions based on superdomains. For example, SWF files from
    # www.example.com and store.example.com were placed in the same sandbox. Flash Player 7 and later
    # have made security sandbox distinctions based on exact domains, so, for example, a SWF file from
    # www.example.com is placed in a different sandbox than a SWF file from store.example.com. The
    # exact-domain behavior is more secure, but occasionally users may encounter a set of cooperating
    # SWF files that were created when the older superdomain rules were in effect, and require the
    # superdomain rules to work correctly. When this occurs, by default, Flash Player shows a dialog
    # box asking users whether to allow or deny access between the two domains. Users may configure a
    # permanent answer to this question by selecting Never Ask Again in the dialog, or by visiting the
    # Settings Manager. The LegacyDomainMatching setting lets you override users' decisions about this
    # situation.  This setting does not have a default value. If it is not included in the mms.cfg
    # file, the user can determine whether to allow the operation in a global manner (using the
    # Settings Manager), or on a case-by-case basis (using an interactive dialog box). The values the
    # user can choose among are "Ask," "Allow," and "Deny." The default value is "Ask". If this value
    # is set to 1, Flash Player behaves as though the user answers "allow" whenever they make this
    # decision. If it is set to 0, Flash Player behaves as though the user answers "deny" whenever they
    # make this decision.
    #
    #LegacyDomainMatching = 0

    # This setting controls how Flash Player determines whether to execute certain local SWF files
    # that were originally produced for Flash Player 7 and earlier. Flash Player 7 and earlier placed
    # all local SWF files in the local-trusted sandbox. Flash Player 8 and later have, by default,
    # placed local SWF files in either the local-with-filesystem or local-with-networking sandbox. In
    # order for a SWF file to be placed in the local-trusted sandbox in Flash Player 8 or later, that
    # SWF file must be designated trusted, using either the Settings Manager or a trust configuration
    # file. This latter behavior is more secure, but occasionally users may encounter an older local
    # SWF file that was created when the older local-trusted behavior was in effect, and must be inthe
    # local-trusted sandbox in order to work correctly. Users are notified of such situations by a
    # dialog box, but the dialog is only a failure notification, not a means to trust the SWF file in
    # question. Users can restore the functionality of such SWF files on a case-by-case basis by
    # designating them trusted in the Settings Manager, but if users encounter a large number of such
    # files, they may also elect in the Settings Manager to place all local SWF files published for
    # Flash Player 7 or earlier into the local-trusted sandbox. The LocalFileLegacyAction setting lets
    # you override users' decisions about this situation. This setting does not have a default value.
    # If it is not included in the mms.cfg file, the user can use the Settings Manager to specify
    # whether to place all older local SWF files into the local-trusted sandbox. If this value is set
    # to 1 (the most permissive setting), Flash Player behaves as though users had elected to place all
    # olderlocal SWF files into the local-trusted sandbox. If this value is set to 0 (the most
    # restrictive setting), Flash Player behaves as though users had elected never to automatically
    # place older local SWF files into the local-trusted sandbox, and also suppresses the failure
    # notification dialog.
    #
    LocalFileLegacyAction = 0

    # This setting lets you prevent users from designating any files on local file systems as trusted
    # (that is, placing them into the local-trusted sandbox). This setting applies to SWF files
    # published for any version of Flash. If this value is set to 1 (the default), Flash Player allows
    # the user to specify whether local files can be placed into the local-trusted sandbox, through the
    # use of the Settings Manager Global Security Settings panel and user trust files. If this value is
    # set to 0, the user cannot place files into the local-trusted sandbox. That is, the Settings
    # Manager Global Security Settings panel and user trust files are ignored.
    #
    AllowUserLocalTrust = 0

    # By default, local security is disabled whenever the ActiveX control is running in a non-browser
    # host application. In rare cases when this causes a problem, you can use this setting to enforce
    # local security rules for the specified application. You can enforce local security for multiple
    # applications by entering a separate EnforceLocalSecurityInActiveXHostApp entry for each
    # application. The filename string must specify the executable filename only, not the full path to
    # the executable; if you specify a full path, the setting is ignored.
    #
    #EnforceLocalSecurityInActiveXHostApp = "executable filename"

    # This option is similar to "EnforceLocalSecurityInActiveXHostApp", but applies to plug-ins as
    # well as the ActiveX control, and imposes stricter security controls. When a plug-in or ActiveX
    # control is running within an application specified, it will be as though the HTML parameter
    # allowNetworking="none" had been specified. That is, no networking or file system access of any
    # kind will be permitted, and the SWF running in the Flash Player will run without the ability to
    # load any additional media or communicate with any servers. You can enforce local security for
    # multiple applications by entering a separate DisableNetworkAndFilesystemInHostApp entry for each
    # application. The filename string must specify the executable filename only, not the full path to
    # the executable; if you specify a full path, the setting is ignored.
    #
    #DisableNetworkAndFilesystemInHostApp = "executable filename"

    # This option enables or disables the use of the Socket.connect() and XMLSocket.connect()
    # methods. If you don’t include this option in the mms.cfg file, or if its value is set to 0,
    # socket connections are permitted to any server. If this value is set to 1, no socket connections
    # are allowed. However, if you want to disable some but not all socket connections, set this value
    # to 1 and then use EnableSocketsTo to specify one or more servers to which socket connections can
    # be made.
    #
    DisableSockets = 1

    # This option is effective only when DisableSockets has a value of 1; it creates a whitelist of
    # servers to which socket connections are allowed. Unlike most other mms.cfg options, you can use
    # this option as many times as is appropriate for your environment. Note that the servers specified
    # are target servers, to which socket connections are made; they are not origin servers, from which
    # the connecting SWF files are served. The values specified here must exactly match the values
    # specified in the ActionScript connect() methods. If you specify an IP address here, but the
    # connect() method specifies a host name, the method fails even if that host name resolves to the
    # specified IP address. Similarly, if you specify a host name here but the connect() method
    # specifies an IP address, the method fails. Using this option does not take the place of a socket
    # policy file on the target server. That is, this option has no effect if the specified server does
    # not have a socket policy file.
    #
    #EnableSocketsTo = test.mydomain.com
    #EnableSocketsTo = 10.1.1.10

    # The GPU compositing feature is gated by the driver version for video cards. If a card and
    # driver combination does not match the requirements needed to implement compositing, set
    # OverrideGPUValidation to 1 to override validation of the driver requirements. For example, you
    # might want GPU compositing enabled during a specific test suite, even if the video driver in the
    # test machine doesn’t meet compositing requirements. This setting overrides driver version
    # gating but still checks for VRAM requirements. Adobe recommends that you use this setting with
    # care. Overriding GPU validation can result in rendering problems or system crashes due to driver
    # issues. After completing the tests or programming tasks that require the use of this setting,
    # consider setting it back to 0 (or removing it from the mms.cfg file) for normal operations.
    #
    OverrideGPUValidation = 0

    # This option specifies how the NetStream constructor connects to a server when a value is
    # specified for peerID, the second parameter passed to the constructor. If RTMFPP2PDisable has a
    # value of 0 or is not present in the mms.cfg file, a peer-to-peer (P2P) connection can be used. If
    # this value is 1, any value specified for peerID is ignored and P2P connections are disabled;
    # NetStream objects can connect only to Flash Media Server.
    #
    RTMFPP2PDisable = 1

    # If this option is present, Flash Player attempts to make RTMFP connections through the
    # specified TURN server in addition to normal UDP sockets. TURN Servers are useful for conveying
    # RTMFP network traffic through firewalls that otherwise block UDP packets.
    #
    #RTMFPTURNProxy = URL of TURN proxy server
