# h6 WiFi

### Tehtävissä käytetty työympäristö
- Lenovo Yoga Slim 7 Pro (AMD Ryzen 7 5800H @ 3.20 GHz, 16 GB DDR4-3200, NVIDIA GeForce RTX 3050 laptop 4 GB GDDR6). WIN11, versio 25H2.
- Oracle VirtualBox 7.2.6
  - WifiChallenge Lab v2.4.2, Debian (64-bit)

### Viikon tehtävänannot
> a) Tutustu wifi challenge lab 2.1 harjoitus ympäristöön ja käytä tarvittaessa hyväksesi jo olemassa olevia ohjeita. \
> b) Kirjoita raportti siitä mitä opit ja mitkä asia yllättivät sinut kun tutustuit harjoitukseen. \
> c) Miten suhtautumisesi WLanin turvallisuuteen muuttui sen jälkeen kun teit harjoitukset?

## a ) Tutustu WiFiChallenge Lab 2.1 harjoitusympäristöön

Tutustuin WiFiChallenge Lab 2.1 harjoitusympäristöön tekemällä tehtäviä mm. Recon (verkon tiedustelun), OPN (avointen verkkojen) ja PSK ( Pre-Shared Key, WPA/WPA2-salasanaverkkojen) tehtävistä.

WifiChallenge Labilta löytyy itseltään enemmän tai vähemmän sekava, joskin vanhentunut mutta kattava [walkthrough](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/) joihinkin tehtäviin.  Raportissa ajankäyttö keskitetty seuraavaan tehtävään, esittäen tärkeimpiä työkaluja ja oppeja.

## b) WiFi Challenge Lab 2.1 ja mitä opin?

### Recon

Ensimmäisessä osiossa tutustuin WiFi-verkon tiedusteluun. Tarkoituksena oli selvittää langattomasta ympäristöstä tukiasemia, kanavia, asiakaslaitteita, probe-pyyntöjä ja mahdollisesti piilotettuja verkon nimiä.

___

#### Airmon-ng
    
``Airmon-ng`` on Aircrack-ng-paketin työkalu, jolla verkkokortti asetetaan monitor modeen. Monitor mode mahdollistaa WiFi-liikenteen kuuntelun ilman, että laite liittyy verkkoon.

    Esimerkki: 
    $ airmon-ng start wlan0

Komento käynnistää monitor moden verkkokortille ``wlan0``. Tämän jälkeen kortin nimeksi tulee usein ``wlan0mon``. (aircrack-ng.org, airmon-ng)

___

#### Airodump-ng

``Airodump-ng`` skannaa WiFi-ympäristöä ja näyttää näkyvät tukiasemat sekä client-laitteet. Sen avulla voidaan selvittää esimerkiksi verkon kanava, BSSID/MAC-osoite, ESSID/nimi ja clienttien probe-kyselyt.

    Esimerkki: 
    $ airodump-ng wlan0mon -w tiedosto --manufacturer --wps --band abg

Komento skannaa 2.4 GHz:n ja 5 GHz:n verkkoja (``--band abg``), näyttää valmistaja (``--manufacturer``)- ja WPS-tietoja (``--wps``) sekä tallentaa tulokset haluttuun kansioon (``-w ~tiedosto``). Lisäämällä esimerkiksi ``-c 11`` voidaan skannaus rajata tietylle kanavalle. (aircrack-ng.org, airodump-ng)

___

#### Mdk4

``mdk4`` on Aircrack-ng-projektiin liittyvä WiFi-testaukseen tarkoitettu proof-of-concept-työkalu. Sen avulla voidaan testata IEEE 802.11 -protokollan heikkouksia esimerkiksi lähettämällä erilaisia Wi-Fi-kehyksiä. (Aircrack-ng Github, mdk4)

Ensimmäisenä tuli luoda uusi sanalista, jossa wifi-sana lisättiin jokaisen sanan alkuun:

    Esimerkki toiminnasta:
    $ cat ~/rockyou-top100000.txt | awk '{print "wifi-" $1}' > ~/wifi-rockyou.txt
    $ sudo iwconfig wlan0mon channel 11
    $ mdk4 wlan0mon p -t F0:9F:C2:6A:88:26 -f ~/wifi-rockyou.txt

