maybe add some entity stack in the game which holds a list of the uuids of all the entities that have run it

there needs to be some difference between world objects and theoretical objects
nah they just ded lamo

object general Object {
	abstract data data;
    // self is replaced with this
	// abstract object self;

	abstract void kill();
	abstract void summon(position pos);
    abstract void instanceTick();
    static void staticTick() {

    }
}

//superclass of all entities
object entity selector @e Entity {
	data compound data = entity @s *;

	void kill() {
		sys.run("kill @s");
	}

    void summon(Position pos) {
        if(selector != null && selector.hasEntityType()) {
            entity.summon(selector.getEntityType(), pos);
        } else {
            entity.summon("area_effect_cloud", pos,  {Age: -2147483648, Duration: 2147483648, WaitTime: 0});
        }
    }

    void summon() {
        summon(~ ~ ~);
    }

    static Entity getAll(Selector sel) {
        return sel.addArguments(selector.getArguments()).getEntity();
    }

    static Entity getAll() {
        return get(@e);
    }

    static Entity getNearest(Selector sel) {
        return get(@e[limit=1,sort=nearest]);
    }

    static Entity getNearest() {
        return getNearest(@e);
    }

    static void staticTick() {
        // execute.as(getAll()).run(instanceTick());
    }
}


//should data paths reference the final path or the root path?
object entity type armor_stand selector @e[tag=raycast.raycaster] Raycaster extends RaycastPart {
	data compound root = entity @s ArmorItems[0].tag;

	score dummy distance = @s distance, 0;
	score dummy max = @s max;
	score dummy stepSize = @s stepSize;

    entity Entity source = @s &root^player;
    data String callback = @s &root^callback

	Raycaster init(score dummy stepSize, score dummy max, entity Entity source, data String callback) {
		this.max = max;
		this.stepSize = stepSize;

        this.source = source;
		this.data.Rotation = source.data.Rotation;

		//quotations can be optional in builtin variable types like entity selectors, positions, and nbt
		entity.tag.add(this, "raycast.raycaster");

        return this;
	}

	//objects only need one summon as everything else will be handled in the constructors
	//this could also technically be optional if someone only wants casts
	Entity summon(Position pos) {
		//nbt will be a standard format
		//as well as positions(~ ~ ~, ^ ^ ^)
		//identifiers, however, cannot be added as they can conflict with variables
		return (Raycaster) entity.summon("armor_stand", pos, {Tags:["raycast.raycaster"]});
	}

	//stepSize is represented in 16ths of a block
	//scores are used as in and out
	void raycast() {
		for(int i = 0; i < 16; i++) {
			this.distance += 1;

			if(this.distance > this.max) distanceOver();

			if(i % this.stepSize == 0) {
                // possibly add a general score for things with no set path
                // but that not only might be confusing but also might include abiguity in the syntax
                // but ambiguity already exists when setting a variable to a value
                // kinda want to remove the ambiguity but could it not just choose with data paths?
				score dummy hit = hit oraycast, execute.positioned(^ ^ ^i*0.0625).run(this.check());
				if(hit > 0) hitObject();
			}
		}	
	}

    void instanceTick() {
        raycast();
    }

    static void staticTick() {
        execute.as(@e[tag=raycast.success.source]).run(addAge());
    }

	abstract void distanceOver();
	abstract void hitObject();

	abstract score dummy check();
}
	
object entity @e[type=armor_stand,tag=raycast.raycaster] GoalRaycaster extends Raycaster {
	// idk about this syntax
    // an alternative to -> could just be ^

	entity Goal goal = entity @s &this.root->goal;

    // so functions need inputs
    // how will the function know where those inputs are coming from
	GoalRaycaster init(score dummy max, score dummy stepSize, entity Goal goal, entity Entity source, data String callback) {
		super(max, stepSize, source, callback);
		this.goal = goal;
	}

	score general check() {
		tag.add(goal, "currentGoal");
        score dummy result = entity.check(@e[tag=currentGoal,distance=..1]));
        tag.remove(goal, "currentGoal");
		return result;
	}	

	void distanceOver() {
		this.kill();
	}

    void hitObject() {
        tag.add(this.goal, "raycast.success.goal");
        tag.add(this, "raycast.success.raycaster");
        tag.add(this.source, "raycast.success.source");
        sys.schedule(this.callback);
    }

    void kill() {
        goal.raycasterDeath();
		super.kill();
    }
}

