# Leiterbahnbreiten

## Einleitung

In diesem Teil wird die Benutzung, der ==Leiterbahnbreiten Tools== erkl&auml;rt. 
Um die verschiedenen Einheiten und Gr&ouml;&szlig;en zu verwalten wurde ein **Unit Tree** benutzt, damit eine gute Performance gew&auml;hrleistet werden kann. 
Desweiteren wird die **Calculator** Klasse benutzt, um die Ergebniseinheiten zu verwalten.

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
            Femto = -15,
            Atto = -18,
            Zepto = -21,
            Yokto = -24,
            Ronto = -27,
            Quekto = -30
        }
```
Danach beginnt auch schon der Unit Tree mit einer Klasse f&uuml;r die **Nodes**. 
Die Klasse besitzt zum einem einen String mit der Unit und auch der Parent Unit. 
Ein Node besteht dabei immer aus einer Unit und dessen Parent Unit.
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
Nach der Node Klasse, folgt die **BackTraceNode** Klasse. 
Diese ist vorallem wichtig, um zu &uuml;berpr&uuml;fen, ob der Unit Tree funktioniert. 
Sie besteht aus einem String, welcher die aktuelle Unit beinhaltet und einer Liste mit den Parent Units.
```c#
        private class BackTraceNode
        {
            public readonly string Unit;
            public List<Node> ParentUnits;

            public BackTraceNode(string unit)
            {
                Unit = unit;
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

Als n&auml;chstes folgt eine Methode, f&uuml;r die **Konvertierung der SI-Pr&auml;fixe**. 
Sie ben&ouml;tigt die Start und Ziel Unit sowie den Start und Target Pr&auml;fix. 
Zudem muss auch ein Wert angegeben werden. 
Dann werden die Start und Ziel Unit an ConvertValue &uuml;bergeben. 
Der angegebene Wert wird dann mit 10^StartPr&auml;fix^ und 10^ZielPr&auml;fix^ multipliziert und auch an ConvertValue &uuml;bergeben.
```c#
        public double ConvertSIPrefixValue(string startUnit, SIPrefix startPrefix, string targetUnit,
            SIPrefix targetPrefix, double value)
        {
            return ConvertValue(startUnit, targetUnit, value * Math.Pow(10.0, (double) startPrefix)) *
                   Math.Pow(10.0, (double) targetPrefix);
        }
```

Hier ist eine Methode, welche dann den **Wert konvertiert**. 
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
Zuerst wird gepr&uuml;ft, ob der Unit Tree auch funktioniert. 
Wenn die Start Unit gleich die Ziel Unit ist, dann wird der Wert direkt ausgegeben. 
Dann wird gepr&uuml;ft, ob die Start und Ziel Unit bereits in _conversion vorhanden sind. 
Danach wird ein functionStack gemacht, mit der Start und Ziel Unit. 
Solange dieser Stack gr&ouml;&szlig;er als 0 ist, wird das oberste Element vom Stack ausgegeben.  


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
```
Zuerst wird eine Queue f&uuml;r die ==Openlist== und eine List f&uuml;r die ==Closedlist== angelegt. Noch dazu kommt ein HashSet f&uuml;r die bearbeiteten Units (ProcessedUnits). 
Angefangen wird damit, dass der ==Openlist== die Start Unit hinzugef&uuml;gt wird. Die Parent Unit ist dabei nat&uuml;rlich Null, da die Start Unit in der Regel kein Parent hat. 
An ProcessedUnits wird die Start Unit auch &uuml;bergeben. 
Solange _conversion nicht die oberste Unit der ==Openlist== und die Ziel Unit beinhaltet, bekommt currentItem die oberste Unit aus der ==Openlist==, welches gleichzeitig aus der Liste entfernt wird. 
Die Unit von currentItem kommt dann in die ==Closedlist==. 
F&uuml;r jedes Element in _conversion wo, x.from das currentItem ist, wird gepr&uuml;ft, ob die processedUnits y.to nicht beinhalten. 
Falls dies der Fall sein sollte, wird der ==Openlist== y.to und currentItem hinzugef&uuml;gt. ProcessedUnits wird auch y.to hizugef&uuml;gt.  
Danach wird der ==Closedlist== die oberste Unit der ==Openlist== hinzugef&uuml;gt und es wird der functionStack erstellt. 
Zudem kommt eine neue Node, die currentNode, welche die Ziel Unit und das oberste Element der ==Openlist== beinhaltet. 
Solange die Parent Unit der currentNode nicht Null ist, wird die currentNode Unit ausgegeben. 
Der functionStack bekommt oben drauf noch _conversion mit der Parent Unit von currentNode und die Unit von currentNode. 
Danach bekommt die neue currentNode die Unit von der vorherigen Parent Unit zugewiesen. 
Zum Schluss wird noch der functionStack zur&uuml;ckgegeben.  

Bei **AddPath** werden Pfade dem Unit Tree hinzugef&uuml;gt. Solange dies passiert, fuktioniert der Unit Tree nicht, weshalb _functional auch false ist. 
Hierbei wird _conversion die Start und Ziel Unit und conversionFunc &uuml;bergeben.
```c#
        public void AddPath(string startUnit, string targetUnit, ConversionFunction conversionFunc)
        {
            _functional = false;
            _conversion.TryAdd((startUnit,targetUnit), conversionFunc);
        }
```

Zum Schluss wird noch &uuml;berpr&uuml;ft, ob der **Unit Tree funktioniert**, damit m&ouml;gliche Fehler fr&uuml;hzeitig erkannt werden.
```c#
        private void checkFunctionality()
        {
            Dictionary<string,List<string>?> traceBook= new Dictionary<string,List<string>>();
            Queue<string> openNodes = new Queue<string>();
            traceBook.Add(_conversion.First().Key.from,null);
            openNodes.Enqueue(_conversion.First().Key.from);
            do
            {
                foreach (var y in _conversion.Keys.Where(x => x.from == openNodes.Peek()))
                {
                    if (!traceBook.ContainsKey(y.to))
                    {
                        traceBook.Add(y.to,new List<string>(){y.from});
                        openNodes.Enqueue(y.to);
                    }
                    else if (traceBook[y.to] != null)
                    {
                        traceBook[y.to].Add(y.from);
                    }
                    else if (traceBook[y.from] != null)
                    {
                        Queue<string> backTrace = new Queue<string>();
                        foreach (string parent in traceBook[y.from])
                        {
                            if (traceBook[parent] != null)
                                backTrace.Enqueue(parent);
                        }
                        traceBook[y.from] = null;

                        while (backTrace.Count > 0)
                        {
                            foreach (string parent in traceBook[backTrace.Peek()])
                            {
                                if (traceBook[parent] != null)
                                    backTrace.Enqueue(parent);
                            }
                            traceBook[backTrace.Dequeue()] = null;
                        }
                    }
                }
                openNodes.Dequeue();

            } while (openNodes.Count > 0);


            if (traceBook.Count(x => x.Value != null) > 0 || 
                traceBook.Count < _conversion.DistinctBy(y => y.Key.from).Count())
            {
                _functional = false;
            }
            else
            {
                _functional = true;
            }
        }
```
Dabei wird zuerst das ==Dictionary tracebook== und eine Queue mit den noch ==unbearbeiteten Nodes (openNodes)== angelegt. 
Zum tracebook wird zuerst das erste Element 'from' von _conversion und Null hinzugef&uuml;gt. Bei OpenNodes wird dieses erste Element auch hinzugef&uuml;gt.  
Danach wird, solange etwas in openNodes vorhanden ist, f&uuml;r jedes Element in _conversion wo x.from gleich das oberste Element von openNodes gepr&uuml;ft, ob tracebook y.to nicht beinhaltet. 
Wenn das der Fall sein sollte, dann wird y.to und eine Liste mit y.from dem tracebook hinzugef&uuml;gt. 
Ansonsten, wenn y.to vom tracebook nicht gleich Null ist, wird zu y.to vom tracebook y.from hinzugef&uuml;gt. 
Und wenn y.from vom tracebook nicht gleich Null ist, dann wird eine neue Queue mit dem Namen ==backtrace== angelegt. 
F&uuml;r jeden parent in backtrace wird &uuml;berpr&uuml;ft, ob dieser Null ist. Wenn dies nicht der Fall sein sollte, dann wird dieser parent dem tracebook hinzugef&uuml;gt. 
Danach wird y.from vom tracebook wieder auf Null gesetzt. 
Solange in backtrace noch Elemente vorhanden sind, wird vom obersten Element in backtrace gepr&uuml;ft ob der parent vom tracebook gleich Null ist. 
Sollte dies nicht der Fall sein, dann wird der parent dem backtrace hinzugef&uuml;gt. Danach werden das backtrace und die openNodes noch geleert.  
Zum Schluss wird noch gepr&uuml;ft, ob die Anzahl an Values aus dem tracebook, welche nicht gleich Null sind gr&ouml;&szlig;er als 0 sind 
oder ob das tracebook weniger Elemente als _conversion y.from Elemente hat. 
Wenn eine der beiden Bedingungen stimmen sollte, dann wird der Unit Tree nicht funktionieren.

Wenn das **bool _functional** false sein sollte, dann wird der Unit Tree gepr&uuml;ft, um zu schauen, ob es wirklich false ist. 
Wenn das bool dann immer noch false sein sollte, dann wird einem in der Konsole deutlich gezeigt, dass etwas nicht ganz funktioniert.
```c#
        public bool IsFunctional()
        {
            if (!_functional)
            {
                checkFunctionality();
            }
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
Au&szlig;erdem gibt es eine Klasse, die die einzelnen Einheiten der Ergebnisse vom Kalkulator verwaltet.  
Dazu geh&ouml;ren sowohl die **ben&ouml;tigte Fl&auml;che** der inneren Schicht als auch der &auml;u&szlig;eren Schicht.
```c#
        public double CalculateAreaIntern(double ampere, double tRise)
        {
            double I = ampere;
            double TRise = tRise;
            const double constantK = 0.024;
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

Und nat&uuml;rlich auch f&uuml;r die **Breite der Leiterbahnen**. Einmal wird dabei mit Mil gerechnet und einmal mit Oz/Ft^2^.
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
            return CalculateWidthMil(area, thickness * 1.378f);

        }
```

Hier wird dann direkt &uuml;berpr&uuml;ft, ob der eingegebene Wert verwendet werden kann oder ob dieser nicht erlaubt ist.
```c#
        public bool CheckInputValue(double ampereValue, double temperatureValue, double ThicknessValue, double traceLengthValue, double temperaturRiseValue)
        {
            double[] InputValue = { ampereValue, temperatureValue, ThicknessValue, traceLengthValue, temperaturRiseValue };

            if(InputValue.Any(x => x <= 0))
            {
                return false;
            }
            return true;
        }
```
Als n&auml;chstes wird auch &uuml;berpr&uuml;ft, ob die eingegebene Unit zul&auml;ssig ist. 
Falls dies nicht der Fall sein sollte, dann gibt das Bool false aus.
```c#
        public bool CheckInputUnit(double targetUnitTemperature, double targetUnitThickness, double targetUnitTraceLength, double targetUnitTemperaturRise, double targetUnitWidthIntern, double targetUnitWidthExtern)
        {
            int tempResult = 0;
            int tempRiseResult = 0;
            int thickResult = 0;
            int traceLenghtResult = 0;
            int widthInternResult = 0;
            int widthExternResult = 0;

            if (targetUnitTemperature > 0 && targetUnitThickness < 2 && targetUnitTemperaturRise > 0 && targetUnitTemperaturRise < 2)
            {
                tempResult = 1;
                tempRiseResult = 1;
            }
            if (targetUnitThickness > 0 && targetUnitThickness < 4)
            {
                thickResult = 1;
            }
            if(targetUnitTraceLength > 0 && targetUnitTraceLength < 7)
            {
                traceLenghtResult = 1;
            }
            if(targetUnitWidthIntern > 0 && targetUnitWidthIntern < 3 && targetUnitWidthExtern > 0 && targetUnitWidthExtern < 3)
            {
                widthInternResult = 1;
                widthExternResult = 1;
            }

            double[] UnitCheck = { tempResult, tempRiseResult, thickResult, traceLenghtResult, widthInternResult, widthExternResult };

            if (UnitCheck.Any(x => x < 1))
            {
                return false;
            }
            return true;
        }
```

## Testing
Um sicher zu stellen, dass alles funktioniert, haben wir mit Unit Tests verschiedene Methoden und Klassen getestet. 
Zum einen wurden der Kalkulator sowie die Pr&auml;fixe getestet und es wurden Tests mit verschiedenen Zahlen gemacht. 
Zudem wird zus&auml;tzlich &uuml;berpr&uuml;ft, ob der Unit Tree funktioniert, da dieser eins der wichtigsten Elemente der Klasse ist.  

**Calculator Test**  
Um den Unit Tree zu testen gibt es viele Funktionen, um sicher zu stellen, dass keine Fehler auftauchen k&ouml;nnen.  
Zuerst wird &uuml;berpr&uuml;ft, ob die Konvertierung der Einheiten funktioniert. 
Dabei werden alle Einheiten benutzt, um alle M&ouml;glichkeiten zu &uuml;berpr&uuml;fen.
```c#
        [TestMethod]
        public void CheckSelfConversion()
        {
            UnitTree Converter = new UnitTree();

            Converter.AddPath("cm", "mym", x => x * 10000);
            Converter.AddPath("cm", "mm", x => x * 10.0);
            Converter.AddPath("cm", "m", x => x / 100.0);
            Converter.AddPath("cm", "in", x => x / 2.54);
            Converter.AddPath("cm", "ft", x => x / 30.48);
            Converter.AddPath("mym", "cm", x => x / 10000);
            Converter.AddPath("m", "cm", x => x * 100.0);
            Converter.AddPath("mm", "cm", x => x / 10.0);
            Converter.AddPath("in", "cm", x => x * 2.54);
            Converter.AddPath("ft", "cm", x => x * 30.48);
            Converter.AddPath("mil", "in", x => x / 1000);
            Converter.AddPath("in", "mil", x => x * 1000);
            Converter.AddPath("oz/ft^2", "mil", x => x / 1.37f);
            Converter.AddPath("mil", "oz/ft^2", x => x * 1.37f);
            Converter.AddPath("mm", "mil", x => x * 39.37f);
            Converter.AddPath("mil", "mm", x => x / 39.37f);
            Converter.AddPath("mm", "mym", x => x * 1000f);
            Converter.AddPath("mym", "mm", x => x / 1000f);
            Converter.AddPath("F", "C", x => (x - 32) * 5 / 9.0);
            Converter.AddPath("C", "F", x => (x * 1.8 + 32));

            string[] StartUnits = new string[] { "mym", "mm", "cm", "m", "in", "ft", "mil","oz/ft^2","F","C" };
            string[] EndUnits = new string[] { "mym", "mm", "cm", "m", "in", "ft", "mil","oz/ft^2","F", "C" };

            for (int i = 0; i < StartUnits.Length; i++)
            {
                double number = Converter.ConvertValue(StartUnits[i], EndUnits[i], 1);
                Assert.AreEqual(1, number);
            }
        }
```
Hier werden dann die Werte der Konvertierungen &uuml;berpr&uuml;ft. 
Wenn ein Ergebnis nicht dem vorgesehenen Wert entspricht, dann schl&auml;gt der Test fehl.
```c#
        [TestMethod]
        public void CheckLengthConversion()
        {
            UnitTree Converter = new UnitTree();

            Converter.AddPath("cm", "mym", x => x * 10000);
            Converter.AddPath("cm", "mm", x => x * 10.0);
            Converter.AddPath("cm", "m", x => x / 100.0);
            Converter.AddPath("cm", "in", x => x / 2.54);
            Converter.AddPath("cm", "ft", x => x / 30.48);
            Converter.AddPath("mym", "cm", x => x / 10000);
            Converter.AddPath("m", "cm", x => x * 100.0);
            Converter.AddPath("mm", "cm", x => x / 10.0);
            Converter.AddPath("in", "cm", x => x * 2.54);
            Converter.AddPath("ft", "cm", x => x * 30.48);
            Converter.AddPath("mil", "in", x => x / 1000);
            Converter.AddPath("in", "mil", x => x * 1000);


            double conversion1 = Converter.ConvertValue("mm", "mym", 1);
            Assert.AreEqual(1000, conversion1);
            double conversion2 = Converter.ConvertValue("mm", "cm", 1);
            Assert.AreEqual(0.1, conversion2);
            double conversion3 = Converter.ConvertValue("mm", "m", 1);
            Assert.AreEqual(0.001, conversion3);
            double conversion4 = Converter.ConvertValue("mm", "in", 1);
            Assert.AreEqual(0.03937007874015748, conversion4);
            double conversion5 = Converter.ConvertValue("mm", "ft", 1);
            Assert.AreEqual(0.0032808398950131233, conversion5);
            double conversion6 = Converter.ConvertValue("mm", "mil", 1);
            Assert.AreEqual(39.37007874015748, conversion6);

            double conversion7 = Converter.ConvertValue("cm", "mm", 1);
            Assert.AreEqual(10, conversion7);
            double conversion8 = Converter.ConvertValue("cm", "mym", 1);
            Assert.AreEqual(10000, conversion8);
            double conversion9 = Converter.ConvertValue("cm", "m", 1);
            Assert.AreEqual(0.01, conversion9);
            double conversion10 = Converter.ConvertValue("cm", "in", 1);
            Assert.AreEqual(0.39370078740157477, conversion10);
            double conversion11 = Converter.ConvertValue("cm", "ft", 1);
            Assert.AreEqual(0.03280839895013123, conversion11);
            double conversion12 = Converter.ConvertValue("cm", "mil", 1);
            Assert.AreEqual(393.70078740157476, conversion12);

            double conversion13 = Converter.ConvertValue("m", "mm", 1);
            Assert.AreEqual(1000, conversion13);
            double conversion14 = Converter.ConvertValue("m", "cm", 1);
            Assert.AreEqual(100, conversion14);
            double conversion15 = Converter.ConvertValue("m", "mym", 1);
            Assert.AreEqual(1e+6, conversion15);
            double conversion16 = Converter.ConvertValue("m", "in", 1);
            Assert.AreEqual(39.37007874015748, conversion16);
            double conversion17 = Converter.ConvertValue("m", "ft", 1);
            Assert.AreEqual(3.2808398950131235, conversion17);
            double conversion18 = Converter.ConvertValue("m", "mil", 1);
            Assert.AreEqual(39370.07874015748, conversion18);
            
            double conversion19 = Converter.ConvertValue("in", "mm", 1);
            Assert.AreEqual(25.4, conversion19);
            double conversion20 = Converter.ConvertValue("in", "cm", 1);
            Assert.AreEqual(2.54, conversion20);
            double conversion21 = Converter.ConvertValue("in", "m", 1);
            Assert.AreEqual(0.0254, conversion21);
            double conversion22 = Converter.ConvertValue("in", "mym", 1);
            Assert.AreEqual(25400, conversion22);
            double conversion23 = Converter.ConvertValue("in", "ft", 1);
            Assert.AreEqual(0.08333333333333333, conversion23);
            double conversion24 = Converter.ConvertValue("in", "mil", 1);
            Assert.AreEqual(1000, conversion24);

            double conversion25 = Converter.ConvertValue("ft", "mm", 1);
            Assert.AreEqual(304.8, conversion25);
            double conversion26 = Converter.ConvertValue("ft", "cm", 1);
            Assert.AreEqual(30.48, conversion26);
            double conversion27 = Converter.ConvertValue("ft", "m", 1);
            Assert.AreEqual(0.3048, conversion27);
            double conversion28 = Converter.ConvertValue("ft", "in", 1);
            Assert.AreEqual(12, conversion28);
            double conversion29 = Converter.ConvertValue("ft", "mym", 1);
            Assert.AreEqual(304800, conversion29);
            double conversion30 = Converter.ConvertValue("ft", "mil", 1);
            Assert.AreEqual(12000, conversion30);

            double conversion31 = Converter.ConvertValue("mil", "mm", 1);
            Assert.AreEqual(0.025400000000000002, conversion31);
            double conversion32 = Converter.ConvertValue("mil", "cm", 1);
            Assert.AreEqual(0.00254, conversion32);
            double conversion33 = Converter.ConvertValue("mil", "m", 1);
            Assert.AreEqual(2.54e-5, conversion33);
            double conversion34 = Converter.ConvertValue("mil", "in", 1);
            Assert.AreEqual(0.001, conversion34);
            double conversion35 = Converter.ConvertValue("mil", "ft", 1);
            Assert.AreEqual(8.333333333333333E-05, conversion35);
            double conversion36 = Converter.ConvertValue("mil", "mym", 1);
            Assert.AreEqual(25.400000000000002, conversion36);
        }
```
Im Folgenden werden die einzelnen Konvertierungen dann nochmal genauer &uuml;berpr&uuml;ft, angefangen bei der Temperatur.
```c#
        [TestMethod]
        public void CheckTemperatureConversion()
        {
            UnitTree Converter = new UnitTree();
            Converter.AddPath("F", "C", x => (x - 32) * 5 / 9.0);
            Converter.AddPath("C", "F", x => (x * 1.8 + 32));

            double conversion1 = Converter.ConvertValue("F", "C", 25);
            Assert.AreEqual(-3.888888888888889, conversion1);
            double conversion2 = Converter.ConvertValue("C", "F", 25);
            Assert.AreEqual(77, conversion2);
        }
```
Danach kommt die Konvertierung der Dicke.
```c#

        [TestMethod]
        public void CheckThicknessConversion()
        {
            UnitTree Converter = new UnitTree();

            Converter.AddPath("oz/ft^2", "mil", x => x / 1.37f);
            Converter.AddPath("mil", "oz/ft^2", x => x * 1.37f);
            Converter.AddPath("mm", "mil", x => x * 39.37f);
            Converter.AddPath("mil", "mm", x => x / 39.37f);
            Converter.AddPath("mm", "mym", x => x * 1000f);
            Converter.AddPath("mym", "mm", x => x / 1000f);

            double conversion1 = Converter.ConvertValue("oz/ft^2", "mil", 1);
            Assert.AreEqual(0.729927004758713, conversion1);
            double conversion2 = Converter.ConvertValue("oz/ft^2", "mm", 1);
            Assert.AreEqual(0.01854018350423585, conversion2);
            double conversion3 = Converter.ConvertValue("oz/ft^2", "mym", 1);
            Assert.AreEqual(18.54018350423585, conversion3);

            double conversion4 = Converter.ConvertValue("mil", "oz/ft^2", 1);
            Assert.AreEqual(1.3700000047683716, conversion4);
            double conversion5 = Converter.ConvertValue("mil", "mm", 1);
            Assert.AreEqual(0.0254000514892096, conversion5);
            double conversion6 = Converter.ConvertValue("mil", "mym", 1);
            Assert.AreEqual(25.4000514892096, conversion6);

            double conversion7 = Converter.ConvertValue("mm", "oz/ft^2", 1);
            Assert.AreEqual(53.93689872441291, conversion7);
            double conversion8 = Converter.ConvertValue("mm", "mym", 1);
            Assert.AreEqual(1000, conversion8);
            double conversion9 = Converter.ConvertValue("mm", "mil", 1);
            Assert.AreEqual(39.369998931884766, conversion9);

            double conversion10 = Converter.ConvertValue("mym", "mil", 1);
            Assert.AreEqual(0.03936999893188477, conversion10);
            double conversion11 = Converter.ConvertValue("mym", "mm", 1);
            Assert.AreEqual(0.001, conversion11);
            double conversion12 = Converter.ConvertValue("mym", "oz/ft^2", 1);
            Assert.AreEqual(0.053936898724412916, conversion12);
        }
```
Als n&auml;chstes wird die Calculator Klasse getestet. 
Dabei werden die Ergebnisse der Methoden mit einem erwarteten Wert verglichen und so &uuml;berpr&uuml;ft, ob bei den Rechungen das Richtige rauskommt.
```c#
        [TestMethod]
        public void CheckLayerCalculation()
        {
            Calculator Calculator = new Calculator();

            double areaIntern = Calculator.CalculateAreaIntern(1, 10);
            double areaExtern = Calculator.CalculateAreaExtern(1, 10);

            Assert.AreEqual(42.39306714892737, areaIntern);
            Assert.AreEqual(16.296001347209486, areaExtern);

            double widthInternMil = Calculator.CalculateWidthMil(areaIntern, 1);
            double widthInternOzFt = Calculator.CalculateWidthOzFt(areaIntern, 1);

            Assert.AreEqual(42.39306714892737, widthInternMil);
            Assert.AreEqual(30.764199204258915, widthInternOzFt);

            double widthExternMil = Calculator.CalculateWidthMil(areaExtern, 1);
            double widthExternOzFt = Calculator.CalculateWidthOzFt(areaExtern, 1);

            Assert.AreEqual(16.296001347209486, widthExternMil);
            Assert.AreEqual(11.825835340416246, widthExternOzFt);
        }
```
Hier wird getestet, ob die Klasse mit negativen Zahlen rechnen w&uuml;rde, da mit diesen kein korrektes Ergebnis geliefert werden k&ouml;nnte.
```c#
        [TestMethod]
        public void CheckInputValueCheck()
        {
            Calculator Calculator = new Calculator();

            Assert.IsTrue(Calculator.CheckInputValue(1, 1, 1, 1, 1));
            Assert.IsFalse(Calculator.CheckInputValue(-1, 1, 1, 1, 1));
            Assert.IsFalse(Calculator.CheckInputValue(1, -1, 1, 1, 1));
            Assert.IsFalse(Calculator.CheckInputValue(1, 1, -1, 1, 1));
            Assert.IsFalse(Calculator.CheckInputValue(1, 1, 1, -1, 1));
            Assert.IsFalse(Calculator.CheckInputValue(1, 1, 1, 1, -1));
        }
```
Als n&auml;chstes wird geschaut was passiert, wenn man 0 als Wert angibt und ob die Klasse mit diesem Wert auch nicht weiter rechnet.
```c#
        [TestMethod]
        public void CheckInputUnitCheck()
        {
           // Exception WrongInput;

            Calculator Calculator = new Calculator();
            Assert.IsTrue(Calculator.CheckInputUnit(1,1,1,1,1,1));
            Assert.IsFalse(Calculator.CheckInputUnit(0,1,1,1,1,1));
            Assert.IsFalse(Calculator.CheckInputUnit(1,0,1,1,1,1));
            Assert.IsFalse(Calculator.CheckInputUnit(1,1,0,1,1,1));
            Assert.IsFalse(Calculator.CheckInputUnit(1,1,1,0,1,1));
            Assert.IsFalse(Calculator.CheckInputUnit(1,1,1,1,0,1));
            Assert.IsFalse(Calculator.CheckInputUnit(1,1,1,1,1,0));
        }
```
In dem n&auml;chsten Test wird &uuml;berpr&uuml;ft, ob der Unit Tree so funktioniert, wie er soll und keine L&uuml;cken aufweist. 
Es wird getestet, was passiert, wenn der Unit Tree keinen Rundlauf hat, wenn es eine schwebene Verbindung gibt und wenn er theoretisch funktionieren sollte. 
Sind die ersten beiden Test falsch und der letzte richtig, dann funktioniert der Unit Tree wie gedacht.
```c#
        [TestMethod]
        public void CheckFunctionality()
        {
            //kein Rundlauf
            UnitTree unitTree = new UnitTree();
            unitTree.AddPath("m", "cm", x => x * 100.0); 
            Assert.IsFalse(unitTree.IsFunctional());

            //schwebende Verbindung
            UnitTree unitTree2 = new UnitTree();
            unitTree2.AddPath("m", "cm", x => x * 100.0);
            unitTree2.AddPath("cm", "m", x => x / 100);
            unitTree2.AddPath("mm", "dm", x => x / 100);
            Assert.IsFalse(unitTree2.IsFunctional());

            //funktionierender Tree
            UnitTree unitTree3 = new UnitTree();
            unitTree3.AddPath("m", "cm", x => x * 100.0);
            unitTree3.AddPath("cm", "m", x => x / 100);
            unitTree3.AddPath("cm", "mm", x => x / 10);
            unitTree3.AddPath("mm", "dm", x => x / 100);
            unitTree3.AddPath("dm", "m", x => x / 10);
            Assert.IsTrue(unitTree3.IsFunctional());
        }
```
In diesem Test werden die Pr&auml;fixe &uuml;berpr&uuml;ft, um zu wissen, ob ihre Werte stimmen.
```c#
        [TestMethod]
        public void CheckPrefixValue()
        {
            SIPrefix[] s = new SIPrefix[] {SIPrefix.Quekto, SIPrefix.Ronto, SIPrefix.Yokto, SIPrefix.Zepto, SIPrefix.Atto
                                            , SIPrefix.Femto, SIPrefix.Piko, SIPrefix.Nani, SIPrefix.Mikro, SIPrefix.Milli, SIPrefix.SIDefault,
                                            SIPrefix.Kilo, SIPrefix.Mega, SIPrefix.Giga, SIPrefix.Tera, SIPrefix.Peta, SIPrefix.Exa, SIPrefix.Zetta
                                            , SIPrefix.Yotta, SIPrefix.Ronna, SIPrefix.Quetta};

            int j = -30;

            foreach(SIPrefix i in s)
            {
                Assert.AreEqual(j,((int)i));
                j += 3;
            }

            SIPrefix[] k = new SIPrefix[] { SIPrefix.Hekto, SIPrefix.Deka, SIPrefix.SIDefault, SIPrefix.Dezi, SIPrefix.Zenti };

            int l = 2;

            foreach(SIPrefix m in k)
            {
                Assert.AreEqual(l, (int)m);
                l -= 1;
            }
        }
```
Es wird auch getestet, ob die Konvertierung der Pr&auml;fixe fehlerfrei funktioniert, damit sich keine Fehler unbemerkt durch das Programm ziehen.
```c#
        [TestMethod]
        public void CheckConvertSIPrefix()
        {
            UnitTree unitTree = new UnitTree();
            SIPrefix[] positiv = new SIPrefix[] {SIPrefix.Kilo, SIPrefix.Mega, 
                                                    SIPrefix.Giga, SIPrefix.Tera, SIPrefix.Peta, SIPrefix.Exa, SIPrefix.Zetta
                                                , SIPrefix.Yotta, SIPrefix.Ronna, SIPrefix.Quetta};

            SIPrefix[] negative = new SIPrefix[] { SIPrefix.Milli, SIPrefix.Mikro, SIPrefix.Nani, SIPrefix.Piko, SIPrefix.Femto,
                                                     SIPrefix.Atto,  SIPrefix.Zepto, SIPrefix.Yokto, SIPrefix.Ronto, SIPrefix.Quekto, };

            SIPrefix[] positiveClose = new SIPrefix[] { SIPrefix.SIDefault, SIPrefix.Deka, SIPrefix.Hekto };

            SIPrefix[] negativeClose = new SIPrefix[] { SIPrefix.SIDefault, SIPrefix.Dezi, SIPrefix.Zenti };

            double k = 1e+3;
            double l = 1e-3;
            double n = 1e+0;
            double m = 1e-0;
            
            foreach (SIPrefix i in positiv)
            {
                 Assert.AreEqual(k, unitTree.ConvertSIPrefixValue("m", SIPrefix.SIDefault, "m", i, 1));
                 k *= 1000;
            }

            foreach (SIPrefix i in negative)
            {
                Assert.AreEqual(l, unitTree.ConvertSIPrefixValue("m", SIPrefix.SIDefault, "m", i, 1), l * 0.001);
                l /= 1000;
            }

            foreach (SIPrefix i in positiveClose)
            {
                Assert.AreEqual(n, unitTree.ConvertSIPrefixValue("m", SIPrefix.SIDefault, "m", i, 1));
                n *= 10;
            }

            foreach (SIPrefix i in negativeClose)
            {
                Assert.AreEqual(m, unitTree.ConvertSIPrefixValue("m", SIPrefix.SIDefault, "m", i, 1));
                m /= 10;
            }
        }        
```