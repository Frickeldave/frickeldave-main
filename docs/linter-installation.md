# Linter Installation und Integration über Husky

Diese Anleitung beschreibt, wie du in einem Astro/Starlight Repository alle notwendigen Linter einbaust und mit Husky Git Hooks automatisierst. 

- [Linter Installation und Integration über Husky](#linter-installation-und-integration-über-husky)
  - [Übersicht der Tools](#übersicht-der-tools)
  - [Wie die Tools zusammenarbeiten](#wie-die-tools-zusammenarbeiten)
  - [Workflow für Entwickler](#workflow-für-entwickler)
    - [Beim Entwickeln](#beim-entwickeln)
    - [Vor dem Commit](#vor-dem-commit)
    - [Bei Fehlern](#bei-fehlern)
  - [Nützliche Befehle für Entwickler](#nützliche-befehle-für-entwickler)
  - [Installationsanleitung](#installationsanleitung)
    - [Pre-commit hooks via husky](#pre-commit-hooks-via-husky)
      - [Voraussetzungen](#voraussetzungen)
      - [Installation](#installation)
      - [Einfachen pre-commit hook anlegen](#einfachen-pre-commit-hook-anlegen)
      - [Testen](#testen)
      - [Wie wird der hook ausgelöst](#wie-wird-der-hook-ausgelöst)
      - [Setup nach dem Clone](#setup-nach-dem-clone)
    - [ESLint Installation \& Konfiguration](#eslint-installation--konfiguration)
      - [ESLint Dependencies installieren](#eslint-dependencies-installieren)
      - [ESLint-Konfiguration erstellen](#eslint-konfiguration-erstellen)
      - [Lint-Script in package.json hinzufügen](#lint-script-in-packagejson-hinzufügen)
      - [ESLint-Ignore-Datei erstellen](#eslint-ignore-datei-erstellen)
      - [Hook erweitern für ESLint](#hook-erweitern-für-eslint)
      - [Testen der ESLint Integration](#testen-der-eslint-integration)
      - [Hinweise für ESLint](#hinweise-für-eslint)
    - [Prettier Installation \& Konfiguration](#prettier-installation--konfiguration)
      - [Prettier Dependencies installieren](#prettier-dependencies-installieren)
      - [Prettier-Konfiguration erstellen](#prettier-konfiguration-erstellen)
      - [Format-Scripts in package.json hinzufügen](#format-scripts-in-packagejson-hinzufügen)
      - [Prettier-Ignore-Datei erstellen](#prettier-ignore-datei-erstellen)
      - [Hook erweitern für Prettier](#hook-erweitern-für-prettier)
      - [Testen der Prettier-Integration](#testen-der-prettier-integration)
      - [Editor-Integration für Prettier](#editor-integration-für-prettier)
      - [Hinweise für Prettier](#hinweise-für-prettier)
    - [Vale Installation \& Konfiguration](#vale-installation--konfiguration)
      - [Installationsvorbereitungen](#installationsvorbereitungen)
      - [Vale-Konfiguration erstellen](#vale-konfiguration-erstellen)
      - [Hook erweitern für Vale](#hook-erweitern-für-vale)
      - [Testen der Vale-Integration](#testen-der-vale-integration)
      - [Vale Styles und Regeln](#vale-styles-und-regeln)
      - [Hinweise für Vale](#hinweise-für-vale)

## Übersicht der Tools

| Tool         | Zweck                            | Dateitypen                              | Fokus                                                              |
| ------------ | -------------------------------- | --------------------------------------- | ------------------------------------------------------------------ |
| **Husky**    | Git Hooks Automatisierung        | Alle (indirekt über andere Tools)       | Automatische Ausführung von Qualitätsprüfungen bei Git-Operationen |
| **ESLint**   | Code-Qualität und Best Practices | `.js`, `.ts`, `.astro`, `.jsx`, `.tsx`  | JavaScript/TypeScript Syntax, Logik, Patterns                      |
| **Prettier** | Code-Formatierung                | Alle unterstützten Dateitypen           | Einheitliche Formatierung und Styling                              |
| **Vale**     | Prose-Linting                    | `.md`, `.mdx`, Text in Code-Kommentaren | Schreibstil, Grammatik, Terminologie                               |

## Wie die Tools zusammenarbeiten

**1. Entwicklungsphase**
- **ESLint**: Läuft im Editor und zeigt Code-Probleme in Echtzeit
- **Prettier**: Formatiert Code automatisch beim Speichern
- **Vale**: Überprüft Dokumentation und Kommentare

**1. Pre-commit Phase**
- Alle drei Tools laufen automatisch vor jedem Git-Commit
- Verhindert, dass problematischer Code/Content committed wird
- Stellt sicher, dass alle Änderungen den Qualitätsstandards entsprechen

**3. CI/CD Pipeline**
- Validiert, dass alle Regeln eingehalten werden
- Blockiert Merges bei Regelverstößen
- Stellt sicher, dass nur qualitativ hochwertiger Code deployed wird

## Workflow für Entwickler

### Beim Entwickeln
1. **Code schreiben** - ESLint zeigt Probleme in Echtzeit
2. **Datei speichern** - Prettier formatiert automatisch
3. **Dokumentation schreiben** - Vale prüft Schreibstil

### Vor dem Commit
1. **Pre-commit Hook** läuft automatisch
2. **ESLint** prüft alle geänderten Code-Dateien
3. **Prettier** formatiert alle Dateien
4. **Vale** prüft alle Markdown-Dateien
5. **Commit wird blockiert** bei Fehlern

### Bei Fehlern
1. **ESLint-Fehler**: Code korrigieren oder `npm run lint` verwenden
2. **Prettier-Fehler**: Automatisch behoben durch `npm run format`
3. **Vale-Fehler**: Text manuell korrigieren oder Regel anpassen

## Nützliche Befehle für Entwickler

- `npm install`: Installiert alle Dependencies und Vale automatisch über postinstall Hook
- `npm run lint`: Versucht den Code automatisch zu korrigieren
- `npm run lint:check`: Findet Fehler im Code und zeigt diese an
- `npm run format`: Formatiert alle Dateien gemäß den Vorgaben
- `npm run format:check`: Findet Formatierungsfehler und zeigt diese an
- `npm run install-vale`: Installiert Vale manuell (falls erforderlich)
- `npm run prose`: Findet alle Prosa-Fehler und zeigt diese an
- `npm run lint:all`: Findet alle Issues im Code und versucht diese soweit es geht automatisch zu korrigieren
- `npm run lint:checkall`: Findet alle Issues im Code und zeigt die Ergebnisse an

## Installationsanleitung

Diese Anleitung beschreibt, wie die einzelnen Tools in ein NEUES astro oder node Projekt eingebunden werden. Dieses Schritte dienen der reinen Dokumentation und müssen NICHT vom Entwickler durchgeführt werden. 

### Pre-commit hooks via husky

Husky ist ein Tool, das **Git Hooks** in Node.js-Projekten vereinfacht und automatisiert. Git Hooks sind Skripte, die automatisch zu bestimmten Zeitpunkten im Git-Workflow ausgeführt werden (z.B. vor einem Commit oder Push).

**Problem ohne Husky:**
- Entwickler vergessen, Linter vor dem Commit auszuführen
- Inkonsistente Code-Qualität erreicht das Repository
- CI/CD-Pipeline schlägt erst spät fehl
- Manuelle Prozesse sind fehleranfällig

**Lösung mit Husky:**
- **Automatische Qualitätsprüfung** bei jedem Commit/Push
- **Verhindert schlechten Code** im Repository
- **Konsistente Standards** für alle Entwickler
- **Frühe Fehlererkennung** vor CI/CD

**Wie funktioniert Husky?**

```bash
# Ohne Husky (manuell):
git add .
npm run lint        # Entwickler vergisst diesen Schritt oft!
git commit -m "fix"

# Mit Husky (automatisch):
git add .
git commit -m "fix" # Husky führt automatisch lint + tests aus
```

**Vorteile von Husky**

| Vorteil                   | Beschreibung                                      |
| ------------------------- | ------------------------------------------------- |
| **Automatisierung**       | Linter, Tests und Formatierung laufen automatisch |
| **Konsistenz**            | Alle Entwickler haben die gleichen Standards      |
| **Frühe Fehlererkennung** | Probleme werden vor dem Push erkannt              |
| **Einfache Einrichtung**  | Funktioniert nach `npm install` automatisch       |
| **Teamweite Standards**   | Hooks werden im Repository geteilt                |



#### Voraussetzungen
- Node.js (>=18) und npm installiert
- Git-Repository initialisiert (oder bestehendes Repo)

#### Installation
   
Öffne ein Terminal im Ziel-Repo und installiere Husky als Dev-Dependency:

```bash
npm install --save-dev husky
```

`prepare`-Script in `package.json` eintragen

Füge in `package.json` unter `scripts` folgendes hinzu (falls noch nicht vorhanden):

```json
"scripts": {
  "prepare": "husky install"
}
```
Husky einmalig initialisieren (setzt `git config core.hooksPath .husky` und erstellt `.husky`). Husky-Versionen (ab v7) setzen die Git-Konfiguration `core.hooksPath` auf das Verzeichnis `.husky/` im Projekt. Git liest Hooks also nicht aus `.git/hooks`, sondern aus dem Pfad, der in `core.hooksPath` konfiguriert ist. Deshalb zeigt ein `ls .git/hooks` häufig nur die mitgelieferten `*.sample`-Dateien, obwohl Husky-Hooks unter `.husky/` aktiv sind.

```bash
npm run prepare
```

#### Einfachen pre-commit hook anlegen


Lege die Datei `.husky/pre-commit` mit folgendem Inhalt an:

```sh
#!/usr/bin/env sh
.
echo "Husky pre-commit hook: hello from husky!"
```

Setze die Datei ausführbar:

```bash
chmod +x .husky/pre-commit
```

#### Testen

Erzeuge eine Änderung, stage und committe:

```bash
git add .
git commit -m "test husky"
```

Beim Commit solltest du die Nachricht vom Hook in der Konsole sehen.

#### Wie wird der hook ausgelöst

Prüfen, wo Git Hooks sucht:

```bash
git config --get core.hooksPath
git config --show-origin core.hooksPath
```

Prüfen, ob der Husky-Hook vorhanden und ausführbar ist:

```bash
ls -l .husky
ls -l .husky/pre-commit
cat .husky/pre-commit
```

#### Setup nach dem Clone

Im Gegensatz zu hooks in `.git/hooks`, müssen Husky-Hooks nach dem Klonen eines Repositories lokal eingerichtet werden, dies passiert über das `prepare` Script in der `package.json`. Führe hierzu bitte folgenden Befehl aus

```bash

# Abhängigkeiten installieren (führt `prepare` aus, wenn es in package.json steht)
npm install

```

   

### ESLint Installation & Konfiguration

ESLint ist ein statisches Code-Analyse-Tool für JavaScript und TypeScript, das potenzielle Probleme, Bugs und stilistische Inkonsistenzen in Ihrem Code identifiziert.

- **Fehlerprävention**: Erkennt potenzielle Bugs und problematische Patterns
- **Code-Konsistenz**: Durchsetzt einheitliche Coding Standards
- **Best Practices**: Warnt vor anti-patterns und schlägt bessere Alternativen vor
- **Framework-Support**: Spezielle Regeln für React, Astro, Node.js etc.

```javascript

// ❌ ESLint würde warnen:
var unusedVariable = 'never used';  // unused variable
if (condition = true) { }            // assignment instead of comparison
function foo() { return; }           // unreachable code

// ✅ ESLint approved:
const usedVariable = 'properly used';
if (condition === true) { }
function foo() { return 'value'; }

```
**Konfiguration**
ESLint wird über `.eslintrc.js` oder `eslint.config.js` konfiguriert und kann projekt-spezifische Regeln definieren.

**Anwendung während der Entwicklung**

Neben Pre-Commit Hooks und Ausführung in der Pipeline kann ESLint wie folgt aufgerufen werden

- `npm run lint:check` Führt einen Check der gesamte Codebase des Repositories für alle .js,.ts,.jsx,.tsx,.astro Dateien aus. 
- `npm run lint` Führt einen Check der gesamte Codebase des Repositories für alle .js,.ts,.jsx,.tsx,.astro Dateien aus und versucht die gefundenen Errors und Warning automatisch zu fixen.
- `npx eslint ./src/lib/taxonomyFilter.ts #--fix` Prüft nur die Datei `/taxonomyFilter.ts` auf Fehler, kann mit dem Parameter `--fix` erweitert werden um ein automatisches fixen.

Automatisch gefixed werden kann: 

- Formatierung (Semicolons, Quotes, Indentation)
- Import-Sortierung
- Trailing Commas
- Spacing-Regeln


#### ESLint Dependencies installieren

Installiere ESLint und die benötigten Plugins für TypeScript und Astro:

```bash
npm install --save-dev eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser astro-eslint-parser eslint-plugin-astro
```

#### ESLint-Konfiguration erstellen

Erstelle eine `.eslintrc.js` im Projekt-Root:

```javascript
module.exports = {
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
  ],
  parser: "@typescript-eslint/parser",
  plugins: ["@typescript-eslint"],
  parserOptions: {
    ecmaVersion: "latest",
    sourceType: "module",
  },
  env: {
    node: true,
    browser: true,
    es2022: true,
  },
  rules: {
    // Allow unused vars that start with underscore
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    // Allow any type occasionally 
    "@typescript-eslint/no-explicit-any": "warn",
    // Allow empty interfaces for extending
    "@typescript-eslint/no-empty-interface": "off",
  },
  overrides: [
    {
      files: ["*.astro"],
      extends: ["plugin:astro/recommended"],
      parser: "astro-eslint-parser",
      parserOptions: {
        parser: "@typescript-eslint/parser",
        extraFileExtensions: [".astro"],
      },
      rules: {
        // Astro specific rules can go here
      },
    },
  ],
};
```

#### Lint-Script in package.json hinzufügen

Füge ein `lint`-Script zu den Scripts in `package.json` hinzu:

```json
"scripts": {
  "lint": "eslint ./src --ext .js,.ts,.jsx,.tsx,.astro --fix",
  "lint:check": "eslint ./src --ext .js,.ts,.jsx,.tsx,.astro"
}
```

Das `--fix` Flag repariert automatisch behebbare Probleme. Mit `lint:check` kannst du nur prüfen, ohne automatische Reparaturen.

#### ESLint-Ignore-Datei erstellen

Erstelle eine `.eslintignore` im Projekt-Root, um bestimmte Dateien und Ordner von der Linting auszuschließen:

```gitignore
# Build outputs
dist/
build/
.astro/

# Dependencies
node_modules/

# Environment files
.env
.env.*

# Generated files
public/
*.config.js
*.config.mjs
*.config.ts

# Cache directories
.cache/
.temp/
.tmp/

# Editor directories and files
.vscode/
.idea/
*.swp
*.swo

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Lock files
package-lock.json
pnpm-lock.yaml
yarn.lock

# Documentation that shouldn't be linted
docs/
README.md
CHANGELOG.md
LICENSE

# Test coverage
coverage/

# Husky hooks (shell scripts)
.husky/

# Wrangler (Cloudflare)
wrangler.toml
```

**Wichtige Ignore-Patterns erklärt:**

| Pattern                | Grund                                                    |
| ---------------------- | -------------------------------------------------------- |
| `dist/`, `build/`      | Build-Ausgaben sollten nicht gelintet werden             |
| `node_modules/`        | Dependencies sind bereits gelintet                       |
| `public/`              | Statische Assets brauchen kein Linting                   |
| `*.config.*`           | Konfigurationsdateien haben oft andere Standards         |
| `.astro/`              | Astro-Build-Cache                                        |
| `docs/`                | Dokumentation braucht meist kein JavaScript-Linting      |
| `.husky/`              | Shell-Scripts, nicht JavaScript                          |
| `wrangler.toml`        | Cloudflare-Konfiguration                                 |

#### Hook erweitern für ESLint

Erweitere den Husky pre-commit Hook, um ESLint auszuführen:

```sh
#!/usr/bin/env sh
echo "Execute linting -> code quality and best practices"
npm run lint:check
```

#### Testen der ESLint Integration

**Zuerst die Dependencies installieren (falls noch nicht geschehen):**

```bash
npm install
```

Dann das ESLint-Setup testen:

```bash
# ESLint manuell ausführen (nur prüfen)
npm run lint:check

# ESLint mit automatischen Reparaturen
npm run lint

# Mit einem Commit testen (führt den Hook aus)
git add .
git commit -m "test eslint integration"
```

#### Hinweise für ESLint

- ESLint wird nun bei jedem Commit automatisch ausgeführt und versucht, Probleme zu beheben
- Falls ESLint Fehler findet, die nicht automatisch behoben werden können, wird der Commit blockiert
- Du kannst einzelne Regeln in der `.eslintrc.js` anpassen oder deaktivieren
- Für größere Projekte empfiehlt sich die Verwendung von `lint-staged`, um nur geänderte Dateien zu prüfen

### Prettier Installation & Konfiguration

Prettier ist ein "opinionated" Code-Formatter, der automatisch eine einheitliche Formatierung für verschiedene Dateitypen durchsetzt. Hier ist eine vollständige Anleitung zur Integration in ein Astro/TypeScript-Projekt.

#### Prettier Dependencies installieren

Installiere Prettier und die benötigten Plugins für Astro und Tailwind:

```bash
npm install --save-dev prettier prettier-plugin-astro prettier-plugin-tailwindcss
```

**Wichtig:** Diese Dependencies müssen installiert sein, bevor Prettier verwendet werden kann!

#### Prettier-Konfiguration erstellen

Erstelle eine `.prettierrc` im Projekt-Root:

```json
{
  "plugins": ["prettier-plugin-astro", "prettier-plugin-tailwindcss"],
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80,
  "useTabs": false,
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "always",
  "endOfLine": "lf",
  "overrides": [
    {
      "files": ["*.astro"],
      "options": {
        "parser": "astro"
      }
    },
    {
      "files": ["*.md", "*.mdx"],
      "options": {
        "printWidth": 100,
        "proseWrap": "always"
      }
    },
    {
      "files": ["*.json", "*.jsonc"],
      "options": {
        "printWidth": 100
      }
    }
  ]
}
```

**Erklärung der wichtigsten Konfigurationsoptionen:**

| Option          | Wert       | Beschreibung                                         |
| --------------- | ---------- | ---------------------------------------------------- |
| `semi`          | `true`     | Semikolons am Ende von Statements                    |
| `singleQuote`   | `false`    | Verwende doppelte Anführungszeichen                  |
| `tabWidth`      | `2`        | Anzahl Leerzeichen pro Einrückungsebene              |
| `trailingComma` | `"es5"`    | Trailing Commas wo ES5 gültig ist (Objekte, Arrays)  |
| `printWidth`    | `80`       | Maximale Zeilenlänge vor Umbruch                     |
| `arrowParens`   | `"always"` | Klammern um Arrow-Function-Parameter                 |
| `endOfLine`     | `"lf"`     | Unix-style line endings (wichtig für cross-platform) |
| `proseWrap`     | `"always"` | Markdown-Text wird umgebrochen                       |

#### Format-Scripts in package.json hinzufügen

Füge `format`-Scripts zu den Scripts in `package.json` hinzu:

```json
"scripts": {
  "format": "prettier -w ./src",
  "format:check": "prettier --check ./src"
}
```

**Script-Erklärung:**
- `format`: Formatiert alle Dateien im `src/` Verzeichnis und schreibt Änderungen zurück (`-w` = write)
- `format:check`: Prüft nur, ob Dateien korrekt formatiert sind, ohne sie zu ändern

#### Prettier-Ignore-Datei erstellen

Erstelle eine `.prettierignore` im Projekt-Root, um bestimmte Dateien von der Formatierung auszuschließen:

```gitignore
# Build outputs
dist/
build/
.astro/

# Dependencies
node_modules/

# Environment files
.env
.env.*

# Generated files
public/sitemap-*.xml
public/rss.xml

# Package files
package-lock.json
pnpm-lock.yaml
yarn.lock

# Cache directories
.cache/
.temp/
.tmp/

# Editor directories and files
.vscode/
.idea/
*.swp
*.swo

# OS generated files
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Documentation that shouldn't be formatted
CHANGELOG.md
LICENSE
```

#### Hook erweitern für Prettier

Erweitere den Husky pre-commit Hook, um Prettier vor ESLint auszuführen:

```sh
#!/usr/bin/env sh
.
echo "Execute formatting (PRETTIER) -> consistent code formatting"
npm run format:check

echo "Execute linting (ESLINT) -> code quality and best practices"
npm run lint:check

if [ $? -ne 0 ]; then
  echo "❌ ESLINT failed! Please fix the linting errors before committing."
  echo "💡 Run 'npm run lint' to automatically fix some issues."
  exit 1
fi

echo "✅ All checks passed successfully!"
```

**Wichtig:** Prettier läuft vor ESLint, da Prettier Formatierung Einfluss auf ESLint-Regeln haben kann.

#### Testen der Prettier-Integration

**Zuerst die Dependencies installieren (falls noch nicht geschehen):**

```bash
npm install
```

Dann das Prettier-Setup testen:

```bash
# Prettier manuell ausführen (nur prüfen)
npm run format:check

# Prettier mit automatischer Formatierung
npm run format

# Einzelne Datei formatieren
npx prettier --write src/components/example.astro

# Mit einem Commit testen (führt den Hook aus)
git add .
git commit -m "test prettier integration"
```

#### Editor-Integration für Prettier

**VS Code Konfiguration (.vscode/settings.json):**

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "prettier.documentSelectors": ["**/*.astro"],
  "[astro]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

**Installiere die VS Code Prettier Extension:**

```bash
code --install-extension esbenp.prettier-vscode
```

#### Hinweise für Prettier

- Prettier formatiert automatisch bei jedem Commit durch den Husky-Hook
- Du kannst einzelne Zeilen von der Formatierung ausschließen mit `// prettier-ignore`
- Prettier und ESLint können konflikte haben - verwende `eslint-config-prettier` um Konflikte zu vermeiden
- Für Markdown-Dateien wird eine größere `printWidth` (100) verwendet für bessere Lesbarkeit
- Das `prettier-plugin-tailwindcss` Plugin sortiert automatisch Tailwind-Klassen

### Vale Installation & Konfiguration

Vale ist ein Syntax-aware Linter für Prosa und Dokumentation, der Schreibstil, Grammatik und Terminologie-Konsistenz überprüft. 

- **Schreibstil-Konsistenz**: Durchsetzt einheitliche Schreibregeln
- **Terminologie-Management**: Stellt sicher, dass Fachbegriffe korrekt verwendet werden
- **Syntax-Aware**: Versteht Markdown, reStructuredText, AsciiDoc etc.
- **Anpassbare Regeln**: Unterstützt verschiedene Style Guides (Google, Microsoft, etc.)

**Beispiele für Vale-Regeln
```markdown
❌ Vale würde warnen:
- "We should of done this differently" (should have, nicht should of)
- "The API is very easy to use" (subjektive Sprache)
- "Click here for more info" (nicht-deskriptive Links)

✅ Vale approved:
- "We should have implemented this differently"
- "The API provides a straightforward interface"
- "Read the complete documentation for detailed examples"
```

#### Installationsvorbereitungen

Da Vale nicht als Node.js-Package verfügbar ist, wird die Installation über Shell-Scripte vorgenommen, die im `scripts/` Ordner des Repositories gespeichert sind:

- **Linux/macOS**: `scripts/install-vale.sh` 
- **Windows**: `scripts/install-vale.ps1`

Diese Scripte laden die jeweils passende Vale-Binary herunter und speichern diese im `tools/` Verzeichnis, welches in der `.gitignore` aufgenommen ist.

**Automatische Installation:**
Vale wird automatisch über einen `postinstall` Hook nach jedem `npm install` installiert:

```json
"scripts": {
  "postinstall": "npm run install-vale",
  "install-vale": "node -e \"const cmd = process.platform === 'win32' ? 'powershell -ExecutionPolicy Bypass -File scripts/install-vale.ps1' : 'bash scripts/install-vale.sh'; require('child_process').exec(cmd, (err, stdout, stderr) => { if (err) { console.error(stderr); process.exit(1); } console.log(stdout); });\"",
  "prose": "node -e \"const vale = process.platform === 'win32' ? './tools/vale.exe' : './tools/vale'; const { exec } = require('child_process'); exec(vale + ' --config=.vale.ini src/content docs', (err, stdout, stderr) => { console.log(stdout); if (stderr) console.error(stderr); if (err) process.exit(1); });\"",
  "prose:check": "npm run prose"
}
```

**Warum dieser Ansatz?**
- ✅ Vale wird automatisch nach dem Klonen installiert  
- ✅ Keine manuellen Installationsschritte erforderlich
- ✅ Korrekte Ausführungsberechtigungen werden automatisch gesetzt
- ✅ Plattformübergreifende Kompatibilität (Windows/Linux/macOS)
- ✅ Vale-Binary wird nicht im Git-Repository gespeichert (siehe `.gitignore`)

#### Vale-Konfiguration erstellen


Vale wird über `.vale.ini` und Style-Dateien im `.vale/` Verzeichnis konfiguriert.
Über die `.vale.ini` im Projekt-Root wird die Konfiguration vorgenommen. Folgend ist eine simple start Konfiguration dargestellt.

```ini
# Vale configuration file
# See: https://vale.sh/docs/topics/config/

StylesPath = .vale

MinAlertLevel = suggestion

# File type associations
[*.{md,mdx}]
# Enable/disable specific styles
BasedOnStyles = Vale, write-good, Microsoft

# Enable rules
Vale.Terms = YES
Vale.Spelling = YES
write-good.Weasel = YES
write-good.TooWordy = YES
Microsoft.Contractions = YES
Microsoft.FirstPerson = NO
Microsoft.Passive = suggestion

# Custom vocabulary
Vale.Vocab = Base

[*.{txt}]
BasedOnStyles = Vale

[*.{astro}]
# For code comments and content within Astro files
BasedOnStyles = Vale
# Transform to extract prose from code comments
Transform = (?s)<!--\s*(.*?)\s*-->
```

Viele Regeln automatisch von vale geprüft oder sind in den rules definiert. Darüber hinaus 
Die Base-Vokabular-Datei prüft darauf, ob Wörter korrekt geschrieben sind.

**Base Vocabulary** (`.vale/styles/Base/accept.txt`):

```
STACKIT
Astro
TypeScript
JavaScript
MDX
Tailwind
GitHub
Cloudflare
API
APIs
CSS
HTML
React
Node.js
npm
CLI
UI
UX
SEO
RSS
JSON
YAML
TOML
Markdown
```

**Base Rejections** (`.vale/styles/Base/reject.txt`):

```
# Common typos and alternatives
Javascript -> JavaScript
Typescript -> TypeScript
Github -> GitHub
```

#### Hook erweitern für Vale

Erweitere den Husky pre-commit Hook um Vale:

```sh
#!/usr/bin/env sh
.

echo ""
echo "🚀 Pre-commit Quality Checks"
echo "=============================="
echo ""

...
...
...

echo "📝 Running Vale (Prose Linting)..."
echo "   ↳ Checking documentation and content for style consistency"
npm run prose:check

if [ $? -ne 0 ]; then
  echo ""
  echo "❌ VALE failed!"
  echo "   ↳ Prose style issues found in documentation"
  echo "   💡 Review the suggestions above and edit the content manually"
  echo "   📋 Vale cannot auto-fix - manual review required"
  echo ""
  exit 1
fi

echo "   ✅ Prose style checks passed successfully"
echo ""

...
...
...

```

#### Testen der Vale-Integration

**Nach der Installation:**

Vale wird automatisch installiert wenn du das Repository klonst und `npm install` ausführst. Falls Vale bereits installiert ist, kannst du die Integration direkt testen:

```bash
# Dependencies installieren (installiert automatisch Vale über postinstall Hook)
npm install

# Vale-Konfiguration testen
npm run prose

# Einzelne Datei testen
./tools/vale --config=.vale.ini src/content/blog/example.md

# Vale-Version überprüfen
./tools/vale --version

# Mit Commit testen (führt den Hook aus)
git add .
git commit -m "test vale integration"
```
#### Vale Styles und Regeln

**Zusätzliche Style-Pakete installieren:**

```bash
# Microsoft Writing Style Guide
vale sync

# Oder manuell herunterladen
mkdir -p .vale/styles
cd .vale/styles
git clone https://github.com/errata-ai/Microsoft.git
git clone https://github.com/errata-ai/write-good.git
```

**Custom Rules erstellen** (`.vale/styles/Custom/NoPassive.yml`):

```yaml
extends: existence
message: "Avoid passive voice: '%s'"
level: suggestion
scope: sentence
tokens:
  - 'was\s+\w+ed'
  - 'were\s+\w+ed'
  - 'is\s+\w+ed'
  - 'are\s+\w+ed'
```

#### Hinweise für Vale

- **Vale kann nicht automatisch fixen** - alle Fehler müssen manuell korrigiert werden
- **Style-Regeln sind anpassbar** - du kannst eigene Regeln für dein Projekt erstellen
- **Vocabulary-Management** - führe projektspezifische Begriffe in `accept.txt` auf
- **Performance** - Vale ist schnell, aber bei sehr großen Repositories kann es länger dauern
- **IDE-Integration** - Es gibt Vale-Extensions für VS Code und andere Editoren
- **CI/CD-Integration** - Vale kann auch in GitHub Actions oder anderen CI-Systemen laufen

**Empfohlene Vale-Einstellungen für technische Dokumentation:**

```ini
# Weniger streng für technische Inhalte
MinAlertLevel = warning

# Projektspezifische Anpassungen
Microsoft.FirstPerson = NO    # "We" und "I" sind in Tutorials OK
Microsoft.Passive = suggestion  # Passive Voice manchmal notwendig
write-good.Weasel = suggestion  # "Easy", "simple" sind in Erklärungen OK
```
