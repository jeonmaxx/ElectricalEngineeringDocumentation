# Leiterbahnbreiten

## Einleitung

In diesem Teil wird die Benutzung, der Leiterbahnbreiten Tools erklaert.

## MasterUnit

```c#
        public abstract class MasterUnit
        {
            public string CurrentUnit;
            public double value;
        }
```

## UnitTree
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

```c#
        public UnitTree()
        {
            _functional = true;
        }

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

```c#
        public double ConvertSIPrefixValue(string startUnit, SIPrefix startPrefix, string targetUnit,
            SIPrefix targetPrefix, double value)
        {
            return ConvertValue(startUnit, targetUnit, value * Math.Pow(10.0, (double) startPrefix)) *
                   Math.Pow(10.0, (double) targetPrefix);
        }
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

```c#
        public void AddPath(string startUnit, string targetUnit, ConversionFunction conversionFunc)
        {
            _functional = false;
            _conversion.TryAdd((startUnit,targetUnit), conversionFunc);
        }

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
```c#
        public Calculator() { }

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