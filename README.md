# Algorithmus für den Split der Aufträge

## Definition

- Auftrag hat noch zu liefernde Positionen P
- Auftrag kann durch den Dealer nicht selber erfüllt werden (Split notwendig)
- es gibt eine Mindestbetrag für den Split U in €
- jede Position hat eine Menge M
- jede Position hat einen Mittleren-Einkaufs-Preis MEK in €
- jeder Supplier hat einen Zuschlag für Versandkosten V in €
- es gibt eine Abbruchbedingung für die Suche S (0,0 - 1,0)
- Jeder Supplier hat einen Faktor für die Auslastung A (0 - 1)

## Ziel

Ein Auftrag soll mit so wenigen wie möglichen Splits erfüllt werden (implizit: Splits sollten die größtmögliche Menger Y' einer Position X besitzen).

## Berechnung

### Quell-Auftrag

``` csharp
var x = 0;
for(var i = 0; i < P; i++) {
    x = x + M[i] * MEK[i];
}
```

Beispiel:

| P   | M   | MEK    |
| --- | --- | ---:   |
| 1   | 1x  | 25,00€ |
| 2   | 3x  | 5,00€  |
| 3   | 4x  | 1,25€  |

``` math
x = (1 x 25,00€) + (3 x 5,00€) + (4 x 1,25€) = 45€
```

### Split-Score

- MSup Lieferbare Menge des Suppliers
- A = Auslastung des Suppliers

``` csharp
var tmp = 0;
for(var i = 0; i < P; i++) {
    tmp = tmp + MSup[i] * MEK[i];
}
y = ((tmp - V) / x) * (1 - A);
```

Beispiel:

|P|M1|M2|MEK
|---|---|---|---:
|1|1x|0x|25,00€
|2|1x|3x|5,00€
|3|1x|4x|1,25€

``` math
V1 = 4,30€
V2 = 1,50€
A1 = 0,8
A2 = 0,2
tmp1 = ((1 x 25,00€) + (1 x 5,00€) + (1 x 1,25€) - 4,30€) / 45,00€ = 0,598
tmp2 = ((3 x 5,00€)) + (4 x 1,25€) - 1,50€) / 45,00€ = 0,411
y1 = 0,598 * (1 - 0,8) = 0,119
y2 = 0,444 * (1 - 0,2) = 0,328
```

## Suche des besten Splits

1. Es werden alle Supplier der Priorität nach gruppiert
2. Jede Prio-Gruppe wird gemischt so das die Supplier zufällig gezogen werden können
3. Für jede Prio-Gruppe wird folgende Schleife durchlaufen:
    1. Der Supplier wird aus der Prio-Gruppe gezogen
    2. Für den gezogen Supplier wird die Split-Score berechnet
    3. Die Split-Score wird mit der Abbruchbedingung vergleichen
        1. Split-Score größer gleich Abbruchbedingung => zwischengespeicherten Supplier ersetzen und Schleife verlassen
    4. Split-Score mit dem zwischengespeicherten Supplier vergleichen
        1. Split-Score größer als der gespeicherte Supplier => zwischengespeicherten Supplier ersetzen
4. Split mit zwischengespeichertem Supplier erstellen
5. Falls Auftrag noch nicht komplett erfüllbar Suche ohne den verwendeten Supplier wiederholen aber mit der gleichen Prio-Gruppe
6. Falls Auftrag noch nicht komplett erfüllbar Suche mit nächster Prio-Gruppe fortführen
7. Falls Auftrag immer noch nicht komplett erfüllbar suche Abbrechen => alle möglichen Splits wurden erstellt und die Restmenge bleibt beim Dealer zur weiteren Verarbeitung.

# Use-Cases

## allgemeine Ausgangssituation

Dealer:

* eleven teamsports GmbH
  * Eigener Bestand wird bevorzugt
  * Kann jegliche Veredelung vornehmen
  * Deutschland

Supplier:

* Vereinsexpress GmbH, Wemding
  * Alias: *VE*
  * Kann Laser und Stick Veredelung vornehmen
  * Prio: 1
  * Deutschland
* eleven teamsports stores GmbH, Berlin
  * Alias: *BER*
  * Kann keine Veredelung vornehmen
  * Prio: 1
  * Deutschland
* Sport Fleck GbR, Trier
  * Alias: *SF*
  * Kann jegliche Veredelung vornehmen
  * Prio: 1
  * Deutschland
* 11teamsports AT GmbH
  * Alias: *AT*
  * Kann keine Veredelung vornehmen
  * Prio: 2
  * Österreich
* 11teamsports CH
  * Alias: *CH*
  * Kann keine Veredelung vornehmen
  * Prio: 3
  * Schweiz

## Auftragstemplates

1. Einpositions-Auftrag Einfach

   Besteht nur aus einer Position, mit Menge 1, die keinen Veredelungs- oder Flockartikel enthält.

2. Mehrpositions-Auftrag Einfach

   Besteht aus zwei Positionen, mit Menge 1, die keine Veredelungs- oder Flockartikel enthalten.

3. Einpositions-Auftrag Mehrfach

   Besteht nur aus einer Position, mit Menge größer 1, die keinen Veredelungs- oder Flockartikel enthält.

4. Mehrpositions-Auftrag Mehrfach

   Besteht aus zwei Positionen, mit Menge größer 1, die keine Veredelungs- oder Flockartikel enthalten.

5. Einpositons-Auftrag Veredelung

   Besteht aus einer Position, mit Menge 1, die einen Veredelungs- oder Flockartikel enthält.

6. Mehrpositions-Auftrag Veredelung

   Besteht aus zwei (unteilbaren) Positionen, mit Menge 1, die einen Veredelungs- oder Flockartikel enthalten.

7. Mehrpositions-Auftrag Mix

   Besteht aus drei Positionen mit jeweils Menge 1. Die erste Position enthält einen normalen Artikel. Die zweite einen Flockartikel und die dritte eine Veredelung.

## 1. Vollständig beim Dealer

| Eigenschaft | Wert
| --- | ---
| Auftragstemplate(s) | alle
| Dealer | kann Auftrag komplett erfüllen
| Supplier | egal

Resultat:

* Der Auftrag muss als **abgelehnt** markiert werden

## 2. Teilweise beim Dealer, komplett bei den Supplier(n)

| Eigenschaft | Wert
| --- | ---
| Auftragstemplate(s) | 2
| Dealer | kann 1. Position erfüllen
| VE | kann die 2. Position erfüllen
| BER | kann die 2. Position erfüllen
| SF | kann Auftrag komplett erfüllen
| AT | kann Auftrag komplett erfüllen
| CH | kann Auftrag komplett erfüllen

Resultat:

* Der Auftrag muss als **zugelassen** markiert werden
* Der komplette Auftrag muss zu Supplier *SF* übertragen werden
* AT, CH scheiden wegen niedriger Prio aus

## 3. Teilweise beim Dealer, Teilweise bei den Supplier(n)

| Eigenschaft | Wert
| --- | ---
| Auftragstemplate(s) | 2
| Dealer | kann 1. Position erfüllen
| VE | kann 2. Position erfüllen
| BER | kann 2. Position erfüllen
| SF | kann 1. Position erfüllen
| AT | kann Auftrag komplett erfüllen
| CH | kann Auftrag komplett erfüllen

Resultat:

* Der Auftrag muss als **zugelassen** markiert werden
* Die Position 2 muss zu einem Supplier (VE, BER) übertragen werden
* Die Position 1 "bleibt" beim Dealer und wird durch diesen erfüllt
* AT, CH scheiden wegen niedriger Prio aus

## 4. Teilmenge beim Dealer, komplett bei den Supplier(n)

| Eigenschaft | Wert
| --- | ---
| Auftragstemplate(s) | 3
| Dealer | kann Position nur zum Teil erfüllen
| VE | kann Postion nur zum Teil erfüllen
| BER | kann Position komplett erfüllen
| SF | kann Position nicht erfüllen
| AT | kann Auftrag komplett erfüllen
| CH | kann Auftrag komplett erfüllen

Resultat:

* Der Auftrag muss als **zugelassen** markiert werde
* Der komplette Auftrag muss zu Supplier *BER* übertragen werden
* AT, CH scheiden wegen niedriger Prio aus.

## 5. Teilmenge beim Dealer, Teilmenge bei den Supplier(n)

| Eigenschaft | Wert
| --- | ---
| Auftragstemplate(s) | 3
| Menge der Pos. | 4 (als Bsp.)
| Dealer | kann Position nur zum Teil erfüllen (2 Stk.)
| VE | kann Postion nur zum Teil erfüllen (3 Stk.)
| BER | kann Postion nur zum Teil erfüllen (2 Stk.)
| SF | kann Position nicht erfüllen
| AT | kann Auftrag komplett erfüllen
| CH | kann Auftrag komplett erfüllen

Resultat:

* Der Auftrag muss als **zugelassen** markiert werde
* Die Position muss mit einer Teilmenge zu einem der Supplier (VE, BER) übertragen werden
* Die Teilmenge, die zum Supplier übertragen wird, ist gleich:

  ``` sql
  MAX((GGBestellt - GGGeliefert) - Dealer.Verfügbar, Supplier.Verfügbar)`
  ```

* Die restliche Teilmenge "bleibt" beim Deauler und wird duch diesen erfüllt
* Keine weiteren Splits erfolgen da Auftrag nach dem 1. Split komplett erfüllt ist
* AT, CH scheiden wegen niedriger Prio aus

## 6. Teilmenge beim Dealer, Teilmenge bei den Supplier(n)

| Eigenschaft | Wert
| --- | ---
| Auftragstemplate(s) | 3
| Menge der Pos. | 4 (als Bsp.)
| Dealer | kann Position nur zum Teil erfüllen (1 Stk.)
| VE | kann Postion nur zum Teil erfüllen (2 Stk.)
| BER | kann Postion nur zum Teil erfüllen (2 Stk.)
| SF | kann Position nicht erfüllen
| AT | kann Auftrag komplett erfüllen
| CH | kann Auftrag komplett erfüllen

Resultat:

* Der Auftrag muss als **zugelassen** markiert werde
* Die Position muss mit einer Teilmenge zu einem der Supplier (VE, BER) übertragen werden
* Die Teilmenge, die zum Supplier übertragen wird, ist gleich:

  ``` sql
  MAX((GGBestellt - GGGeliefert) - Dealer.Verfügbar, Supplier.Verfügbar)
  ```

* Die restliche Teilmenge "bleibt" beim Deauler und wird duch diesen erfüllt
* Es wird ein weiterer Split generiert, für den Supplier der vorher nicht ausgewählt wurde, da der Auftrag nicht komplett erfüllt worden ist. Die Menge für den Split wird wie oben berechnet (DropLi wurde schon erstellt und somit GGGeliefert erhöht)
* AT, CH scheiden wegen niedriger Prio aus