Ensimmäisessä komennossa:
- ``cat ~/rockyou-top100000.txt`` lukee alkuperäisen sanalistan.
- ``|`` putki siirtää sanalistan seuraavalle komennolle käsiteltäväksi.
- ``awk '{print "wifi-" $1}'`` lisää jokaisen sanan eteen tekstin wifi-.
  - ``awk`` on tekstin käsittelyyn tarkoitettu komentorivityökalu. Se lukee syötettä oletuksena rivi kerrallaan ja jakaa rivit kenttiin. (man7.org, awk(1p)).
  - ``awk`` lukee putkesta tulevan sanalistan rivi kerrallaan.
  - ``awk`` lisää sanan ``wifi-`` jokaisen rivin ensimmäiseen kenttään ``$1``.
  - esimerkiksi sana ``testi`` muuttuisi muotoon ``wifi-testi``.
- ``> ~/wifi-rockyou.txt`` tallentaa muokatun sanalistan uuteen tiedostoon.

Toinen komento lukitsi monitor mode -verkkokortin kanavalle 11, koska kohdeverkko toimi kyseisellä kanavalla. ``iwconfig`` syystä tai toisesta tarvitsi sudo-voimat, syytä en selvittänyt.

Kolmas komento käynnisti MDK4:n probe-tilassa:
- ``wlan0mon`` on monitor mode -verkkokortti.
- ``p`` tarkoittaa probe test -tilaa.
- ``-t F0:9F:C2:6A:88:26`` määrittää kohteeksi piilotetun tukiaseman MAC-osoitteen eli ``BSSID``:n.
- ``-f ~/wifi-rockyou.txt`` kertoo, mitä sanalistaa käytetään ESSID-ehdokkaiden, eli verkon nimien testaamiseen.

(Aircrack-ng Github, mdk4)

___
### OPN

Tässä osiossa hyödynsin Recon-vaiheessa löydettyä piilotetun verkon ESSID:tä. Tarkoituksena oli yhdistää avoimeen piilotettuun WiFi-verkkoon ja tarkistaa sen takana olevan reitittimen hallintasivu.

Recon-vaiheessa piilotetun verkon ESSID:ksi selvisi: ``wifi-free``.

___

#### Wpa_supplicant

``wpa_supplicant`` on langattoman client-laitteen WPA/802.11i-yhteydenhallintaan tarkoitettu työkalu. Lisäksi se hallitsee langattoman verkon tunnistautumista ja yhdistämistä. (NetBSD man-pages, wpa_supplicant)

``free.conf`` on ``wpa_supplicant``-työkalun ``.conf``-asetustiedosto. Sen avulla määritetään, mihin WiFi-verkkoon yhdistetään ja millä asetuksilla. (NetBSD man-pages, wepa_supplicant.conf)

    Esimerkki .conf-tiedostosta:
    network={
        ssid="wifi-free"
        key_mgmt=NONE
        scan_ssid=1
    }

- ``network={ ... }`` määrittelee yhden verkon asetukset. Kaikki aaltosulkeiden sisällä oleva koskee tätä yhtä WiFi-verkkoa.
- ``ssid="wifi-free"`` määrittelee verkon nimen, johon halutaan yhdistää. Tässä verkon nimi on ``wifi-free``.
- ``key_mgmt=NONE`` määrittelee avainhallinnan. Arvo ``NONE`` tarkoittaa, että verkko on avoin eikä siihen tarvita salasanaa.
- ``scan_ssid=1`` määrittelee, että verkkoa etsitään aktiivisesti sen nimellä. Tätä käytetään piilotetuissa verkoissa, joissa SSID ei näy tavallisessa skannauksessa.

Komento, joka käynnistää ``wpa_supplicant``-työkalun ja yhdistää laitteen ``free.conf``-tiedostossa määriteltyyn WiFi-verkkoon:

    $ sudo wpa_supplicant -Dnl80211 -iwlan2 -c free.conf

