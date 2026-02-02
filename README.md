# selenium-fritzbox
Demo für GUI-Tests mit Fokus auf Barrierefreiheit (WCAG), Cross-Browser-Support und Usability über Playwright Testautomatisierung mit TypeScript und Selenium-Basis

basiert auf [Perplexity](https://www.perplexity.ai/search/fokus-auf-barrierefreiheit-wca-aI8hBWyuRyiVXDMmVXP1gw#0)

---
[Playwright](https://playwright.dev/) mit [TypeScript](https://playwright.dev/docs/test-typescript) ist aktuell der modernere Standard für [Web‑UI‑Tests](https://aqua-cloud.io/de/ui-tests-ein-umfassender-leitfaden/), während Selenium vor allem als „Legacy‑/Kompatibilitäts‑Arbeitspferd“ weiter wichtig bleibt. Eine kombinierte Umgebung ergibt Sinn, wenn du vorhandene Selenium‑Tests weiterbetreiben musst, aber für neue Features auf Playwright setzt (insbesondere für Cross‑Browser, moderne SPA‑UIs, ARIA‑Lokatoren und integrierten Testrunner).

## Heutige Bedeutung von Playwright vs. Selenium (Stand: 01/2026)
- Selenium ist de‑facto‑Standard in vielen Unternehmen, extrem breit bei Sprachen und Browsern (inkl. älterer Browser, Remote‑Grids, Cloud‑Anbieter) und deshalb für heterogene Landschaften und lange Historie weiterhin relevant.  
- Playwright bietet mit seiner modernen Architektur (DevTools‑basiert/WebSocket, auto‑wait, besseres Locator‑System inkl. ARIA‑Rollen) meist stabilere und schnellere Tests, vor allem für moderne Single‑Page‑Applications, und hat einen integrierten Test Runner mit Parallelisierung.  
- Für WCAG/Barrierefreiheit ist Playwright attraktiv, weil du sehr leicht über Rollen, Labels und Text lokalisieren kannst und so nah an der Accessibility‑Semantik der Anwendung arbeitest. Selenium kann das natürlich auch, aber mit mehr Boilerplate und weniger „Erstklassigkeit“ für ARIA‑Lokatoren.  
- Cross‑Browser‑Support:  
  - Selenium: „alles“, inkl. exotischer Kombinationen und älterer Browser (wenn Treiber existieren).  
  - Playwright: Chromium, Firefox, WebKit – also praktisch Chrome/Edge/Safari und damit die für moderne Web‑Apps relevanten Engines, plus sehr gute Headless‑Performance.  
- Usability‑Tests: Für echte UX‑Forschung bleiben manuelle Tests/UX‑Lab Pflicht; Automatismen prüfen eher „interaktionale Usability“ (Fokus‑Reihenfolge, Tastatur‑Bedienbarkeit, Off‑by‑one‑Fehler, Regressionen), und da punktet Playwright mit stabileren, wartbaren Tests.

Fazit zur Architektur: Für ein neues Setup mit TypeScript auf Ubuntu würde ich Playwright als „First‑Class“ wählen und Selenium nur als Layer für bestehende Tests/Legacy‑Browser nutzen. Ein reines Selenium‑Setup in TypeScript lohnt sich kaum noch, wenn du nicht zu Selenium Clouds/Grids oder sehr exotischen Browsern musst.

***

## Grundidee der kombinierten Umgebung
Zielbild:
- Ubuntu + Node.js + [pnpm/npm](https://www.perplexity.ai/search/fokus-auf-barrierefreiheit-wca-aI8hBWyuRyiVXDMmVXP1gw?sm=d#4) + TypeScript.  
- Playwright als Haupt‑Testframework (inkl. `@playwright/test`).  
- Selenium‑WebDriver in einer separaten Test‑Suite, ebenfalls in TypeScript, um:
  - Alte Selenium‑Tests wiederzuverwenden/migrierbar zu halten.  
  - Ggf. spezielle Fälle abzudecken (z.B. bestimmte Remote‑Grids, IE/Enterprise‑Legacy).  
- Gemeinsame Bausteine:
  - Gemeinsame ESLint/Prettier/TSConfig‑Basis.  
  - Gemeinsame Reporting‑Schicht (z.B. Allure, HTML‑Reports) oder Integration in CI/CD (GitHub Actions, GitLab CI, Jenkins).  
  - Optional: gemeinsame Page‑Objects/Screen‑Modelle, die intern auf Playwright oder Selenium abstrahieren (z.B. über Interface‑Layer).

***

## Schritt‑für‑Schritt‑Einrichtung auf Ubuntu (Playwright + TypeScript)

### 1. System vorbereiten
- Ubuntu aktualisieren:  
  - `sudo apt update && sudo apt upgrade`  
- Node.js LTS installieren (z.B. via `nvm`):  
  - `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash`  
  - Terminal neu laden, dann: `nvm install --lts`  
  - Prüfen: `node -v  # print v24.13.0`, `npm -v # print v11.6.2; use pnpm`  

Optional: `pnpm` installieren, wenn du das bevorzugst: 
```bash
# based at https://www.perplexity.ai/search/fokus-auf-barrierefreiheit-wca-aI8hBWyuRyiVXDMmVXP1gw?sm=d#4
# 1. pnpm installieren (Ubuntu)
npm install -g npm@11.8.0 # update befor npm v11.6.2
npm install -g pnpm
# oder: curl -fsSL https://get.pnpm.io/install.sh | sh -
pnpm -v # print v10.28.2

# 2. Core-Vorteile für Tests nutzen
pnpm init -y
#pnpm add -D @playwright/test @axe-core/playwright typescript
#pnpm add -D selenium-webdriver @types/selenium-webdriver
#pnpm dlx playwright install

# based at part 'Konkrete Einrichtung für dich' und [Konfiguration mit 'cross-env dotenv'](https://www.perplexity.ai/search/hallo-thema-playwright-umgebun-o95aEGP6TAa73NOcTgCSCw#2)
# 3. Deine Test-Deps (schnell!)
pnpm add -D @playwright/test @axe-core/playwright \
  selenium-webdriver @types/selenium-webdriver @types/node \
  typescript tsx cross-env dotenv
pnpm dlx playwright install --with-deps

# 4. Browser & Start
npx tsc --init  # Erstellt Standard-Vorlage
find . -name tsconfig.json
pnpm run env:init  # Erstellt .env.development; ** ERR_PNPM_NO_SCRIPT  Missing script: env:init
```

### 2. Projekt initialisieren (npm)
```bash
mkdir selenium-fritzbox && cd $_
# oder: git clone https://github.com/technomage77/selenium-fritzbox.git && cd selenium-fritzbox/
npm init -y
npm install --save-dev typescript ts-node @types/node
npx tsc --init
```

Wichtige TS‑Compiler‑Optionen im `tsconfig.json` (vereinfacht):
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "moduleResolution": "node",
    "outDir": "dist",
    "types": ["node", "jest"]
  },
  "include": ["tests", "src"]
}
```

(Types für Jest o.ä. nur, wenn du sie brauchst.)

### 3. Playwright mit Test Runner installieren
```bash
pnpm install --save-dev @playwright/test
pnpx playwright install
```

- `pnpx playwright install` lädt die Browser (Chromium, Firefox, WebKit) und richtet die Binaries ein.  
- `npx playwright install-deps`, wenn dir System‑Dependencies fehlen (auf cleanem Server sinnvoll).

Konfigurationsdatei erzeugen:
```bash
pnpx playwright init  # error: unknown command 'init'
```

Das legt u.a. an:
- `playwright.config.ts`  
- Beispieltests unter `tests/`  

In der Config:
- Browser‑Matrix für Cross‑Browser: Chromium, Firefox, WebKit.  
- Parallelisierung, Reporter (HTML, JSON, Allure etc.).  
- Base URL deiner AUT.  

### 4. Erste Tests (mit Blick auf WCAG / Usability)

Beispielhafter Test mit ARIA‑Lokatoren und Tastatur‑Navigation:

```ts
import { test, expect } from '@playwright/test';

test.describe('Login – barrierefrei', () => {
  test('kann rein mit Tastatur bedient werden', async ({ page }) => {
    await page.goto('https://deine-app.example/login');

    // Über ARIA-Role/Label lokalisieren
    const email = page.getByRole('textbox', { name: /e-mail/i });
    const password = page.getByRole('textbox', { name: /passwort/i });
    const submit = page.getByRole('button', { name: /anmelden/i });

    await email.fill('user@example.com');
    await password.fill('geheim123');
    await submit.press('Enter');

    await expect(page).toHaveURL(/dashboard/);
    await expect(page.getByRole('heading', { name: /willkommen/i })).toBeVisible();
  });
});
```

Ergänzend:

- WCAG‑Checks via integrierte Assertions (Focus order, visible labels, `aria-*`) und linters in deiner App.  
- Für automatisierte Accessibility‑Scans integrieren viele Teams `axe-core` oder `@axe-core/playwright`:

```bash
npm install --save-dev @axe-core/playwright
```

Dann im Test:

```ts
import { test } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('Seite erfüllt grundlegende WCAG-Regeln', async ({ page }) => {
  await page.goto('https://deine-app.example/start');
  const accessibilityScanResults = await new AxeBuilder({ page }).analyze();
  // Ergebnisse bewerten, z.B. in Datei/Report schreiben oder spezifische Rule-IDs abprüfen
});
```

***

## Selenium‑Basis in derselben TypeScript‑Umgebung

### 1. Selenium‑Abhängigkeiten

```bash
npm install --save-dev selenium-webdriver
npm install --save-dev @types/selenium-webdriver
```

Browser‑Treiber:

- Chrome/Edge: `chromedriver`/`msedgedriver` (oft nicht mehr händisch nötig, wenn z.B. ChromeDriver mitgeliefert wird bzw. du Remote‑Grid nutzt).  
- Firefox: `geckodriver` (z.B. via `sudo apt install firefox-geckodriver` oder manuell).  

### 2. Einfacher Selenium‑Test in TypeScript

**`scripts/fritzbox.selenium.ts`**
```ts
import { Builder, By, Key, until } from 'selenium-webdriver';
import * as chrome from 'selenium-webdriver/chrome.js';
import fs from "fs";

// based at https://www.perplexity.ai/search/hallo-thema-playwright-umgebun-o95aEGP6TAa73NOcTgCSCw#0
const BASE_URL = process.env.BASE_URL ?? 'http://fritz.box';

async function fritzboxLoginTest() {
  // Chrome mit lokaler IP (kein Headless für Sichtbarkeit)
  const options = new chrome.Options();
  options.addArguments('--disable-web-security'); // Fritz!Box CORS

  const driver = await new Builder()
    .forBrowser('chrome')
    .setChromeOptions(options)
    .build();

  try {
    // Navigate to a URL
    await driver.get(BASE_URL);
    console.log("Navigated to Fritz!Box");

    // Sprache auf Deutsch (falls nötig)
    const langSelect = await driver.wait(
      until.elementLocated(By.css('#language_select')), 5000
    );
    await langSelect.click();
    await driver.findElement(By.css('option[value="deutsch"]')).click();
    console.log("Select language german");

    // Login-Feld (Fritz!Box 7.x/FRITZ!OS 7.5+)
    const usernameField = await driver.wait(
      until.elementLocated(By.name('username')), 10000
    );
    await usernameField.sendKeys(FRITZ_USER); // Aus config!

    const passwordField = driver.findElement(By.name('password'));
    await passwordField.sendKeys(FRITZ_PASS, Key.RETURN);

    // Erfolgreichen Login prüfen (Dashboard oder Übersicht)
    await driver.wait(until.elementLocated(By.id('main_content')), 10000);
    const overviewTitle = await driver.findElement(By.css('h1'));
    const titleText = await overviewTitle.getText();
    console.log('Dashboard-Titel:', titleText);

  } catch (error) {
    console.error('Login-Test fehlgeschlagen:', error);
  } finally {
    // based at https://www.hyperbrowser.ai/docs/sessions/selenium
    // Screenshot
    await driver.takeScreenshot().then((data) => {
      fs.writeFileSync("last_view_results.png", data, "base64");
    });
    console.log("Screenshot saved");

    await driver.quit();
    await client.sessions.stop(session.id);
  }
}

fritzboxLoginTest();
```

Für WCAG/Usability‑Checks müsstest du hier eher extern integrieren (z.B. `axe-core` via `executeScript` in der Seite ausführen) oder z.B. ein Node‑Script schreiben, das HTML‑Snapshots an Axe übergibt. Das ist deutlich mehr Handarbeit als bei Playwright.

***

## Selenium-Suite ausführen

```bash
pnpm env:init  # create '.env.development'
BASE_URL=http://192.168.77.2 pnpm exec tsx scripts/fritzbox.selenium.ts  #passed

DOTENV_CONFIG_PATH=.env.development pnpm exec tsx scripts/fritzbox.selenium.ts  #failed
DOTENV_CONFIG_PATH=.env.development pnpm exec tsx scripts/fritzbox.selenium.ts --trace on
NODE_ENV=dev DOTENV_CONFIG_PATH=.env.development pnpm exec tsx scripts/fritzbox.selenium.ts

TEST_ENV=test pnpm test  # start 'pnpm run test:dev && pnpm run test:selenium' see 'package.json'
TEST_ENV=development pnpm playwright test
TEST_ENV=development pnpm playwright test --debug
pnpm exec playwright show-report
```

***

## CI/CD‑Integration und paralleler Betrieb

- Beide Testarten (Playwright, Selenium) kannst du per NPM‑Scripts und CI‑Jobs orchestrieren, z.B.:

```json
{
  "scripts": {
    "test": "npm run test:pw && npm run test:selenium",
    "test:pw": "playwright test",
    "test:selenium": "ts-node tests/selenium/index.ts"
  }
}
```

- In der Pipeline kannst du z.B. in einem Stage nur Playwright laufen lassen (schnelle Feedback‑Schleife) und Selenium‑Suites seltener (Nacht‑Runs, spezielle Browser/Grids).

***

## Etablierte Tutorials / Ressourcen (Community‑Anerkennung, deutsch/englisch)

Ich kann dir momentan keine konkreten Links aus der Community heraussuchen, aber die folgenden Typen von Quellen haben sich erfahrungsgemäß bewährt:

### Playwright + TypeScript + Ubuntu

- Offizielle Dokumentation „Getting started“ von Playwright mit TypeScript (schrittweise Einrichtung inkl. `npx playwright init`, Cross‑Browser‑Config, CI‑Snippets).  
- Mehrteilige YouTube‑Playlisten „Playwright TypeScript Tutorial“ von bekannten Testing‑Kanälen (z.B. Fokus auf Page‑Object‑Model, Fixtures, Testdaten‑Handling).  
- Artikel in deutschsprachigen Entwickler‑Magazinen/Blogs („End‑to‑End‑Tests mit Playwright“ etc.), die sehr praxisnah zeigen, wie du eine TS‑Codebasis samt Config aufbaust und in CI integrierst.  
- Blogposts von QA‑Dienstleistern, die Selenium durch Playwright ersetzt haben und konkrete Migrationstipps geben (Locator‑Strategie, Test‑Stabilität, Reporting).

### Selenium + TypeScript

- Offizielle Selenium‑Dokumentation und Beispiel‑Repos zur Nutzung mit JavaScript/TypeScript (WebDriver‑API, Treiber‑Handling, Remote‑Grids).  
- Tutorials von Testing‑Blogs, die zeigen, wie du Selenium‑Tests in einer Node.js/TS‑Codebasis strukturierst und parallel ausführst (inkl. Docker‑basierter Grids).  
- Playlists „Selenium WebDriver with TypeScript“ von bekannten Online‑Schulen, die die Grundstruktur (Builder, Page Objects, Waits) sauber vorleben.

### Accessibility und Usability

- Tutorials, die Playwright mit `axe-core` kombinieren und zeigen, wie du automatisierte WCAG‑Checks in deine Tests integrierst.  
- Artikel über „Testing with ARIA roles in Playwright“, in denen Locator‑Strategie, Fokus‑Reihenfolge und Tastatur‑Interaktionen demonstriert werden.

***

## Empfehlung für dein Setup

- Primär: Playwright + TypeScript als vollwertige E2E‑/Cross‑Browser‑Lösung mit Accessibility‑Checks (ARIA‑Lokatoren, `@axe-core/playwright`).  
- Sekundär: Selenium‑Layer nur dort, wo:
  - Alte Tests existieren,  
  - bestimmte Browser/Grids zwingend über WebDriver laufen,  
  - oder du bewusst Technologie‑Diversität brauchst.  
- Langfristig: Schrittweise Migration von Selenium‑Tests auf Playwright, vor allem dort, wo du Barrierefreiheit, moderne SPA‑Verhalten und Stabilität/Tempo optimieren willst.

Wenn du magst, skizziere ich dir im nächsten Schritt ein konkretes Repository‑Layout (Ordnerstruktur, Naming‑Konventionen, gemeinsame Page‑Objects) passend zu deiner CI/CD‑Landschaft (GitLab/Jenkins/GitHub Actions).
