# Linter Installation und Integration über Husky

Diese Anleitung beschreibt, wie du in einem Astro/Starlight Repository alle notwendigen Linter einbaust und mit Husky Git Hooks automatisierst. 

- [Linter Installation und Integration über Husky](#linter-installation-und-integration-über-husky)
  - [Übersicht der Tools](#übersicht-der-tools)
  - [Was ist Husky?](#was-ist-husky)
    - [Warum Husky verwenden?](#warum-husky-verwenden)
    - [Wie funktioniert Husky?](#wie-funktioniert-husky)
    - [Vorteile von Husky](#vorteile-von-husky)
  - [ESLint - JavaScript/TypeScript Code Quality](#eslint---javascripttypescript-code-quality)
    - [Was macht ESLint?](#was-macht-eslint)
    - [Hauptfunktionen](#hauptfunktionen)
    - [Beispiele für ESLint-Regeln](#beispiele-für-eslint-regeln)
    - [Konfiguration](#konfiguration)
    - [Anwendung während der Entwicklung](#anwendung-während-der-entwicklung)
  - [Prettier - Code Formatierung](#prettier---code-formatierung)
    - [Was macht Prettier?](#was-macht-prettier)
    - [Hauptfunktionen](#hauptfunktionen-1)
    - [Beispiele für Prettier-Formatierung](#beispiele-für-prettier-formatierung)
    - [Konfiguration](#konfiguration-1)
  - [Vale - Prose und Dokumentations-Linting](#vale---prose-und-dokumentations-linting)
    - [Was macht Vale?](#was-macht-vale)
    - [Beispiele für Vale-Regeln](#beispiele-für-vale-regeln)
    - [Konfiguration](#konfiguration-2)
  - [Wie die Tools zusammenarbeiten](#wie-die-tools-zusammenarbeiten)
    - [1. Entwicklungsphase](#1-entwicklungsphase)
    - [2. Pre-commit Phase](#2-pre-commit-phase)
    - [3. CI/CD Pipeline](#3-cicd-pipeline)
  - [Workflow für Entwickler](#workflow-für-entwickler)
    - [Beim Entwickeln](#beim-entwickeln)
    - [Vor dem Commit](#vor-dem-commit)
    - [Bei Fehlern](#bei-fehlern)
  - [Installationsanleitung](#installationsanleitung)
    - [Pre-commit hooks via husky](#pre-commit-hooks-via-husky)
      - [Voraussetzungen](#voraussetzungen)
      - [Installation](#installation)
      - [Einfachen pre-commit hook anlegen](#einfachen-pre-commit-hook-anlegen)
      - [Testen](#testen)
      - [Wie wird der hook ausgelöst](#wie-wird-der-hook-ausgelöst)
      - [Setup nach dem Clone](#setup-nach-dem-clone)
      - [Doing nach dem clone](#doing-nach-dem-clone)
    - [ESLint Installation \& Konfiguration](#eslint-installation--konfiguration)
      - [ESLint Dependencies installieren](#eslint-dependencies-installieren)
      - [ESLint-Konfiguration erstellen](#eslint-konfiguration-erstellen)
      - [Lint-Script in package.json hinzufügen](#lint-script-in-packagejson-hinzufügen)
      - [Hook erweitern](#hook-erweitern)
      - [Testen](#testen-1)
      - [Troubleshooting](#troubleshooting)
      - [Hinweise](#hinweise)
    - [Prettier Installation \& Konfiguration](#prettier-installation--konfiguration)
      - [Prettier Dependencies installieren](#prettier-dependencies-installieren)
      - [Prettier-Konfiguration erstellen](#prettier-konfiguration-erstellen)
      - [Prettier-Ignore-Datei erstellen](#prettier-ignore-datei-erstellen)
      - [Format-Scripts in package.json hinzufügen](#format-scripts-in-packagejson-hinzufügen)
      - [Hook erweitern für Prettier](#hook-erweitern-für-prettier)
      - [Testen der Prettier-Integration](#testen-der-prettier-integration)
      - [Editor-Integration für Prettier](#editor-integration-für-prettier)
      - [Troubleshooting Prettier](#troubleshooting-prettier)
      - [Hinweise für Prettier](#hinweise-für-prettier)

## Übersicht der Tools

| Tool         | Zweck                            | Dateitypen                              | Fokus                                                              |
| ------------ | -------------------------------- | --------------------------------------- | ------------------------------------------------------------------ |
| **Husky**    | Git Hooks Automatisierung        | Alle (indirekt über andere Tools)       | Automatische Ausführung von Qualitätsprüfungen bei Git-Operationen |
| **ESLint**   | Code-Qualität und Best Practices | `.js`, `.ts`, `.astro`, `.jsx`, `.tsx`  | JavaScript/TypeScript Syntax, Logik, Patterns                      |
| **Prettier** | Code-Formatierung                | Alle unterstützten Dateitypen           | Einheitliche Formatierung und Styling                              |
| **Vale**     | Prose-Linting                    | `.md`, `.mdx`, Text in Code-Kommentaren | Schreibstil, Grammatik, Terminologie                               |

## Was ist Husky?

Husky ist ein Tool, das **Git Hooks** in Node.js-Projekten vereinfacht und automatisiert. Git Hooks sind Skripte, die automatisch zu bestimmten Zeitpunkten im Git-Workflow ausgeführt werden (z.B. vor einem Commit oder Push).

### Warum Husky verwenden?

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

### Wie funktioniert Husky?

```bash
# Ohne Husky (manuell):
git add .
npm run lint        # Entwickler vergisst diesen Schritt oft!
git commit -m "fix"

# Mit Husky (automatisch):
git add .
git commit -m "fix" # Husky führt automatisch lint + tests aus
```

### Vorteile von Husky

| Vorteil                   | Beschreibung                                      |
| ------------------------- | ------------------------------------------------- |
| **Automatisierung**       | Linter, Tests und Formatierung laufen automatisch |
| **Konsistenz**            | Alle Entwickler haben die gleichen Standards      |
| **Frühe Fehlererkennung** | Probleme werden vor dem Push erkannt              |
| **Einfache Einrichtung**  | Funktioniert nach `npm install` automatisch       |
| **Teamweite Standards**   | Hooks werden im Repository geteilt                |

## ESLint - JavaScript/TypeScript Code Quality

### Was macht ESLint?
ESLint ist ein statisches Code-Analyse-Tool für JavaScript und TypeScript, das potenzielle Probleme, Bugs und stilistische Inkonsistenzen in Ihrem Code identifiziert.

### Hauptfunktionen
- **Fehlerprävention**: Erkennt potenzielle Bugs und problematische Patterns
- **Code-Konsistenz**: Durchsetzt einheitliche Coding Standards
- **Best Practices**: Warnt vor anti-patterns und schlägt bessere Alternativen vor
- **Framework-Support**: Spezielle Regeln für React, Astro, Node.js etc.

### Beispiele für ESLint-Regeln
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

### Konfiguration

ESLint wird über `.eslintrc.js` oder `eslint.config.js` konfiguriert und kann projekt-spezifische Regeln definieren.

### Anwendung während der Entwicklung

Neben Pre-Commit Hooks und Ausführung in der Pipeline kann ESLint wie folgt aufgerufen werden

`npm run lint:check` Führt einen Check der gesamte Codebase des Repositories für alle .js,.ts,.jsx,.tsx,.astro Dateien aus. 

`npm run lint` Führt einen Check der gesamte Codebase des Repositories für alle .js,.ts,.jsx,.tsx,.astro Dateien aus und versucht die gefundenen Errors und Warning automatisch zu fixen.

`npx eslint ./src/lib/taxonomyFilter.ts #--fix` Prüft nur die Datei `/taxonomyFilter.ts` auf Fehler, kann mit dem Parameter `--fix` erweitert werden um ein automatisches fixen.

Automatisch gefixed werden kann: 

- Formatierung (Semicolons, Quotes, Indentation)
- Import-Sortierung
- Trailing Commas
- Spacing-Regeln

## Prettier - Code Formatierung

### Was macht Prettier?
Prettier ist ein "opinionated" Code-Formatter, der automatisch eine einheitliche Formatierung für verschiedene Dateitypen durchsetzt.

### Hauptfunktionen
- **Automatische Formatierung**: Konsistente Einrückung, Zeilenlänge, Quotes etc.
- **Zero-Configuration**: Funktioniert out-of-the-box mit sinnvollen Defaults
- **Multi-Language**: Unterstützt JavaScript, TypeScript, CSS, HTML, Markdown, JSON und mehr
- **Editor-Integration**: Kann beim Speichern automatisch formatieren

### Beispiele für Prettier-Formatierung
```javascript
// Vorher (inkonsistent formatiert):
const obj={name:"John",age:30,city:"Berlin"};
if(condition){doSomething();} else{doSomethingElse();}

// Nachher (Prettier formatiert):
const obj = {
  name: "John",
  age: 30,
  city: "Berlin",
};

if (condition) {
  doSomething();
} else {
  doSomethingElse();
}
```

### Konfiguration
Prettier wird über `.prettierrc` oder `prettier.config.js` konfiguriert.

## Vale - Prose und Dokumentations-Linting

### Was macht Vale?
Vale ist ein Syntax-aware Linter für Prosa und Dokumentation, der Schreibstil, Grammatik und Terminologie-Konsistenz überprüft.


- **Schreibstil-Konsistenz**: Durchsetzt einheitliche Schreibregeln
- **Terminologie-Management**: Stellt sicher, dass Fachbegriffe korrekt verwendet werden
- **Syntax-Aware**: Versteht Markdown, reStructuredText, AsciiDoc etc.
- **Anpassbare Regeln**: Unterstützt verschiedene Style Guides (Google, Microsoft, etc.)

### Beispiele für Vale-Regeln
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

### Konfiguration
Vale wird über `.vale.ini` und Style-Dateien im `.vale/` Verzeichnis konfiguriert.

## Wie die Tools zusammenarbeiten

### 1. Entwicklungsphase
- **ESLint**: Läuft im Editor und zeigt Code-Probleme in Echtzeit
- **Prettier**: Formatiert Code automatisch beim Speichern
- **Vale**: Überprüft Dokumentation und Kommentare

### 2. Pre-commit Phase
- Alle drei Tools laufen automatisch vor jedem Git-Commit
- Verhindert, dass problematischer Code/Content committed wird
- Stellt sicher, dass alle Änderungen den Qualitätsstandards entsprechen

### 3. CI/CD Pipeline
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
1. **ESLint-Fehler**: Code korrigieren oder `eslint --fix` verwenden
2. **Prettier-Fehler**: Automatisch behoben durch `prettier --write`
3. **Vale-Fehler**: Text manuell korrigieren oder Regel anpassen

## Installationsanleitung

### Pre-commit hooks via husky

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

Im Gegensatz zu hooks in `.git/hooks`, müssen Husky-Hooks nach dem Klonen eines Repositories lokal eingerichtet werden. Dies geschieht automatisch mit dem `prepare`-Script.

```bash
# Abhängigkeiten installieren (führt `prepare` aus, wenn es in package.json steht)
npm install

# Falls nötig Husky manuell einrichten (sollte aber durch npm install erledigt sein)
npm run prepare

```

#### Doing nach dem clone

Nachdem du das Repository geklont hast, sind folgende Schritte erforderlich:

1. **Alle Dependencies installieren:**
   ```bash
   npm install
   ```

2. **Git Hooks automatisch einrichten:**
   Nach dem `npm install` werden die Husky-Hooks automatisch aktiviert. Dies passiert über das `prepare` Script in der `package.json`.

3. **Linting-Funktionalität testen:**
   ```bash
   # Alle Dateien auf Linting-Fehler prüfen
   npm run lint:check
   
   # Automatische Fehlerkorrektur versuchen
   npm run lint:fix
   ```

4. **Git-Hooks testen:**
   ```bash
   # Commit mit Linting-Check testen
   git add .
   git commit -m "test: Git-Hook Funktionalität"
   ```
   Bei Linting-Fehlern wird der Commit automatisch abgebrochen.

5. **Entwicklungsserver starten:**
   ```bash
   npm run dev
   ```

**Wichtig:** Husky und alle Linting-Tools sind bereits vollständig konfiguriert. Es ist keine weitere Installation oder Konfiguration erforderlich!

### ESLint Installation & Konfiguration

ESLint ist ein statisches Code-Analyse-Tool für JavaScript und TypeScript, das Code-Qualität und Best Practices durchsetzt. Hier ist eine vollständige Anleitung zur Integration in ein Astro/TypeScript-Projekt.

#### ESLint Dependencies installieren

Installiere ESLint und die benötigten Plugins für TypeScript und Astro:

```bash
npm install --save-dev eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser astro-eslint-parser eslint-plugin-astro
```

**Wichtig:** Diese Dependencies müssen installiert sein, bevor ESLint verwendet werden kann!

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

#### Hook erweitern

Erweitere den Husky pre-commit Hook, um ESLint auszuführen:

```sh
#!/usr/bin/env sh
echo "Execute linting -> code quality and best practices"
npm run lint:check
```

#### Testen

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

#### Troubleshooting

**Fehler: "ESLint couldn't find the config '@typescript-eslint/recommended'"**

Das bedeutet, die TypeScript ESLint-Plugins sind nicht installiert. Führe aus:

```bash
npm install --save-dev @typescript-eslint/eslint-plugin @typescript-eslint/parser astro-eslint-parser eslint-plugin-astro
```

#### Hinweise

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

| Option              | Wert      | Beschreibung                                                    |
| ------------------- | --------- | --------------------------------------------------------------- |
| `semi`              | `true`    | Semikolons am Ende von Statements                               |
| `singleQuote`       | `false`   | Verwende doppelte Anführungszeichen                             |
| `tabWidth`          | `2`       | Anzahl Leerzeichen pro Einrückungsebene                         |
| `trailingComma`     | `"es5"`   | Trailing Commas wo ES5 gültig ist (Objekte, Arrays)            |
| `printWidth`        | `80`      | Maximale Zeilenlänge vor Umbruch                                |
| `arrowParens`       | `"always"`| Klammern um Arrow-Function-Parameter                             |
| `endOfLine`         | `"lf"`    | Unix-style line endings (wichtig für cross-platform)           |
| `proseWrap`         | `"always"`| Markdown-Text wird umgebrochen                                  |

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

#### Hook erweitern für Prettier

Erweitere den Husky pre-commit Hook, um Prettier vor ESLint auszuführen:

```sh
#!/usr/bin/env sh
.
echo "Execute formatting (PRETTIER) -> consistent code formatting"
npm run format

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

#### Troubleshooting Prettier

**Fehler: "Prettier couldn't find the plugin 'prettier-plugin-astro'"**

Das bedeutet, die Prettier-Plugins sind nicht installiert. Führe aus:

```bash
npm install --save-dev prettier-plugin-astro prettier-plugin-tailwindcss
```

**Fehler: "Error: Cannot find module 'prettier-plugin-tailwindcss'"**

Installiere das fehlende Plugin:

```bash
npm install --save-dev prettier-plugin-tailwindcss
```

**Code wird nicht formatiert in VS Code:**

1. Überprüfe, ob die Prettier-Extension installiert ist
2. Setze Prettier als Standard-Formatter in VS Code
3. Aktiviere "Format on Save" in den VS Code Einstellungen

#### Hinweise für Prettier

- Prettier formatiert automatisch bei jedem Commit durch den Husky-Hook
- Du kannst einzelne Zeilen von der Formatierung ausschließen mit `// prettier-ignore`
- Prettier und ESLint können konflikte haben - verwende `eslint-config-prettier` um Konflikte zu vermeiden
- Für Markdown-Dateien wird eine größere `printWidth` (100) verwendet für bessere Lesbarkeit
- Das `prettier-plugin-tailwindcss` Plugin sortiert automatisch Tailwind-Klassen
