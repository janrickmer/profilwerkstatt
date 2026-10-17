# KI-Berufsberater einrichten (Cloudflare Workers AI)

Der KI-Berufsberater in Schritt 5c der Profilwerkstatt läuft über einen kleinen Dienst bei Cloudflare: einen
**Worker** (`ki-berater.js` in diesem Ordner), der die Nachrichten an das Sprachmodell **Mistral Small 3.1** in
**Cloudflare Workers AI** weitergibt. Ein API-Schlüssel ist nicht nötig; der Worker nutzt das Sprachmodell über eine
Bindung im eigenen Cloudflare-Konto.

## Inhalt

- [Was der Berater sendet und was nicht](#was-der-berater-sendet-und-was-nicht)
- [Zuerst: alten Berater abschalten](#zuerst-alten-berater-abschalten)
- [Kosten und Grenzen](#kosten-und-grenzen)
- [Einrichtung im Cloudflare-Dashboard](#einrichtung-im-cloudflare-dashboard)
- [Worker aktualisieren](#worker-aktualisieren)
- [Alternative: Einrichtung per Kommandozeile](#alternative-einrichtung-per-kommandozeile)
- [Bedingungen und Datenschutz](#bedingungen-und-datenschutz)
- [Häufige Probleme](#häufige-probleme)

## Was der Berater sendet und was nicht

- Erst wenn eine Schülerin oder ein Schüler das Kästchen „Ich bin mindestens 16 Jahre alt oder meine Eltern sind
  einverstanden …“ ankreuzt und eine Nachricht abschickt, sendet der Browser an den Worker: die Nachricht, die letzten
  Nachrichten des Gesprächs (höchstens 16) und ein Kurzprofil (Schwerpunkt und Punkte im Interessencheck, vorgeschlagene Berufe, wichtigste
  Werte, Stärken laut Selbst- und Fremdeinschätzung, was die Person gern tut).
- **Nicht** gesendet werden Name, Klasse, die Anmerkungen der befragten Personen und die Angaben zum Praktikum. Der Worker
  übernimmt vom Profil nur die genannten Felder und ignoriert alles andere.
- Der Worker gibt ebenfalls höchstens die letzten 16 Nachrichten weiter und kürzt zu lange Texte. Er speichert nichts und schreibt
  keine Inhalte in Protokolle. Gegen Massenanfragen merkt er sich im Arbeitsspeicher, wann welche IP-Adresse zuletzt
  angefragt hat (höchstens 6 Nachrichten je Minute), und nimmt nur Anfragen von `https://profilwerkstatt.janrickmer.de`
  an.
- Der Systemtext im Worker legt fest, dass der Berater nur über Berufswahl, Praktikum und Bewerbung spricht, nicht nach
  persönlichen Daten fragt, nichts für Jugendliche Ungeeignetes schreibt und bei Krisen auf Vertrauenspersonen, die Nummer
  gegen Kummer (116 111), die TelefonSeelsorge (0800 111 0 111) und den Notruf 112 hinweist.
- Nach Angaben von Cloudflare werden Eingaben und Antworten bei Workers AI weder zum Training von KI-Modellen noch zur
  Verbesserung von Diensten genutzt ([Your Data and Workers AI](https://developers.cloudflare.com/workers-ai/platform/data-usage/)).

## Zuerst: alten Berater abschalten

Der frühere Berater lief über einen anderen Worker (`profilwerkstatt-ki`) mit der Gemini API von Google. Deren
Bedingungen erlauben keinen Einsatz in Angeboten, die von unter 18-Jährigen genutzt werden.

1. Im Cloudflare-Dashboard unter „Workers & Pages“ den Worker **`profilwerkstatt-ki`** öffnen, dann „Settings“ →
   ganz unten „Delete“. Damit ist der alte Berater sofort aus – auch für Browser, die noch die alte Fassung der Seite
   geladen haben.
2. In Google AI Studio (aistudio.google.com) unter „API keys“ den Schlüssel des alten Beraters löschen.

## Kosten und Grenzen

- Workers AI enthält im kostenlosen Tarif **10.000 „Neurons“ am Tag**; der Zähler beginnt um 00:00 Uhr UTC neu
  (in Deutschland 1 Uhr bzw. im Sommer 2 Uhr). Im kostenlosen Tarif entstehen keine Kosten: Ist das Tageskontingent
  aufgebraucht, antwortet der Berater bis zum nächsten Tag mit „Der KI-Berater ist für heute ausgelastet …“.
- Grob geschätzt kostet eine Nachricht je nach Länge des Gesprächs 50 bis 180 Neurons. Das reicht für etwa **60 bis
  180 Nachrichten am Tag**. Den Verbrauch zeigt das Dashboard unter „AI“ → „Workers AI“.
- Reicht das nicht, kann das Konto auf „Workers Paid“ umgestellt werden (Grundgebühr pro Monat; Neurons über dem
  Tageskontingent kosten dann zusätzlich). Die aktuellen Preise stehen unter
  [Workers AI – Pricing](https://developers.cloudflare.com/workers-ai/platform/pricing/).

## Einrichtung im Cloudflare-Dashboard

### Schritt 1: Anmelden

Melden Sie sich unter [dash.cloudflare.com](https://dash.cloudflare.com) an – am besten mit dem Konto, in dem auch der
alte Worker lag. Dann stimmt die Adresse in `index.html` schon (`https://profilwerkstatt-berater.feindt-cloudflare.workers.dev/`).

### Schritt 2: Worker anlegen

1. Wählen Sie im linken Menü „Workers & Pages“ (ggf. unter „Compute“).
2. Klicken Sie auf „Create“ (bzw. „Create application“) und wählen Sie „Worker“ bzw. „Start with Hello World!“.
3. Name: **`profilwerkstatt-berater`**. Aus dem Namen ergibt sich die Adresse,
   z. B. `https://profilwerkstatt-berater.feindt-cloudflare.workers.dev`.
4. Klicken Sie auf „Deploy“. Cloudflare legt zunächst ein Beispielprogramm an.
5. Klicken Sie auf „Edit code“ und löschen Sie den gesamten Text im Editor.
6. Öffnen Sie die Datei `worker/ki-berater.js` aus diesem Projekt (auf GitHub: Datei öffnen und über den Knopf
   „Copy raw file“ kopieren) und fügen Sie den **gesamten** Inhalt in den Editor ein.
7. Klicken Sie auf „Deploy“.

### Schritt 3: Workers AI verbinden

1. Öffnen Sie den Worker `profilwerkstatt-berater` und dort „Settings“ → „Bindings“ (bei manchen Konten ist
   „Bindings“ ein eigener Reiter).
2. Klicken Sie auf „Add binding“ (bzw. „Add“) und wählen Sie **„Workers AI“**.
3. Variablenname („Variable name“): **`AI`** – genau so, in Großbuchstaben.
4. Speichern bzw. „Deploy“.

### Schritt 4 (empfohlen): Protokolle ausschalten

Bei neuen Workern sind die **Workers Logs** standardmäßig eingeschaltet. Chat-Inhalte landen darin nicht – der
Worker schreibt nur bei Störungen die Fehlermeldung von Workers AI in das Protokoll, und der Inhalt einer Anfrage
wird von Cloudflare nicht mitgeschrieben. Die Logs halten aber zu jeder Anfrage Metadaten (Zeitpunkt, aufgerufene
Adresse, Status) bis zu sieben Tage fest. Datensparsamer ist es ohne:

1. Öffnen Sie den Worker und dort den Reiter „Settings“. Scrollen Sie bis zum Abschnitt **„Observability“** (er
   steht unterhalb von „Bindings“ und „Variables and Secrets“).
2. Stellen Sie **„Workers Logs“** auf „Disabled“ (Knopf „Edit“ bzw. Schalter). Falls dort auch „Traces“ oder
   „Issues“ angeboten werden: ebenfalls ausschalten. Logpush und „AI Gateway“ werden nicht gebraucht.

Finden Sie die Einstellung nicht, ist das kein Sicherheitsproblem – die Datenschutzerklärung nennt die Verarbeitung
von IP-Adresse und Zeitpunkt durch Cloudflare bereits. Alternativ schaltet `npx wrangler deploy` (siehe unten) die
Logs über die Datei `wrangler.toml` aus.

### Schritt 5: Testen

1. Rufen Sie die Adresse des Workers direkt im Browser auf. Erwartete Antwort:
   `{"error":"Diese Adresse ist nur für die Profilwerkstatt freigegeben."}` – das ist richtig, denn der Worker nimmt
   nur Anfragen von der Profilwerkstatt an.
2. Öffnen Sie die Profilwerkstatt, gehen Sie zu Schritt 5c, kreuzen Sie das Kästchen an und schicken Sie eine
   Nachricht, z. B. „Welche Berufe passen zu mir?“. Nach einigen Sekunden erscheint die Antwort.

### Schritt 6: Adresse prüfen

Steht im Dashboard eine andere Adresse als `https://profilwerkstatt-berater.feindt-cloudflare.workers.dev/` (anderes
Konto), tragen Sie sie in `index.html` bei `KI_ENDPOINT` ein. Für Tests von einer anderen Adresse aus (z. B.
`http://localhost:8080`) können Sie im Worker unter „Settings“ → „Variables and Secrets“ die Variable
`ALLOWED_ORIGINS` anlegen (mehrere Adressen durch Kommas getrennt).

## Worker aktualisieren

Ändert sich `worker/ki-berater.js`, öffnen Sie den Worker, klicken auf „Edit code“, ersetzen den gesamten Text durch den
neuen Inhalt und klicken auf „Deploy“. Die Bindung `AI` bleibt erhalten.

## Alternative: Einrichtung per Kommandozeile

Im Ordner `worker/`: `npx wrangler login`, dann `npx wrangler deploy`. Die Datei `wrangler.toml` legt Name, Bindung `AI`
und ausgeschaltete Protokolle fest.

## Bedingungen und Datenschutz

- Der Worker läuft unter den Bedingungen Ihres Cloudflare-Kontos (Self-Serve Subscription Agreement mit den
  Service-Specific Terms und der Acceptable Use Policy unter [cloudflare.com/terms](https://www.cloudflare.com/terms/)).
  Cloudflare verarbeitet die Daten dabei im Auftrag des Betreibers (Data Processing Addendum, Teil der Bedingungen).
  Anders als bei der Gemini API war bei der Prüfung keine Regel zu finden, die Angebote für unter 18-Jährige ausschließt;
  lesen Sie die aktuellen Bedingungen bitte trotzdem selbst, bevor Sie den Berater freischalten.
- Das Modell Mistral Small 3.1 steht unter der Apache-2.0-Lizenz.
- Wird die Profilwerkstatt im Unterricht eingesetzt, sollte die Schule (bzw. ihre Datenschutzbeauftragte) über den
  freiwilligen KI-Berater informiert sein.
- Die Datenschutzerklärung der Seite (`impressum.html`, Abschnitt 5) beschreibt den Berater; ändert sich der Anbieter
  oder das Modell, muss sie angepasst werden.

## Häufige Probleme

- **„Der KI-Berater ist gerade nicht erreichbar …“:** Der Worker ist nicht angelegt, die Adresse in `KI_ENDPOINT`
  stimmt nicht oder die Bindung `AI` fehlt (Schritt 3).
- **„Der KI-Berater ist für heute ausgelastet …“:** Das Tageskontingent von Workers AI ist aufgebraucht (siehe
  [Kosten und Grenzen](#kosten-und-grenzen)).
- **„Diese Adresse ist nur für die Profilwerkstatt freigegeben.“** im Chat: Die Seite läuft unter einer anderen
  Adresse als `https://profilwerkstatt.janrickmer.de`. Tragen Sie die Adresse in `ALLOWED_ORIGINS` ein (Schritt 6).
