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

Verktøyet har to visninger, valgt med fanene øverst:

- **Sammenlign miljøer** – forskjeller mellom to miljøer (beskrevet under).
- **Kvalitetsanalyse** – sjekker innholdet i *ett* valgt miljø: om de
  maskinlesbare reglene i `Regel`-feltet følger fasit, og andre
  uregelmessigheter i innholdet. Se «Kvalitetsanalyse» lenger ned.

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

## Kvalitetsanalyse

Fanen **Kvalitetsanalyse** henter én sjekkliste fra ett miljø og sjekker
innholdet. Velg **sjekkliste** og **miljø som analyseres** (Prod, Test eller
Dev), og klikk **Analyser**. Forskjeller mellom miljøene ser du i fanen
**Sammenlign miljøer**.

Analysen har to faner: **Maskinlesbare regler** og **Innholdskvalitet**. Begge
har et sammendrag der du kan klikke på en rad eller et fargefelt for å filtrere
listen under, og valgene lagres i URL-en på samme måte som i sammenligningen.

Fasiten står alltid synlig øverst i hver fane (grønn kant, merket **FASIT**),
med eksempler på riktig og feil skrivemåte. I listene viser kolonnen **Fasit**
hele verdien slik den skal stå, med endringen fra dagens verdi under. For
innholdet viser verktøyet også hvilke skrivemåter som er valgt som fasit i den
aktuelle sjekklisten (f.eks. `pbl.` framfor `pbl`), sammen med antall.

### Fasit for Regel-feltet

Det finnes ingen vedtatt standard for `Regel`. Fasiten er tolket fra
skrivemåten flest regler allerede bruker.

En regel har alltid formen `If (betingelse) then (utfall)`. Det finnes **to
gyldige regeltyper**, og det er utfallet som skiller dem:

| Regeltype | Fasit | Betyr |
| --- | --- | --- |
| **1. Peker på sjekkpunkt** | `If (1.73 = true) then (1.28)` | Når betingelsen er oppfylt, blir 1.28 aktuelt og skal vurderes. |
| **2. Preutfylling** | `If (norskSvenskDansk = true) then (1.1 = true)` | Når betingelsen er oppfylt, besvares 1.1 automatisk med ja. Brukes normalt på sjekkpunkter av typen `Auto`, ofte med et datafelt i betingelsen. |

```
Regel         = "If (" Betingelse ") then (" Utfall ")"
Utfall        = Sjekkpunkt-Id                             peker på sjekkpunkt
              | Sjekkpunkt-Id " = " ("true" | "false")    preutfylling
Betingelse    = Ledd { (" & " | " || ") Ledd }
Ledd          = (Sjekkpunkt-Id | datafelt) " = " ("true" | "false")
Sjekkpunkt-Id = tall "." tall              f.eks. 1.28
datafelt      = camelCase                  f.eks. norskSvenskDansk
```

Skrivemåten er den samme for begge typene:

- Hele betingelsen står i én parentes. Blandes `&` og `||`, må gruppene ha
  egen parentes: `If (2.12 = false & (3.13 = true || 3.20 = true)) then (2.9)`.
- Utfallet er ett sjekkpunkt-Id, normalt sjekkpunktets egen Id.
- Én regel per felt. En vurderingsregel og en preutfyllingsregel skal ikke stå
  sammen i samme felt.
- Ett mellomrom rundt `=`, `&` og `||`, ingen mellomrom innenfor parentesene,
  og ingen linjeskift.

Hver regel får én av tre vurderinger:

| Vurdering | Betyr |
| --- | --- |
| **Følger fasit** | Regelen er skrevet nøyaktig etter fasit. |
| **Kan rettes automatisk** | Regelen er entydig. Verktøyet viser et forslag med fargelagt diff (gjennomstreket = fjernes, uthevet = legges til). |
| **Må vurderes manuelt** | Fritekst, flere regler i ett felt, uklar logikk eller syntaksfeil. Et menneske må ta stilling til hva som er ment. |

