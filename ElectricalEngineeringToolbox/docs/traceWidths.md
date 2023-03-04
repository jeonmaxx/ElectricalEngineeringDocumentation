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
Sie besteht aus einem string, welcher die aktuelle Unit beihaltet und einer Liste mit dem Parent Units.
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
Sie ben&ouml;tigt die Start und Ziel Unit, sowie den Start und Target Pr&auml;fix. 
Zudem muss auch ein Wert angegeben werden. 
Dann werden die Start und Ziel Unit an ConvertValue &uuml;bergeben. 
Der angegebene Wert wird dann mit 10^StartPr&auml;fix^ mal 10^ZielPr&auml;fix^ mal genommen und auch an ConvertValue &uuml;bergeben.
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
Zuerst wird eine Queue f&uuml;r die ==Openlist== und eine List f&uuml;r die ==Closedlist== angelegt. Noch dazu kommt ein HashSet f&uuml;r die bearbeiteten Units. 
Angefangen wird damit, dass der ==Openlist== die Start Unit hinzugef&uuml;gt wird. Die Parent Unit ist dabei nat&uuml;rlich Null, da die Start Unit in der Regel kein Parent hat. 
ProcessedUnits wird die Start Unit auch &uuml;bergeben. 
Solange _conversion nicht die oberste Unit der ==Openlist== und die Ziel Unit beinhaltet, bekommt currentItem die oberste Unit aus der ==Openlist==, welches gleichzeitig aus der Liste entfernt wird. 
Die Unit von currentItem kommt dann in die ==Closedlist==. 
F&uuml;r jedes Element in _conversion wo, x.from das currentItem ist, wird gerpr&uuml;ft ob die processedUnits y.to nicht beinhalten. 
Falls dies der Fall sein sollte, wird der ==Openlist== y.to und currentItem hinzugef&uuml;gt. ProcessedUnits wird auch y.to hizugef&uuml;gt.  
Danach wird der ==Closedlist== die oberste Unit der ==Openlist== hinzugef&uuml;gt und es wird der functionStack gemacht. 
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
                _functional =true;
            }
        }
```
Dabei wird zuerst das ==Dictionary tracebook== angelegt und eine Queue mit den noch ==unbearbeiteten Nodes (openNodes)==. 
Zum tracebook wird zuerst das erste Element 'from' von _conversion und Null hinzugef&uuml;gt. Bei OpenNodes wird dieses erste Element auch hinzugef&uuml;gt.  
Danach wird, solange etwas in openNodes drin ist, f&uuml;r jedes Element in _conversion wo x.from gleich das oberste Element von openNodes ist gepr&uuml;ft, ob tracebook y.to nicht beinhaltet. 
Wenn das der Fall sein sollte, dann wird y.to und eine Liste mit y.from dem tracebook hinzugef&uuml;gt. 
Ansonsten, wenn y.to vom tracebook nicht gleich Null ist, dann wird zu y.to vom tracebook y.from hinzugef&uuml;gt. 
Und wenn y.from vom tracebook nicht gleich Null ist, dann wird eine neue Queue mit dem Namen ==backtrace== angelegt. 
F&uuml;r jeden parent in backtrace wird &uuml;berpr&uuml;ft, ob dieser Null ist. Wenn dies nicht der Fall sein sollte, dann wird dieser parent dem tracebook hinzugef&uuml;gt. 
Danach wird y.from vom tracebook wieder auf Null gesetzt. 
Solange in backtrace noch Elemente drin sind, wird das oberste Element in backtrace gepr&uuml;ft ob der parent vom tracebook gleich Null ist. 
Sollte dies nicht der Fall sein, dann wird der parent dem backtrace hinzugef&uuml;gt. Danach werden das backtrace und die openNodes noch geleert.  
Zum Schluss wird noch gepr&uuml;ft, ob die Anzahl an Values aus tracebook, welche nicht gleich Null sind gr&ouml;&szlig;er als 0 ist 
oder ob das tracebook weniger Elemente hat als _conversion y.from Elemente hat. 
Wenn eine der beiden Bedingungen stimmen sollte, dann wird der Unit Tree nicht funktionieren. Ansonsten wird er funktionieren.

Wenn das **bool _functional** false sein sollte, dann wird der Baum gepr&uuml;ft um zu schauen ob es wirklich false ist. 
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
Um sicher zu stellen, dass alles funktioniert, haben wir mit Unit Tests verschiedene Methoden und Klassen getestet. 
Zum einen wurde der Calculator getestet, die Pr&auml;fixe wurden getestet und es wurden Tests mit ultimativen Zahlen gemacht. 
Zudem wurde, dass der Unit Tree funktioniert, bereits in der Klasse selbst &uuml;berpr&uuml;ft.