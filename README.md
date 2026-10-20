# Affaldskort

Interaktive kort over den offentlige debat om affald i Danmark. Repoet indeholder de færdige kort, så de kan lægges direkte på en hjemmeside. Der er ingen serverkode, ingen database og intet byggetrin.

Seneste opdatering er 17. oktober 2026. Det fulde kort har 80.401 dokumenter, og fokuskortet har 43.535.

En forhåndsvisning af repoets indhold ligger på https://simnyborg.github.io/affaldskort/

## Indhold

| Sti | Indhold |
|---|---|
| `index.html` | Oversigtssiden med fanerne Overblik, Debatkort, Aktørkort, Aktører, Netværk, Hvem driver debatten, Tone, Tidslinje og Kilder & metode. Under Debatkort kan man skifte mellem hele debatten og fokuskortet. |
| `debatkort.html` og `debatkort_data_*.js` | Det fulde kort med alle dokumenter. Siden henter selv sine datafiler. |
| `fokus/index.html` og `fokus/index_data_*.js` | Fokuskortet med dokumenter fra 2017 og frem, udvalgt til kommuner og affaldsselskaber |
| `aktoerkort.html` | Kortet over aktørerne. Data ligger i selve filen. |
| `data/` | Data til oversigtssiden. `summary.js` har nøgletal og grafer, `actors.js` aktørerne, `documents.js`, `documents_2.js` og `documents_3.js` dokumentlisten, `network.js` netværket og `docs.js` kildelisten. |
| `stats.json` og `version.json` | Nøgletal og tidspunktet for seneste opdatering. Siderne bruger dem ikke. |
| `indlejring-eksempel.html` | Eksempel på en side, der viser begge kort i en ramme |
| `.nojekyll` | Bruges kun til forhåndsvisningen på GitHub og skal ikke med på serveren |

## Sådan lægges kortene på Affaldsviden

Kopiér mappens indhold til webserveren og bevar filernes indbyrdes placering. Mappen `.git` og filerne `README.md` og `.nojekyll` skal ikke med, for `.git` rummer hele historikken og kan ellers hentes af alle. Den nemmeste vej er at hente repoet som ZIP via Code og Download ZIP, som ikke indeholder `.git`. Alle stier er relative, så mappen kan ligge hvor som helst, for eksempel under `/affaldskort/`. Alle links peger på konkrete filer, så serveren behøver ikke vise `index.html` for en mappe.

Mappen fylder ca. 70 MB. Den største fil er ca. 10 MB, og 7 filer er over 5 MB. Det betyder noget, hvis filerne lægges op gennem et CMS med en grænse for filstørrelse.

Kortene kan vises på tre måder.

**1. Hele oversigtssiden.** Link til `/affaldskort/index.html`. Tilføjes `?kort=fokus`, åbner siden på fokuskortet.

**2. Et kort på en eksisterende side.** Sæt kortet ind med en iframe. Se også `indlejring-eksempel.html`.

```html
<iframe src="/affaldskort/debatkort.html" title="Kort over affaldsdebatten"
        style="width:100%;height:85vh;border:0" loading="lazy"></iframe>

<iframe src="/affaldskort/fokus/index.html" title="Fokuskort over affaldsdebatten"
        style="width:100%;height:85vh;border:0" loading="lazy"></iframe>
```

Når debatkortet eller fokuskortet vises i en ramme, skjuler de deres titel og informationsknap. Aktørkortet skjuler sin titel. Giv derfor siden en overskrift og gerne denne korte forklaring.

> Hvert punkt er et dokument fra den offentlige debat om affald. Placeringen er beregnet ud fra dokumentets tekst med algoritmen UMAP, så dokumenter om samme emne ligger tæt og danner øer. Farven viser temaet, og klyngerne er navngivet efter, hvad dokumenterne i dem handler om. Klik på et punkt for at åbne kilden.

Rammen bør være synlig og have sin størrelse, når siden indlæses. Læg den helst ikke i en skjult fane eller en boks, der først foldes ud senere. Kortet bliver også tegnet der, men filterpanelet starter så sammenfoldet. Behold `loading="lazy"`, så kortet først hentes, når brugeren når ned til det.

**3. Kortene alene i hele vinduet.** Link til `/affaldskort/debatkort.html` eller `/affaldskort/fokus/index.html`.

Fokuskortet har et link med teksten "Se hele debatten", som kun ses, når fokuskortet åbnes alene. Det peger på oversigtssiden i mappen ovenover (`../index.html#debatkort`). Bruges fokuskortet uden oversigtssiden, kan linket fjernes i `fokus/index.html` (elementet med `id="akf-back"`).

