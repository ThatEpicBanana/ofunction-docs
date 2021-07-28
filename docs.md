# Variables
## Data
### Syntax
```
data <type> <name> [in ...] = x;
    ... entity [<selector>] <path> = x;
    ... block [<position>] <path> = x;
    ... storage <identifier> <path> = x;
```
Examples:
```
data int detail = 0;
data UUID uuid in entity @s UUID;
data compound inventory in entity @s Inventory = [];
data string callback in storage test:ray settings.callback;
```
### What is it?
Data values are varaibles that are stored either in an entity's, block's, or storage's nbt. They support a wide range of types of variables such as strings, numbers, objects, and lists. 

It is written to using other data paths / variables or [SNBT](https://minecraft.fandom.com/wiki/NBT_format#SNBT_format) which is partly explained below. 

The path can also be specified by using an [NBT Path](https://minecraft.fandom.com/wiki/NBT_path_format) which is also explained below.

ex: `data UUID uuid in entity @s UUID;`

### Types
#### Numbers
- Byte
    - A signed 8-bit integer, ranging from -128 to 127 (inclusive). 
    - Defined with a "b" &nbsp; ex: `1b`
- Short
    - A signed 16-bit integer, ranging from -32,768 to 32,767 (inclusive). 
    - Defined with an "s" &nbsp; ex: `30000s`
- Integer
    - A signed 32-bit integer, ranging from -2,147,483,648 and 2,147,483,647 (inclusive). 
    - Defined with no letter &nbsp; ex: `31415926`
- Long
    - A signed 64-bit integer, ranging from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 (inclusive).
    - Defined with an "l" &nbsp; ex: `31415926l`
- Float
    - A 32-bit, single-precision floating-point number, ranging from -3.4E+38 to +3.4E+38. 
    - Definied with an "f" &nbsp; ex: `3.1415926f`
    - [This video](https://youtu.be/p8u_k2LIZyo?t=258) has a very nice explanation as to how these work
- Double
    - A 64-bit, double-precision floating-point, ranging from -1.7E+308 to +1.7E+308. 
    - Definied with no letter &nbsp; ex: `3.1415926`

All of these extend a base number class and will be converted to scores for arithmetic.
> **Note:** Floats and Doubles will be floored when converting to a score
#### Strings
Defined by text surrounded by either single quotes or double quotes

ex: `"text"` `'text'`
#### Compounds
Definied using [SNBT](https://minecraft.fandom.com/wiki/NBT_format#SNBT_format), attributes can be accessed through [dot notation](#DotNotation) or [bracket notation](#BracketNotation).

ex: `{id:"minecraft:tnt",Slot:1b,Count:1b}`
##### Dot notation <a name="DotNotation"></a>
`variable.attribute`
##### Bracket notation <a name="BracketNotation"></a>
`variable['attribute']` `variable["attribute"]`
##### Function calling
Functions in compounds can be used as normal 

ex: `variable.attribute.attributeFunction()`
#### Lists
Definied with brackets containing the items of the list seperated with commas

ex: `["a","b","c"]`

Can be accessed by providing the index in brackets

ex: `variable[i]`
##### With compounds
Can also check for a compund that matches set SNBT. For instance while dealing with inventories.

ex: `Inventory[{Slot:1b}]`
##### Special number arrays
There are three types of arrays for different types of numbers
- A byte array (Denoted by a B)
- An integer array (Denoted by an I)
- A long array (Denoted by a L)

These are notated by a list with the type's letter and a semicolon at the start

ex: `[B;1b,2b,3b]` `[I;1,2,3]` `[L;1l,2l,3l]`
#### UUIDs <a name="DataUUID"></a>
Technically integer lists under the hood, you can access parts of the UUID by using the syntax for lists.

ex: `playerUUID[0]`

See more in the entity section
#### Booleans
ex: `true` `false`

These will be converted into a byte tag by the game.

## Scores
### Syntax
```
score [<type>] [<precision> | float] <name> [in ...] = x;
    ... <scoreboard> [<score holder>] = x;
```
**Examples:**
```
score distance = 0;
score minecraft.used:minecraft.carrot_on_a_stick uses = 0;
score detail in raycast detail;
score trigger toggle in togglesetting @s = 0;
```
### What are they?
Scoreboards are a way to store custom integers, [fractions](#ScorePrecision), or [floats](#ScoreFloat) per entity. They can also store values for "fake players" by just inputting a name.

[Here's](https://youtu.be/EkJiiICCK-s) a good video that explains these. 

These can use arithmetic natively without casting to a different type. 

### What's that precision thing? <a name="ScorePrecision"></a>
The precision number allows the use of fractions with scores, but does not allow for multiplying or dividing. 

It does this by using the precision number kinda like a denominator for the stored number, inflating it.

ex: 
 - The value of 1 with the precision of 2 would store as 2
 - The value of 1/2 with the precision of 2 would store as 1
 - The value of 5 with the precision of 5 would store as 25

#### Syntax
```
score [<type>] <precision> <name> [in ...] = x;
    ... <scoreboard> [<score holder>] = x;
```

Examples:

```
score dummy 16 step = 1/16;
score 2 threehalves = 1.5;
```

### How do floats work? <a name="ScoreFloat"></a>
Floats are achieved through a heavily modified version of [this library by emeraldfyr3](https://www.youtube.com/watch?v=e6OrClOPO_M) and allow for decimal numbers in scoreboards. 

These support full arithmetic.

#### Syntax
```
score [<type>] float <name> [in ...] = x;
    ... <scoreboard> [<score holder>] = x;
```

**Examples:**

```
score dummy float step = 1/16;
score float pi = 3.14159265359;
```

## Macro Variables <a name="Macro"></a>
### Syntax
```
<type> <name> = x;
```
**Examples:**
```
int i = 0;
Selector self = @s;
CommandText command = "say hi";
```
### Definition
"Macro variables" are all variables that do not fit under data values or scores. This means that they only exist during compilation, and allow you to [do repetitive stuff easier]() and [edit functions]() among other things.

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

#### 

## General keywords
### Full variable syntax
```
[static] [persistent] ...
    ... data <type> <name> [in ...] = x;
        ... entity [<selector>] <path> = x;
        ... block [<position>] <path> = x;
        ... storage <identifier> <path> = x;
    ... score [<type>] [<precision> | float] <name> [in ...] = x;
        ... <scoreboard> [<score holder>] = x;
```
### Persistent
Persistent tells the compiler not to delete the variable after it is used.
### Static
Static, used in classes, makes the variable a global variable instead of pertaining to a specific instance of the class.

## Storage
### Local vs Global
So you might notice that in data and scores, there are usually global (storage in data, fake players in scores) and local (entities / blocks in data, entities in scores) types of storage. 

Generally, local variables are only used in objects and structs without the static keyword, and global variables are always used in functions (even when in an object.)
### Garbage collection
All global variables are garbage collected after use unless:
 - The variable is marked as [Persistent]()
 - The variable is marked as [Static]()
 - The variable is in a [Module]()
 - The variable is in another non-temporary variable
 - The variable is actually global (specified on load)


