# Sjekkliste-diff-viewer

Verktøy for å se ulikheter mellom sjekklistene i de ulike miljøene til DIBK sine
sjekkliste-API-er. Hele verktøyet er én selvstendig HTML-fil
([sjekkliste-diff-viewer.html](sjekkliste-diff-viewer.html)) uten byggesteg
eller avhengigheter – den henter data direkte fra API-ene i nettleseren.

Verktøyet støtter flere sjekklister:

| Sjekkliste | API |
| ---------- | --- |
| **DiBK Bygg** | `sjekkliste-bygg-api` |
| **Arbeidstilsynet** | `sjekkliste-arbeidstilsynet-api` |

Sjekklistene har samme datamodell (`Id`, `Navn`, `Tema`, `Prosesskategori`,
`Undersjekkpunkter` osv.), så all sammenligning, statistikk og filtrering
fungerer likt uansett hvilken sjekkliste som er valgt.

Miljøene som kan sammenlignes er **Prod**, **Test** og **Dev**. Øverst i
verktøyet velger du først **sjekkliste**, deretter hvilke to miljøer som skal
sammenlignes: det venstre miljøet fungerer som fasit, og det høyre
sammenlignes mot det.

## Publisert versjon

Verktøyet er publisert via GitHub Pages og kan åpnes direkte i nettleseren:

- https://arkitektum.github.io/Sjekkliste-diff-viewer/

Siden oppdateres automatisk hver gang endringer pushes til `main`-grenen.

## Slik bruker du det

1. Velg hvilken **sjekkliste** du vil sammenligne (DiBK Bygg eller
   Arbeidstilsynet).
2. Velg **fasit** (venstre) og **miljø** (høyre) som skal sammenlignes. De to
   velgerne kan ikke peke på samme miljø.
3. Klikk **Last inn data**. Verktøyet henter begge sjekklistene samtidig fra
   API-ene og bygger sammenligningen.
4. Bla gjennom resultatet, bruk statistikken til å drille inn på avvik, og
   filtrer/søk etter behov.

Bytter du sjekkliste, nullstilles et allerede innlastet resultat – tallene og
radene hører til den forrige sjekklisten. Klikk **Last inn data** på nytt.

Valgene dine (sjekkliste, miljøer, filtre, søk, visning) lagres i URL-en og i
nettleseren, slik at en lenke kan deles og gjenskape akkurat den visningen – og
slik at tilstanden huskes til neste besøk.

### API-endepunkter

**DiBK Bygg**

| Miljø | Endepunkt |
| ----- | --------- |
| Prod | `https://sjekkliste-bygg-api.ft.dibk.no/api/sjekkliste` |
| Test | `https://sjekkliste-bygg-api.ft-test.dibk.no/api/sjekkliste` |
| Dev  | `https://sjekkliste-bygg-api.ft-dev.dibk.no/api/sjekkliste` |

**Arbeidstilsynet**

| Miljø | Endepunkt |
| ----- | --------- |
| Prod | `https://sjekkliste-arbeidstilsynet-api.ft.dibk.no/api/sjekkliste` |
| Test | `https://sjekkliste-arbeidstilsynet-api.ft-test.dibk.no/api/sjekkliste` |
| Dev  | `https://sjekkliste-arbeidstilsynet-api.ft-dev.dibk.no/api/sjekkliste` |

Fordi dataene hentes direkte fra nettleseren, må API-et tillate `CORS` for den
origin verktøyet kjøres fra, og du må ha tilgang til det valgte miljøet. Hvis
lastingen feiler får du en beskjed som peker på nettopp dette.

## Funksjonalitet

### Sammenligning

- Grupperer sjekkpunkter etter **Prosesskategori**, med valgfri undergruppering
  etter **Tema**.
- Sammenligner hele hierarkiet: hovedsjekkpunkt og **Undersjekkpunkter**,
  matchet på `Id`.
- Felt som er miljøspesifikke eller tidsstempler (`SjekkId`, `Oppdatert`,
  `Updated`, `LastModified`, `Timestamp`) ignoreres i sammenligningen.
- Hvert sjekkpunkt får en status:
  - **Identisk** – ingen forskjell.
  - **Forskjellig** – finnes i begge, men med endret innhold.
  - **Kun i fasit** – mangler i miljøet det sammenlignes mot.
  - **Kun i miljø** – finnes bare i miljøet det sammenlignes mot.
  - **Flyttet** – finnes i begge miljøene, men på ulikt nivå i hierarkiet.
    Verktøyet slår opp punktet i hele treet på kategori + `Id`, sammenligner
    innholdet der det faktisk ligger, og viser hvor det ligger i hvert miljø.
- **Ekstra innhold** i det sammenlignede miljøet behandles som forventet og
  markeres informativt – ikke som feil.
- Forskjeller vises på ordnivå med fargelagt inline-diff, og lister
  sammenlignes på identitet framfor rekkefølge.

### Statistikk og drill-down

- Et oppsummeringspanel viser hvor like miljøene er (samsvarsprosent), en
  fordeling av statusene, en oversikt over hvilke felt som avviker
  (mangler/endret/ekstra), og forskjeller per prosesskategori.
- Klikk på et tall, et fargefelt eller en rad for å hoppe rett til de aktuelle
  sjekkpunktene – de åpnes og markeres automatisk, med et banner du kan
  nullstille fra.

### Filtrering og søk

- **Søk** på ID eller navn.
- **Kategori**-filter.
- **Status**-filter: kun problemer, kun forskjellige, kun identiske, kun i
  fasit, kun i miljø.
- **Avviksklasse**-filter: må rettes (mangler + endret), mangler helt, flyttet,
  endret/mangelfullt innhold, kun ekstra innhold, finnes bare i miljøet,
  identiske.
- **Felt med endringer**-filter.

### Visning

- Veksle mellom **kun overskrifter**, **vis alle detaljer** og
  **kun problem-detaljer**.
- **Utvid alle** / **Kollaps alle**.
- Slå **gruppering etter tema** av og på.
- Tastaturvennlig navigasjon.

### Eksport

- **Eksporter forskjeller** laster ned en JSON-fil
  (`sjekkliste-api-diff-<sjekkliste>-<dato>.json`) med tidsstempel, hvilken
  sjekkliste og hvilke miljøer som ble sammenlignet, oppsummering og alle
  sjekkpunkter som ikke er identiske.
