# SPEC.md · Restaurant Bella Vista (K1)

**Quelle:** Interview mit Inhaber Marco Ferretti, Meeting 1 (Nachrichten 1–12) und Meeting 2 (Nachrichten 13–20).
**Stand:** 07.11.2026 · überarbeitete Fassung
**Art:** Anforderungsmodell der Geschäftswelt. Keine Feature-Liste, keine UI-Skizze, keine Technikentscheidung. Technische IDs und Verknüpfungsfelder sind als *Modellierung* gekennzeichnet, soweit Marco sie nicht selbst genannt hat.

## Entitäten

| Entität | Attribute |
|---|---|
| **Standort** | `standort_id`, `name` (Kreuzberg / Spandau), `restaurantschluss` (23:00 / 22:00), `regulaerer_kuechenschluss` (22:30 / 21:30), `sonder_kuechenschluss` (optional, nur für einen Sonderabend) |
| **Tisch** | `tisch_id`, `standort_id → Standort`, `tischnummer`, `kapazitaet` (Wert offen), `status` (frei / reserviert / besetzt; zeitfensterbezogen aus Reservierungen und tatsächlicher Belegung ermittelt) |
| **Gast** | `gast_id`, `name`, `telefonnummer` (eindeutig, zur Wiedererkennung), `besuchsanzahl`, `bella_card_berechtigt` (ab 10 Besuchen), `ist_firmenkunde` (ja/nein; ergänzende Kennzeichnung für späteres Catering, Privatgast und Firmenkunde schließen einander nicht aus) |
| **Reservierung** | `reservierung_id`, `gast_id → Gast` (optional bei Walk-in), `tisch_id → Tisch`, `beginn`, `geplantes_ende` (beginn + 2 h), `tatsaechlich_freigegeben_am` (optional, nach Bestätigung durch Personal), `personenanzahl`, `quelle` (Telefon / Walk-in), `status` (reserviert / bestätigt / storniert / no-show), `ankunft` (optional), `gruppenmenue_ausnahme` (ja/nein, nur durch Inhaber) |
| **Gericht** | `gericht_id`, `name`, `kategorie` (u. a. Pasta, Vorspeise, Grill, Dessert, Getränk), `restaurantpreis` |
| **Bestellung** | `bestellung_id`, `tisch_id → Tisch`, `aufgenommen_von → Mitarbeiter`, `aufgenommen_am`, `status` (Werte offen) |
| **Bestellposition** | `position_id`, `bestellung_id → Bestellung`, `gericht_id → Gericht`, `menge`, `einzelpreis_zum_zeitpunkt`, `abrechnungsart` (À-la-carte / Gruppenmenü) |
| **Mitarbeiter** | `mitarbeiter_id`, `name`, `rolle` (Inhaber / Manager / Bedienung) |

**Verknüpfungen der n:m-Beziehungen** (keine eigenständigen Geschäftsobjekte, aber mit eigenen Attributen):

| Verknüpfung | Attribute |
|---|---|
| **Standortangebot** (Standort ↔ Gericht) | `standort_id → Standort`, `gericht_id → Gericht` |
| **Gastzuordnung** (Gast ↔ Bestellposition) | `position_id → Bestellposition`, `gast_id → Gast` (oder `platz` bei unbekanntem Mitesser), `anteil` (manuell bei gemeinsamer Position, sonst 100 %) |
| **Einsatz** (Mitarbeiter ↔ Standort) | `mitarbeiter_id → Mitarbeiter`, `standort_id → Standort`, `datum`, `manager_vom_dienst` (ja/nein) |

**Modellierungsgrenzen:** Es gibt genau zwei Standorte, weitere sind nicht geplant (Nachr. 11). Eine eigene Rechnungs- oder Zahlungsentität ist nicht bestätigt. Getrennte Zahlung wird über die Gastzuordnung der Bestellpositionen abgebildet (Beziehung 7). Bei einem Walk-in darf ein namentlich bekannter Gast fehlen. Catering-Objekte gehören nicht zu diesem Modell (siehe „Neue Anforderung“). (Nachr. 1, 13, 18–20)

## Beziehungen

