[![Build Status](https://travis-ci.org/incrediblesound/story-graph.svg?branch=master)](https://travis-ci.org/incrediblesound/story-graph)

### Index
[Overview](#overview)  
[World Generator](#generator)  
[API Documenation](#docs)  

<a name="overview"/>

# StoryGraph

The Graph that Generates Stories

StoryGraph is a library that allows you to generate a narrate based on random interactions between actors in a world. StoryGraph provides classes for actors, types, and rules that can model interactions between different classes of entities. Rules represent possible interactions between actors and, in addition to generating text output, can also create new actors, remove actors from the world, and move actors between locations.

Story graph is inspired by [programming interactive worlds with linear logic](http://www.cs.cmu.edu/~cmartens/thesis/) by [Chris Martens](http://www.cs.cmu.edu/~cmartens/index.html) although it doesn't realize any of the specific principles she develops in that thesis.

My own worlds are available in the examples directory. You can run them directly with node.js:
```shell
$ node examples/forest.js
```
You will see output something like this:
```
The river joins with the shadow for a moment. The river does a whirling dance with the shadow. A bluejay discovers the river dancing with the shadow. A bluejay observes the patterns of the river dancing with the shadow. A bluejay dwells in the stillness of life. A duck approaches the whisper. A duck and the whisper pass each other quietly.
```
This project is licensed under the terms of the MIT license.

<a name="generator"/>

## StoryGraph World Generator

The StoryGraph world generator translates a plain english description of a StoryGraph world into a working StoryGraph program. StoryGraph objects, especially rules, can be cumbersome to write out by hand, so this program allows you to easily define your world and generate the code automatically. The program generated by the World Generator will be somewhat dull and shouldn't be considered a finished product. Its purpose is to generate the boilerplate and make it easy to go in and build your StoryGraph world.

## How to Use

First, write a description of your world using the grammar below and save it to disk. Go to the root directory of StoryGraph in your console and use the following command:
```shell
node generateWorld path/to/description.txt myWorld.js
```
The second parameter is the output file name and it is important that it has a .js extension. Now you can modify your world as you see fit. Generated worlds automatically console.log a four step story so you can immediately test you world like this:
```shell
node myWorld.js
```

## Grammar

Here is the grammar of the world generator. Note that the formats provided here are not flexible. Only the parts inside curly braces may be replaced with your custom text.

### Basic Types

FORMAT: There is a type called {typename}.  
```code
There is a type called person. 
There is a type called ghost.
```

### Type Extensions

FORMAT: A {new type} is a {base type}.  
```code
A woman is a person. 
A cat is an animal. 
A skeleton is a ghost.
```

### Type Decorators

FORMAT: Some actors are {typename}.  
OPTIONAL FORMAT: Some actors are {type one} and some are {type two}.  
```code
Some people are smart and some are stupid. 
Some people are scary.
```
### Locations

FORMAT: There is a place called <{place name}>.   
```code
There is a place called <the red house>.
There is a place called <the field of wheat>.
```

### Actors

Note that the placeholder {type} here may be a basic type or extended type preceded by any number of type decorators. After the actor's description there is a second clause listing out what locations this actor may enter. You may list any number of locations separated by the word "or". Note that only actors with multiple locations can enter into transitions that describe their change of location.

FORMAT: There is a {type} called {name}, [ he | she |it ] is  [ in | on ] <{place}>.
OPTIONAL FORMAT: There is a {type} called {name}, [ he | she |it ] is [ in | on ] <{place}> or <{place two}>.
```code
There is a ghost called Slimer, it is in <the red house>.
There is a smart kind man named Joe, he is in <the red house> or <the field of wheat>.
There is a beautiful woman named Angelina, she is in <the red house> or <the law office>.
```

### Rules

Again, the placeholder {type} may be preceded by any number of decorators.

FORMAT: If a {type one} <{encounter text}> a {type two} then the {type one|two} <{result text}>.  
OPTIONAL FORMAT: If a {type one} <{encounter text}> a {type two} then the {type one|two} <{result text}> the {type one|two}  
```code
If a boy <is startled by> a ghost then the boy <starts to cry>. If a man <sees> a ghost then the man <stares in disbelief at> the ghost.
```

### Transitions

FORMAT: The {type or actor} <{transitions to}> <{destination}> from <{origin}>.
```code
The man <goes out to> <the field of wheat> from <the red house>.
The bird <swoops down into> <the field of wheat> from <the sky>.
```

### Full Example

For a full working example look at example.txt in the root directory of this repository.

<a name="docs"/>

## StoryGraph API Documentation

All of the StoryGraph classes and constants are accessible via the file in `dist/story.js` which is generated by webpack. To generate this file run the following commands:
```shell
$ npm install -g webpack
$ npm install
$ webpack
```

## Types

The first step is to define types. Types are how the story graph engine determines whether or not an actor matches a given rule. While it is possible to make rules that apply to specific actors, you'll probably want to make general rules that apply to classes of actors, and for that you use types. First the basics:
```javascript
const Type = require('./dist/story.js').Type;

// we can start with a basic type and extend it with more specific types
const person = new Type('person');

// An actor with the adult type will match rules for either "person" or "adult"
const adult = person.extend('adult');

// An actor with the man type will match rules for "person", "adult" and "man"
const man = adult.extend('male');
```
I like to use this helper that allows me to easily create actors with complex types:
```javascript
const extendType = function(typeName){
  return function(type){
    return type.extend(typeName);
  }
}
// now I can do this:
const smart = extendType('smart');
const sneaky = extendType('cunning');
const spy = new Actor({
  type: smart(sneaky(person)),
  name: 'spy'
});
```
Another strategy you might want to use is to make every type extend from one base type which allows you to match on specific types while ignoring more general types. For example, if everything in the world extends from the "person" type, then you can match on ( sneaky && person ) which will match any sneaky person young or old, male or female.

## Actors

Making actors is really straightforward. They take a type, which defines what rules they match, and a name which is used to create the narrative output. Actors can also be given a lifetime; after the graph goes through a number of time steps equal to the lifetime of an actor, the actor will be removed from the graph. Lifetimes are more useful for consequent actors discussed below. Here are some basic examples:
```javascript
const Actor = require('dist/story').Actor

const bob = new Actor({
  type: smart(man),
  name: 'Bob'
});

const sally = new Actor({
  type: smart(girl),
  name: 'Sally'
});
```
You can add actors to the world one by one or as an array of actors. You can save the id of an actor in the world by adding them individually and saving the return value in a variable.
```javascript
const World = require('dist/story').World

const world = new World()

const bobId = world.addActor(bob)
const sallyId = world.addActor(sally)
world.addActor([tim, larry, david, moe])
```

## Rules

Rules are added via the addRule method on the world, and have the following structure:

```javascript
world.addRule({
  cause: { type: [A, B, C], value: [A, B, C]},
  consequent: { type: [A, B, C], value: [A, B, C] },
  isDirectional: boolean,
  locations: [],
  mutations: function(source, target){
    target.type.add('newType');
  },
  consequentActor: {
    type: type,
    name: "name"
  }
});
```
**Cause**    
Cause is a description of the event that triggers the rule. The cause type has the following structure
```javascript
[ (id || type), ACTION, (id || type) ]
```
where id is the id of an actor in the world, action is one of the actions described in ./src/constants.js, and type is an instance of Type. Note that the first position in the cause type is referred to as the "source" and the third position as "target". It helps to think of it as describing vertex-edge->vertex structure in a directed graph.

The cause value is an array that can be any mix of strings and references to the source and target that triggered the rule. Constants are provided that allow you to refer to source and target. Here is an example cause:
```javascript
const c = require('dist/story').constants;

const cause = {
  type: [ dancer, c.ENCOUNTER, dancer ],
  value: [ c.SOURCE, 'dances with', c.TARGET]
}
```
When the cause is matched with actors StoryGraph will replace the source and target constants in the value portion of the cause with the name of the actors. That means if actors named "Bob" and "Sally" are matched with the rule above, StoryGraph will add "Bob dances with Sally." to the output.

**consequent**  
The type property of consequent is different from the type property of cause: it is the type of event that is triggered by the rule as a consequence of the rule being matched, like a chain reaction. If you want a rule that results in an actor being removed from the world you can use constants.VANISH in the consequent type, but all other action types will trigger a search for a matching rule. The value of the consequent is any mix of strings and references to the actors that triggered the rule. The consequent value will produce a string describing the outcome of the interaction when the rule is triggered.

This distinction is important: The consequent type describes an event that will trigger a rule after the current rule is done, the consequent value describes a sentence that will be rendered by the current rule as part of its execution. This means that a rule can render both the cause and effect of an event as output but it can also serve as the cause itself if the consequent type triggers another rule. Here are two examples, one to match the cause given above and one to demonstrate the vanish constant:

```javascript
consequent: {
    type: [], // this rule doesn't trigger anything else
    value: ['The crowd admires the dancing skill of', c.SOURCE, ' and ', c.TARGET]
}

consequent: {
  type: [ c.SOURCE, c.VANISH ], // this rule removes the source from the graph
  value: [ c.SOURCE, 'disappears into thin air' ]
}
```

**directionality**  
The isDirectional property must be set to tell the graph how to match rules. If this property is set to false then the order of the actors will be ignored when finding a match. When the source and target are qualitatively different actors and the action is truly directional you should set this property to true.

**mutations**  
If you want to mutate the actors involved in an event you can add a mutations function to your rule. The mutations function takes the source actor and target actor as parameters. This allows you to alter the types of actors as a part of the consequence of the rule being activated, for which you can use the remove, add, and replace helpers on the actors type:
```javascript
{
  cause: [ handsome(boy), meets, pretty(girl) ],
  consequent: [ c.SOURCE, 'starts dating', c.TARGET ],
  isDirectional: false,
  mutations: function(source, target){
    source.type.replace('single', 'dating');
    target.type.replace('single', 'dating');
  }
}
```

**consequent actor**  
We've already seen how a rule can trigger another event, but a rule can also create a new actor in the world. If you want a rule to produce an actor add the actor's definition in the consequentActor property of the rule. Consequent actors have a couple of special properties: first they can have members. Because the consequent actor is a product of some other set of specific actors triggering a rule it makes sense that those actors might merge or compose to create the consequent actor. Moreover, the name of the consequent actor might involve the names of the actors that triggered the rule, so there is an initializeName function that takes the instance of the new actor and the world instance as parameters and returns a string that will be set as the name of the new actor instance. This allows you to use the members of the new actor to set the name. Here is a full rule example:
```javascript
world.addRule({
  cause: {
    type: [smart(person), c.ENCOUNTER, smart(person)],
    value: [c.SOURCE, 'meets', c.TARGET]
  },
  consequent: {
    type: [],
    value: [c.SOURCE, 'and', c.TARGET, 'start chatting']
  },
  isDirectional: false,
  consequentActor: {
    type: casual(discussion(gathering)),
    name: 'having a discussion with',
    members: [c.SOURCE, c.TARGET],
    lifeTime: Math.floor(Math.random()*3),
    initializeName: (actor, world) => `${this.members[0]} ${this.name} ${this.members[1]}`
    }
  }
});

```
If this rule was matched with two actors "Bob" and "Tom" it would produce the following output:

"Bob meets Tom. Bob and Tom start chatting."

And it would create a new actor with the name:

"Bob having a discussion with Tom"


**locations**
Locations are an optional feature of StoryGraph that add a lot of complexity and narrative possibilities. I think of locations as named graphs. When you add a location to your StoryGraph world, you are saying that there is a specific named place where actors can reside and where specific rules may apply.

Adding a location to a world is as simple as this:
```javascript
const world = new StoryGraph.World();
world.addLocation({ name: 'the house'});
world.addLocation({ name: 'the garden'});
```
When creating actors you can provide a list of possible locations for that actor and an optional starting location.
```javascript
const Bob = world.addActor(new Actor({
  type: human,
  name: 'Robert',
  locations: ['the house', 'the garden'],
  location: 'the house' // optional; defaults to first location in the locations array
}));
```
There are two ways to use locations in your StoryGraph rules. First of all, if you have an actor that can exist in multiple different locations you have to create rules to model the transitions between those locations.
```javascript
world.addRule({
  cause: {
    type: [human, c.MOVE_OUT, 'the house'],
    value: ['']
  },
  consequent: {
    type: [c.SOURCE, c.MOVE_IN, 'the garden'],
    value: [c.SOURCE, 'walks out into the garden']
  },
})
world.addRule({
  cause: {
    type: [human, c.MOVE_OUT, 'the garden'],
    value: ['']
  },
  consequent: {
    type: [c.SOURCE, c.MOVE_IN, 'the house'],
    value: [c.SOURCE, 'enters the house']
  },
})
```
As you can see from these examples we have special constants move_out and move_in for modeling location transitions. An actor will only match a location transition if it is currently in the location indicated in the rule type with move_out, and if the location indicated in the type with move_in is contained in the possible locations of that actor. So, for the first example above, an actor will match the rule if its current location is "the house" and if "the garden" is contained in its potential locations.

The second way to use locations in your rules is to localize them. Some rules might describe events that make sense if they happen in one location but not in another. To configure this, simply add a locations property to the rule with an array of locations where that rule can occur. Here is an example of a rule I wrote that is localized:
```javascript
world.addRule({
  cause: {
    type: [human, c.ENCOUNTER, ghoul],
    value: [c.SOURCE, 'sees', c.TARGET]
  },
  consequent: {
    type: [],
    value: [c.SOURCE, 'turns pale and runs away']
  },
  locations: ['the graveyard'],
  isDirectional: true,
  mutations: null,
});
```
## Generate Stories

To generate a narrative use the runStory method on the world object. This method takes a number which determines how many time steps the graph will run for and an optional array of events that happen at specific time steps. Events have the following structure:
```javascript
{
    step: 1 // this is the time step when this event will be rendered
    event: [ (id || type), action, (id || type) ] //this is the type of the rule to be rendered
}
```
 At each time step the runStory method will check to see if there is an event set for that step, and if not it will generate a random event. The result will be stored on the output property of the world object.
```javascript
world.runStory(4, [
    { step: 1, event: [ bob, c.ENCOUNTER, tom ]},
    { step: 4, event: [ bob, c.MOVE_OUT, cafe ]}
]);
console.log(world.output); // "Bob meets Tom..."
```
