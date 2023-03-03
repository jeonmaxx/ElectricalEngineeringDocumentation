# Leiterbahnbreiten

## Einleitung

In diesem Teil wird die Benutzung, der ==Leiterbahnbreiten Tools== erkl&auml;rt. 
Um die verschiedenen Einheiten und Gr&ouml;&szlig;en zu verwalten wurde ein **Unit Tree** benutzt, damit eine gute Performance zu gew&auml;hrleistet wird. 
Desweiteren wird die **Calculator** Klasse benutzt, um die Ergebniss Einheiten zu verwalten.

## UnitTree
Als erstes besitzt der Unit Tree ein Enum, welches alle vorhandenen **SI-Pr&auml;fixe** enth&auml;lt. 
Diese w&auml;ren nat&uuml;rlich erweiterbar, falls jemals welche dazu kommen sollten.
Zudem besitzt das Enum einen Default, da die Pr&auml;fixe nicht immer ben&ouml;tigt werden.
```c#
        public enum SIPrefix
        {
            Quetta = 30,
            Ronna = 27,
            Yotta = 24,
            Zetta = 21,
            Exa = 18,
            Peta = 15,
            Tera = 12,
            Giga = 9,
            Mega = 6,
            Kilo = 3,
            Hekto = 2,
            Deka = 1,
            SIDefault = 0,
            Dezi = -1,
            Zenti = -2,
            Milli = -3,
            Mikro = -6,
            Nani = -9,
            Piko = -12,
            Femot = -15,
            Atto = -18,
            Zepto = -21,
            Yokto = -24,
            Ronto = -27,
            Quekto = -30
        }
```
Danach beginnt auch schon der Unit Tree mit einer Klassen f&uuml;r die **Nodes**.
```c#
        private class Node
        {
            public readonly string Unit;
            public readonly Node? ParentUnit;

            public Node(string unit, Node? parentUnit)
            {
                Unit = unit;
                ParentUnit = parentUnit;
            }

            public override string ToString()
            {
                return Unit;
            }

            public bool HasUnit(string unit)
            {
                return unit == Unit;
            }
        }
```
Als n&auml;chstes folgt eine Methode, f&uuml;r die Konvertierung der SI-Pr&auml;fixe.
```c#
        public double ConvertSIPrefixValue(string startUnit, SIPrefix startPrefix, string targetUnit,
            SIPrefix targetPrefix, double value)
        {
            return ConvertValue(startUnit, targetUnit, value * Math.Pow(10.0, (double) startPrefix)) *
                   Math.Pow(10.0, (double) targetPrefix);
        }
```

Hier ist eine Methode, welche den Wert dann konvertiert.
```c#
        public double ConvertValue(string startUnit, string targetUnit, double value)
        {
            IsFunctional();
            if (startUnit.Equals(targetUnit))
            {
                return value;
            }

            if (_conversion.ContainsKey((startUnit,targetUnit)))
            {
                return _conversion[(startUnit, targetUnit)].Invoke(value);
            }

            Stack<ConversionFunction> functionStack = CreateConversionStack(startUnit, targetUnit);

            while (functionStack.Count > 0)
            {
                value = functionStack.Pop().Invoke(value);
                Console.WriteLine(value);
            }
            
            return value;
        }
```
Hier wird viel mit dem Unit Tree gearbeitet, um von der ==Start Unit zur Ziel Unit== zu gelangen.
```c#
        Stack<ConversionFunction> CreateConversionStack(string startUnit, string targetUnit)
        {
            Queue<Node> openQueue = new Queue<Node>();
            List<Node> closedList = new List<Node>();
            HashSet<string> processedUnits = new HashSet<string>();

            openQueue.Enqueue(new Node(startUnit, null));
            processedUnits.Add(startUnit);

            while (!_conversion.ContainsKey((openQueue.Peek().Unit, targetUnit)))
            {
                Node currentItem = openQueue.Dequeue();
                closedList.Add(currentItem);
                foreach (var y in _conversion.Keys.Where(x => x.from == currentItem.ToString()))
                {
                    if (!processedUnits.Contains(y.to))
                    {
                        openQueue.Enqueue(new Node(y.to, currentItem));
                        processedUnits.Add(y.to);
                    }
                }
            }

            closedList.Add(openQueue.Peek());
            Stack<ConversionFunction> functionStack = new Stack<ConversionFunction>();
            Node currentNode = new Node(targetUnit, openQueue.Peek());

            while (currentNode.ParentUnit != null)
            {
                Console.WriteLine(currentNode.Unit);
                functionStack.Push(_conversion[(currentNode.ParentUnit.Unit, currentNode.Unit)]);
                currentNode = currentNode.ParentUnit;
            }

            return functionStack;
        }

        public void AddPath(string startUnit, string targetUnit, ConversionFunction conversionFunc)
        {
            _functional = false;
            _conversion.TryAdd((startUnit,targetUnit), conversionFunc);
        }
```

Zum Schluss wird noch &uuml;berpr&uuml;ft, ob der Unit Tree funktioniert. 
Falls dies nicht der Fall sein sollte, dann wird einem in der Konsole deutlich gezeigt, dass etwas nicht ganz funktioniert.
```c#
        private void checkFunctionality()
        {
        }

        public bool IsFunctional()
        {
            if (!_functional)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("!!!Using not functional UnitTree!!!");
                Console.ResetColor();
            }
            return _functional;
        }
```

## Calculator
Au&szlig;erdem gibt es eine Klasse, welche die einzelnen Einheiten verwaltet, die Ergebnisse vom Kalkulator sind.  
Zum einem die **ben&ouml;tigte Fl&auml;che**, der inneren Schicht, wie auch der &auml;u&szlig;eren Schicht.
```c#
        public double CalculateAreaIntern(double ampere, double tRise)
        {
            double I = ampere;
            double TRise = tRise;
            double constantK = 0.024;
            double constantB = 0.44;
            double constantC = 0.725;
            double area;

            area = Math.Pow(I / (constantK * Math.Pow(TRise, constantB)), (1 / constantC));

            return area;
        }
```
```c#
        public double CalculateAreaExtern(double ampere, double tRise)
        {
            double I = ampere;
            double TRise = tRise;
            double constantK = 0.048;
            double constantB = 0.44;
            double constantC = 0.725;
            double area;

            area = Math.Pow(I / (constantK * Math.Pow(TRise, constantB)), (1 / constantC));

            return area;
        }
```

Und nat&uuml;rlich auch f&uuml;r die **Breite der Leiterbahnen**. Einmal wird dabei mir Mil gerechnet und einmal mit Oz/Ft^2^.
```c#
        public double CalculateWidthMil(double area, double thickness)
        {
            double A = area;
            double t = thickness;
            double width;

            width = A / t;

            return width;
        }
```
```c#
        public double CalculateWidthOzFt(double area, double thickness)
        {
            double A = area;
            double t = thickness;
            double width;

            width = A / (t * 1.378);

            return width;
        }
```

## Testing
Hier wird in ein paar S&auml;tzen das Testing beschrieben (oder auch nicht, wie ihr wollt).