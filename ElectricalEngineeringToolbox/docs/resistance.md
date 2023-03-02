# Widerstandsfarbcode

## Einleitung

In diesem Teil wird die Benutzung, der Widerstand Tools erklaert.
Es gibt fuer die Widerstand Tools momentan Drei Klassen. 
Einmal die **Resistor** Klasse, die hauptsaechlich die Werte verwaltet. 
Dann die **Factor** Klasse, welche den Factorring bearbeitet. 
Und die **Tolerance** Klasse, die den Toleranzring bearbeitet.

## Resistor
Zuerst gibt es einen Enum, fuer die einzelnen Farben der Ringe, welchen man erweitern koennte, falls noch Farben dazu kommen sollten.
```c#
    public enum ResistorRing
    {
        Black = 0, Brown = 1, Red = 2, Orange = 3, Yellow = 4,
        Green = 5, Blue = 6, Purple = 7, Gray = 8, White = 9,
        Gold = 10, Silver = 11, Pink = 12
    }
```
Um den Farbcode von **Farben zu Wert** zu konvertieren, gibt es momentan zwei Konstruktoren. Einmal fuer Vier Ringe und natuerlich auch fuer Fuenf Ringe. 
Als Beispiel wird der fuer Vier Ringe gezeigt:
```c#
        public Resistor(ResistorRing ring1, ResistorRing ring2, ResistorRing factor, ResistorRing tolerance)
        {
            InvalidNumber(ring1);
            InvalidNumber(ring2);

            _ring1 = ring1;
            _ring2 = ring2;
            _factor = new Factor(factor);
            _tolerance = new Tolerance(tolerance);
            _ringCount = RingCount.Four;
            UpdateValueFromColors();
        }
```
Bei dieser Funktion muss man angeben, welche Farben die ersten beiden Ringe, der Faktorring und der Toleranzring hat.
Zuerst wird geprueft, ob die Farben, die man eingetragen hat, auch angenommen werden koennen. 
Der Factorring wird an die Faktor Klasse und der Toleranzring an die Toleranz Klasse uebergeben.
Dann sagt die Funktion, dass hier nur Vier Ringe verwendet werden (Beim anderen Konstruktor sagt er an der Stelle Fuenf Ringe) 
und zum Schluss wird zum finalen berechnen noch die Funktion "UpdateValueFromColors" verwendet.  

Natuerlich gibt es auch einen Konstruktor, wenn man von einem **Wert zu Farben** konvertieren will:
```c#
        public Resistor(int value, Tolerance tolerance, RingCount ringCount)
        {
            _ringCount = ringCount;
            _tolerance = tolerance;

            int relevantDigits = value;
            int cFactor = 0;

            switch (ringCount)
            {
                case RingCount.Four:
                    while (relevantDigits >= 100)
                    {
                        relevantDigits /= 10;
                        cFactor++;
                    }

                    _ring2 = (ResistorRing)(relevantDigits % 10);
                    _ring1 = (ResistorRing)(relevantDigits / 10);

                    break;

                case RingCount.Five:
                    while (relevantDigits >= 1000)
                    {
                        relevantDigits /= 10;
                        cFactor++;
                    }

                    _ring3 = (ResistorRing)(relevantDigits % 10);
                    _ring2 = (ResistorRing)(relevantDigits / 10 % 10);
                    _ring1 = (ResistorRing)(relevantDigits / 100 % 10);
                    break;

                default:
                    throw new ArgumentOutOfRangeException(nameof(ringCount), ringCount, null);

            }

            _factor = new Factor(Math.Pow(10, cFactor));
            UpdateValueFromColors();
        }
```
Hierbei muss man angeben, welchen Wert man braucht, welche Toleranz man haben moechte und wie viele Ringe der Widerstandsfarbcode haben soll.
Dann gibt es einen Switch fuer beide Faelle der Ringanzahl, also sowohl fuer Vier, als auch fuer Fuenf Ringe. 
Dabei wird der Wert dann gekuerzt, da nur die ersten beiden, beziehungsweise Drei Zahlen wichtig fuer die Umrechnung sind.
Dabei wird der cFaktor bei jedem kuerzen erhoeht, damit das Program bestimmen kann, welche Faktorfarbe benoetigt wird.
Dann berechnet er die einzelnen Ziffern, der relevanten Zahl, mit der Hilfe von geteilt und Modulo.
So kann er also bestimmen, welche Farbe sowohl der erste, als auch der zweite und ggf. dritte Ring benoetigt.
Der Faktor wird dann mit Hilfe der Faktor Klasse und cFaktor berechnet und die Toleranz musste man hier selbst bestimmen.

Und zu guter Letzt gibt es noch die "UpdateValueFromColors" Funktion, welche in allen Konstruktoren fuer die Umrechnung benutzt wird:
```c#
        private void UpdateValueFromColors()
        {
            int tmp = _ringCount switch
            {
                RingCount.Four => (int) (_ring2 + 10 * (int) _ring1),
                RingCount.Five => (int) (_ring3 + 10 * (int) _ring2 + 100 * (int) _ring1),
                _ => throw new ArgumentOutOfRangeException()
            };
            _value = (long) (tmp * _factor.Value);
        }
```
Auch hier gibt es wieder eine Moeglichkeit, den Wert fuer Vier, als auch fuer Fuenf Ringe auszurechnen. 
Der Wert, der dabei raus kommt wird dann noch mit dem Wert des Faktors multipliziert und schon hat man den benoetigten Wert.

