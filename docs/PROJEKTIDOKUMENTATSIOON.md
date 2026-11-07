# Arduino Mängulaud

**Õppeaine:** IFI6203.DT Robootika  
**Autorid:** Marcus Puust, Jan Aaron Einloo  
**Semester:** 2026 sügis

---

## Projekti eesmärk ja loodava seadme lühikirjeldus

Arduino Mängulaud on ruudukujuline elektrooniline mängulaud, mille igas nurgas on nupp ja selle juures LED-tuli. Laual on kaks mängurežiimi, mille vahel saab valida LCD-ekraanil kuvatavast menüüst. Menüüs liigutakse pöördenupu abil, millega valitakse ka raskusaste.

- **Simon Says (mälumäng).** Seade näitab tulede ja helide abil järjestikku süttivate nurkade jada. Mängija peab sama jada nuppudega õiges järjekorras kordama. Iga õnnestunud vooru järel lisandub jadasse üks samm. Mäng lõpeb esimese vea korral ja LCD kuvab saavutatud taseme.
- **Reaktsioonimäng (kiirusmäng).** Pärast juhusliku pikkusega ooteaega süttib ühe nurga tuli ja mängija peab võimalikult kiiresti vajutama selle nurga nuppu. Seade mõõdab reaktsiooniaja millisekundites ja kuvab selle LCD-l koos parima tulemusega. Liiga varajane või vale nupu vajutus loetakse veaks.

**Miks me seda teeme?** Projekti eesmärk on luua lihtne, aga mitmekülgne interaktiivne seade, mis kasutab mitut erinevat sisend- ja väljundseadet ning nõuab läbimõeldud juhtloogikat: jadade genereerimist ja salvestamist, ajastust, nuppude lugemist (sh põrke kõrvaldamist), menüüsüsteemi ning helilist ja visuaalset tagasisidet. Kaks mängurežiimi ühes seadmes näitavad, kuidas sama riistvara saab tarkvara abil kasutada erinevatel eesmärkidel. Valmis seade on ühtlasi meelelahutuslik ja sobib hästi esitlemiseks, sest publik saab seda ise proovida.

---

## Sisendite loetelu

| Sisend | Mida tuvastatakse | Millega tuvastatakse |
|---|---|---|
| Mängunupu vajutus (4 tk, iga nurk eraldi) | Milline nurk vajutati ja millal (ajatempel ms täpsusega) | 4 nuppu, ühendatud Arduino digitaalsisenditega (`INPUT_PULLUP`), loetakse `digitalRead()` abil |
| Menüüs liikumine, raskusastme valik, mängu alustamine ja taaskäivitamine | Pöörlemise suund (järgmine või eelmine valik), lühike ja pikk vajutus | Pöördenupp KY-040: CLK ja DT signaalide järjekorrast tuvastatakse pöörlemise suund, nupp (SW) loetakse `digitalRead()` abil |

*Märkus: reaktsiooniaja mõõtmiseks kasutatakse Arduino sisemist taimerit (`millis()`), mis ei ole eraldi füüsiline sisendseade.*

---

## Väljundite loetelu

| Väljund | Mida edastatakse | Millega edastatakse |
|---|---|---|
| Valgussignaal (4 tk, iga nurk eraldi) | Simon Says jada näitamine, reaktsioonimängu signaal, vajutuse kinnitus | 4 LED-i (soovitavalt eri värvi), digitaalväljundid läbi takistite |
| Heli | Iga nurga oma toon, vea- ja võiduhelid, menüüklõpsud | Passiivne piezo, juhitakse `tone()` funktsiooniga |
| Tekstiinfo | Menüü, raskusaste, tase, skoor, reaktsiooniaeg, parim tulemus, mängu lõpu teade | LCD 16x2 ekraan I2C-mooduliga |

---

## Nõuded loodavale seadmele

### Üldised

- Seadme käivitamisel kuvatakse LCD-l menüü, kus saab valida Simon Says ja Kiiruse mang vahel.
- Menüüs liigutakse pöördenupu (KY-040) keeramisega: päripäeva järgmine valik, vastupäeva eelmine valik.
- Pärast mängu valikut kuvatakse raskusastme valik (Lihtne, Keskmine, Raske), mida muudetakse samuti keeramisega.
- Pöördenupu lühike vajutus kinnitab valiku või alustab mängu.
- Pöördenupu pikk vajutus (umbes 2 s) taaskäivitab mängu ajal sama mängu ja viib menüüs eelmise sammu juurde.
- Mängu lõpuekraanil alustab lühike vajutus uue mängu sama raskusega ja pikk vajutus viib tagasi menüüsse.
- Raskusaste mõjutab Simon Says režiimis jada näitamise kiirust ja vastamise ajalimiiti (Lihtne 7 s, Keskmine 5 s, Raske 3 s) ning reaktsioonimängus ooteaja vahemikku.
- Mängu ajal kuvab LCD valitud raskusastet (nt „Tase: 5  Raskus: K“).
- Nurganuppe kasutatakse ainult mängimiseks.
- Iga nupuvajutusega kaasneb sama nurga LED-i süttimine ja sellele nurgale omane toon.
- Nupuvajutuste puhul kõrvaldatakse kontaktipõrge (debounce), et üks vajutus loetaks ainult üheks.

### Simon Says režiim

