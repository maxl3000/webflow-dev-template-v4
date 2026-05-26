# 🚀 Mein Start-Guide: Neues Webflow-Dev-Projekt

Eine Schritt-für-Schritt-Anleitung für mich (Anfänger), um jedes Mal sauber ein neues Projekt aufzusetzen — mit allen Stolperfallen, die ich schon kenne.

> **Kurz-Begriffe:**
> - **bun** = schneller Paketmanager + Tool zum Ausführen von JS/TS (Ersatz für npm/node)
> - **degit** = lädt ein GitHub-Repo herunter, *ohne* dessen Git-Historie (sauberer Start)
> - **Dev-Server** = läuft lokal auf meinem Mac unter `https://localhost:6545` und liefert meinen Code live an Webflow
> - **mkcert / SSL** = erstellt ein lokales HTTPS-Zertifikat, damit Webflow (HTTPS) meinen lokalen Server laden darf

---

## ✅ Einmalig (schon erledigt, nur zur Info)

Diese Tools sind global installiert und müssen **nicht** pro Projekt neu installiert werden:
- `bun`, `bunx` (über pnpm)
- `vercel` (über pnpm)
- `gh` (GitHub CLI, über Homebrew)

Prüfen kann ich das jederzeit mit:
```bash
bun --version
vercel --version
gh auth status      # bin ich bei GitHub eingeloggt?
```

Falls `gh` „not logged in" sagt → in einem **normalen Terminal** (nicht über Tools) einloggen:
```bash
gh auth login
# GitHub.com → HTTPS → Login with a web browser → Code eingeben → Authorize
```

> ⚠️ **Mein GitHub-Username ist `maxl3000`** (mit kleinem **L**, nicht die Ziffer 1). Verwechsle ihn nicht mit `max13000` — das führt zu „Repository not found".

---

## 1. Projekt anlegen & Vorlage laden

```bash
# Ordner erstellen und reingehen
mkdir mein-neues-projekt
cd mein-neues-projekt

# Vorlage herunterladen (ohne Git-Historie)
degit vallafederico/webflow-dev-setup

# Abhängigkeiten installieren
bun install
```

> 💡 Wenn der Ordner schon Dateien enthält, meckert degit. Dann: `degit vallafederico/webflow-dev-setup --force`

---

## 2. `.env` einrichten (WICHTIG: keine geschweiften Klammern!)

```bash
cp .env.example .env
```

Dann `.env` öffnen und ausfüllen. **Die `{ }`-Klammern sind nur Platzhalter und müssen WEG.**

❌ **Falsch** (Klammern stehen lassen):
```
VERCEL_URL="{webflow-dev-template-v4.vercel.app}"
```

✅ **Richtig** (volle URL, mit https, ohne Klammern):
```
VERCEL_URL="https://mein-projekt.vercel.app"
USE_SSL=true
```

> Diese Klammern haben mich beim ersten Mal ~30 Min gekostet (404-Fehler `…/%7B…%7D/app.css`). Immer entfernen!

---

## 3. HTTPS-Zertifikat einrichten (einmal pro Projekt)

Webflow läuft über HTTPS und darf meinen lokalen Server nur laden, wenn der auch HTTPS mit einem **vertrauenswürdigen** Zertifikat hat.

```bash
bun setup-ssl
```

Das Skript fragt nach meinem **Mac-Passwort** (`sudo`), um die lokale CA als vertrauenswürdig einzutragen.

> ⚠️ Falls `bun setup-ssl` nicht durchläuft (z. B. über ein Tool ohne sudo-Eingabe), reicht auch:
> 1. Zertifikate erzeugen (ohne sudo):
>    ```bash
>    mkdir -p certs
>    node_modules/.bin/mkcert create-ca && mv -f ca.key ca.crt certs/
>    node_modules/.bin/mkcert create-cert --key certs/localhost-key.pem --cert certs/localhost.pem --ca-key certs/ca.key --ca-cert certs/ca.crt localhost 127.0.0.1 ::1
>    ```
> 2. CA vertrauen (fragt nach Passwort):
>    ```bash
>    sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain "$(pwd)/certs/ca.crt"
>    ```
> 3. **Browser komplett neu starten** (Cmd+Q), nicht nur Tab schließen.

