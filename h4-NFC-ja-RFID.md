# h4 - NFC ja RFID

## a) Käytössäni olevat RFID-tuotteet

*Tarkastele käytössäsi olevia RFID tuotteita, mieti miten hyvin olet suojautunut RFID urkinnalta?*

Mielestäni olen suojautunut RFID-urkintaa vastaan melko hyvin. Käytössäni olevat merkittävät tuotteet:
- Pankkikortit, joita säilytän aivan tavallisessa lompakossa.
  - Suojaus perustuu lähinnä kortteihin asetettuihin mataliin käyttörajoihin, sekä sananlaskuun "tyhjästä on paha nyhjästä".
- Puhelimen NFC-toiminto ei yleensä ole päällä, joten se ei muodosta merkittävää riskiä.
- Passi on kotona jemmassa.

Käytössäni ei ole kulkutageja, matkakortteja, älyavaimia tai älykelloja, jotka lisäisivät RFID-urkinnan mahdollisuutta. Suurin mahdollinen riskikohta ovat pankkikortit, ja tämänkin pienen riskin 

## b) APDU?

*Tutustu APDU komentojen rakenteeseen (voit käyttää tekoälyä tutustumiseen).*

- APDU (Application Protocol Data Unit) on älykorttien käyttämä viestimuoto.
- APDU:n avulla kortinlukija tai muu laite lähettää komentoja kortille ja vastaanottaa kortin vastauksia.
  - APDU-viestien päätyypit:
    - Command APDU - kortille lähtevä komento.
    - Response APDU - kortin palauttama vastaus.
- APDU-rakenne perustuu ISO/IEC 7816-4 -standardiin

https://www.cardlogix.com/glossary/apdu-application-protocol-data-unit-smart-card/
https://www.passninja.com/tutorials/nfc-protocols/what-are-apdus
https://docs.oracle.com/en/java/javacard/3.1/jc_api_srvc/api_classic/javacard/framework/APDU.html?utm_source=chatgpt.com

## c) Tutki ja kerro minkä mielenkiintoisen RFID hakkerointi uutiset löysit.

## Lähteet
