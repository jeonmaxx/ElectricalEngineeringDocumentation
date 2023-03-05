# Widerstandsfarbcode

## Einleitung

In diesem Teil wird die Benutzung, der ==Widerstand Tools== erkl&auml;rt.
Es gibt f&uuml;r die Widerstand Tools momentan drei Klassen. 
Einmal die **Resistor** Klasse, die haupts&auml;chlich die Werte verwaltet. 
Dann die **Factor** Klasse, welche den Factorring bearbeitet. 
Und die **Tolerance** Klasse, die den Toleranzring bearbeitet.

## Resistor
Zuerst gibt es ein Enum, f&uuml;r die einzelnen Farben der Ringe, welchen man erweitern k&ouml;nnte, falls noch Farben dazu kommen sollten.
```c#
    public enum ResistorRing
    {
        Black = 0, Brown = 1, Red = 2, Orange = 3, Yellow = 4,
        Green = 5, Blue = 6, Purple = 7, Gray = 8, White = 9,
        Gold = 10, Silver = 11, Pink = 12
    }
```
Um den Farbcode von **Farben zu Wert** zu konvertieren, gibt es momentan zwei Konstruktoren. Einmal f&uuml;r vier Ringe und nat&uuml;rlich auch f&uuml;r Ffuuml;nf Ringe. 
Als Beispiel wird der f&uuml;r vier Ringe gezeigt:
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
Bei dieser Funktion muss man angeben, ==welche Farben die ersten beiden Ringe, der Faktorring und der Toleranzring haben==.
Zuerst wird gepr&uuml;ft, ob die Farben, die man eingetragen hat, auch angenommen werden k&ouml;nnen. 
Der Factorring wird an die Faktor Klasse und der Toleranzring an die Toleranz Klasse &uuml;bergeben.
Dann sagt die Funktion, dass hier nur vier Ringe verwendet werden (Beim anderen Konstruktor sagt er an der Stelle f&uuml;nf Ringe) 
und zum Schluss wird zum finalen berechnen noch die Funktion "UpdateValueFromColors" verwendet.  

Nat&uuml;rlich gibt es auch einen Konstruktor, wenn man von einem **Wert zu Farben** konvertieren will:
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
Hierbei muss man angeben, ==welchen Wert man braucht, welche Toleranz man haben m&ouml;chte und wie viele Ringe der Widerstandsfarbcode haben soll==.
Dann gibt es einen Switch f&uuml;r beide F&auml;lle der Ringanzahl, also sowohl f&uuml;r vier, als auch f&uuml;r f&uuml;nf Ringe. 
Dabei wird der Wert dann gek&uuml;rzt, da nur die ersten beiden, beziehungsweise drei Zahlen wichtig f&uuml;r die Umrechnung sind.
Dabei wird der cFaktor bei jedem K&uuml;rzen erh&ouml;ht, damit das Programm bestimmen kann, welche Faktorfarbe ben&ouml;tigt wird.
Dann berechnet er die einzelnen Ziffern, der relevanten Zahl, mit der Hilfe von Division und Modulo.
So kann er also bestimmen, welche Farbe sowohl der erste, als auch der zweite und ggf. dritte Ring ben&ouml;tigt.
Der Faktor wird dann mit Hilfe der Faktor Klasse und cFaktor berechnet und die Toleranz musste man hier selbst bestimmen.

Und zu guter Letzt gibt es noch die "UpdateValueFromColors" Funktion, welche in allen Konstruktoren f&uuml;r die Umrechnung benutzt wird:
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
Auch hier gibt es wieder eine M&ouml;glichkeit, den Wert f&uuml;r vier, als auch f&uuml;r f&uuml;nf Ringe auszurechnen. 
Der Wert, der dabei rauskommt wird dann noch mit dem Wert des Faktors multipliziert und schon hat man den ben&ouml;tigten Wert.

## Factor
Die Faktor Klasse hat nur Drei Hauptteile, die relativ simpel und einfach zu erkl&auml;ren sind.
Hier werden zuerst die einzelnen Faktoren aufgelistet, welche auch hier erweiterbar w&auml;ren, falls noch welche dazu kommen sollten.
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
Wenn man von **Farbe zu Wert** rechnen m&ouml;chte, dann kann man das ganz einfach mit dieser Funktion machen, indem man ihr die gew&uuml;nschte Farbe &uuml;bergibt. 
In dem Switch wird festgelegt, welche Farbe welchen Faktor bekommt und dieser Faktorwert wird dann zur&uuml;ckgegeben.
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

