# SPEC.md · Restaurant Bella Vista (K1) · v1

**Quelle:** Interview mit Inhaber Marco Ferretti, Meeting 1 (Nachrichten 1–12).
**Stand:** Zwischenstand nach Meeting 1. Punkte, die noch nicht geklärt sind, stehen unter „Offene Fragen“ und werden in Meeting 2 nachgefragt.
**Art:** Anforderungsmodell der Geschäftswelt. Keine Feature-Liste, keine UI-Skizze, keine Technikentscheidung. Technische IDs und Verknüpfungsfelder sind als *Modellierung* gekennzeichnet, soweit Marco sie nicht selbst genannt hat.

## Entitäten

| Entität | Attribute |
|---|---|
| **Standort** | `standort_id`, `name` (Kreuzberg / Spandau), `restaurantschluss` (Werte offen), `kuechenschluss` (= Restaurantschluss − 30 min) |
| **Tisch** | `tisch_id`, `standort_id → Standort`, `tischnummer`, `kapazitaet` (Werte offen; z. B. Fenstertisch für Paare, runder Tisch für Familien), `status` (frei / reserviert / besetzt) |
| **Gast** | `gast_id`, `name`, `telefonnummer` (zur Wiedererkennung), `besuchsanzahl`, `bella_card_berechtigt` (ab 10 Besuchen) |
| **Reservierung** | `reservierung_id`, `gast_id → Gast` (optional bei Walk-in), `tisch_id → Tisch`, `beginn` (Datum, Uhrzeit), `geplantes_ende` (beginn + 2 h), `tatsaechlich_freigegeben_am` (optional, nach Bestätigung durch Personal), `personenanzahl`, `quelle` (Telefon / Walk-in), `status` (reserviert / bestätigt / storniert / no-show) |
| **Gericht** | `gericht_id`, `name`, `kategorie` (u. a. Pasta, Vorspeise, Grill, Dessert, Getränk), `preis` |
| **Bestellung** | `bestellung_id`, `tisch_id → Tisch`, `gast_id → Gast` (optional, für Bella-Card), `aufgenommen_von → Mitarbeiter`, `aufgenommen_am`, `status` (Werte offen) |
| **Bestellposition** | `position_id`, `bestellung_id → Bestellung`, `gericht_id → Gericht`, `menge`, `preis` |
| **Mitarbeiter** | `mitarbeiter_id`, `name`, `rolle` (u. a. Inhaber, Bedienung, Küche; weitere offen), `standort_id → Standort` |

**Modellierungsgrenzen:** Eine eigene Rechnungsentität ist noch nicht bestätigt, die Abrechnung ergibt sich zunächst aus den Bestellpositionen. Bei einem Walk-in darf ein namentlich bekannter Gast fehlen. Catering ist nur angekündigt und noch nicht modelliert. (Nachr. 1, 5, 11)

## Beziehungen

1. **Standort 1:n Tisch.** Gleiche Tischnummern an verschiedenen Standorten sind verschiedene Tische („Tisch 4 in Kreuzberg und Tisch 4 in Spandau“). (Nachr. 3)
2. **Tisch 1:n Reservierung** über die Zeit. Eine Reservierung bekommt einen Tisch. Wie zusammengeschobene Tische abgebildet werden, ist offen. (Nachr. 1, 3, 4)
3. **Gast 1:n Reservierung.** Stammgäste reservieren wiederholt. Walk-ins kommen ohne vorherigen Anruf und ohne Gastdatensatz. (Nachr. 1, 6, 9)
4. **Standort n:m Gericht** über das Standortangebot. Pasta, Vorspeisen, Desserts und Getränke gibt es an beiden Standorten, Spandau hat insgesamt weniger Gerichte und keine Grillgerichte. (Nachr. 5, 7)
5. **Tisch 1:n Bestellung** über die Zeit. (Nachr. 5)
6. **Bestellung n:m Gericht** über Bestellposition: pro Position ein Gericht, eine Menge, ein Preis. (Nachr. 5)
7. **Gast 1:n Bestellung.** Wird ein Stammgast über die Telefonnummer erkannt, wird seine Bestellung ihm zugeordnet, damit die Bella-Card greift (*Modellierung*). (Nachr. 5, 6)
8. **Mitarbeiter 1:n Bestellung.** Wer welche Bestellung aufgenommen hat. (Nachr. 12)
9. **Standort 1:n Mitarbeiter.** Wer an welchem Standort arbeitet. Ob Mitarbeiter zwischen den Standorten wechseln, ist offen. (Nachr. 12)

## Business Rules

- **BR1 · Gruppenmenü ab 8 Personen:** Ab **8 Personen** gilt das Gruppenmenü statt À-la-carte („Acht oder mehr, das ist Gruppenmenü, fertig“). Das muss dem Gast schon bei der telefonischen Reservierung gesagt werden, das System zeigt dafür einen Hinweis. **Ausnahme:** Marco entscheidet persönlich, etwa bei langjährigen Stammgästen, ob eine Gruppe doch À-la-carte bestellen darf. (Nachr. 1, 2)

