# SPEC.md · Restaurant Bella Vista (K1) · Ergänzung Meeting 2

**Quelle:** Interview mit Inhaber Marco Ferretti, Meeting 2 (Nachrichten 13–20).
**Stand:** Änderungen gegenüber v1 (Meeting 1). Hier steht nur, was in Meeting 2 **neu** dazukam oder **präzisiert** wurde. v1 und diese Ergänzung zusammen ergeben die finale SPEC.md.
**Art:** Anforderungsmodell der Geschäftswelt. Keine Feature-Liste, keine UI-Skizze, keine Technikentscheidung. Felder, die Marco nicht selbst genannt hat, sind als *Modellierung* gekennzeichnet.

## Entitäten (Änderungen)

| Entität | Neu oder geändert |
|---|---|
| **Standort** | `restaurantschluss` jetzt bekannt: Kreuzberg 23:00, Spandau 22:00 · `regulaerer_kuechenschluss` 22:30 / 21:30 · **neu** `sonder_kuechenschluss` (optional, nur für einen Sonderabend) (Nachr. 14) |
| **Gast** | **neu** `ist_firmenkunde` (ja/nein). Privatgast und Firmenkunde schließen sich nicht aus (Nachr. 13) |
| **Reservierung** | **neu** `gruppenmenue_ausnahme` (ja/nein, nur durch Inhaber) (Nachr. 16) |
| **Gericht** | `preis` wird zu `restaurantpreis`, einheitlich an allen Standorten, die das Gericht anbieten (Nachr. 15) |
| **Bestellung** | `gast_id` entfällt, die Zuordnung zum Gast läuft jetzt pro Position (Nachr. 18, 19) |
| **Bestellposition** | **neu** `abrechnungsart` (À-la-carte / Gruppenmenü) (Nachr. 17) · `preis` wird zu `einzelpreis_zum_zeitpunkt` (*Modellierung*) |
| **Mitarbeiter** | `rolle` jetzt festgelegt: Inhaber / Manager / Bedienung · `standort_id` entfällt, ersetzt durch Einsatz (Nachr. 16) |

**Neue Verknüpfungen** (keine eigenständigen Geschäftsobjekte, aber mit eigenen Attributen):

| Verknüpfung | Attribute |
|---|---|
| **Gastzuordnung** (Gast ↔ Bestellposition) | `position_id → Bestellposition`, `gast_id → Gast` (oder `platz` bei unbekanntem Mitesser, *Modellierung*), `anteil` (manuell bei gemeinsamer Position, sonst 100 %) (Nachr. 18, 19) |
| **Einsatz** (Mitarbeiter ↔ Standort) | `mitarbeiter_id → Mitarbeiter`, `standort_id → Standort`, `datum`, `manager_vom_dienst` (ja/nein) (Nachr. 16) |

## Beziehungen (Änderungen)

- **Ersetzt v1-Beziehung 7 („Gast 1:n Bestellung“):** **Gast n:m Bestellposition** über Gastzuordnung. Jede Position wird beim Aufnehmen einem Gast zugewiesen, soweit möglich. Gemeinsame Positionen (z. B. eine Flasche Wein) teilt die Bedienung manuell auf. (Nachr. 18, 19)
- **Ersetzt v1-Beziehung 9 („Standort 1:n Mitarbeiter“):** **Mitarbeiter n:m Standort** über Einsatz. Mitarbeiter springen zwischen den Filialen ein (z. B. aus Kreuzberg nach Spandau), erfasst wird, wer an welchem Abend wo gearbeitet hat. (Nachr. 16)
- **Präzisiert, v1-Beziehung 4 (Standort n:m Gericht):** Ein Gericht, das an beiden Standorten angeboten wird, hat einen gemeinsamen Preis. (Nachr. 15)

## Business Rules (Präzisierungen)

- **BR1 · Gruppenmenü:** Die Ausnahme genehmigt **nur Marco**. Manager und Bedienung dürfen das nicht selbst entscheiden. (Nachr. 16)

- **BR2 · Reservierungen:** Reservierungen annehmen, Tische zuweisen, stornieren und umplanen darf der **Manager vom Dienst** (in Kreuzberg meist Sofia, in Spandau Thomas). (Nachr. 16)

- **BR3 · Bella-Card** (in v1 offen, jetzt verbindlich): 15 % gelten **nur auf die eigenen À-la-carte-Positionen** des Stammgastes. **Nie** auf Gruppenmenü oder Catering. Bei getrennter Zahlung bekommt kein anderer Gast den Rabatt. Bei gemeinsamen Positionen wird nur der Anteil des Stammgastes rabattiert. Andere Rabatte darf die Bedienung nicht geben, darüber entscheidet der Manager. (Nachr. 16–19)

- **BR4 · Preise:** Ein Gericht kostet an allen Standorten, die es anbieten, **denselben Preis**. Eine Preisänderung an einem gemeinsamen Gericht gilt automatisch für beide Standorte, bei einem Gericht nur aus Kreuzberg nur dort. **Nur Marco** darf Preise ändern. Catering hat eigene Preise. (Nachr. 15, 16)