- Vooru alguses näidatakse kogu senine jada: iga samm on LED + toon, seejärel lühike paus.
- Iga voor lisab jadasse ühe juhusliku nurga.
- Kui mängija kordab kogu jada õigesti, kuvatakse LCD-l uus tase ja algab järgmine voor.
- Kui mängija vajutab vale nuppu või ei vajuta määratud aja jooksul, kõlab veaheli, kõik LED-id vilguvad ja LCD kuvab teate „Mang labi!“ koos saavutatud tasemega.
- Kõrgemal tasemel lüheneb jada näitamise kiirus.

### Reaktsioonimängu režiim

- LCD kuvab „Valmis...“ ja seade ootab juhusliku aja (nt 1–4 s).
- Seejärel süttib juhuslikult valitud nurga LED ja kõlab signaal.
- Kui mängija vajutab õiget nuppu, kuvab LCD reaktsiooniaja millisekundites („Aeg: 287 ms“). Kui see on parim tulemus, kuvatakse ka „Uus rekord!“.
- Kui mängija vajutab enne signaali, kuvatakse „Liiga vara!“ ja kõlab veaheli.
- Kui mängija vajutab vale nuppu, kuvatakse „Vale nupp!“ ja kõlab veaheli.
- Parim tulemus jääb meelde seni, kuni seade on sisse lülitatud.

---

## Süsteemi füüsiliste komponentide loetelu

*Esialgne loetelu, mudelinumbrid ja andmelehtede viited täpsustatakse komponentide hankimisel.*

| Komponent | Kogus | Mudel / märkused | Andmeleht |
|---|---|---|---|
| Mikrokontroller | 1 | Arduino Uno R3 (või ühilduv) | |
| Nupp | 4 | Taktiilne nupp 12x12 mm või arkaadnupp, täpsustamisel | lisada |
| LED | 4 | 5 mm LED (punane, roheline, sinine, kollane) | lisada |
| Takisti LED-idele | 4 | 220 Ω | – |
| Piesokõlar | 1 | Piesokõlar (passiivne piesoelement), mudel täpsustamisel | lisada |
| LCD-ekraan | 1 | 16x2 LCD (HD44780) + I2C moodul (PCF8574) | lisada |
| Pöördenupp (rotary encoder) | 1 | KY-040, sisseehitatud nupuga | lisada |
| Arenduslaud ja juhtmed | – | Breadboard, jumper-juhtmed | – |
| Mängulaua korpus | 1 | Ruudukujuline alus, nupud nurkades, materjal 3D prinditud | – |

### Põhjendus komponentide valiku kohta

- **Arduino Uno R3:** kursusel kasutatav platvorm, piisavalt digitaalviike kõigi komponentide jaoks, palju dokumentatsiooni ja teeke.
- **Nupud:** lihtsaim ja töökindlaim sisendseade. Sisemise pull-up takisti (`INPUT_PULLUP`) kasutamisel pole lisatakisteid vaja.
- **Eri värvi LED-id:** igal nurgal on oma värv, mis teeb Simon Says jada meeldejätmise lihtsamaks.
- **Piesokõlar:** juhitakse `tone()` funktsiooniga, nii et saab mängida eri sagedusega toone ja igal nurgal on oma heli. Piesokõlar on õhuke, odav ja tarbib vähe voolu, seega saab selle ühendada otse Arduino viiku.
- **LCD 16x2 I2C-ga:** kuvab menüüd ja tulemusi. I2C-ühendus vajab ainult kahte andmeviiku (SDA, SCL), mis jätab ülejäänud viigud vabaks.
- **Pöördenupp KY-040:** üks komponent menüüs liikumiseks, raskusastme valikuks ning mängu alustamiseks ja taaskäivitamiseks. Erinevalt tavalisest nupust tuvastab see ka pöörlemissuuna, seega on see eraldi tüüpi sisendseade.

---

## Süsteemi füüsiliste komponentide ühendusskeem

*Fritzingu skeem lisatakse faili `hardware/wiring-diagram.png`. Esialgne viikude jaotus:*

| Komponent | Arduino viik |
|---|---|
| Nupp 1 (ülemine vasak) | D2 |
| Nupp 2 (ülemine parem) | D3 |
| Nupp 3 (alumine vasak) | D4 |
| Nupp 4 (alumine parem) | D5 |
| LED 1 (ülemine vasak) | D6 → 220 Ω → LED → GND |
| LED 2 (ülemine parem) | D7 → 220 Ω → LED → GND |
| LED 3 (alumine vasak) | D8 → 220 Ω → LED → GND |
| LED 4 (alumine parem) | D9 → 220 Ω → LED → GND |
| Piesokõlar | D10 (+), GND (−) |
| LCD I2C – SDA | A4 |
| LCD I2C – SCL | A5 |
| LCD I2C – VCC / GND | 5V / GND |
| Pöördenupp – CLK | D11 |
| Pöördenupp – DT | D12 |
| Pöördenupp – SW (nupp) | A1 (digitaalsisendina, `INPUT_PULLUP`) |
| Pöördenupp – + / GND | 5V / GND |

Nupud on ühendatud viigu ja GND vahele. Vajutamata olekus loeb viik `HIGH` ja vajutatult `LOW`.

---

## Süsteemi juhtiv (pseudo)kood koos kommentaaridega

*Lisatakse järgmises etapis.*