- ``Dnl80211`` määrittää käytettäväksi ajuriksi ``nl80211``. ``-D``-valitsimella kerrotaan, mitä ajuria käytetään. (linux.die.net, wpa_supplicant)
- ``iwlan2`` määrittää käytettävän verkkoliitännän. Tässä yhteys muodostetaan verkkokortilla ``wlan2``. ``-i``-valitsin kertoo, mitä interfacea kuunnellaan ja käytetään. (linux.die.net, wpa_supplicant)
- ``-c free.conf`` määrittää käytettävän asetustiedoston. ``-c``-valitsimella annetaan polku konfiguraatiotiedostoon. (linux.die.net, wpa_supplicant)

___

#### Dhclient

``dhclient`` on DHCP-asiakasohjelma, jolla laite pyytää DHCP-palvelimelta IP-osoitteen ja muita verkon tietoja, kuten oletusreitittimen ja nimipalvelimen tiedot (linux.die.net, dhclient).

    Esimerkkikomento:
    sudo dhclient wlan2 -v

- ``sudo dhclient`` käynnistää DHCP-asiakasohjelman pääkäyttäjän oikeuksilla. DHCP-asiakasohjelman tehtävä on pyytää verkosta IP-osoite ja muut tarvittavat verkkoasetukset.
- `` wlan2`` määrittelee verkkoliitännän, jolle IP-osoite haetaan.
- ``-v`` tarkoittaa verbose-tilaa eli näyttää tarkemmin, mitä komento tekee.

___

### PSK

Tässä osiossa harjoiteltiin WPA/WPA2-PSK-suojatun verkon tutkimista. PSK tarkoittaa Pre-Shared Key -mallia, jossa tukiasemalla ja client-laitteilla on käytössä sama ennalta jaettu avain eli käytännössä WiFi-salasana. Tätä jaettua avainta/salasanaa käytetään clientin ja tukiaseman välisen liikenteen salaamiseen ja purkamiseen. Toisin kuin OPN-verkossa, PSK-verkossa liikenne on siis salattua, joten pelkkä liikenteen kuuntelu ei riitä sisällön tarkasteluun. (Portnox, Exploring WPA-PSK and WiFi Security)

___

#### Aireplay-ng

``Aireplay-ng`` on Aircrack-ng-paketin työkalu, jota käytetään WiFi-kehysten lähettämiseen eli injektiointiin. Työkalun tarkoitus on tuottaa liikennettä, jota voidaan myöhemmin käyttää esimerkiksi WEP- ja WPA-PSK-avainten murtamisessa. Työkalu tukee useita eri toimintoja, joista yksi on ``deauthentication``. (Aircrack-ng, aireplay-ng)

``Deauthenticationin`` tarkoituksena oli pakottaa client-laite irtoamaan tukiasemasta hetkeksi, jolloin se yhdistää takaisin verkkoon. Uudelleenyhdistämisen aikana ``airodump-ng`` voi kaapata WPA/WPA2-handshaken. Tätä menetelmää voidaan käyttää WPA/WPA2-handshaken kaappaamiseen. (Aircrack-ng, deauthentication)

    Esimerkki:
    $ sudo aireplay-ng -0 10 -a F0:9F:C2:71:22:12 wlan0mon

Komennossa:
- ``sudo aireplay-ng`` käynnistää kehyksien lähettämiseen käytettävän työkalun pääkäyttäjän oikeuksilla.
- ``-0`` tarkoittaa deauthentication-toimintoa.
- ``10`` määrittää lähetettävien deauthentication-pakettien määrän.
- ``-a F0:9F:C2:71:22:12`` määrittää kohteena olevan tukiaseman BSSID:n.
- ``wlan0mon`` on monitor mode -tilassa oleva verkkokortti.

___

#### Aircrack-ng

``Aircrack-ng`` on työkalu, jolla voidaan yrittää murtaa WEP- ja WPA/WPA2-PSK-avaimia kaapatuista tiedostoista. WPA/WPA2-PSK-verkoissa tämä perustuu siihen, että handshake on saatu kaapattua ja salasanaa kokeillaan sanalistasta. (Aircrack-ng, aircrack-ng)

    Esimerkki:
    $ aircrack-ng ~/wifi/scanc6-02.cap -w ~/rockyou-top100000.txt

Komennossa:
- ``aircrack-ng`` käynnistää salasanan murtamiseen käytettävän työkalun.
- ``~/wifi/scanc6-02.cap`` on kaapattu tiedosto, jossa handshake sijaitsee.
- ``-w ~/rockyou-top100000.txt`` määrittää käytettävän sanalistan.

