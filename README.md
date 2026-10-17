# Profilwerkstatt – Berufsorientierung Klasse 11

Eine Website zur Berufsorientierung vor dem Betriebspraktikum im Januar. Schülerinnen und Schüler reflektieren in fünf Schritten ihre **Vorlieben**, ihr **Selbstbild**, ihr **Fremdbild** und ihre **Interessen** (RIASEC-Modell nach John L. Holland) und leiten daraus **passende Berufe** für die engere Wahl und das Praktikum ab. Am Ende steht ein PDF-Export des persönlichen Profils.

## Eigenschaften

- Eine einzige Datei (`index.html`), komplett eigenständig – keine externen Fonts oder Skripte; nur der freiwillige
  KI-Berater nutzt einen Dienst (siehe unten)
- Alle Eingaben bleiben lokal im Browser des jeweiligen Geräts (automatisches Zwischenspeichern, Lösch-Knopf für Schulrechner)
- PDF-Export des Profils sowie ein druck- und verschickbarer Fremdwahrnehmungs-Fragebogen als PDF
- Freiwilliger KI-Berufsberater (Schritt 5c) über einen Cloudflare Worker mit Workers AI (Sprachmodell Mistral Small 3.1):
  Er erhält nur die Nachrichten und ein Kurzprofil ohne Namen, erst nach einem Einverständnis-Kästchen (ab 16 Jahren oder
  mit Einverständnis der Eltern). Quelltext und Einrichtung: `worker/` und **worker/ANLEITUNG.md**. Der frühere Berater
  mit der Gemini API ist abgeschaltet, weil deren Bedingungen keinen Einsatz bei unter 18-Jährigen erlauben.
- Verifizierte Links zu Check-U und BERUFENET der Bundesagentur für Arbeit

## Installation

Schritt-für-Schritt-Anleitung (GitHub Pages, Strato-Subdomain): siehe **INSTALLATION.md**. Den KI-Berater richten Sie
nach **worker/ANLEITUNG.md** ein.

## Lizenzen eingebetteter Komponenten

- Schrift **Carlito** (metrisch kompatibel zu Calibri) – SIL Open Font License 1.1
- **jsPDF** – MIT-Lizenz (Lizenzhinweis im eingebetteten Code enthalten)

---

made by Jan-Rickmer Feindt
