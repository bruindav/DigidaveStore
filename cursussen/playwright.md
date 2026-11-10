## Hoe deze cursus werkt

In 8 modules ga je van je eerste test naar een complete Playwright-testsuite in TypeScript die automatisch draait bij elke push naar GitHub. Elke module heeft drie delen: **uitleg**, **voorbeeldcode** en een **oefening op een echte webapp**. Reken op 1 tot 2 uur per module.

**Werkwijze per module**

1. Lees de uitleg en typ de voorbeelden zelf over (niet kopiëren: dan blijft het beter hangen).
2. Doe de oefening. Loop je vast, kijk dan pas bij de hints.
3. Kom je er niet uit? Lees eerst de foutmelding goed (module 5 legt uit hoe). Bespreek het daarna met een collega, of plak je code en foutmelding in een AI-assistent en vraag om uitleg.

**De oefenapps**

De oefeningen gebruiken apps van [DigidaveStore](https://bruindav.github.io/DigidaveStore/), een verzameling webapps die online staan.

| Module | App | Waarom deze app |
| --- | --- | --- |
| 1–3 | [DigidaveStore](https://bruindav.github.io/DigidaveStore/) | Eenvoudige pagina met groepen, kaarten en links: ideaal om te beginnen |
| 3–4 | [QR Lezer](https://bruindav.github.io/qr-lezer/) | Eén duidelijke functie, inclusief bestand uploaden |
| 5–6 | [Mijn Tuin](https://bruindav.github.io/mijntuin/) | Meer schermen en formulieren: geschikt voor Page Objects |
| 7 | [HoornaarZoeken](https://bruindav.github.io/HornetApp/) | Kaart (Leaflet), inloggen en netwerkverkeer |
| 8 + eind | [Operationeel Portaal Scheepvaart](https://bruindav.github.io/DigidaveStore/prototypes/operationeel-portaal.html) | Lijkt het meest op ons werk: modules, formulieren, rollen |

> **Let op:** voor een deel van de oefeningen in module 5 tot en met 7 is een account nodig. We zoeken nog uit hoe iedereen daar kan oefenen. Kun je niet inloggen, doe dan het deel van de oefening dat zonder account kan.

**Spelregel:** deze apps draaien met echte gegevens. Schrijf geen tests die gegevens aanmaken, wijzigen of wissen, tot je in module 7 leert hoe je dat veilig doet (mocken).

**Voorbereiding**

- [ ] Node.js LTS geïnstalleerd (`node -v` geeft versie 20 of hoger)
- [ ] VS Code geïnstalleerd, met de extensie *Playwright Test for VSCode* (van Microsoft)
- [ ] Git geïnstalleerd (`git --version` werkt) en een GitHub-account
- [ ] Een lege map gemaakt, bijvoorbeeld `C:\dev\playwright-cursus`, en in VS Code geopend via *File > Open Folder…*

We werken in **TypeScript**. Ken je weinig TypeScript? Geen probleem. Playwright-tests gebruiken maar een klein deel van de taal, en het wordt uitgelegd waar het nodig is.

## Module 1 – Installatie en je eerste test

Na deze module heb je een werkend Playwright-project en schrijf je zelf een test die een pagina opent en de titel controleert.

**Wat is Playwright?** Een framework van Microsoft dat een echte browser (Chromium, Firefox, WebKit/Safari) bestuurt vanuit code. Jij schrijft wat een gebruiker doet (klikken, typen) en wat je verwacht te zien. Playwright voert dat uit en meldt wat er misgaat.

### 1.1 Project aanmaken

Open een terminal in je lege map (in VS Code: *Terminal > New Terminal*) en voer uit:

```bash
npm init playwright@latest
```

Kies: TypeScript, map `tests`, GitHub Actions workflow **ja**, browsers installeren **ja**. Je krijgt:

- `playwright.config.ts` – de instellingen
- `tests/example.spec.ts` – een voorbeeldtest
- `package.json` – je project en afhankelijkheden
- `.github/workflows/playwright.yml` – voor module 8

### 1.2 Tests draaien

```bash
npx playwright test              # alle tests, onzichtbaar (headless)
npx playwright test --headed     # met zichtbare browser
npx playwright test --ui         # UI-modus: het fijnste om mee te leren
npx playwright show-report       # het HTML-rapport bekijken
npx playwright test --project=chromium   # alleen in Chromium
```

Standaard draait elke test in drie browsers (Chromium, Firefox en WebKit), dus in het rapport zie je elke test drie keer. Tijdens het leren is `--project=chromium` sneller en overzichtelijker.

### 1.3 Anatomie van een test

```ts
import { test, expect } from '@playwright/test';

test('pagina heeft de juiste titel', async ({ page }) => {
  await page.goto('https://bruindav.github.io/DigidaveStore/');
  await expect(page).toHaveTitle(/Digidave/i);
});
```

- `test('naam', ...)` – één testgeval met een beschrijvende naam.
- `async ({ page })` – Playwright geeft je een verse browsertab: de `page`-*fixture*.
- `await` – elke browseractie kost tijd. Vergeet je `await`, dan krijg je vreemde fouten. Dit is de meest gemaakte beginnersfout.
- `expect(...)` – de controle. Zonder `expect` test je eigenlijk niets.
- `/Digidave/i` – een reguliere expressie: ‘bevat Digidave’, hoofdletterongevoelig. Met aanhalingstekens (`'Digidave'`) moet de titel precies gelijk zijn.

**VS Code-tips:** sla op met **Ctrl+S** (een wit bolletje op het tabblad betekent: niet opgeslagen, en dan draait Playwright de oude versie) of zet *File > Auto Save* aan. Met de Playwright-extensie staat er een groen driehoekje naast elke test; klik erop om alleen die test te draaien.

### Oefening 1 – DigidaveStore

Maak `tests/digidavestore.spec.ts` (rechtsklik op de map `tests` > *New File…*) en schrijf:

1. Een test die de pagina opent en controleert dat de titel klopt (zoek eerst de echte titel op in het browsertabblad).
2. Een test die controleert dat de URL `DigidaveStore` bevat (`toHaveURL`).
3. Draai beide in de UI-modus. Verander daarna bewust de verwachte titel in iets fouts en bekijk hoe de foutmelding eruitziet.

**Klaar als:** 2 groene tests, en je kunt in eigen woorden uitleggen wat `await` en `expect` doen.

*Hint:* zie je `Timeout 5000ms exceeded`, dan was je verwachting fout: Playwright heeft 5 seconden gewacht tot de pagina eraan voldeed.

## Module 2 – Locators en acties

Na deze module kun je elk element op een pagina betrouwbaar vinden en ermee werken zoals een gebruiker dat doet.

Een **locator** is een beschrijving van hoe je een element vindt. Het element wordt pas gezocht op het moment dat je er iets mee doet. Daardoor werkt een locator ook als de pagina tussendoor verandert.

### 2.1 De voorkeursvolgorde

Zoek elementen zoals een gebruiker ze ziet, niet via de interne opbouw van de pagina. Dat maakt tests robuust bij een herontwerp.

1. `page.getByRole('button', { name: 'Opslaan' })` – rol + zichtbare naam. **Altijd eerst proberen.**
2. `page.getByLabel('E-mailadres')` – formuliervelden via hun label.
3. `page.getByPlaceholder('Zoek...')` – als er geen label is.
4. `page.getByText('Welkom terug')` – tekst die geen knop of link is.
5. `page.getByAltText('logo')` / `getByTitle(...)` – afbeeldingen en tooltips.
6. `page.getByTestId('melding-opslaan')` – een `data-testid`-attribuut dat het ontwikkelteam toevoegt. Goed om met je team af te spreken.
7. `page.locator('#id')` of `page.locator('article.card')` – CSS. Laatste redmiddel: breekt snel.

Veelgebruikte rollen: `button`, `link`, `heading`, `textbox`, `checkbox`, `combobox` (dropdown), `listitem`, `article`, `dialog`, `tab`, `row`, `main`, `navigation`.

**CSS-selectors in het kort:** `article` is een tag, `.card` een klasse, `article.card` (zonder spatie) een element dat beide is, `#rev-title` een id, `.card h3` (met spatie) een `h3` ergens binnen een `.card`, `button[type="button"]` een attribuut. In Chrome DevTools (F12 > Elements) kun je met Ctrl+F een selector uitproberen.

### 2.2 Verfijnen en combineren

```ts
// binnen een ander element zoeken
const kaart = page.getByRole('article').filter({ hasText: 'QR Lezer' });
await kaart.getByRole('link', { name: /Open app/ }).click();

// alleen binnen het hoofdgedeelte van de pagina zoeken
const main = page.getByRole('main');
await main.getByRole('button', { name: /Handige tools/ }).click();

// het eerste, laatste of n-de element
page.getByRole('article').first();
page.getByRole('article').nth(2);   // telt vanaf 0

// exacte naam in plaats van 'bevat'
page.getByRole('button', { name: 'Opslaan', exact: true });
```

### 2.3 Acties

```ts
await page.getByLabel('Naam').fill('Jan');              // veld leegmaken en invullen
await page.getByRole('button', { name: 'Zoek' }).click();
await page.getByLabel('Status').selectOption('Open');  // native <select>
await page.getByRole('checkbox', { name: 'Akkoord' }).check();
await page.getByLabel('Zoek').press('Enter');
await page.getByText('Melding').hover();
```

**Angular Material:** een `mat-select` is géén native `<select>`, dus `selectOption` werkt niet. Je klikt de dropdown open en kiest dan een optie: `await page.getByRole('combobox', { name: 'Status' }).click();` gevolgd door `await page.getByRole('option', { name: 'Open' }).click();`.

### Oefening 2 – DigidaveStore verkennen

1. Tel hoeveel apps er op de pagina staan en controleer dat aantal met `toHaveCount(n)`.
2. Klap één groep open, controleer hoeveel apps er dan zichtbaar zijn, klik bij QR Lezer op *Open app* en controleer dat je op de QR Lezer-pagina uitkomt.
3. Controleer dat de groep ‘Persoonlijk’ bestaat (`getByRole('heading', ...)`).
4. Uitdaging: schrijf voor elke locator in je test op welke plek van de lijst in 2.1 hij staat. Gebruik je CSS? Kijk of het met een rol kan.

*Hints:*

- Ingeklapte groepen zijn verborgen. `getByRole` telt alleen wat een gebruiker kan zien; `page.locator('article.card')` telt alles in de HTML, ook verborgen kaarten.
- Elke groepsnaam staat twee keer op de pagina: als filterknop bovenaan en als groepstitel. Krijg je *strict mode violation … resolved to 2 elements*, zoek dan binnen `page.getByRole('main')`.
- Een link opent in een nieuw tabblad? Zie module 7.4.
- Open de pagina met `npx playwright codegen https://bruindav.github.io/DigidaveStore/` en wijs elementen aan: je ziet direct welke locator Playwright voorstelt. Codegen komt uitgebreid terug in module 5.

## Module 3 – Assertions en automatisch wachten

Na deze module weet je hoe je controleert wat er op het scherm staat, en waarom je in Playwright (bijna) nooit zelf hoeft te wachten.

### 3.1 Auto-waiting

Voor elke actie wacht Playwright automatisch tot het element zichtbaar, stabiel en klikbaar is. Een `expect` op een locator probeert het steeds opnieuw, standaard tot 5 seconden. Daarom geldt:

- **Nooit** `await page.waitForTimeout(3000)` gebruiken om ‘even te wachten’. Het maakt tests traag én onbetrouwbaar.
- Wacht in plaats daarvan op wat je wilt zien: `await expect(page.getByText('Opgeslagen')).toBeVisible();`

### 3.2 De belangrijkste assertions

| Assertion | Controleert |
| --- | --- |
| `toBeVisible()` / `toBeHidden()` | Element wel of niet zichtbaar |
| `toHaveText('...')` / `toContainText('...')` | Exacte tekst / tekst bevat |
| `toHaveValue('...')` | Waarde van een invoerveld |
| `toHaveCount(n)` | Aantal gevonden elementen |
| `toBeEnabled()` / `toBeDisabled()` | Knop of veld bruikbaar |
| `toBeChecked()` | Checkbox aangevinkt |
| `toHaveAttribute('href', /qr/)` | Attribuut met waarde |
| `toHaveClass(/actief/)` | CSS-klasse aanwezig |
| `toHaveURL(/.../)` / `toHaveTitle(...)` | Op de pagina zelf |

Omkeren doe je met `.not`: `await expect(foutmelding).not.toBeVisible();`

**Soft assertions** laten de test doorlopen na een fout, zodat je alle afwijkingen in één keer ziet: `await expect.soft(titel).toHaveText('Mijn Tuin');`

### 3.3 Let op het verschil

```ts
// GOED – wacht en probeert opnieuw
await expect(page.getByRole('heading')).toHaveText('Resultaat');

// FOUT – leest één keer en vergelijkt direct, geen wachten
const tekst = await page.getByRole('heading').textContent();
expect(tekst).toBe('Resultaat');
```

### 3.4 Bestand uploaden

```ts
await page.getByLabel('Kies afbeelding').setInputFiles('tests/fixtures/qr-test.png');
```

### Oefening 3 – QR Lezer

Maak eerst met een gratis QR-generator een afbeelding met de tekst `https://example.com` en sla die op als `tests/fixtures/qr-test.png`.

1. Test dat de startpagina laadt en dat de uploadknop zichtbaar en bruikbaar is.
2. Upload de afbeelding en controleer dat het resultaat `https://example.com` toont.
3. Upload een afbeelding *zonder* QR-code (bijv. een foto) en controleer de foutmelding die de app dan toont.
4. Uitdaging: zoek in je test naar één plek waar je zou willen ‘wachten’ en vervang dat door een assertion.

*Hint:* is het uploadveld verborgen achter een mooie knop? `setInputFiles` werkt ook op een verborgen `<input type="file">`: `page.locator('input[type=file]')`. Dit is een van de weinige plekken waar CSS prima is.

## Module 4 – Tests structureren: groepen, hooks en fixtures

Na deze module zijn je tests overzichtelijk gegroepeerd, zonder herhaalde code, en draaien ze onafhankelijk van elkaar.

### 4.1 Gouden regel: elke test staat op zichzelf

Elke test krijgt een schone browser (nieuwe *context*: geen cookies, geen localStorage). Een test mag nooit afhangen van een eerdere test. Playwright draait tests bovendien **parallel**, dus de volgorde is niet gegarandeerd.

### 4.2 Groeperen en hooks

```ts
import { test, expect } from '@playwright/test';

test.describe('QR Lezer – uploaden', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('https://bruindav.github.io/qr-lezer/');
  });

  test('leest een geldige QR-code', async ({ page }) => { /* ... */ });
  test('meldt een afbeelding zonder QR-code', async ({ page }) => { /* ... */ });
});
```

- `beforeEach` / `afterEach` – voor en na elke test in de groep.
- `beforeAll` / `afterAll` – één keer per groep (bijv. testdata klaarzetten).
- `test.skip()`, `test.only()`, `test.fixme()` – tijdelijk overslaan, alleen deze draaien, of markeren als bekend probleem. Laat `.only` nooit staan bij het committen.

### 4.3 baseURL

Zet in `playwright.config.ts` onder `use`: `baseURL: 'https://bruindav.github.io'`. Dan schrijf je in tests kort `await page.goto('/qr-lezer/');`. Later wissel je met één regel tussen test- en productieomgeving.

### 4.4 Eigen fixtures

Een fixture levert iets aan een test, zoals `page`. Je kunt er zelf een maken, bijvoorbeeld een pagina die al op de QR Lezer staat:

```ts
// tests/fixtures.ts
import { test as base, type Page } from '@playwright/test';

export const test = base.extend<{ qrPagina: Page }>({
  qrPagina: async ({ page }, use) => {
    await page.goto('/qr-lezer/');   // klaarzetten
    await use(page);                 // hier draait de test
    // hier eventueel opruimen
  },
});
export { expect } from '@playwright/test';
```

In je test importeer je dan `test` uit `./fixtures` in plaats van uit `@playwright/test` en schrijf je `async ({ qrPagina }) => ...`.

### Oefening 4 – Opruimen

1. Herschrijf je QR Lezer-tests met `test.describe` en `beforeEach`.
2. Zet `baseURL` in de config en maak alle `goto`-aanroepen kort.
3. Zet je DigidaveStore- en QR Lezer-tests elk in een eigen bestand; draai daarna alles en bekijk in het rapport dat ze parallel liepen.
4. Uitdaging: maak de fixture `qrPagina` uit 4.4 en gebruik die in één test.

## Module 5 – Debuggen, codegen en de trace viewer

Na deze module vind je binnen een paar minuten uit waarom een test faalt, en laat je Playwright het eerste concept van een test voor je opnemen.

### 5.1 Codegen: opnemen in plaats van typen

```bash
npx playwright codegen https://bruindav.github.io/mijntuin/
```

Je klikt door de app; Playwright schrijft de code mee. Handig als startpunt, maar **altijd nabewerken**: codegen voegt geen of weinig `expect`s toe, kiest soms zwakke locators en neemt elke misklik op. In VS Code kan hetzelfde met *Record new* in het Testing-paneel.

### 5.2 Debugmogelijkheden

| Hulpmiddel | Hoe | Wanneer |
| --- | --- | --- |
| UI-modus | `npx playwright test --ui` | Stap voor stap terugkijken, locators uitproberen |
| Debug-modus | `npx playwright test --debug` | Regel voor regel door een test lopen |
| Pauze in code | `await page.pause();` | Op één plek stoppen en zelf rondkijken |
| VS Code | Breakpoint zetten, rechtsklik op het driehoekje > *Debug Test* | Als je gewend bent aan breakpoints |
| Trace viewer | `npx playwright show-trace trace.zip` | Een mislukte test achteraf analyseren, ook uit CI |

### 5.3 De trace viewer

Een trace is een opname van een test: per stap een screenshot, het DOM, netwerkverzoeken en console-meldingen. Zet in de config onder `use`:

```ts
trace: 'on-first-retry',     // trace bij de herhaling van een mislukte test
screenshot: 'only-on-failure',
video: 'retain-on-failure',
```

In het HTML-rapport klik je bij een mislukte test op *Trace*. Kijk vooral naar het tabblad **Before/After** van de stap die faalde, en naar **Console** en **Network**: vaak zie je daar de echte oorzaak (een JavaScript-fout, een verzoek dat 403 geeft).

### 5.4 Een foutmelding lezen

Playwright vertelt je drie dingen: welke locator, wat het verwachtte en wat het kreeg. Lees altijd eerst de regel `Locator:` en de `Call log`. *Strict mode violation* betekent: je locator vond meerdere elementen, maak hem specifieker.

### Oefening 5 – Mijn Tuin

1. Neem met codegen een korte route op: naar het plantenoverzicht en één plant openen (inloggen alleen als je een account hebt).
2. Bewerk de opname: verbeter zwakke locators, haal misklikken weg en voeg minstens 3 `expect`s toe.
3. Maak de test bewust kapot (verkeerde verwachte tekst), draai hem met `--trace on` en zoek in de trace de stap waar het misgaat.
4. Uitdaging: open de trace, ga naar het tabblad Network en zoek een verzoek naar Firestore. Wat zie je?

*Let op:* zet nooit een wachtwoord in je testcode. In module 7 leer je hoe het netjes moet.

## Module 6 – Het Page Object Model

Na deze module staan je locators en schermacties op één plek, zodat een wijziging in de app maar één aanpassing in je tests kost. Dit is de standaardopzet in vrijwel elk professioneel testteam.

### 6.1 Het idee

Zonder Page Objects staat `page.getByLabel('Plantnaam')` in twintig tests. Hernoemt iemand het label, dan pas je twintig plekken aan. Met een Page Object staat die locator in één klasse, en lezen je tests als een verhaal: `await planten.zoek('Lavendel')`.

### 6.2 Een Page Object schrijven

```ts
// pages/PlantenPagina.ts
import { type Page, type Locator } from '@playwright/test';

export class PlantenPagina {
  readonly page: Page;
  readonly zoekveld: Locator;
  readonly plantkaarten: Locator;

  constructor(page: Page) {
    this.page = page;
    this.zoekveld = page.getByPlaceholder('Zoek plant');
    this.plantkaarten = page.getByRole('listitem');
  }

  async open() {
    await this.page.goto('/mijntuin/');
  }

  async zoek(naam: string) {
    await this.zoekveld.fill(naam);
  }

  kaart(naam: string): Locator {
    return this.plantkaarten.filter({ hasText: naam });
  }
}
```

```ts
// tests/planten.spec.ts
import { test, expect } from '@playwright/test';
import { PlantenPagina } from '../pages/PlantenPagina';

test('zoeken toont alleen de gezochte plant', async ({ page }) => {
  const planten = new PlantenPagina(page);
  await planten.open();
  await planten.zoek('Lavendel');
  await expect(planten.kaart('Lavendel')).toBeVisible();
});
```

De locators in dit voorbeeld zijn ter illustratie: zoek met codegen of DevTools uit hoe de echte elementen heten.

**TypeScript-opfrisser:** `class` is een bouwtekening, `constructor` wordt uitgevoerd bij `new`, `readonly` betekent ‘niet meer overschrijven’, en `: string` of `: Locator` vertelt welk type iets is.

### 6.3 Vuistregels

- Eén klasse per scherm of duidelijk onderdeel (bijv. ook een `Navigatie`-klasse voor het menu).
- Methodes beschrijven wat een gebruiker doet (`voegPlantToe`), niet hoe (`klikOpKnop3`).
- Zet `expect`s bij voorkeur in de test, niet in het Page Object. Dan zie je in de test wat er gecontroleerd wordt.
- Combineer met een fixture (module 4) zodat je `new PlantenPagina(page)` niet steeds hoeft te typen: `async ({ plantenPagina }) => ...`.

### Oefening 6 – Mijn Tuin herbouwen

1. Maak een map `pages/` met minstens twee Page Objects voor Mijn Tuin, bijvoorbeeld het plantenoverzicht en de tuinplattegrond of het bloeiwiel.
2. Herschrijf je test uit module 5 zodat er geen enkele `getBy...` meer in het testbestand staat.
3. Schrijf 3 nieuwe tests die alleen de Page Objects gebruiken (bijv. zoeken, een plant openen, wisselen tussen maanden in het bloeiwiel).
4. Uitdaging: maak een fixture die je Page Objects levert, en vergelijk hoe leesbaar je test nu is ten opzichte van de codegen-versie.

## Module 7 – Moeilijkere situaties: inloggen, netwerk en kaarten

Na deze module test je apps met een login, controleer en vervals je netwerkverkeer, en kun je omgaan met lastige onderdelen zoals een Leaflet-kaart en pop-ups.

### 7.1 Geheimen buiten je code

Zet de gegevens van je testaccount in een bestand `.env` (en voeg `.env` toe aan `.gitignore`!):

```bash
TEST_EMAIL=test@voorbeeld.nl
TEST_WACHTWOORD=...
```

Installeer `npm i -D dotenv` en zet bovenaan `playwright.config.ts`: `import 'dotenv/config';`. In je code gebruik je dan `process.env.TEST_EMAIL!`. Gebruik nooit een beheerdersaccount voor tests.

### 7.2 Eén keer inloggen, overal ingelogd (storage state)

Inloggen in elke test is traag. Log daarom één keer in een *setup*-test in, sla de sessie op en hergebruik die:

```ts
// tests/auth.setup.ts
import { test as setup, expect } from '@playwright/test';

setup('inloggen', async ({ page }) => {
  await page.goto('/HornetApp/');
  await page.getByLabel('E-mail').fill(process.env.TEST_EMAIL!);
  await page.getByLabel('Wachtwoord').fill(process.env.TEST_WACHTWOORD!);
  await page.getByRole('button', { name: 'Inloggen' }).click();
  await expect(page.getByText('Uitloggen')).toBeVisible();
  await page.context().storageState({ path: 'playwright/.auth/user.json', indexedDB: true });
});
```

In de config maak je een project `setup` en laat je andere projecten daarvan afhangen met `dependencies: ['setup']` en `use: { storageState: 'playwright/.auth/user.json' }`. Zet `playwright/.auth` ook in `.gitignore`. **Let op:** Firebase Auth bewaart de sessie in IndexedDB, daarom staat `indexedDB: true` erbij.

### 7.3 Netwerk controleren en vervalsen

```ts
// wachten op een verzoek en het antwoord controleren
const antwoord = page.waitForResponse(r => r.url().includes('firestore') && r.status() === 200);
await page.getByRole('button', { name: 'Vernieuwen' }).click();
await antwoord;

// een externe dienst vervangen door een nepantwoord (mock)
await page.route('**/api.open-meteo.com/**', route =>
  route.fulfill({ json: { current: { temperature_2m: 21 } } })
);

// een fout simuleren: hoe reageert de app als een dienst plat ligt?
await page.route('**/tiles/**', route => route.abort());
```

Mocken is dé manier om situaties te testen die je in het echt moeilijk kunt opwekken (lege lijst, serverfout, trage verbinding) zonder echte data aan te raken. Het verkeer van de Firestore-SDK zelf laat zich lastig mocken; daarvoor gebruik je in de praktijk een apart testproject of de Firebase Emulator.

### 7.4 Lastige onderdelen

- **Pop-ups van de browser** (`confirm`, `alert`): `page.once('dialog', d => d.accept());` vóór de klik.
- **Nieuw tabblad**: `const nieuw = page.waitForEvent('popup'); await link.click(); const tab = await nieuw;`
- **Kaart (Leaflet)**: markers zijn gewone elementen: `page.locator('.leaflet-marker-icon')`. Klikken op een plek op de kaart: `await page.locator('#map').click({ position: { x: 200, y: 150 } });`
- **Visuele vergelijking**: `await expect(page).toHaveScreenshot();` maakt de eerste keer een referentiebeeld en vergelijkt daarna. Handig voor een kaart of SVG-plattegrond, maar kies dan een vaste schermgrootte.

### Oefening 7 – HoornaarZoeken

1. Zorg voor een testaccount met de laagste rol en zet de gegevens in `.env`.
2. Bouw de storage-state-setup uit 7.2 en controleer dat een test direct ingelogd start.
3. Test dat er markers op de kaart verschijnen en dat een klik op een marker de details van een waarneming toont.
4. Blokkeer de kaarttegels met `page.route` en controleer dat de app nog bruikbaar blijft.
5. Uitdaging: test dat een gebruiker met de laagste rol géén beheerknoppen ziet (rolgebaseerde toegang).

*Geen account?* Doe opdracht 4 en de oefeningen met `page.route` uit 7.3 op de pagina's die zonder inloggen werken, en oefen 7.4 met de links in DigidaveStore die een nieuw tabblad openen.

## Module 8 – Een echte testsuite: configuratie en CI

Na deze module heb je een testsuite die op meerdere browsers en schermformaten draait, en automatisch bij elke push op GitHub.

### 8.1 Mappenstructuur

```text
playwright-cursus/
  pages/              Page Objects
  tests/
    fixtures.ts       eigen fixtures
    fixtures/         testbestanden (afbeeldingen e.d.)
    auth.setup.ts
    portaal/          tests per app of module
  playwright.config.ts
  .env                (niet in Git)
```

### 8.2 De config: projecten, retries, parallel

```ts
import { defineConfig, devices } from '@playwright/test';
import 'dotenv/config';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,       // op GitHub 2 herkansingen
  forbidOnly: !!process.env.CI,          // vergeten .only laat CI falen
  reporter: [['html'], ['list']],
  use: {
    baseURL: process.env.BASE_URL ?? 'https://bruindav.github.io',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'setup', testMatch: /.*\.setup\.ts/ },
    { name: 'chromium', use: { ...devices['Desktop Chrome'] }, dependencies: ['setup'] },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] }, dependencies: ['setup'] },
    { name: 'mobiel',   use: { ...devices['Galaxy S9+'] },     dependencies: ['setup'] },
  ],
});
```

Eén project draaien: `npx playwright test --project=mobiel`. Alleen tests met een label: geef de test een tag, `test('opslaan', { tag: '@smoke' }, ...)`, en draai `npx playwright test --grep @smoke`.

Chrome, Edge, Vivaldi en Brave zijn allemaal op Chromium gebouwd; slaagt een test in `chromium`, dan werkt hij daar vrijwel zeker ook. Wil je toch de echte Chrome of Edge gebruiken, zet dan `channel: 'chrome'` of `channel: 'msedge'` in `use`.

### 8.3 GitHub Actions

Het bestand `.github/workflows/playwright.yml` uit module 1 doet al het meeste: Node installeren, `npm ci`, browsers installeren, tests draaien en het rapport als *artifact* bewaren. Voeg geheimen toe via GitHub > repo > Settings > Secrets and variables > Actions, en geef ze door in de workflow:

```yaml
      - name: Run Playwright tests
        run: npx playwright test
        env:
          TEST_EMAIL: ${{ secrets.TEST_EMAIL }}
          TEST_WACHTWOORD: ${{ secrets.TEST_WACHTWOORD }}
```

Faalt een test op GitHub? Download het artifact `playwright-report`, pak het uit en open de trace (module 5).

### 8.4 Flaky tests

Een test die soms slaagt en soms faalt heet *flaky*. Oorzaken zijn bijna altijd: vaste wachttijden, tests die van elkaar afhangen, of gedeelde testdata. Zoek ze op met `npx playwright test --repeat-each=10`. Los ze op; verhoog niet zomaar de time-outs.

### Oefening 8 – Operationeel Portaal Scheepvaart

1. Maak een nieuwe GitHub-repo `playwright-cursus`, zet je werk erin en controleer dat de workflow groen draait.
2. Voeg tests toe voor het portaal: de dashboardpagina laadt, en je kunt wisselen tussen de modules Melding, Storing en Incident.
3. Draai alles ook op het project `mobiel` en los op wat daar anders gaat.
4. Markeer 3 snelle, belangrijke tests als `@smoke`.
5. Uitdaging: draai je suite 10 keer achter elkaar en repareer elke test die één keer faalt.

## Eindopdracht – een testsuite voor een volledige werkstroom

Bouw voor het Operationeel Portaal Scheepvaart een suite die de werkstroom van een **Storing** van begin tot eind test, zoals je dat in een echt project zou doen.

- [ ] Page Objects voor het dashboard, het Storing-formulier en het journaal
- [ ] Een test die een storing aanmaakt (velden invullen), opslaat en terugvindt in het overzicht
- [ ] Validatietests: verplichte velden leeg laten en de foutmeldingen controleren
- [ ] Een test per rol: wat mag een operator wel en niet zien of doen
- [ ] Minstens één test die een netwerkfout simuleert met `page.route`
- [ ] Testdata wordt na afloop opgeruimd, of de test gebruikt alleen nepdata
- [ ] Draait groen in GitHub Actions op Chromium en mobiel

Klaar? Vraag een collega om een code review van je repo, en review die van hem of haar.

## Spiekbrief

| Wat | Code |
| --- | --- |
| Pagina openen | `await page.goto('/pad/')` |
| Knop klikken | `await page.getByRole('button', { name: 'X' }).click()` |
| Veld invullen | `await page.getByLabel('X').fill('tekst')` |
| Dropdown (native) | `await page.getByLabel('X').selectOption('waarde')` |
| Bestand uploaden | `await locator.setInputFiles('pad')` |
| Zichtbaar? | `await expect(locator).toBeVisible()` |
| Tekst? | `await expect(locator).toHaveText('X')` |
| Aantal? | `await expect(locator).toHaveCount(3)` |
| Filteren | `locator.filter({ hasText: 'X' })` |
| Binnen een deel zoeken | `page.getByRole('main').getByRole('button', ...)` |
| Mocken | `await page.route('**/api/**', r => r.fulfill({ json: {} }))` |
| Pauzeren | `await page.pause()` |
| Alles draaien | `npx playwright test` |
| Eén browser | `npx playwright test --project=chromium` |
| UI-modus | `npx playwright test --ui` |
| Opnemen | `npx playwright codegen <url>` |
| Rapport | `npx playwright show-report` |

Officiële documentatie: [playwright.dev](https://playwright.dev/docs/intro). Vooral de pagina's over [Locators](https://playwright.dev/docs/locators) en [Best Practices](https://playwright.dev/docs/best-practices) zijn de moeite waard.