object entity @e[type=armor_stand,tag=raycast.Goal] Goal extends RaycastPart {
	data compound root = entity @s ArmorItems[0].tag;
    // change syntax to score dummy raycasters = @s raycasters = 0?
    score dummy raycasters = @s raycasters, 0;
    data String callback = root^callback;

	Goal init(int maxLength = [1,6,10,20], score dummy stepSize, data String callback) {
		// this is why i have it as curly bracket
        tag.add(@s, "currentGoal");

        // the static variables could be used like this by adding a bunch
        // of execute ifs, the accepted inputs set by the array of values
		execute.as(@a[distance=..maxLength]).at(@s).run(() -> {
            entity Goal goal = Goal.getNearest(@e[tag=currentGoal]);
            ((GoalRaycaster) GoalRaycaster.summon()).init(maxLength, stepSize, goal, this, callback);
            goal.onRaycasterSpawn();
        });

        tag.remove(@s, "currentGoal");
	}



    Entity summon(Position pos) {
        entity.summon("armor_stand", pos, {Tags:["raycast.Goal"]});
        return this;
    }

    static void startRaycast(int maxLength = [1,6], score dummy stepSize, data String callback) {
        ((Goal) Goal.summon()).init(maxLength, stepSize, callback);
    }

    void raycasterDeath() {
        this.raycasters--;
    }

    void onRaycasterSpawn() {
        this.raycasters++;
    }

    static Goal getNearest(Selector sel) {
        return (Goal)super.getNearest(sel);
    }
}

object entity @e RaycastPart {
    score dummy age = @s age, 0;

    static void staticTick() {
        Goal.getAll(@e[tag=raycast.success.goal]).addAge();
        RaycastPart.getAll(@e[tag=raycast.success.source]).addAge();
        Raycaster.getAll(@e[tag=raycast.success.raycaster]).addAge();
    }

    void addAge() {
        age++;
        if(age > 1) kill();
    }
}

	

need to add some command modifier functions
the entity library maybe could be included in this/

if you add java-like paths (com.mojang.minecraft.nbt etc) then they could be viewed as the paths in functions (thatepicbanana.place.command etc to thatepicbanana:place/command)
 - import thatepicbanana:general/uuid/getuuid
 - copy everything from python including the init stuff
 instead of init files running their code every time they get imported, just add the code to the onload function.

find a way to include normal mcfunction files in these modules

blocks will be interesting
two types of blocks
 - aoe cloud ones
	- these will be used with an area of effect cloud marking their position and acting as the runner entity
	- they can be used by any module, including ones outside of it's own
 - set ones
	- these will be marked by a set of cooridnates set on compile automatically or set by the user
	- these can only be used in it's own module since you would have to compile other modules 
	alongside it's own for it to work

consumer.toCommand() will return either a function with the consumer's commands 
in it or just a single command, think of it as an if statement or while loop.

maybe make variable syntax more of a mov command in the zachtronics games
ex: data distance = &root^distance, 10
the compiler can also just use a singular address if it doesn't want to initialize the value
ex: data x = pos.x
you could also even take this further by giving adresses more of a syntax like this:
entity @s root.distance
with entity @s being optional optionally
this could make it so variables could instance different things over time such as in a loop iterating over multiple entities
why do we need variables that change their source? this could lead to ambiguity as to which it is in the future
yea but this doesn't mean that the syntax for starting a function couldn't be like that
also, it could just be restricted to variables that are not stored in objects

use comments in compiled mcfunction files to denote functions

```
$root^distance - >@e[type=armor_stand]<
$root->distance - ^@e[type=armor_stand]^ - $@e[type=armor_stand]$ - |@e[type=armor_stand]| - ~@e[type=armor_stand]~
```

or could just replace everything with strings and switch it to stuff like system.eval("command")
would this not already happen when interfacing with command blocks? why not just make it easier

some ofunction libraries could be written in java for speed and to use minecraft lists

for and while loop variables can still be from runtime because of recursion

add a located thing for functions so operators can use it ingame

make it so "static" functions (functions that input and return built in variables such as strings and ints) cannot use
"dynamic" variables or functions. These static functions then are only run in compilation.

all entity data types are technically selectors which means you could do weird stuff with them

entites can be reprensented a lot of ways, namely UUIDs, Tags, and Selectors.
you might notice that these are not all "data," so why is it under data?
wouldn't it make more sense to put it under something like entity?

