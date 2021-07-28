### Types
#### Table of contents 
> - [Integer](#MacroInteger)
> - [Float](#MacroFloat)
> - [String](#MacroString)
>   - [CommandText](#MacroCommandText)
>       - [Command](#MacroCommand)
>       - [Selector](#MacroSelector)
>           - [Entity](#MacroEntity)

#### Integer <a name="MacroInteger"></a>
##### Syntax
```
int i = 0;
Integer i = 10;
```
##### Definition
This represents a simple non-decimal number useful for things such as for loops.

These can be converted to scores and data values as needed.

#### Float <a name="MacroFloat"></a>
##### Syntax
```
float distance = .5;
Float threehalves = 1.5;
```
##### Definition
This represents a float designed to be used for float scores.

#### String <a name="MacroString"></a>
##### Syntax
```
string name1 = "John";
String name2 = "Doe";
```
##### Definition
This represents simple string of characters.

#### CommandText <a name="MacroCommandText"></a>
##### Syntax
```
CommandText hi = new CommandText("hi");
CommandText hi = commandText "hi";
CommandText hi = cmdtxt "hi";
```

##### Definition
Represents any part of a command

#### Command <a name="MacroCommand"></a>
##### Syntax
```
Command sayhi = new Command("say hi");
Command sayhi = command "say hi";
Command sayhi = cmd "say hi";
```
##### Definition
Represents a full command

#### Selector <a name="MacroSelector"></a>
##### Syntax
```
Selector self = new Selector("@s");
Selector self = selector "@s";
Selector self = sel "@s";
Selector self = @s;
```

#### Entity <a name="MacroEntity"></a>
##### Syntax
```
Entity self = new Entity(@s);
Entity self = entity @s;
Entity self = ent @s;
Entity self = @s;
```
##### Definition
Technically a selector, just with more abilities.

Use a [UUID](#DataUUID) to preserve selectors between ticks and entity sources. 

[abc](/../notes.md)