___

## c) Onko suhtautumiseni WLanin turvallisuuteen muuttunut?

Ennen harjoitusta en ollut miettinyt WLAN-turvallisuutta kovin syvällisesti. WiFi oli minulle ehkä lähinnä sellainen mystinen kodin näkymätön taikakenttä, joka joko toimii tai sitten reititin irrotetaan seinästä ja toivotaan parasta. Turvallisuuden osalta ajatus oli melko yksinkertainen: jos verkossa on salasana, naapurit pysyvät poissa, eikä kukaan pääse varastamaan nettiä tai tekemään televisiosta naapurin omaa YouTube-soitinta.

Harjoituksen jälkeen ymmärsin paremmin, että langaton verkko ei ole täysin suljettu tai näkymätön ympäristö. Vaikka verkkoon ei liittyisi, siitä voi silti havaita paljon metatietoa. Esimerkiksi tukiasemat, kanavat, MAC-osoitteet ja asiakaslaitteet voivat näkyä jo pelkän passiivisen kuuntelun perusteella.

Piilotettu verkko hieman yllätti. Olin aiemmin ajatellut, että piilotettu SSID tarjoaisi hieman enemmän lisäsuojaa. Harjoituksen perusteella ymmärsin, että näin ei käytännössä ole. Näkymättömyysviitan sijaan todellisuudessa se onkin vähän kuin ihminen, joka yrittää piiloutua verhon taakse, mutta jättää kengät näkyviin. Verkon nimeä ei näytetä suoraan, mutta sen olemassaolo, tukiaseman tiedot ja muu metatieto voidaan silti havaita.

Salasanojen merkitys ei sinänsä tullut uutena asiana, sillä olen ollut jo aiemmin tietoinen vahvojen salasanojen tärkeydestä. Harjoitus kuitenkin konkretisoi asian WLAN-verkkojen näkökulmasta. Salasana on edelleen tärkeä, mutta jos se on tyyliä salasana123, qwerty tai koiran nimi syntymävuodella, tietoturva seisoo lähinnä toivon ja kohtalon varassa. Harjoitus osoitti käytännössä, että heikko salasana voi löytyä sanalistasta, jolloin “suojattu verkko” ei välttämättä olekaan niin suojattu kuin olohuoneessa istuessa helposti ajattelisi.

Kokonaisuutena suhtautumiseni WLAN-turvallisuuteen muuttui huolettomasta selvästi varovaisemmaksi. WiFi ei ole enää vain taustalla huriseva kodin mukavuuspalvelu, vaan oikea verkko, joka juttelee ympäristölleen enemmän kuin ehkä olisi tarpeen. Naapurit ehkä pysyvät edelleen poissa, mutta nyt ymmärrän paremmin, ettei tietoturvaa kannata rakentaa pelkän hyvän tuurin, oletusasetusten ja reitittimen vilkkuvien valojen varaan.

___

## Lähteet

WifiChallenge Labs
- Walkthrough v2.0: https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/

Aircrack-ng
- Airmon-ng: https://www.aircrack-ng.org/doku.php?id=airmon-ng
- Airodump-ng: https://www.aircrack-ng.org/doku.php?id=airodump-ng
- Aireplay-ng: https://www.aircrack-ng.org/doku.php?id=aireplay-ng
- Deauthentication: https://www.aircrack-ng.org/doku.php?id=deauthentication
- Aircrack-ng: https://www.aircrack-ng.org/doku.php?id=aircrack-ng
  

Aircrack-ng Github
- mdk4: https://github.com/aircrack-ng/mdk4

man7.org Linux man-pages
- awk(1p): https://man7.org/linux/man-pages/man1/awk.1p.html

NetBSD Manual Pages
- wpa_supplicant(8): https://man.netbsd.org/wpa_supplicant.8
- wpa_supplicant.conf(5): https://man.netbsd.org/wpa_supplicant.conf.5

Linux Die.net - Linux man page
- wpa_supplicant(8): https://linux.die.net/man/8/wpa_supplicant
- dhclient(8): https://linux.die.net/man/8/dhclient

Portnox, Exploring WPA-PSK and WiFi Security
- What is PSK in WPA?: https://www.portnox.com/cybersecurity-101/network-security/wpa-psk/