functions need inputs
how will the inputs know where they're stored?
i might have to change a lot about this
but couldn't they be set automatically?
the others can't but that's because you need a lot of control 
over where they are stored due to different entity types and such
but function inputs could just be stored in a few dummy storage paths and scores
it could also be allowed to be specified where they are for non-ofunction libraries

add data structures for things such as the settings thing

tempted to add public and private

better way of doing contructors and stuff
object Raycaster raycaster = entity @s root^raycaster
object Raycaster raycaster = entity @s root^raycaster, new Raycaster();

add get and set from c#
https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/virtual

add Tickable and Loadable interfaces (or however they will be named)

add subtract and stuff instead do the operation then return the score path
data.getFreeDataPath();
data.getFreeScorePath();

object Score alias b, brd {
    public Score() {

    }

    public Score(Score copy) {

    }

    public Score add(Score a, Score b) {
        // this is a problem, how do we specify the difference between the path and the value?
        // how do we implement soft copying (things that happen to the copy happen to the source) and deep copying?
        // maybe borrow some stuff from pointers? specify path with &, value with *, idk if comma should exist
        score dummy temp = &data.getFreeScorePath(), *a;

        sys.run("scoreboard players operation " + temp + " += " + b)

        return temp;
    }
}

good pointer explanation: https://stackoverflow.com/questions/2094666/pointers-in-c-when-to-use-the-ampersand-and-the-asterisk

Commands and such should not have to be intepreted
well there should be two steps to compiling, the "static expansion" and the normal "compilation"
the static expansion compiles all of the static classes and funtions then runs them on the source code
the normal compilation then compiles it all
this is making me want to seperate the difference between static and dynamic things more
but there isn't a clear difference between the two as shown by the example above
something like nbt - named strings
s"@e" or "@e"s - s for selector
and of course the syntax with no quotes stays the same

"count"s s"count" s:count: :count:s
"count"blk blk"count" blk:count: :count:blk
s - sel - selector
c - cmd - command
d - dat - data path
de - den - entity path
db - dbk - block path
ds - dst - storage path
b - brd - scoreboard
i - ide - identifier
ie - ien - entity type
ib - ibk - block type
is - ist - storage id
p - pos - position

implement them as single variable constructors with aliases
Score:"position" would also be correct syntax


two constructors, one for initiating and one for summoning
nah three, another for initiating without inputs
there can be a built in tag for all classes that identifies them
this can be used to check if it has been initiated before
however, I am not certain about this as I'd rather have three separate syntax for using them
though the tag can still be used in the typecast constructor
this should be explicitly said as it may confuse people
there also should be options to turn off the tag check in the constructor and the typecast constructor when typecasting, separately
constructors shouldn't have return types
summon(a,b,c) {} initiate(a,b,c) {} cast {}
summon automatically calls initiate
Class.summon(a,b,c) or Class.summon(pos,a,b,c)
Entity.summon(pos,Class(a,b,c)) or 
Entity.summon(pos,new Class(a,b,c))
there's no syntax for setting an entity to a class currently, and there really should be. Think a good idea could be to separate uuids and selectors, and provide a way to convert between the two.
entity Class class = @s; class.initiate(a,b,c) or also
entity Class class = @s, initiate(a,b,c) or entity Class class = @s, (a,b,c)
also very much could shorten initiate to init.
nvm no more casting
entity Class class = (Class(a,b,c)) @s;
entity Class class = (Class()) @s;

with the seperation of entities, uuids, and selectors, this syntax is completely destroyed
object Raycaster raycaster = entity @s root^raycaster
this would require the summoning of a raycaster, obtaining the uuid, then storing it
object Raycaster raycaster = (Raycaster(a,b,c)) Entity.summon(ie:"minecraft:armor_stand");
data UUID raycasterUUID = raycaster.getUUID();
data UUID raycasterUUID = UUID.getUUID(raycaster);



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
    positions = "10..20 1 10" // special type of string for this?
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

&nbsp;

This will probably be put into a different part
# Defining other parts of the datapack:
## loot tables:
## advancements:
## item modifiers:
## recipes:
## tags:
### block tags:
### entity tags:

# Variables
## Data
### Definition
```
data Type name = x
```
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
#### UUIDs
Technically integer lists under the hood, you can access parts of the UUID by using the syntax for lists.

ex: `playerUUID[0]`

See more in the entity section
#### Booleans
ex: `true` `false`

These will be converted into a byte tag by the game.