1. **Standort 1:n Tisch.** Gleiche Tischnummern an verschiedenen Standorten sind verschiedene Tische („Tisch 4 in Kreuzberg und Tisch 4 in Spandau“). (Nachr. 3)
2. **Tisch 1:n Reservierung** über die Zeit. Eine Reservierung bekommt zunächst einen Tisch. Wie zusammengeschobene Tische abgebildet werden, ist offen. (Nachr. 1, 3, 4)
3. **Gast 1:n Reservierung.** Ein bekannter Gast reserviert mehrfach. Walk-ins können ohne Gastdatensatz erfasst werden. (Nachr. 1, 9, 20)
4. **Standort n:m Gericht** über das Standortangebot. Pasta, Vorspeisen, Desserts und Getränke gibt es an beiden Standorten, Spandau hat insgesamt weniger Gerichte und keine Grillgerichte. (Nachr. 5, 7, 15)
5. **Tisch 1:n Bestellung** über die Zeit. (Nachr. 5, 18)
6. **Bestellung n:m Gericht** über Bestellposition: pro Position ein Gericht, eine Menge, ein Preis. (Nachr. 5)
7. **Gast n:m Bestellposition** über Gastzuordnung, je Zuordnung mit dem manuell festgelegten Anteil. Jede Position wird beim Aufnehmen einem Gast zugewiesen, soweit möglich. Gemeinsame Positionen (z. B. eine Flasche Wein) werden auf mehrere Gäste aufgeteilt. Nicht namentlich bekannte Mitesser werden vorläufig über einen Platzbezeichner unterschieden (*Modellierung*). (Nachr. 18, 19)
8. **Mitarbeiter 1:n Bestellung.** Die Bedienung, die die Bestellung aufnimmt. (Nachr. 12, 16)
9. **Mitarbeiter n:m Standort** über Einsatz (Standort, Datum). Mitarbeiter springen zwischen den Filialen ein, erfasst wird, wer an welchem Abend wo gearbeitet hat. (Nachr. 12, 16)

## Business Rules

- **BR1 · Gruppenmenü ab 8 Personen:** Ab **8 Personen** ist das Gruppenmenü die **verbindliche Standardregel** statt À-la-carte („Acht oder mehr, das ist Gruppenmenü, fertig“). Die App zeigt bei der Reservierung einen deutlichen Hinweis, damit das Personal den Gast darüber informiert. **Ausnahme:** Nur Marco (Inhaber) darf für eine solche Gruppe À-la-carte genehmigen (`gruppenmenue_ausnahme`); Manager und Bedienung dürfen dies nicht eigenständig entscheiden. Ob zusätzlich eine technische Sperre für À-la-carte erforderlich ist, hat Marco nicht abschließend festgelegt. (Nachr. 1, 2, 16)

- **BR2 · Tischbelegung und Freigabe:** Jede Reservierung wird mit **2 Stunden** geplanter Belegung ab Beginn kalkuliert. Der Tisch bleibt **besetzt, bis das Personal die Freigabe manuell bestätigt** (Gäste gegangen, Tisch abgeräumt), auch wenn die Planzeit abgelaufen ist. Erscheint ein Gast ohne Nachricht nicht, wird der Tisch nach **30 Minuten** freigegeben. **Ausnahme:** Bei Stammgästen zeigt das System einen Hinweis, über längeres Freihalten entscheidet ein Mensch. Eine Stornierung gibt den Tisch sofort wieder für den Abend frei. Reservierungen annehmen, Tische zuweisen, stornieren und umplanen darf der Manager vom Dienst. (Nachr. 4, 8–10, 16)

- **BR3 · Bella-Card-Rabatt:** Ab **10 Besuchen** erhält ein über seine Telefonnummer erkannter Gast **15 % Rabatt**, **nur auf seine eigenen À-la-carte-Positionen**. Bei getrennter Zahlung bekommt kein anderer Gast am Tisch diesen Rabatt. Gemeinsam bestellte Positionen teilt die Bedienung manuell auf, rabattiert wird nur der Anteil des Stammgastes. **Nie** auf Gruppenmenü oder Catering. Andere Rabatte darf die Bedienung nicht geben, darüber entscheidet der Manager. (Nachr. 5, 6, 16–19)

- **BR4 · Speisekarte und Preise je Standort:** Bestellt werden darf nur, was am jeweiligen Standort angeboten wird. **Grillgerichte gibt es nicht in Spandau** (kein Grill). Ein Gericht kostet an allen Standorten, die es anbieten, **denselben Restaurantpreis**. Eine Preisänderung an einem gemeinsamen Gericht gilt automatisch für beide Standorte, bei einem Gericht nur aus Kreuzberg nur dort. **Nur Marco** darf Preise ändern. Catering hat eine eigene Preislogik. (Nachr. 5, 7, 15, 16)

- **BR5 · Küchenschluss:** Neue Küchenbestellungen sind **30 Minuten vor Restaurantschluss** automatisch gesperrt: **Kreuzberg ab 22:30** (Schluss 23:00), **Spandau ab 21:30** (Schluss 22:00). **Ausnahme:** Nur Marco darf für einen Sonderabend (z. B. Silvester, Valentinstag) einen späteren Küchenschluss festlegen, die Sperre verschiebt sich dann mit. Gelegentliche Kulanz des Kochs für Stammgäste ist keine offizielle Ausnahme. (Nachr. 12, 14, 16)

## Widersprüche

- **W1 · „Nach zwei Stunden gilt der Tisch als frei, auch wenn die Leute noch sitzen“ (Nachr. 4) vs. Geburtstagsgruppe sitzt fast drei Stunden, nächste Gruppe wartet (Nachr. 4).**
  **Aufgelöst (Nachr. 10):** Die 2 Stunden sind nur die Planung. Frei ist der Tisch erst, wenn die Bedienung die Freigabe bestätigt.
  → **Modell:** `Reservierung.geplantes_ende` und `Reservierung.tatsaechlich_freigegeben_am` getrennt; der Tisch bleibt für die tatsächliche Belegung als besetzt markiert, bis das Personal die Freigabe bestätigt. Dadurch können mehrere Reservierungen desselben Tischs im Zeitverlauf korrekt dargestellt werden.