- **BR5 · Küchenschluss:** Gesperrt wird **Kreuzberg ab 22:30** (Schluss 23:00), **Spandau ab 21:30** (Schluss 22:00). **Ausnahme:** Nur Marco darf für einen Sonderabend (z. B. Silvester, Valentinstag) einen späteren Küchenschluss festlegen, die Sperre verschiebt sich dann mit. Gelegentliche Kulanz des Kochs für Stammgäste ist keine offizielle Ausnahme. (Nachr. 14, 16)

## Widersprüche

- **W2 · „15 Prozent auf die gesamte Rechnung“ (Nachr. 6) vs. Gruppenmenü mit fixen Preisen (Nachr. 6). Jetzt aufgelöst.**
  Auf die Nachfrage bestätigt Marco den Widerspruch (Nachr. 17): Auf das Gruppenmenü gibt es keinen Bella-Card-Rabatt, die Bella-Card gilt nur À-la-carte, beim Catering gar nicht („drei verschiedene Welten“). Bei getrennter Zahlung war er zunächst unsicher (Nachr. 18) und legt dann fest: nur auf den eigenen Anteil (Nachr. 19).
  → **Modell:** `Bestellposition.abrechnungsart` und Gastzuordnung mit Anteil.

## Neue Anforderung · Catering

- **Herkunft:** Auf die Zukunftsfrage in Meeting 1 angekündigt (Nachr. 11), in Meeting 2 erhoben (Nachr. 13).
- **Ablauf:** Catering für Firmenevents, z. B. TechCorp-Weihnachtsfeier für 60 Personen, geliefert ins Büro des Kunden. Marco schreibt ein Angebot (Buffet, Preis pro Person, Lieferdatum). Ein Auftrag durchläuft **angefragt → bestätigt → geliefert → bezahlt**. (Nachr. 13)
- **Preislogik:** **Ab 30 Personen Pauschalpreis pro Person**, darunter individuelle Kalkulation. Konkrete Preise wurden nicht genannt. (Nachr. 13)
- **Kunden:** Eine gemeinsame Liste für alle Gäste, **Firmenkunden werden markiert** (`Gast.ist_firmenkunde`). Niemand soll doppelt erfasst werden (z. B. der TechCorp-Chef, der auch privat essen kommt). (Nachr. 13)
- **Umfang:** **Nicht in V1.** Bis dahin bleibt Excel. Eigene Objekte wie `CateringAuftrag` und `Angebot/Rechnung` werden später spezifiziert. (Nachr. 20)

## Prioritäten (von Marco bestätigt)

1. **V1:** Reservierungen für beide Standorte, Tischzuweisung, Telefon und Walk-in. Das war bisher der größte Schmerz. (Nachr. 20)
2. **V1:** Bestellungen digital aufnehmen, mit Speisekarte je Standort. (Nachr. 20)
3. **Nach V1:** Stammgäste über die Telefonnummer erkennen, Bella-Card automatisch („mir persönlich wichtig, aber der Betrieb läuft auch ohne“), Mitarbeiterrechte. (Nachr. 20)
4. **Später:** Catering, „Excel reicht noch ein paar Monate“. (Nachr. 20)

## Offene Fragen

**In Meeting 2 beantwortet (aus v1):**
- Catering-Ablauf, Preise, Kunden → beantwortet (Nachr. 13)
- Bella-Card auf Gruppenmenü und bei getrennter Zahlung (W2) → beantwortet (Nachr. 17–19)
- Schließzeiten und Sonderabende → beantwortet (Nachr. 14)
- Preise an beiden Standorten → beantwortet (Nachr. 15)
- Rollen, Rechte, Standortwechsel → beantwortet (Nachr. 16)

**Weiterhin offen:**
- Welche **Kapazität** hat jeder Tisch, und wie werden **zusammengeschobene Tische** einer Reservierung zugeordnet? (Nachr. 4)
- Zählen die **10 Besuche** insgesamt oder pro Jahr? Ist jeder Bella-Card-Gast auch „Stammgast“ für den Wartezeit-Hinweis? (Nachr. 5, 6, 9)
- Wie wird **Trinkgeld** erfasst und auf die Mitarbeiter eines Abends aufgeteilt? (Nachr. 12, 16, 18)
- Was gehört zum **Gruppenmenü** (Inhalt, Preis)? (Nachr. 2, 17)
- Welche **Öffnungszeiten am Tagesbeginn**, Ruhetage und Feiertagsregeln gelten? (Nachr. 14)
- Welche **Zahlungsarten** und **Bestellstatus** werden gebraucht? (Nachr. 5, 18)
- Wie werden **Sondereinstellungen** wie ein verlängerter Küchenschluss festgehalten? (Nachr. 14, 16)
- Welche Angaben braucht das spätere **Catering** für Firmenkunden, Angebot und Rechnung? (Nachr. 13)