---

## 4. Dev-Server starten

```bash
bun dev
```

Läuft dann auf **https://localhost:6545**. Diese Seite im Browser öffnen — dort steht der **Loader-Code** zum Kopieren (Klick auf den Kasten = kopiert).

> Der Server läuft weiter und baut bei jeder Änderung in `src/` automatisch neu (Live-Reload). Stoppen mit `Ctrl + C` im Terminal.

---

## 5. Mit Webflow verbinden

1. In Webflow: **Project Settings → Custom Code → Head Code**
2. Den **Loader-Block** von `https://localhost:6545` einfügen (nur diesen einen!)
3. **Save** → **Publish** (auf die `.webflow.io`-Staging-Domain)
4. Staging-Seite öffnen und Konsole prüfen (Rechtsklick → Untersuchen → Console)

> ⚠️ **Niemals** zusätzlich alte einzelne Tags wie
> `<script src="http://localhost:6545/app.js">` stehen lassen!
> - Der Loader macht das alles schon.
> - `http://localhost` wird von Chrome blockiert („loopback address space"). Der Loader nutzt korrekt `https://localhost`.
> - Wenn ich Fehler sehe: in Webflow überall nach `localhost` suchen und alte `http://`-Tags löschen.

### Was in der Konsole OK ist
- `App local …` → mein Code läuft lokal ✅
- Gelbe Warnung „…vercel.app/app.js preloaded but not used…" → **harmlos** (verschwindet nach echtem Deploy)
- „message channel closed…" → kommt von einer **Browser-Extension**, nicht von meinem Code

---

## 6. Auf GitHub sichern

```bash
git init
git add .
git commit -m "Initial commit"

# Privates Repo erstellen + verbinden (Username: maxl3000!)
gh repo create maxl3000/mein-neues-projekt --private --source=. --remote=origin --push
```

> Falls die Remote schon existiert oder falsch ist:
> ```bash
> git remote set-url origin https://github.com/maxl3000/mein-neues-projekt.git
> git push -u origin main
> ```

---

## 7. (Optional) Auf Vercel deployen

Damit die Produktiv-URL (`VERCEL_URL`) echte Assets ausliefert und die Preload-Warnungen verschwinden:
```bash
vercel          # erstes Mal: Projekt verknüpfen, Fragen mit Enter bestätigen
# oder, wenn ein Deploy-Hook in .env steht:
bun run dep
```

---

## 🆘 Häufige Fehler & schnelle Lösung

| Fehler in der Konsole | Ursache | Lösung |
|---|---|---|
| `…/%7B…%7D/app.css` 404 | `{ }`-Klammern in `.env` oder Loader | Klammern entfernen (Schritt 2) |
| `ERR_CERT_AUTHORITY_INVALID` | CA nicht vertraut | `bun setup-ssl` + Browser neu starten (Schritt 3) |
| `blocked …loopback address space` | alter `http://localhost`-Tag in Webflow | alte Tags löschen, nur Loader behalten (Schritt 5) |
| `Repository not found` (push) | falscher Username `max13000` | richtig: `maxl3000` |
| pnpm: „global bin dir not in PATH" | PATH-Setup | `~/.zshrc` lädt `$PNPM_HOME` — neues Terminal öffnen |

---

## 📋 Spickzettel (die wichtigsten Befehle)

```bash
degit vallafederico/webflow-dev-setup   # Vorlage laden
bun install                              # Pakete installieren
cp .env.example .env                     # .env anlegen (dann Klammern entfernen!)
bun setup-ssl                            # HTTPS-Zertifikat (Mac-Passwort nötig)
bun dev                                  # Dev-Server: https://localhost:6545
# → Loader von localhost:6545 in Webflow Custom Code einfügen, Publish
git init && git add . && git commit -m "Initial commit"
gh repo create maxl3000/PROJEKT --private --source=. --remote=origin --push
```
