# Handleiding: Zendure HA-GP013 (nul op de meter)

[← README](../README.md)

Deze handleiding gaat stap voor stap van installatie tot batterijen die je aansluiting rond 0 W
houden. Kom je van Zendure-HA, lees dan eerst [stap 3](#3-overstappen-van-zendure-ha). Reken op 20 minuten.

**Inhoud**

1. [Wat je nodig hebt](#1-wat-je-nodig-hebt)
2. [Hoe het werkt](#2-hoe-het-werkt)
3. [Overstappen van Zendure-HA](#3-overstappen-van-zendure-ha)
4. [Nieuwe installatie](#4-nieuwe-installatie)
5. [De P1-meter instellen](#5-de-p1-meter-instellen)
6. [Dagelijks bedienen: Zendure Manager](#6-dagelijks-bedienen-zendure-manager)
7. [Wolkenmodus (wisselvallig weer)](#7-wolkenmodus-wisselvallig-weer)
8. [Het optiemenu](#8-het-optiemenu)
9. [Meerdere omvormers en richtingwissel](#9-meerdere-omvormers-en-richtingwissel)
10. [Zonneverwachting](#10-zonneverwachting)
11. [Meldingen, diagnostiek en opnames](#11-meldingen-diagnostiek-en-opnames)
12. [Problemen oplossen](#12-problemen-oplossen)

---

## 1. Wat je nodig hebt

- Home Assistant **2025.4 of nieuwer**.
- Een **Zendure-app-token** ([zo krijg je die](https://github.com/Zendure/Zendure-HA/wiki/Installation)).
- Voor *Slimme Matching*: een P1-vermogenssensor (W) of, het liefst, een **HomeWizard P1-meter**.
- Aanrader: een vast IP-adres (DHCP-reservering) voor elke omvormer en voor de P1-meter.

## 2. Hoe het werkt

De **Zendure Manager** leest je P1-meter (bij een HomeWizard elke seconde) en verdeelt het
benodigde vermogen over je omvormers:

- Minstens elke 4 seconden een regelcyclus, en **direct** een nieuwe bij een sprong van 400 W
  (waterkoker, kookplaat).
- Zo **weinig mogelijk omvormers** tegelijk; één omvormer staat klaar om te laden, één om te ontladen.
- **Zuinig voor de hardware:** opdrachten gaan naar het RAM (flash hooguit eens per uur), bij elke
  richtingwissel eerst een bevestigde 0 W, en omvormers die niets doen gaan slapen (≈30 W → 5 W).

## 3. Overstappen van Zendure-HA

Beide integraties gebruiken het domein `zendure_ha`. Zendure HA-GP013 neemt de bestaande
configuratie over: je apparaten, entiteit-id's, geschiedenis, energiedashboard en instellingen
blijven. Ze kunnen niet naast elkaar draaien.

> ⚠️ **Gebruik niet *Verwijderen* in HACS, en verwijder de integratie nooit onder *Instellingen →
> Apparaten & diensten*.** Recente versies van HACS en Home Assistant verwijderen dan ook de
> configuratie, en daarmee je instellingen en entiteiten.

Je laat HACS daarom stoppen met het volgen van Zendure-HA en downloadt deze integratie over de bestaande bestanden heen:

1. **Maak een volledige back-up** (*Instellingen → Systeem → Back-ups*).
2. **Voeg deze repository toe aan HACS**, maar download nog niet: HACS → ⋮ → *Aangepaste repositories* →
   `https://github.com/GruttePier013/Zendure-HA-GP013`, type *Integratie*.
3. **Stop Home Assistant** en markeer Zendure-HA als niet geïnstalleerd in de opslag van HACS. Met de
   add-on *Terminal & SSH*:

   ```sh
   ha core stop
   cd /config/.storage
   cp hacs.repositories hacs.repositories.bak
   jq '(.data[] | select(.full_name == "Zendure/Zendure-HA")) |= (.installed = false | del(.version_installed, .installed_commit))' \
     hacs.repositories > /tmp/hacs.repositories && mv /tmp/hacs.repositories hacs.repositories
   ha core start
   ```

   Home Assistant moet gestopt zijn terwijl je dit aanpast, anders schrijft HACS bij het afsluiten de
   oude stand terug. Zonder `jq`: open `hacs.repositories`, zoek de regel met
   `"full_name": "Zendure/Zendure-HA"` en zet `"installed": false`.
4. **HACS → Zendure HA-GP013 → Downloaden** (nieuwste release). HACS schrijft de bestanden over de
   bestaande `custom_components/zendure_ha`; er wordt niets verwijderd.
5. **Herstart Home Assistant.** De integratie laadt met de bestaande configuratie. Loop nieuwe
   instellingen na onder *Configureren*.

Zendure-HA staat in HACS daarna als niet geïnstalleerd. **Download het nooit opnieuw**: dan wordt deze
integratie overschreven.

**Is de configuratie toch al verwijderd?**

1. Zet een **back-up van vóór het verwijderen** terug (alleen het Home Assistant-deel) en volg daarna de stappen hierboven.
2. **Geen back-up:** download Zendure HA-GP013, herstart en voeg de integratie opnieuw toe met hetzelfde
   Zendure-token. Home Assistant onthoudt verwijderde entiteiten een tijd en geeft ze hun oude id terug,
   zodat geschiedenis en energiedashboard doorlopen. Controleer of geen entiteit een `_2` kreeg.
3. Vul de instellingen opnieuw in onder *Configureren*. Stond de P1-opname aan, dan staan al je
   instellingen in `<config>/zendure_ha/p1_<datum>.meta.json`. Zet ook de zekeringgroep per apparaat en
   de entiteiten van de Zendure Manager (modus, P1-doel, handmatig vermogen) terug.

Meer (Engels): [Switching from Zendure-HA](migration.md).

## 4. Nieuwe installatie

[![Open in HACS](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=GruttePier013&repository=Zendure-HA-GP013&category=integration)

1. HACS → ⋮ → *Aangepaste repositories* → `https://github.com/GruttePier013/Zendure-HA-GP013`, type *Integratie*.
2. Zoek **Zendure HA-GP013** → *Downloaden* → herstart Home Assistant.
3. [![Integratie toevoegen](https://my.home-assistant.io/badges/config_flow_start.svg)](https://my.home-assistant.io/redirect/config_flow_start/?domain=zendure_ha)
   en vul je Zendure-app-token in.

Instellen gebeurt op twee plekken:

| Waar | Wat | Wanneer |
|---|---|---|
| 🎛️ **Apparaatpagina van de *Zendure Manager*** | modus en doelen; entiteiten voor je dashboard en automatiseringen | dagelijks |
| ⚙️ **Optiemenu** (*Zendure HA-GP013 → Configureren*) | hoe de regeling werkt | eenmalig |

## 5. De P1-meter instellen

*Configureren → P1-meter*:

1. Kies je **P1-sensor** (vermogen in W). Die is altijd de terugval.
2. **HomeWizard elke seconde uitlezen** (aanrader): vul het IP-adres van de P1-meter in. Het adres uit
   de HomeWizard-integratie wordt voorgesteld.
   - Gebruikt de HomeWizard-integratie al API v2, dan wordt dat token hergebruikt.
   - Anders: druk op de knop van de P1-meter en kies binnen 30 seconden *Versturen*.
   - Of vink *API v1* aan en zet *Lokale API* aan in de HomeWizard-app.

Zet daarna op de Zendure Manager de **Bedrijfsmodus** op **Slimme Matching**.

## 6. Dagelijks bedienen: Zendure Manager

| | Entiteit | Waarden | Wat het doet |
|---|---|---|---|
| 🔀 | **Bedrijfsmodus** | Uit · Handmatig Vermogen · Slimme Matching · Slim ontladen · Slim opladen · Zonne-energie opslaan | *Slimme Matching* houdt de meter op 0 W in beide richtingen. *Slim ontladen* / *Slim opladen* doen één richting. *Zonne-energie opslaan* laadt alleen met overschot. *Handmatig Vermogen* draait een vast vermogen. |
| ✋ | **Handmatig vermogen** | −12000 … +12000 W | **+** ontlaadt naar het huis, **−** laadt. Verdeeld over de omvormers naar laadniveau; een aandeel verschuift pas als het meer dan 5 % van deze waarde verandert. |
| 🎯 | **P1-doel (band)** | −200 … +200 W, standaard 0 | Waar de regeling op mikt. **0**: nul op de meter. **−50**: tot 50 W teruglevering mag tijdens laden of stilstand (minder opdrachten). **+50**: tot 50 W afname mag tijdens ontladen. |
| 🌥️ | **Wolkenmodus (wisselvallig) bediening** | Automatisch (gedetecteerd) · Aan · Uit | Zie stap 7. |
| ☁️ | **P1-doel (wolken)** | 0 … −500 W, standaard −100 | De teruglevering waar het laden op mikt in de wolkenmodus. |

Het zijn entiteiten: een automatisering kan ze aanpassen, bijvoorbeeld het P1-doel op basis van de
zonneverwachting of de terugleverprijs.

## 7. Wolkenmodus (wisselvallig weer)

Bij snel wisselende bewolking springt je zonne-opbrengst heen en weer. De wolkenmodus vangt dat op:

- **Automatisch:** de manager telt flanken in de P1-waarde over 10 minuten. Bij 6 of meer gaat de
  modus aan; bij 4 of minder (of geen zon) uit.
- **Aan** / **Uit:** jij beslist; de manager schakelt pas weer zelf als je *Automatisch* kiest.

Als de modus aan staat:

- ☁️ mikt het laden op het **P1-doel (wolken)**: een voorbijdrijvende wolk kost dan wat teruglevering in plaats van afname;
- ⏱️ laadt een omvormer na 10 s weer in plaats van na 60 s;
- 🔌 start een tekort onder 150 W geen omvormer;
- 🔋 blijft één omvormer klaarstaan om te ontladen.

De binaire sensor **Wolkenmodus (wisselvallig)** (diagnose) laat zien of de modus nu aan staat,
met de attributen `source` (`auto` / `user`) en `detected`. Beide doelen werken samen: de band geldt
altijd, in de wolkenmodus telt ook het wolkendoel, en de grootste teruglevering geldt (ze tellen niet op).

## 8. Het optiemenu

*Zendure HA-GP013 → Configureren*. De standaardwaarden zijn een goed begin.

**⚡ Vermogensregeling**

| Instelling | Standaard | Wat het doet |
|---|---|---|
| Normaal updateinterval | 4 s | minstens zo vaak een regelcyclus |
| Snel updateinterval | 2,2 s | de volgende cyclus na een duidelijke P1-verandering |
| Minimale P1-verandering | 15 W | kleinere veranderingen tellen niet als "duidelijk" |
| Minimaal vermogen om een apparaat te starten | 50 W | een stilstaande omvormer start pas vanaf dit vermogen |
| Vermogenstolerantie apparaat | 15 W | alleen een nieuwe opdracht als die meer dan dit afwijkt van het gemeten vermogen |
| Updateinterval in handmatige modus | 15 s | hoe vaak *Handmatig vermogen* wordt bijgewerkt |
| Piekbegrenzing | 0 W (uit) | de batterijen ontladen pas boven deze afname |
| Piekdetectie | 400 W | een P1-sprong van deze grootte start direct een cyclus (0 = uit) |

**🔄 Richtingwissel-bescherming**

| Instelling | Standaard | Wat het doet |
|---|---|---|
| Wachttijd bij richtingwissel | 6 s | *Via 0 W*: tussen laden en ontladen eerst 0 W; met een gemeten 0 W mag hij na 2 s draaien, zonder meting wacht hij zo lang. Niet gebruikt bij *Richtingswissel: Direct* (stap 9). |
| Minimale looptijd van een gestart apparaat | 10 s | een gestarte omvormer houdt zo lang een laag vermogen, zodat het relais niet klappert |

Verder in het menu: **meerdere omvormers** (stap 9), **zonvoorspelling** (stap 10), **P1-meter**
(stap 5), **Account / token** en **MQTT & Wi-Fi** (cloud- of lokale broker), **apparaatadressen** (vast IP per
zenSDK-omvormer) en **logging, simulatie en testdata** (stap 11).

Alle details (Engels): [Configuration](configuration.md).

## 9. Meerdere omvormers en richtingwissel

| Instelling | Standaard | Wat het doet |
|---|---|---|
| Spreiddrempel | 85 % | een volgende omvormer start pas als de draaiende dit deel van hun limiet leveren (één SolarFlow 2400 tot ongeveer 2000 W) |
| Rolwisselmarge | 10 % | een rol (laden/ontladen) verhuist pas als een andere omvormer zoveel meer laadniveau heeft (en na 5 min) |
| Veilige stand (0 W) in flash | aan | 0 W eenmaal in de flash, zodat een herstarte omvormer op 0 W begint (hooguit eens per uur) |
| Stilstaande omvormers laten slapen na | 10 min | een omvormer die zo lang op 0 W staat, gaat slapen (0 = uit); de ontlader blijft wakker |
| Ontlader alleen wakker houden om | leeg (altijd) | uren waarin de ontlader wakker blijft, bijv. `7-9, 12-13, 17-19` |

**Richtingwissel per omvormer (nieuw in 1.7.0):** elke zenSDK-omvormer (SolarFlow 800 / 1600 / 2400 / 4000)
heeft naast *AC Input Limit*, *Output Limit* en *AC Operation Mode* de keuze **Richtingswissel**:

- **Via 0 W** (standaard): stoppen, wachten op een gemeten 0 W (minstens 2 s), dan starten in de andere
  richting. Dat starten kost de omvormer 9–11 s.
- **Direct:** een draaiende omvormer schakelt alleen zijn AC-modus om en loopt vanzelf door 0 W heen
  (≈ 230 W/s). De manager draait zo'n omvormer om in plaats van een andere te starten.

In de simulatie gaf *Direct* 4–8 % minder afname + teruglevering bij koken en in het voorjaar,
ongeveer hetzelfde op de opgenomen avonden, en 1 % meer op een novemberdag. In een huis is het pas twee
keer gemeten; probeer het met een opname (stap 11) en vergelijk.

**Batterijgrenzen:** geeft de batterij (BMS) van een omvormer 30 s lang duidelijk minder dan
gevraagd (laden dat afneemt bij bijna vol, een koude batterij), dan plant de manager met wat hij echt
geeft en nemen de andere omvormers de rest over.

**Zekeringgroepen:** per apparaat *niet gebruikt*, *eigen groep* (3600 W) of een groep van
600 tot 5000 W. Apparaten in één groep delen de limiet. Zie de
[Zendure-HA-wiki](https://github.com/Zendure/Zendure-HA/wiki/Fuse-Group).

## 10. Zonneverwachting

*Configureren → Zonvoorspelling*: een sensor met de verwachting van vandaag, bijvoorbeeld
`sensor.solcast_pv_forecast_forecast_today` of `sensor.solcast_pv_forecast_forecast_remaining_today`.

- **Batterijen worden vandaag niet vol:** geen bewuste teruglevering (geen wolkenmarge en geen
  negatief P1-doel), zodat elke Wh zon wordt opgeslagen.
- **Geen zon meer vandaag:** de omvormer met de laadrol wordt klaargezet om te ontladen.

## 11. Meldingen, diagnostiek en opnames

**Meldingen onder Reparaties:**

| Melding | Wanneer |
|---|---|
| **{naam} reageert niet** | een zenSDK-omvormer is niet bereikbaar |
| **Geen adres bekend voor {naam}** | geen IP-adres gevonden; vul het in bij *Apparaatadressen* |
| **De P1-meter stuurt geen waarden** | een minuut geen P1-waarde: de batterijen houden hun vermogen, na 5 min vallen ze terug op het basisverbruik |
| **Een andere regelaar wijzigt {apparaten}** | iets anders (energieplan in de Zendure-app, andere integratie) wijzigt de limieten |
| **Zendure-cloud-MQTT is niet verbonden** / **Lokale MQTT-broker is niet verbonden** | MQTT-verbinding weg |

**Diagnostiek:** *Diagnostiek downloaden* op de integratiepagina geeft de stand van manager,
apparaten, rollen en instellingen (geheimen en serienummers afgeschermd). Per omvormer zijn er
diagnosesensoren (activiteit, rol, flash-schrijfacties, adresbron).

**Opname:** *Configureren → Logging, simulatie en testdata → P1 elke seconde opnemen tot middernacht*
schrijft de P1-waarde, de regelstand en een gebeurtenissenlog naar `<config>/zendure_ha/`. Die kun je
naspelen in de [simulatie](simulation.md). Details (Engels): [Recording and diagnostics](recording.md).

**Waakhond:** in de slimme modi grijpt hij in als er 10 s lang meer dan 300 W wordt afgenomen of
teruggeleverd terwijl er niets gebeurt. Hij logt de regelstand en dwingt een regelcyclus af.

## 12. Problemen oplossen

| Probleem | Oplossing |
|---|---|
| HACS biedt aan Zendure-HA te downloaden | Nooit doen: dan wordt deze integratie overschreven. Zendure-HA hoort op *niet geïnstalleerd* te staan (stap 3). |
| Instellingen en entiteiten weg na overstappen | De configuratie is verwijderd. Zie *Is de configuratie toch al verwijderd?* in stap 3. |
| P1-waarde maar om de paar seconden | Vul het IP-adres van de HomeWizard in onder *Configureren → P1-meter*. |
| Een omvormer verdwijnt steeds | Geef hem een vast IP-adres (DHCP-reservering of *Apparaatadressen*). |
| *Een andere regelaar wijzigt …* | Zet het energieplan in de Zendure-app of de andere integratie uit. |
| Meter blijft niet op 0 W | Download de diagnostiek of maak een opname; het gebeurtenissenlog laat zien waarom opdrachten werden tegengehouden. |
| Er draait maar één omvormer | Zo bedoeld: zo weinig mogelijk. Een volgende start boven de spreiddrempel (85 %). |

Open bij problemen een [issue](https://github.com/GruttePier013/Zendure-HA-GP013/issues) met de
diagnostiek erbij. Meld problemen met deze integratie niet bij Zendure-HA.
