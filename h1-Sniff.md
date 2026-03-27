# h1 Sniff

Tarkemmat tehtävänannot viikon läksyihin löytyvät [täältä](https://terokarvinen.com/verkkoon-tunkeutuminen-ja-tiedustelu/#h1-sniff).

## x) Lue ja tiivistä
### x-1) Tero Karvinen, Wireshark - Getting Started

Ensimmäinen lue ja tiivistä -tehtävän artikkeli oli Karvisen [Wireshark - Getting Started](https://terokarvinen.com/wireshark-getting-started/).
- #### Artikkelin tavoite
  - Opettaa asentamaan Wireshark Debianiin tai Debian-pohjaiseen distroon.
  - Opettaa kaappaamaan verkkoliikennettä ja tarkastelemaan tilastoja käyttäen suodattimia DNS- ja HTTP-liikenteen analysointiin.
- #### Asennuksesta ja käyttöönototsta
  - Asennus päivitysten jälkeen komennolla ``$ sudo apt-get install wireshark``.
  - Sallitaan ei-root -käyttäjien kaapata paketteja.
    - Sertifikaattivirheet voivat johtua väärästä kellonajasta, korjaa käyttäen ``$ timedatectl``-komentoa.
  - Lisää käyttäjä wireshark -ryhmään.
    - ``$ sudo adduser <käyttäjä> wireshark``
    - ``$ whoami``-komennolla selvität kuka olet, jos syystä tai toisesta olet unohtanut kyseisen tiedon.
  - Ryhmän saa käyttöön reloggaamalla tai käyttämällä komentoa ``$ newgrp wireshark``.
  - Viimeisenä käynnistetään ohjelman GUI komennolla ``$ wireshark``.
- #### Sniffaus, eli liikenteen kaappaus
  -  Valitaan verkkoliitäntä (interface), jossa on liikennettä.
  -  Vaihtoehto: käytä ``any`` kaikkeen näkyvään liikenteeseen.
  -  Sininen hain evä -ikoni käynnistää kaappauksen, punainen neliö -ikoni pysäyttää kaappauksen.
- #### Vianmääritys
  - Ei liikennettä? Luo liikennettä esimerkiksi selaamalla verkkoa.
  - Ei vieläkään? Tarkista ryhmäjäsenyys.
- #### Tallennus ja lataus
  - Kaappaukset tallennetaan .pcap -tiedostoina.
  - Tallennettua dataa voidaan analysoida kuten live-liikennettä.
- #### Tilastot ja analyysi
  - Endpoints - listaa hostit.
  - I/O Graphs - Liikenteen ajoitus.
  - Protocol hierarchy - protokollajakauma.
    - Huom! Poistamalla suodattimet näkee koko datan.
- #### Filtterit, suodattimet
  - Yleisimmät esimerkkisuodattimet:
    - dns - DNS-kyselyt.
    - tls - salattu liikenne (yleisin webissä).
    - http - salaamaton HTTP.
    - tcp.port == 443 - porttipohjainen suodatus.
    - ip.addr == x.x.x.x - IP-osoitteella suodatus.
    - frame contains "teksti" - merkkijonohaku
- #### TCP-keskustelu tekstinä
  - Pakettia hiiren oikealla klikattaessa voi nähdä koko TCP-yhteyden keskustelun tekstimuotoisena näkymänä.
  - Toimii erityisesti salaamattomassa liikenteessä.
  - Kätevä esimerkiksi hyökkääjien käyttämien TCP-yhteyksien (esim. netcat) tarkasteluun.
- #### Mitä tästä opin?
  - Wiresharkilla voidaan analysoida verkkoliikennettä tehokkaasti.
  - Tarkkuus paranee rajaamalla dataa käyttäen suodattimia.
  - Salaamaton liikenne on suoraan luettavissa, salattuun tarvitaan lisää työkaluja.
### x-2) Tero Karvinen, Network Interface Names on Linux

## Lähteet

Tero Karvinen
- https://terokarvinen.com/verkkoon-tunkeutuminen-ja-tiedustelu/#h1-sniff
- https://terokarvinen.com/wireshark-getting-started/
- https://terokarvinen.com/network-interface-linux/
