# Widerstandsfarbcode

## Einleitung

In diesem Teil wird die Benutzung, der Widerstand Tools erklaert.

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
Um den Farbcode von Farben zu Wert zu konvertieren, gibt es momentan zwei Konstruktoren. Einmal fuer Vier Ringe und natuerlich auch fuer Fuenf Ringe. Als Beispiel wird der fuer Vier Ringe gezeigt:
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


Natuerlich gibt es auch einen Konstruktor, wenn man von einem Wert zu Farben konvertieren will:
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

## Factor

## Tolerance