## Krav til serveren

- Der er kun statiske filer. Webserveren skal sende `.html` som `text/html; charset=utf-8`, `.js` som `text/javascript` og `.json` som `application/json`, hvilket de fleste gør af sig selv.
- Slå gzip eller brotli til. Med gzip overfører det fulde kort ca. 12 MB, fokuskortet ca. 7 MB og oversigtssiden ca. 10 MB. Kortenes datafiler (`*_data_*.js`) er allerede pakket og bliver kun lidt mindre, mens html-filerne og `data/` bliver meget mindre.
- Datafilerne hentes med et versionsnummer efter indholdet (`?v=...`), så de kan caches længe. Html-filerne bør have kort cachetid, så en opdatering slår igennem med det samme.

### Content Security Policy

Kortene henter biblioteker og skrifttyper fra eksterne servere, bruger scripts og styles direkte i siderne og pakker deres data ud i Web Workers, som oprettes fra `blob:`. Har Affaldsviden en Content Security Policy, er dette den politik, kortene er testet med.

```
default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net https://cdnjs.cloudflare.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://maxcdn.bootstrapcdn.com; font-src 'self' https://fonts.gstatic.com https://maxcdn.bootstrapcdn.com; img-src 'self' data:; worker-src 'self' blob:
```

Skal kortene vises i en iframe, skal `frame-ancestors` eller `X-Frame-Options` tillade siderne fra samme domæne (`'self'` eller `SAMEORIGIN`). Har serveren CSP-rapportering slået til, kommer der én rapport om `wasm-eval` pr. kort. Den er harmløs, for deck.gl prøver at bruge WebAssembly og falder selv tilbage til almindelig JavaScript, så politikken skal ikke udvides.

| Domæne | Bruges til |
|---|---|
| `cdn.jsdelivr.net` | deck.gl 9.1.15 og Apache Arrow 21.2.0, som tegner kortene |
| `cdnjs.cloudflare.com` | d3 7.9.0, som tegner graferne på oversigtssiden |
| `maxcdn.bootstrapcdn.com` | To stylesheets, som kortbiblioteket indlæser |
| `fonts.googleapis.com` og `fonts.gstatic.com` | Skrifttyper |

Bibliotekernes versioner er låst fast, og de tre scriptfiler har en integrity-hash. En ny udgave af et bibliotek kan derfor ikke ændre kortene, og browseren afviser en scriptfil, der ikke er den forventede. Stylesheets og skrifttyper har ingen hash.

Ved hvert besøg henter browseren filer fra disse servere, som dermed ser brugerens IP-adresse. Det gælder også Google Fonts. Skal det undgås, kan bibliotekerne, stylesheets og skrifttyperne lægges på Affaldsvidens egen server, og adresserne i html-filerne rettes til.

## Krav til browseren

- Kortene tegnes med WebGL og pakker deres data ud med browserens indbyggede DecompressionStream. Det kræver Chrome eller Edge 80, Firefox 113 eller Safari 16.4 og nyere, også på mobil.
- Siderne sætter ingen cookies og har ingen statistik. De gemmer kun visningsvalg i browseren (localStorage) under nøglerne `ak-tab` og `ak-kort` (fanen og kortet, der sidst var valgt på oversigtssiden) og `ak-panel-collapsed`, `ak-time-collapsed`, `akf-panel-collapsed` og `akf-time-collapsed` (om filterpanel og periodevælger er foldet sammen).
- Siderne beder søgemaskiner om ikke at indeksere dem. Skal kortene kunne findes via søgning, fjernes linjen `<meta name="robots" content="noindex">` i `index.html`, `debatkort.html`, `aktoerkort.html` og `fokus/index.html`.

## Opdateringer

Repoet opdateres, hver gang kortene bygges igen. Nye udgaver lægges oven på de gamle i historikken, så `git pull` giver altid den nyeste. Uden git kan repoet hentes som ZIP via Code og Download ZIP. Bruges git, så kopiér filerne til serveren uden `.git`. Erstat hele mappen på serveren i stedet for at kopiere de nye filer oven i de gamle, så datafiler, der ikke bruges mere, ikke bliver liggende. `version.json` viser, hvornår kortene sidst blev opdateret.

## Indholdet

Kortene viser titler, datoer, kilder og links til de oprindelige dokumenter. Nyhedsartiklernes egen tekst vises ikke, og et klik på et punkt åbner dokumentet hos kilden. Dokumenterne kommer fra offentligt tilgængelige kilder som Folketinget, ministerier, kommuner, affaldsselskaber, organisationer, forskning og medier.
