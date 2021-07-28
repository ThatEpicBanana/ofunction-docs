# variable redesign:

so blank slate here. 

I'd like the ability to add variables with paths that are guessed.
There's two types of "guessed" variables:
 - the local ones
    - scores stored per entity (@s)
    - data paths on the entity or block
 - and the global ones
    - scores with a named entity 
    - scores in storage

However, "global" ones can be used in times when data does not need to be preserved between ticks

## global scoreboards:
### why not?:
You might ask, why not just have one "ofunction" scoreboard with a bunch of values?
Well, player names have a 40 character limit, which could introduce problems.
It would also be really memory intensive
### why yes?:
Temporary ones wouldn't work because how will we know which variables a function that is calling this one would use?
I think the best way to do it *would* be to have one scoreboard, except with some sort of garbage collection
### potential problem:
this could mess up with the obliteration of functions with only two lines of functions
if sent out to multiple entities, they could mess up with each other's variables

## the usage of the types:
objects and direct access would be the only ones to use local variables
but you could use global variables in objects with the static keyword

## so how does this get implemented?:
### in general:
we first need to make it so setting the location is optional 

ex: <sub><sup>(note that &root includes the @s)</sup></sub>
```
score count = 0;
score count in &root^count = 0;
```
### how will the local data paths know where to go?:
#### the problem:
I need to add a better way of setting things such as the
 - selector
 - entity type
 - data root

while setting up objects as they're all technically optional.

#### possible solutions:
##### the python way?:
I could seperate all of these into seperate functions that the object has to overwrite such as
```java
entity Raycaster {
    private Selector getSelector() {
        return sel:"@e[type=armor_stand]";
    }

    private EntityIdentifier getEntityType() {
        return ie:"armor_stand";
    }

    private DataPath getRootPath() {
        return dat:"ArmorItems[0].tag";
    }
}
```
but this strikes me as
 - too hidden
 - beginner unfriendly
 - and just way too much code

##### as variables?:
```java
entity Raycaster {
    private Selector selector = sel:"@e[type=armor_stand]";
    private EntityIdentifier entityType = ie:"armor_stand";
    private DataPath root = dat:"ArmorItems[0].tag";
}
```
fixes the too much code problem but still has the others

##### in declaration?:
```
entity selector @e[type=armor_stand] entityType armor_stand root ArmorItems[0].tag Raycaster {

}

entity 
    selector @e[type=armor_stand] 
    entityType armor_stand 
    root ArmorItems[0].tag 
Raycaster {

}
```
this comes back with the too much code problem but looks fine if you format it correctly


##### near declaration?:
```
entity Raycaster options {
    selector = sel:"@e[type=armor_stand]",
    type = ie:"armor_stand",
    root = dat:"ArmorItems[0].tag"
} {

}

entity Raycaster options {
    selector @e[type=armor_stand]
    type armor_stand
    root ArmorItems[0].tag
} {

}

// block examples

block Scheduler options {
    type = "fixed",
    <!-- positions = "10..20 1 10" // special type of string for this? Or maybe a operator overload?
    positions = [
        pos:"10 1 10",
        pos:"11 1 10",
        pos:"12 1 10",
        pos:"13 1 10",
        pos:"14 1 10"
    ]
    root = dat:"*"
} {

}

block Shop options {
    type = "marked",
    markerType = ie:"armor_stand",
    markerData = nbt:"{Marker:1b,Invisible:1b,ArmorItems:[{id:'tnt',Count:1b}]}"
    root = dat:"*"
} {

}
```
this is my current favorite

##### passed in constructor?:
```
entity Raycaster {
    constructor() {
        super(sel:"@e[type=armor_stand]", ie:"armor_stand", dat:"ArmorItems[0].tag");
    }
}
```
way too janky
##### java environment way?:
```
@options(
    selector = sel:"@e[type=armor_stand]",
    type = ie:"armor_stand",
    root = dat:"ArmorItems[0].tag"
)
entity Raycaster {

}
```
this is my second favorite

