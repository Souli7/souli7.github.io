# souli7.github.io

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