## Factor
Die Faktor Klasse hat nur Drei Hauptteile, die relativ simpel und einfach zu erklaeren sind.
Hier werden zuerst die einzelnen Faktoren aufgelistet, welche auch hier erweiterbar waeren, falls noch welche dazu kommen sollten.
```c#
        public static Factor One = new Factor(1);
        public static Factor Ten = new Factor(10);
        public static Factor Hundred = new Factor(100);
        public static Factor Thousand = new Factor(1_000);
        public static Factor TenThousand = new Factor(10_000);
        public static Factor HundredThousand = new Factor(100_000);
        public static Factor Million = new Factor(1_000_000);
        public static Factor TenMillion = new Factor(10_000_000);
        public static Factor HundredMillion = new Factor(100_000_000);
        public static Factor Billion = new Factor(1_000_000_000);

        public ResistorRing Color { get; }
        public double Value { get; }
```
Wenn man von **Farbe zu Wert** rechnen moechte, dann kann man das ganz einfach mit dieser Funktion machen, indem man ihr die gewuenschte Farbe uebergibt. 
In dem Switch wird festgelegt welche Farbe welchen Faktor bekommt und dieser Faktorwert wird dann zuruekgegeben.
```c#
        public Factor(ResistorRing color)
        {
            Color = color;

            Value = color switch
            {
                ResistorRing.Black => 1,
                ResistorRing.Brown => 10,
                ResistorRing.Red => 100,
                ResistorRing.Orange => 1_000,
                ResistorRing.Yellow => 10_000,
                ResistorRing.Green => 100_000,
                ResistorRing.Blue => 1_000_000,
                ResistorRing.Purple => 10_000_000,
                ResistorRing.Gray => 100_000_000,
                ResistorRing.White => 1_000_000_000,
                ResistorRing.Gold => 0.1,
                ResistorRing.Silver => 0.01,
                ResistorRing.Pink => 0.001,
                _ => throw new ArgumentOutOfRangeException(nameof(color), color, null)
            };
        }
```

Hiermit kann man von **Wert zu Farbe** rechnen, indem man den Wert uebergibt. 
Auch hier wird ein Switch benutzt, um den gewuenschten Wert in die dazugehoerige Farbe umzuwandeln.
```c#
        public Factor(double value)
        {
            Value = value;
            Color = value switch
            {
                1 => ResistorRing.Black,
                10 => ResistorRing.Brown,
                100 => ResistorRing.Red,
                1_000 => ResistorRing.Orange,
                10_000 => ResistorRing.Yellow,
                100_000 => ResistorRing.Green,
                1_000_000 => ResistorRing.Blue,
                10_000_000 => ResistorRing.Purple,
                100_000_000 => ResistorRing.Gray,
                1_000_000_000 => ResistorRing.White,
                0.1 => ResistorRing.Gold,
                0.01 => ResistorRing.Silver,
                0.001 => ResistorRing.Pink,
                _ => throw new OutOfSpecException()
            };
        }
```