- **BR2 · Tischbelegung und Freigabe:** Jede Reservierung wird mit **2 Stunden** geplanter Belegung ab Beginn kalkuliert. Der Tisch bleibt **besetzt, bis das Personal die Freigabe manuell bestätigt** (Gäste gegangen, Tisch abgeräumt). Erscheint ein Gast ohne Nachricht nicht, wird der Tisch nach **30 Minuten** freigegeben. **Ausnahme:** Bei Stammgästen zeigt das System einen Hinweis, über längeres Freihalten entscheidet ein Mensch. Eine Stornierung gibt den Tisch sofort wieder für den Abend frei. Eine Anzahlung gibt es nicht. (Nachr. 4, 8–10)

- **BR3 · Bella-Card:** Ab **10 Besuchen** bekommt ein Gast die Bella-Card. Er wird über seine **Telefonnummer** erkannt („Name kann man vergessen, Telefonnummer nicht“). Der Rabatt beträgt **15 %**. Worauf genau er gilt, ist noch nicht geklärt (siehe W2). (Nachr. 5, 6)

- **BR4 · Speisekarte je Standort:** Bestellt werden darf nur, was am jeweiligen Standort angeboten wird. **Grillgerichte (z. B. Bistecca, Salsiccia, Mixed Grill) gibt es nicht in Spandau**, weil es dort baulich keinen Grill gibt. Grillgerichte werden bei der Bestellung in Spandau gar nicht erst angeboten. (Nachr. 5, 7)

- **BR5 · Küchenschluss:** **30 Minuten vor Restaurantschluss** nimmt die Küche keine neuen Bestellungen mehr an. Das System sperrt neue Bestellungen ab diesem Zeitpunkt automatisch. Die Schließzeiten je Standort und mögliche Ausnahmen sind noch offen. (Nachr. 12)

## Widersprüche

- **W1 · „Nach zwei Stunden gilt der Tisch als frei, auch wenn die Leute noch sitzen“ (Nachr. 4) vs. Geburtstagsgruppe sitzt fast drei Stunden, nächste Gruppe wartet (Nachr. 4).**
  **Aufgelöst (Nachr. 10):** Die 2 Stunden sind nur die Planung. Frei ist der Tisch erst, wenn die Bedienung die Freigabe bestätigt.
  → **Modell:** `Reservierung.geplantes_ende` und `Reservierung.tatsaechlich_freigegeben_am` getrennt, der Tisch bleibt besetzt bis zur Bestätigung.

- **W2 · „15 Prozent auf die gesamte Rechnung“ bzw. „Rabatt auf alles“ (Nachr. 5, 6) vs. Gruppenmenü mit fixen Preisen und enger Marge (Nachr. 6).**
  **Noch offen:** Marco hat selbst gesagt, dass er das nie geregelt hat und es vor der Umsetzung festlegen muss. Wird in Meeting 2 angesprochen.
  → **Modell (vorläufig):** Bestellpositionen müssen erkennen lassen, ob sie À-la-carte oder Gruppenmenü sind.

## Prioritäten

1. **V1:** Reservierungen für Kreuzberg und Spandau in einem System, Telefon und Walk-in, Tischzuweisung, geplante und tatsächliche Verfügbarkeit (BR1, BR2). (Nachr. 1, 3, 10, 11)
2. **V1:** Bestellungen digital aufnehmen und an die Küche geben, mit Speisekarte je Standort und Küchenschluss (BR4, BR5). (Nachr. 5, 7, 11, 12)
3. **Danach:** Stammgäste über die Telefonnummer erkennen, Bella-Card automatisch abziehen (BR3). (Nachr. 5, 6)
4. **Weniger dringend:** Mitarbeiter, Standortzuordnung und Trinkgeld. (Nachr. 12)
5. **Später:** Catering für Firmenevents. (Nachr. 11)

## Offene Fragen (für Meeting 2)

- **Catering** (neu angekündigt): Wie läuft ein Auftrag ab, von der Anfrage über das Angebot bis zur Bezahlung? Welche Preise gelten, und sind Catering-Kunden auch Restaurantgäste? (Nachr. 11)
- **W2:** Gilt der Bella-Card-Rabatt auch auf das Gruppenmenü? Und was gilt, wenn am Tisch getrennt bezahlt wird? (Nachr. 6)
- Wann schließen **Kreuzberg und Spandau**, und gibt es Abende mit anderem Küchenschluss? (Nachr. 12)
- Kosten gemeinsame Gerichte an **beiden Standorten gleich viel**, und wie werden Preisänderungen gepflegt? (Nachr. 5, 7)
- **Mitarbeiter:** Welche Rollen gibt es, wer darf was entscheiden (Rabatte, Ausnahmen, Preise), arbeiten Mitarbeiter auch am anderen Standort, und wie wird Trinkgeld aufgeteilt? (Nachr. 2, 12)
- Welche **Kapazität** hat jeder Tisch, und wie werden **zusammengeschobene Tische** einer Reservierung zugeordnet? (Nachr. 4)
- Zählen die **10 Besuche** insgesamt oder pro Jahr („zwölfter Besuch dieses Jahr“)? (Nachr. 5, 6)
- Was gehört zum **Gruppenmenü** (Inhalt, Preis)? (Nachr. 1, 2)
- Welche **Bestellstatus** und Zahlungsarten werden gebraucht? (Nachr. 5)