I tillegg kryssjekkes reglene mot resten av sjekklisten (**kryssjekk**):
Id-er som ikke finnes i samme prosesskategori, utfall som peker på et annet
sjekkpunkt enn regelens eget, betingelser som bruker utfallet selv, ringer der
regler avhenger av hverandre, om regeltypen passer sjekkpunkttypen
(preutfylling på et sjekkpunkt som ikke er `Auto`, eller et `Auto`-sjekkpunkt med en regel
som bare peker på sjekkpunktet), og om `HarMaskinlesbarRegel` stemmer med
innholdet.

Alle avvikstypene med forklaring står i verktøyet under **Fasit for
Regel-feltet**. Et par av forslagene er tolkninger og bør bekreftes:

- **Ledd uten «= true/false»** – `(1.26 || 1.27 = true)` blir
  `(1.26 = true || 1.27 = true)`.
- **Datafelt ikke i camelCase** – `TegningNyPlan` blir `tegningNyPlan`. Sjekk
  navnet mot datamodellen før du retter.

Regellisten kan vises per sjekkpunkt eller med **Slå sammen like regler**, som
viser hver regeltekst én gang med alle sjekkpunktene som bruker den.
**Mønstre** grupperer reglene på struktur (Id-er byttes med `ID`, datafelt med
`felt`), slik at sjeldne varianter blir synlige. Listen kan også filtreres på
**regeltype**.

### Innholdskvalitet

| Kontroll | Type | Hva sjekkes |
| --- | --- | --- |
| Samme Id med ulikt innhold | Feil | Samme Id flere steder i samme prosesskategori, men med ulikt navn. Gjenbruk av samme sjekkpunkt under flere foreldre er greit. |
| HarMaskinlesbarRegel uten regel | Bør sjekkes | Flagget er `true`, men `Regel` er tomt. |
| Lovhjemmel uten gyldig lenke | Bør sjekkes | `LovhjemmelUrl` mangler eller er noe annet enn én URL. |
| Mangler verdi | Bør sjekkes | `Sjekkpunkttype`, `Milepel` eller `Tema` er tomt. |
| Mangler nynorsk navn | Bør sjekkes | `NavnNynorsk` er tomt. |
| Lik Rekkefolge blant søsken | Bør sjekkes | Sjekkpunkter på samme nivå med samme `Rekkefolge`. |
| Id følger ikke mønsteret tall.tall | Bør sjekkes | `Id` er ikke på formen `1.28`. |
| Utfalltype ulik for samme kode | Bør sjekkes | Samme `Utfalltypekode` har ulik `Utfalltype`-tekst. Forslaget er den vanligste teksten. |
| Mellomrom/linjeskift i tekst | Kan rettes | Mellomrom/linjeskift i start eller slutt, eller doble mellomrom, i navn, tema, lovhjemmel og utfalltype. `Beskrivelse` sjekkes bare i start/slutt. |
| Samme verdi skrevet ulikt | Kan rettes | F.eks. `auto`/`Auto` i `Sjekkpunkttype`. Fasit er den vanligste skrivemåten. |
| Lovhenvisning skrevet ulikt | Kan rettes | Samme lov skrevet på flere måter (`pbl`/`pbl.`/`Pbl`/`Plan- og bygningsloven`, `SAK`/`SAK10`), `§` uten mellomrom og `jfr.`/`jf`. Fasit er den vanligste skrivemåten med `§ ` foran paragrafen. |
| Nynorsk lik bokmål | Info | Kan være riktig, men er ofte tegn på manglende oversettelse. |
| Ingen tiltakstyper / Ingen utfall | Info | Listen er tom. |

### Eksport

- **Eksporter utvalg (CSV)** laster ned det som vises i aktiv fane, med
  filtrene som er valgt. Filen er semikolonseparert og åpnes direkte i Excel.
- **Eksporter alle funn (JSON)** laster ned alle regler med vurdering, forslag
  og avvik, og alle innholdsfunn
  (`sjekkliste-analyse-<sjekkliste>-<miljø>-<dato>.json`).

Verktøyet endrer ingenting i sjekklistene. Forslagene må legges inn i
sjekklisteadministrasjonen.
