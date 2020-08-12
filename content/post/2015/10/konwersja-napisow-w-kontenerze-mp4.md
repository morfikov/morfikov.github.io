---
author: Morfik
categories:
- Linux
date: "2015-10-19T19:31:49Z"
date_gmt: 2015-10-19 17:31:49 +0200
published: true
status: publish
tags:
- video
title: Konwersja napisów w kontenerze MP4
---

Podczas ogarniania kolekcji filmów i przerabiania jej w taki sposób by został nam tylko jeden plik,
tj. [kontener MKV]({{< baseurl >}}/post/kontener-multimedialny-mkv/), możemy czasem napotkać
problemy, które mogą nam uniemożliwić to zadanie. Może się zdarzyć tak, że będziemy mieli do
czynienia z innymi kontenerami niż MKV, np. MP4. Ten wpis będzie poświęcony właśnie tego rodzaju
kontenerom.

<!--more-->
## Uzyskiwanie informacji o kontenerze MP4

Teoretycznie można skorzystać z `mkvmerge` do wyciągnięcia informacji o kontenerze MP4. Problem w
tym, że nie zostaną pokazane wszystkie ścieżki. Przykładowo:

    $ mkvmerge -i film.mp4
    File 'film': container: QuickTime/MP4
    Track ID 0: video (MPEG-4p10/AVC/h.264)
    Track ID 1: audio (AAC)
    Chapters: 45 entries

Mamy co prawda informacje na temat rodzaju kontenera, jak i ścieżek video i audio. Są nawet
rozdziały. Jedyne czego brakuje to napisów. Innym narzędziem, które może nam pomóc w opisaniu
kontenera MP4 jest `mediainfo` . W odróżnieniu od `mkvmerge` potrafi podać informacje o napisach, a
te są w dość egzotycznym formacje:

    Format                                   : Timed Text
    Codec ID                                 : tx3g

Będzie trzeba te napisy jakoś wyciągnąć i przekonwertować do formatu SRT ale o tym później.