Hiermit kann man von **Wert zu Farbe** rechnen, indem man den Wert &uuml;bergibt. 
Auch hier wird ein Switch benutzt, um den gew&uuml;nschten Wert in die dazugeh&ouml;rige Farbe umzuwandeln.
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
Auch hier gibt es wieder einen Enum. Dieser verf&uuml;gt &uuml;ber die drei verschiedenen Rundungs-Modi: ==Abrunden, Aufrunden und zum n&auml;chsten runden==. 
Danach werden den verschiedenen Toleranzm&ouml;glichkeiten noch ihre jeweilige Ringfarbe zugeteilt.
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
Dieser Konstruktor ist daf&uuml;r zust&auml;ndig, dass wenn man eine Farbe angibt, man den passenden Toleranzwert ausgegeben bekommt. 
Falls man eine Farbe angeben sollte, f&uuml;r welche es keinen Wert gibt, dann wird eine Exception getriggert, damit daraus keine Fehler entstehen k&ouml;nnen.
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
Dieser Konstruktor ist f&uuml;r das Kovertieren von Wert zu Farbe zust&auml;ndig. 
Zudem muss man auch den Rundungsmodus ausw&auml;hlen, damit einer der m&ouml;glichen Farben bei der Rechnung als Ergebnis herauskommen kann. Default ist dabei das Abrunden.
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
Zun&auml;chst gibt es ein Array, wo alle erlaubten Ergebnisse eingetragen sind. Dieses ist f&uuml;r die folgenden Rechungen n&uuml;tzlich.  
Als n&auml;chstes folgt ein Switch f&uuml;r die verschiedenen Rundungsmodi. 
Beim **Abrunden** werden die erlaubten Zahlen zuerst vom ==Gr&ouml;&szlig;ten bis zum Kleinsten== sortiert. 
Allerdings werden dabei nur die Werte beachtet, die kleiner als der angegebene Wert sind.
Danach wird der erste von diesen Werten (also der gr&ouml;&szlig;te Wert, von denen die kleiner als der angegebene sind) zu dem Wert von lowTargetValue. 
Wenn alle erlaubten Werte gr&ouml;&szlig;er als der angegebene Wert sind, dann wird lowTargetValue zu 0. 
Wenn lowTargetValue 0 ist, wird der kleinste Wert aus dem erlaubten Werte Array genommen, ansonsten bleibt es beim Wert von lowTargetValue.  
Beim **Aufrunden** ist es so &auml;hnlich, nur dass vom ==Kleinsten bis zum Gr&ouml;&szlig;ten== sortiert wird. 
Die Werte, welche kleiner als der angegebene Wert sind, werden dabei &uuml;bersprungen und dann wird der kleinste Wert von denen, die gr&ouml;&szlig;er als der angegebene Wert highTargetValue zugewiesen. 
Wenn dieser Wert auch hier 0 sein sollte, wird highTargetValue zum gr&ouml;&szlig;ten Wert von den erlaubten Werten.  
Beim **Runden zum N&auml;chsten** wird Aggregate benutzt, um herauszufinden, welcher Wert am n&auml;chsten an dem angegebenen ist. 
Dabei wird &uuml;berpr&uuml;ft, ob die Differenz zu x oder y kleiner ist. 
Ist der Abstand zu x kleiner, dann bleibt der x Wert beim n&auml;chsten Durchlauf x und wenn nicht, dann wird der y Wert zum neuen x Wert. 
Das geht solange, bis alle Werte vom Array durchgelaufen sind. 
Der letzte x Wert bleibt dann der Wert von Value und ist somit der n&auml;chste Wert an dem angegebenen.  

Hier werden den einzelnen numerischen Werten die jeweilige Farbe zugewiesen, falls man die Farbe durch einen angegebenen Wert ausrechnen m&ouml;chte.
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
Durch Tests haben wir &uuml;berpr&uuml;ft ob ColorToValue, ValueToColor und die Toleranz Methoden so funktioren, wie sie funktionieren sollten.  
Dabei haben wir f&uuml;r jeden dieser Testkategorien eine eigene Klasse gemacht, damit es &uuml;bersichtlich bleibt.  

**ColorToValue Test**  
Zuerst haben wir die Methoden getestet, mit denen man von Farbe zu Wert konvertieren kann. 
Dabei wurde zum einen getestet, was passiert, wenn man eine ung&uuml;ltige Farbe eingeben w&uuml;rde. 
```c#
        [TestMethod]
        public void InvalidNummeric()
        {
            ResistorRing r = ResistorRing.Red;

            Resistor RFour = new Resistor(r, r, r, r);
            Resistor RFive = new Resistor(r, r, r, r, r);
        }
```
Dann wurde auch direkt getestet, ob das Konvertieren von Farbe zu Faktor vern&uuml;nftig funktioniert und auch keine Fehler auftauchen.
```c#
        [TestMethod]
        public void ColorToFactor()
        {
            ResistorRing[] RingFactor = new ResistorRing[] {ResistorRing.Pink,ResistorRing.Silver,ResistorRing.Gold,ResistorRing.Black, ResistorRing.Brown,
                                                             ResistorRing.Red,ResistorRing.Orange,ResistorRing.Yellow,
                                                             ResistorRing.Green,ResistorRing.Blue,ResistorRing.Purple,ResistorRing.Gray,ResistorRing.White,
                                                             };
            double j = 0.001;

            for (int i = 0; i < RingFactor.Length; i++)
            {
                Assert.AreEqual(j, new Factor(RingFactor[i]).Value);
                j *= 10;
            }
        }
```