## Tolerance
Auch hier gibt es wieder einen Enum. Dieser verfuegt ueber die Drei verschiedenen Rundungs-Modi: Abrunden, Aufrunden und zum naechsten runden. 
Danach werden den verschiedenen Toleran Moeglichkeiten noch ihre jeweilige Ringfarbe zugeteilt.
```c#
        public enum RoundMode { ToLower, ToUpper, Nearest }

        public static Tolerance One => new Tolerance(ResistorRing.Brown);
        public static Tolerance Two => new Tolerance(ResistorRing.Red);
        public static Tolerance Half => new Tolerance(ResistorRing.Green);
        public static Tolerance Quarter => new Tolerance(ResistorRing.Blue);
        public static Tolerance Tenth => new Tolerance(ResistorRing.Purple);
        public static Tolerance Twentieth => new Tolerance(ResistorRing.Gray);
        public static Tolerance Five => new Tolerance(ResistorRing.Gold);
        public static Tolerance Ten => new Tolerance(ResistorRing.Silver);

        public ResistorRing Color { get; private set; }
        public double Value { get; }
```
Dieser Konstruktor ist dafuer zustaendig, dass wenn man eine Farbe angibt, man den passenden Toleranzwert ausgegeben bekommt. 
Falls man eine Farbe angeben sollte, fuer welche es keinen Wert geben sollte, dann wird eine Exception getriggert, damit daraus keine Fehler entstehen koennen.
```c#
        public Tolerance(ResistorRing ring)
        {
            Color = ring;
            Value = ring switch
            {
                ResistorRing.Black => throw new OutOfSpecException(),
                ResistorRing.Brown => 1,
                ResistorRing.Red => 2,
                ResistorRing.Orange => 0.05,
                ResistorRing.Yellow => 0.02,
                ResistorRing.Green => 0.5,
                ResistorRing.Blue => 0.25,
                ResistorRing.Purple => 0.1,
                ResistorRing.Gray => 0.01,
                ResistorRing.White => throw new OutOfSpecException(),
                ResistorRing.Gold => 5,
                ResistorRing.Silver => 10,
                ResistorRing.Pink => throw new OutOfSpecException(),
                _ => throw new ArgumentOutOfRangeException(nameof(ring), ring, null)
            };
        }
```
Dieser Konstruktor ist fuer das Kovertieren von Wert zu Farbe zustaending. 
Zudem muss man auch den Rundungsmodus auswaehlen, damit einer der moeglichen Farben bei der Rechnung als Ergebnis herauskommen kann. Default ist dabei das Abrunden.
```c#
        public Tolerance(double value, RoundMode mode = RoundMode.ToLower)
        {
            double[] allowedValues = { 1, 2, 0.5, 0.25, 0.1, 0.01, 5, 10, 0.02, 0.05 };
            value = Math.Abs(value);

            switch (mode)
            {
                case RoundMode.ToLower:
                    double lowTargetValue = allowedValues.OrderByDescending(v => v)
                        .SkipWhile(v => v > value)
                        .FirstOrDefault();
                    Value = lowTargetValue == 0 ? allowedValues.Min() : lowTargetValue;
                    break;

                case RoundMode.ToUpper:
                    double highTargetValue = allowedValues.OrderBy(v => v)
                        .SkipWhile(v => v < value)
                        .FirstOrDefault();
                    Value = highTargetValue == 0 ? allowedValues.Max() : highTargetValue;
                    break;

                case RoundMode.Nearest:
                    Value = allowedValues.Aggregate((x, y) => Math.Abs(x - value) < Math.Abs(y - value) ? x : y);
                    break;

                default:
                    throw new ArgumentOutOfRangeException(nameof(mode), mode, null);
            }

            CalculateColorFromValue();
        }
```
Zunaechst gibt es ein Array, wo alle erlaubten Ergebnisse eingetragen sind. Dieses ist fuer die folgenden Rechungen nuetzlich.  
Als naechstes folgt ein Switch fuer die verschiedenen Rundungs Modi. 
Beim **Abrunden** werden die erlaubten Zahlen zuerst vom Groessten zum Kleinsten sortiert. 
Allerdings werden dabei nur die Werte beachtet, die kleiner als der angebene Wert sind. 
Danach wird der erste von diesen Werten (also der groesste Wert, von denen die kleiner als der angebene sind) zu dem Wert von lowTargetValue. 
Wenn alle erlaubten Werte groesser als der angebene Wert sind, dann wird lowTargetValue zu 0. 
Wenn lowTargetValue 0 ist, wird der kleinste Wert aus dem erlaubten Werte Array genommen, ansonsten bleibt es beim Wert von lowTargetValue.  
Beim **Aufrunden** ist es so aehnlich, nur dass vom Kleinsten bis zum Groessten sortiert wird. 
Die Werte, welche kleiner als der angebene Wert sind werden dabei uebersprungen und dann wird der kleinste Wert von denen, die groesser als der angegebene Wert highTargetValue zugewiesen. 
Wenn dieser Wert auch hier 0 sein sollte, wird highTargetValue zum groessten Wert von den erlaubten Werten.  
Beim **Runden zum Naechsten** wird Aggregate benutzt um herauszufinden, welcher Wert am naechsten an dem angegebenen ist. 
Dabei wird geprueft ob der x Wert minus dem angebenen Wert kleiner als der y Wert minus dem angegebenen Wert ist. 
Wenn ja, dann bleibt der x Wert beim naechsten Durchlauf x und wenn nicht, dann wird der y Wert zum neuen x Wert. 
Das geht solange, bis alle Werte vom Array durchgelaufen sind. 
Der letzte x Wert bleibt dann der Wert von Value und ist somit der naechste Wert an dem angegebenen.  

Hier werden den einzelnen numerischen Werten die jeweilige Farbe zugewiesen, falls man die Farbe durch einen angegebenen Wert ausrechnen moechte.
```c#
        private void CalculateColorFromValue()
        {
            Color = Value switch
            {
                1 => ResistorRing.Brown,
                2 => ResistorRing.Red,
                0.5 => ResistorRing.Green,
                0.25 => ResistorRing.Blue,
                0.1 => ResistorRing.Purple,
                0.01 => ResistorRing.Gray,
                5 => ResistorRing.Gold,
                10 => ResistorRing.Silver,
                0.02 => ResistorRing.Yellow,
                0.05 => ResistorRing.Orange,
                _ => throw new ArgumentOutOfRangeException()
            };
        }
```

## Testing
Durch Tests haben wir ueberprueft ob ColorToValue, ValueToColor und die Toleranz Methoden so funktioren, wie sie funktionieren sollten.  
Dabei haben wir fuer jeden dieser Testkategorien eine eigene Klasse gemacht, damit es uebersichtlich bleibt.