Po dłuższych poszukiwaniach okazało się, że do kontenerów MP4 wykorzystywane jest narzędzie `MP4Box`
, które w debianie jest dostępne w [pakiecie gpac](https://gpac.wp.imt.fr/) . No to teraz już bez
problemu możemy uzyskać informacje na temat kontenera MP4:

    $ MP4Box -info film.mp4
    * Movie Info *
            Timescale 600 - Duration 03:16:02.900
            5 track(s)
            Fragmented File: no
            File suitable for progressive download (moov before mdat)
            File Brand isom - version 1
            Created: GMT Wed Oct 23 14:07:50 2013

    File has root IOD (9 bytes)
    Scene PL 0xff - Graphics PL 0xff - OD PL 0xff
    Visual PL: AVC/H264 Profile (0x15)
    Audio PL: AAC Profile @ Level 2 (0x29)
    No streams included in root OD

    Chapters:
            Chapter #1 - 00:00:00.000 - "Foreword"
            Chapter #2 - 00:01:30.090 - "Awaiting Achilles"
            Chapter #3 - 00:08:16.329 - ""Is There No One Else?""
            Chapter #4 - 00:11:06.332 - "Secret Lovers"
            Chapter #5 - 00:18:22.101 - "Brothers' Pledges"
            Chapter #6 - 00:25:33.365 - "Greatest War"
            Chapter #7 - 00:27:41.493 - "Recruiter Odysseus"
            Chapter #8 - 00:33:23.335 - "Glory and Doom"
            Chapter #9 - 00:36:39.864 - "Royal Welcome"
            Chapter #10 - 00:40:01.566 - "They're Coming for Me"
            Chapter #11 - 00:44:50.187 - "Immortality Is Yours"
            Chapter #12 - 00:52:32.316 - "Beach Combat"
            Chapter #13 - 00:58:00.143 - "Too Early in the Day"
            Chapter #14 - 01:03:48.992 - "No Need to Fear"
            Chapter #15 - 01:07:41.224 - "Spoils of War"
            Chapter #16 - 01:12:36.352 - "The Way I Love Helen"
            Chapter #17 - 01:17:55.838 - "Power, Not Love"
            Chapter #18 - 01:21:31.720 - "Soldiers Obey"
            Chapter #19 - 01:23:55.864 - "Gathering Forces"
            Chapter #20 - 01:28:49.324 - "Brave Offer"
            Chapter #21 - 01:32:23.371 - "Paris vs. Menelaus"
            Chapter #22 - 01:36:58.146 - "Battle Cry"
            Chapter #23 - 01:41:39.093 - "Greek Retreat"
            Chapter #24 - 01:44:56.957 - "Weak Morale"
            Chapter #25 - 01:48:15.322 - "Everyone Dies"
            Chapter #26 - 01:54:20.687 - "Born for This War"
            Chapter #27 - 01:57:26.039 - "Priam's Order"
            Chapter #28 - 02:00:05.698 - "Flaming Attack"
            Chapter #29 - 02:03:47.086 - "Hector's Adversary"
            Chapter #30 - 02:08:40.212 - "Tragic Mistake"
            Chapter #31 - 02:10:42.668 - "Night of Torments"
            Chapter #32 - 02:15:10.436 - "Summoned to Fight"
            Chapter #33 - 02:20:47.773 - "Now You Know"
            Chapter #34 - 02:23:08.246 - "Hector vs. Achilles"
            Chapter #35 - 02:26:25.110 - "Desecrating the Dead"
            Chapter #36 - 02:28:29.734 - "A Father's Plea"
            Chapter #37 - 02:34:48.446 - "Achilles' Word"
            Chapter #38 - 02:37:28.939 - "Inspirations and Honors"
            Chapter #39 - 02:41:20.170 - "Parting Gift"
            Chapter #40 - 02:45:40.764 - "Troy Under Siege"
            Chapter #41 - 02:51:09.759 - "We Will Be Together"
            Chapter #42 - 02:55:39.028 - "Royal Bloodshed"
            Chapter #43 - 02:58:42.712 - "Brought to Hell"
            Chapter #44 - 03:02:24.433 - "I Walked With Giants"
            Chapter #45 - 03:05:12.601 - "End Credits"

    Track # 1 Info - TrackID 1 - TimeScale 24000 - Media Duration 03:16:02.751
    Media Info: Language "Undetermined" - Type "vide:avc1" - 282024 samples
    Visual Track layout: x=0 y=0 width=1280 height=528
    MPEG-4 Config: Visual Stream - ObjectTypeIndication 0x21
    AVC/H264 Video - Visual Size 1280 x 528
            AVC Info: 1 SPS - 1 PPS - Profile High @ Level 4.1
            NAL Unit length bits: 32
            Pixel Aspect Ratio 1:1 - Indicated track size 1280 x 528
            Chroma format 1 - Luma bit depth 8 - chroma bit depth 8
    Self-synchronized

    Track # 2 Info - TrackID 2 - TimeScale 48000 - Media Duration 03:16:02.901
    Media Info: Language "English" - Type "soun:mp4a" - 551386 samples
    MPEG-4 Config: Audio Stream - ObjectTypeIndication 0x40
    MPEG-4 Audio AAC LC - 2 Channel(s) - SampleRate 48000
    Synchronized on stream 1

    Track # 3 Info - TrackID 3 - TimeScale 1000 - Media Duration 03:05:05.671
    Media Info: Language "English" - Type "text:tx3g" - 2748 samples
    3GPP/MPEG-4 Timed Text - Size 1280 x 528 - Translation X=0 Y=0 - Layer 0

    Track # 4 Info - TrackID 4 - TimeScale 1000 - Media Duration 03:06:09.928
    Media Info: Language "Romanian; Moldavian; Moldovan" - Type "text:tx3g" - 2689 samples
    3GPP/MPEG-4 Timed Text - Size 1280 x 528 - Translation X=0 Y=0 - Layer 0

    Track # 5 Info - TrackID 5 - TimeScale 1000 - Media Duration 03:05:05.619
    Media Info: Language "Russian" - Type "text:tx3g" - 2796 samples
    3GPP/MPEG-4 Timed Text - Size 1280 x 528 - Translation X=0 Y=0 - Layer 0

I jak widzimy powyżej, mamy trochę więcej ścieżek niż nam pokazał `mkvmerge`.

## Wydobywanie i konwersja napisów

Narzędzie `MP4Box` potrafi wydobyć i przekonwertować napisy bez większego problemu, tak jak to miało
miejsce w przypadku kontenerów MKV. Wystarczy wydać to poniższe polecenie, podając format napisów (
`srt` ) oraz numer ścieżki ( `3` ), przykładowo:

    $ MP4Box -srt 3 film.mp4

Napisy zostaną przekonwertowane do formatu SRT, czyli do tego jedynie słusznego formatu, który
powinien w końcu zapanować nad światem.