**ValueToColor Test**  
Es wurde auch getestet, ob das Konvertieren von Wert zu Farbe funktioniert. 
Dabei wurde als erstes getestet, ob die Funktion den Wert richtig rundet.
```c#
        [TestMethod]
        public void ValueTest()
        {
            long test = new Resistor(373500, Tolerance.One, RingCount.Four).Value;

            Assert.AreEqual(370000, test);
        }
```

Danach wurde getestet, ob die Konvertierung mit vier Ringen so funktioniert, wie gedacht.
```c#
        [TestMethod]
        public void FourRingTest()
        {
            ResistorRing One = new Resistor(373500, Tolerance.One, RingCount.Four).Ring1;
            ResistorRing Two = new Resistor(373500, Tolerance.One, RingCount.Four).Ring2;
            ResistorRing Three = new Factor(1).Color;
            ResistorRing Four = new Tolerance(One).Color;
            string fourRings = One.ToString() + " " + Two.ToString() + " " + Three.ToString() + " " + Four.ToString();

            string result = "Orange Purple Black Orange";

            Assert.AreEqual(result, fourRings);
        }
```

Nat&uuml;rlich wurde das Gleiche auch mit f&uuml;nf Ringen getestet.
```c#
        [TestMethod]
        public void FiveRingTest()
        {
            ResistorRing One = new Resistor(373500, Tolerance.One, RingCount.Five).Ring1;
            ResistorRing Two = new Resistor(373500, Tolerance.One, RingCount.Five).Ring2;
            ResistorRing Three = new Resistor(373500, Tolerance.One,RingCount.Five).Ring3;
            ResistorRing Four = new Factor(1).Color;
            ResistorRing Five = new Tolerance(One).Color;
            string fiveRings = One.ToString() + " " + Two.ToString() + " " + Three.ToString() + " " + Four.ToString() + " " + Five.ToString();

            string result = "Orange Purple Orange Black Orange";

            Assert.AreEqual(result, fiveRings);
        }
```

Als n&auml;chstes wurde &uuml;berpr&uuml;ft, ob auch der Faktorring funktioniert und die richtige Farbe ausgibt.
```c#
        [TestMethod]
        public void FactorRingTest()
        {
            int val = Rounding(120);
            Assert.AreEqual(ResistorRing.Red, new Factor(val).Color);
        }

        public int Rounding(int value)
        {
            int digits = value.ToString().Length;
            int newValue = 1;

            while (digits > 1)
            {
                newValue *= 10;
                digits -= 1;
            }
            return newValue;
        }
    }
```

**Tolerance Test**  
Zum Schluss wurde noch getestet, ob die Toleranzklasse so funktioniert, wie gedacht. 
Dabei wurden die verschiedenen Rundungsmodi mit verschiedenen Zahlen getestet, um Fehler m&ouml;glichst fr&uuml;hzeitig zu erkennen.
```c#
        [TestMethod]
        public void TestMethod1()
        {
            (double input, double expectedValue, Tolerance.RoundMode roundMode)[] testList =
            {
                (0.00001, 0.01, Tolerance.RoundMode.ToLower),
                (0, 0.01, Tolerance.RoundMode.ToLower),
                (-1, 1, Tolerance.RoundMode.ToLower),
                (-10_000_000_000, 10, Tolerance.RoundMode.ToLower),
                (10_000_000_000, 10, Tolerance.RoundMode.ToLower),
                (3.5,2,Tolerance.RoundMode.ToLower),

                (0.00001, 0.01, Tolerance.RoundMode.ToUpper),
                (0, 0.01, Tolerance.RoundMode.ToUpper),
                (-1, 1, Tolerance.RoundMode.ToUpper),
                (-10_000_000_000, 10, Tolerance.RoundMode.ToUpper),
                (10_000_000_000, 10, Tolerance.RoundMode.ToUpper),
                (3.5,5,Tolerance.RoundMode.ToUpper),

                (0.00001, 0.01, Tolerance.RoundMode.Nearest),
                (0, 0.01, Tolerance.RoundMode.Nearest),
                (-1, 1, Tolerance.RoundMode.Nearest),
                (-10_000_000_000, 10, Tolerance.RoundMode.Nearest),
                (10_000_000_000, 10, Tolerance.RoundMode.Nearest),
                (3.5,5,Tolerance.RoundMode.Nearest)
            };

            foreach((double input, double expectedValue, Tolerance.RoundMode roundMode) in testList)
            {
                Assert.AreEqual(expectedValue, new Tolerance(input, roundMode).Value);
            }
        }
```