#### what if these options aren't specified?
There will be a simple set of defaults

##### for entities:
- selector = sel:"@e",
- type = null,
- root = dat:"*"
##### for blocks:
- type = "marked",
- markerType = ie:"area_of_effect_cloud",
- root = dat:"*"

#### how do we access these options?

`class.options.root`
`class.root`

## Variable assignment thing
What if we think of these variables not as keywords leading up to an equals sign, but as a seperate value such as a 0 or 1? This allows us to use these things in other places such as on storage.

ex: `data String callback = data in storage test:raycast settings.callback;`

We could also allow for other variables to override assignment

We could also not do any of this and just use similar syntax.

&nbsp;

# Stuff stolen from spwn and rust
> [video](https://www.youtube.com/watch?v=cQyNar6rgW8)

## Macros
The name is great

## Interfaces
The idea of forcing an interface onto something is also quite nice, it could introduce a better way of doing the constructors and stuff

## Let
```
let uuid = data entity @s UUID;
```

## Modules

# Optimization
## Library-specific temporary variables
How about there are a set of temporary variables per library that are checked to make sure fit together

# Command blocks

```
{}
{}.getCommand() // returns a list of either one or two commands or a function
{}.getCommands() // returns a list of all of the commands
{}.getFunction() // returns a function
```

A for loop is made of a command block.
You could even think of it as it's own function.

```
for(int i = 0; i < 5; i++) {}
for(int i = 0; i < 5; i++) do {}
for(int i = 0; i < 5; i++).do({})
```

# Operator overloading

Steal from scala
```
When you write...

sum = 2 + 3,
...you're actually calling a method called + on a RichInt type with a value of 2. You could even rewrite it as...

sum = 2.+(3)
...if you really really wanted to.
```
(https://dzone.com/articles/operator-overloading-scala)

## Paratheseesless functions
Would only work with one argument due to functions

`test(identifier minecraft, armor_stand, 2)`

### nospace
Could add a no space option for bitwise operators and stuch

```
nospace function !(Object obj) {
    return obj.falsehood();
}
```

Other applications

```
/"say hi"
```

Imply nospace with all functions that are comprised of no letters or lines (a-zA-Z\\-_)

## Weird things with loops and command blocks

```
for(int i = 0; i < 10; i++) do {

}

// wouldn't work, how would the checker get accessed?
function for()
```

or maybe...

```
object ForLoopBuilder {
    private CommandPart start;
    private CommandPart check;
    private CommandPart end;

    public ForLoopBuilder(CommandPart start, CommandPart check, CommandPart end) {
        this.start = start;
        this.check = check;
        this.end = end;
    }

    public do(CommandBlock block) {
        // will automatically do .toCommand()
        run(this.start);
        CommandBlock loop = {
            if(this.check.toCondition()) {
                run(block);
                run(this.end);
                // this is really weird
                run(loop);
            }
        }
        run(loop);
    }

    // this is also weird
    public (CommandBlock block) {
        do(block);
    }
}

function for(CommandPart start, CommandPart check, CommandPart end) {
    return new ForLoopBuilder(start, check, end);
}

for(score i = 0, i < 10, i++) do {}
for(score i = 0, i < 10, i++) {}
```

but this doesn't work with the integer for loops

# Macro variables in functions
```
function a(int x matches 1..16) {}
function a(int x = 1 matches 1..16) {}
function a(int x matches [1, 2, 4, 8, 16]) {}
```

# Score floats
So normally in scores, you could only add a precision modifier for fractions. However, what's to stop us from adding in floats using multiple score values?

Could use [this library](https://youtu.be/e6OrClOPO_M) after updating it for 1.16 and commenting it for ofunction or completely remaking it in ofunction.

[Library](https://youtu.be/gDObl5lCF1w) seems to already be updated to 1.16 - nvm

![Image](https://cdn.discordapp.com/attachments/642730973942906931/870048761823514714/unknown.png)

[Better teleport](https://youtu.be/OINJSgKWQpg)

# Garbage collection
free(x)