- **W2 · „15 Prozent auf die gesamte Rechnung“ (Nachr. 6) vs. Gruppenmenü mit fixen Preisen und enger Marge (Nachr. 6).**
  **Aufgelöst (Nachr. 17, 19):** Marco bestätigt den Widerspruch. 15 % gelten nur auf eigene À-la-carte-Positionen, nie auf Gruppenmenü oder Catering („drei verschiedene Welten“). Bei getrennter Zahlung nur auf den Anteil des Stammgastes.
  → **Modell:** `Bestellposition.abrechnungsart` und Gastzuordnung mit Anteil (Beziehung 7).

## Neue Anforderung aus Meeting 2 · Catering

- **Herkunft:** Auf die Zukunftsfrage erstmals erwähnt (Nachr. 11), Ablauf in Meeting 2 erhoben (Nachr. 13).
- **Ablauf:** Catering für Firmenevents, z. B. TechCorp-Weihnachtsfeier für 60 Personen, geliefert ins Büro des Kunden. Marco schreibt ein Angebot (Buffet, Preis pro Person, Lieferdatum). Ein Auftrag durchläuft **angefragt → bestätigt → geliefert → bezahlt**. (Nachr. 11, 13)
- **Preislogik:** **Ab 30 Personen Pauschalpreis pro Person**, darunter individuelle Kalkulation. Konkrete Preise wurden nicht genannt. (Nachr. 13)
- **Kunden:** Eine gemeinsame Liste für alle Gäste, **Firmenkunden werden zusätzlich gekennzeichnet** (`Gast.ist_firmenkunde`), ohne ihre Nutzung als Privatgast auszuschließen. Niemand soll doppelt erfasst werden (z. B. der TechCorp-Chef, der auch privat essen kommt). Die spätere Zuordnung von Kontaktpersonen zu Unternehmen ist noch offen (*Modellierung*). (Nachr. 13)
- **Umfang:** **Nicht in V1.** Bis dahin bleibt Excel. Eigene Objekte wie `CateringAuftrag` und `Angebot/Rechnung` werden später spezifiziert. (Nachr. 11, 20)

## Prioritäten

1. **V1:** Reservierungen für Kreuzberg und Spandau in einem System, Telefon und Walk-in, Tischzuweisung, geplante und tatsächliche Verfügbarkeit (BR1, BR2). (Nachr. 3, 10, 20)
2. **V1:** Bestellungen digital aufnehmen und an die Küche geben, mit Speisekarte je Standort, einheitlichen Preisen und Küchenschluss (BR4, BR5). (Nachr. 5, 7, 12, 15, 20)
3. **Nach V1:** Stammgäste per Telefonnummer erkennen, Bella-Card automatisch, getrennte Rechnung (BR3). Mitarbeiterrechte und Erfassung, wer wann wo gearbeitet hat. Die Rollenentscheidungen gelten fachlich schon jetzt. (Nachr. 16–20)
4. **Später:** Catering mit Angebot, Auftragsphasen und Abrechnung. (Nachr. 11, 13, 20)

## Offene Fragen

- Welche **Kapazität** hat jeder Tisch, und wie werden **zusammengeschobene Tische** einer Reservierung zugeordnet? (Nachr. 4)
- Soll bei Reservierungen ab acht Personen zusätzlich zum verbindlichen Gruppenmenü-Hinweis eine **technische À-la-carte-Sperre** gelten, oder reicht die dokumentierte Entscheidung von Marco? (Nachr. 2, 16)
- Zählen die **10 Besuche** für die Bella-Card insgesamt oder pro Jahr („zwölfter Besuch dieses Jahr“)? Ist jeder Bella-Card-Gast auch „Stammgast“ für den Wartezeit-Hinweis? (Nachr. 5, 6, 9)
- Wie werden **unbekannte Mitesser** bei getrennter Zahlung zugeordnet? (Nachr. 18, 19)
- Wie wird **Trinkgeld** erfasst und auf die Mitarbeiter eines Abends aufgeteilt? (Nachr. 12, 16, 18)
- Was gehört zum **Gruppenmenü** (Inhalt, Preis), und wird es als Gericht oder eigenes Angebot geführt? (Nachr. 2, 17)
- Welche **Öffnungszeiten am Tagesbeginn**, Ruhetage und Feiertagsregeln gelten? Bestätigt sind nur die Schließzeiten und Sonderabende. (Nachr. 14)
- Welche **Zahlungsarten** und **Bestellstatus** werden gebraucht? (Nachr. 5, 18)
- Soll es künftig eine **Anzahlung** bei Reservierungen geben? Heute nicht, Marco hält es für möglich. (Nachr. 8)
- Wie werden **Sondereinstellungen** wie ein verlängerter Küchenschluss festgehalten (wer, wann, wie lange)? (Nachr. 14, 16)
- Welche Angaben braucht das spätere **Catering** für Firmenkunden, Angebot und Rechnung? (Nachr. 13)
