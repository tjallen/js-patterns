# Design pattern shite
Compiling some JS design pattern info, use cases etc

## References
 - https://addyosmani.com/resources/essentialjsdesignpatterns/book/
 - http://leoasis.github.io/posts/2013/01/24/javascript-object-creation-patterns/
 - http://robdodson.me/javascript-design-patterns-singleton/
 - http://code.tutsplus.com/tutorials/understanding-design-patterns-in-javascript--net-25930
 - https://www.youtube.com/watch?v=HkFlM73G-hk&list=PLoYCgNOIyGABs-wDaaxChu82q_xQgUb4f
 - https://john-dugan.com/object-oriented-javascript-pattern-comparison/

## Contents
- [References](#references)
- [Modules](#modules)
  - [The Module pattern](#the module pattern)
  - [Revealing module pattern](#revealing module pattern)
- [Singletons](#singletons)
- [Factories](#factories)
- [Constructors](#constructor)
- [Combination Constructor/Prototype](#combination constructor/prototype)
- ~~[Dynamic Prototype](#dynamic prototype)~~
- [OLOO](#oloo)
- [Mixins/decorators](#mixins/decorators)
- [Observer](#observer)
- [IIFEs](#iifes)
- ~~[jQuery examples](#jquery examples)~~


## Categories of patterns
- **Creational**: Focus on creating situationally appropriate objects.
- Eg: Constructor, Factory, Abstract, Prototype, Singleton, Builder
- **Structural**: Focus on object composition & realising relationships between objects; changing correct parts of a system
- Eg: Decorator, Facade, Flyweight, Adapter, Proxy
- **Behavioral**: Focus on improving the communication between objects in a system
- Eg: Iterator, Mediator, Observer, Visitor

## Modules
Several options for implementing modules including:
- The Module pattern
- Object literals
- AMD, CommonJS, EMCAScript Harmony etc etc zzzz

### The Module pattern

The module pattern is based in part on object literals.

- Encapsulates 'privacy', state, organization w/ closures
- Stops pieces leaking into global scope
- Return a public API, keep everything else private in closure
- Similar to IIFE but an object is returned rather than a function

Example:
```JavaScript
var testModule = (function () {
 
  var counter = 0;
 
  return {
 
    incrementCounter: function () {
      return counter++;
    },
 
    resetCounter: function () {
      console.log( "counter value prior to reset: " + counter );
      counter = 0;
    }
  };
 
})();
 ```
 Usage:
 ```JavaScript
// Increment our counter
testModule.incrementCounter();
 
// Check the counter value and reset
// Outputs: counter value prior to reset: 1
testModule.resetCounter();
```

### Revealing module pattern

- Define our vars and functions in the private scope and return an anonymous object with pointers to the private functionality we want to reveal as public

Example:
```JavaScript
var myRevealingModule = (function () {
 
var privateCounter = 0;

function privateFunction() {
  privateCounter++;
}

function publicFunction() {
  publicIncrement();
}

function publicIncrement() {
  privateFunction();
}

function publicGetCount(){
  return privateCounter;
}

// Reveal public pointers to
// private functions and properties

return {
  start: publicFunction,
  increment: publicIncrement,
  count: publicGetCount
};

})();
 
myRevealingModule.start();
```

Positives: 
- Consistent syntax
- Clear deliniation between public and private

Negatives:
- Modules created this way can be more fragile due to the public/private interaction

### Singletons
- All objects in JS are singletons
- Good for optimization / using right away

Examples: 
- jQuery: `$`
- Object literals:
```JavaScript
var user = {  
  firstName: 'Cunty',
  middleName: 'The'
  lastName: 'Sardine',
  sayName: function() {
      return this.firstName + ' ' + this.middleName + ' ' +  this.lastName;
  },
  theDarndestLittleFellow: true
};
```

### Factories
Factory functions return objects that have methods (esp objects w/ varying prototypes). Conceived as a DRY way to abstract the object creation process.

- Good for deferred object construction
- Good for creating complex objects

- Potential inefficiency: methods created on the factory function are copied to all objects
- Lacking type determination: the new object instances created by the factory are typed as 'Object'; no indication of their content (solved by the Constructor pattern)

Examples:
```JavaScript
function createCar( make, model ) {
  var o = new Object();
  
  o.make = make;
  o.model = model;
  o.sayCarInfo = function() {
    alert( 'I own this abomination: ' + this.make + ' ' + this.model );
  }
  
  return o;
}

var bobCar = createCar( 'Austin', 'Allegro' );
var daveCar = createCar( 'FSO', 'Polonez' );
```

### Constructor

- The constructor pattern solves the Factory pattern's type determinism issue -
`alert( bobCar.constructor === Car );` is true in the Constructor pattern, but would be false in Factory
- Similar inefficiency issues to the Factory pattern, which leads to the combination Constructor/Prototype pattern

### Combination Constructor/Prototype

- Solves efficiency issues of Factory / Constructor patterns by using prototypal inheritance (prototypal delegation really)
- Allows for unique (non-shared) instance properties created within a constructor function as well as shared props and mewthods on the constructor functions prototype

Example:

```javascript
// constructor function to make car objects
function Car( make, model ) {
  this.make = make;
  this.model = model;
}

// constructor prototype to share props and methods
Car.prototype.sayCar = function() {
  alert( 'I have a spicy ' + this.make + ' ' this.model );
}

// create 2 car objects
var bobCar = new Car( 'Pagani', 'Zonda' ),
    daveCar = new Car( 'Ferrari', 'F50' );

// call method on bob's car
bobCar.sayCar(); // el z0nd
```

**Note about prototypes**: if you assign your constructor's prototype to an object literal instead of using dot notation, you'll overwrite the default `constructor` property. The object literal tells JS to _replace_ the prototype with a new object, rather than _augment_ the existing prototype. Because `constructor` is a default property on all prototypes, it gets removed when you replace the prototype with a new object. IF you want to assign a new object to a prototype and maintain the constructor relationship, you need to recreate the `constructor` property and assign it the proper value -- see below:

Create prototype using object literal without an explicit constructor:

```js


function Car( make, model ) {
  this.make = make;
  this.model = model;
}

// create constructor prototype with object literal
Car.prototype = {
  sayCar = function() {
    alert( 'I have a spicy ' + this.make + ' ' this.model );
  }
}

var johnCar = new Car( 'Ford', 'F150' );

// get constructor on john's car
alert( johnCar.constructor === Car ); // false
alert( johnCar.constructor === Object ); // true
```
Create prototype using object literal and create an explicit constructor:
```js
function Car( make, model ) {
  this.make = make;
  this.model = model;
}

// create constructor prototype with object literal
Car.prototype = {
  constructor: Car,
  sayCar: function() {
    alert( 'I have a ' + this.make + ' ' + this.model );
  }
};

var johnCar = new Car( 'Ford', 'F150' );

// get constructor on john's car
alert( johnCar.constructor === Car ); // true
alert( johnCar.constructor === Object ); // false
```
- Cuts down on repetition from using the dot notation
- However assigning a new object to a prototype makes your code more complicated; you lose the ability to determine type

...

- Confusing separation of constructor and prototype. Leads to the Dynamic Prototype pattern

### Dynamic Prototype

...

### OLOO

A recent pattern that uses prototypical inheritance to create objects directly from other objects, rather than using constructor functions

```javascript
// constructor
var Car = {
  init: function( make, model ) {
    this.make = make;
    this.model = model;
  },
  sayCar: function() {
    alert( 'I have this abomination: ' + this.make + ' ' + this.model );
  }
};

// create 2 car objects
var bobCar = Object.create(Car),
    daveCar = Object.create(Car);
    
//call init method on each
bobCar.init( 'Morris', 'Marina' );
daveCar.init( 'AvtoVAZ', 'Lada' );

// call method on daves car
daveCar.sayCar(); // the mighty Lada
```

Ex:
```javascript
function Car( make, model ) {
  this.make = make;
  this.model = model;
  this.sayCar = function() {
    alert('I own this abomination: ' + this.make + ' ' + this.model );
  };
}

var bobCar = new Car( 'Chrysler', 'PT Cruiser Cabriolet' );
var daveCar = new Car( 'REVA' , 'G-Wiz' );
```


### Mixins / decorators(OO)
- Functions taking an object and adding aditional behaviour

### Observer
- Observables, event emitters, PubSub
- Communicate over system boundaries

### IIFEs

A popular approach to encapsulation from the global namespace

Basic IIFE example:
```JavaScript
// anon
( function() { // code })();

// named
(function foo() { // code })();
```
Expanding on the first example:

```JavaScript
var namespace = namespace || {};
/*
in this example a namespace object is passed as a function parameter and public methods and properties are assigned to it
*/
(function ( o ) {
  o.foo = "foo";
  o.bar = function() {
    return "bar";
  };
})( namespace );
```

An even more expanded example:
```JavaScript
/*
semicolon before function invocation is a safety net against concat scripts / other plugins which mightn't be closed properly
namespace and undefined are passed to ensure:
1. namespace can be modified locally and isn't overwritten outside of function context
2. the value of undefined is guaranteed as truly undefined, to avoid issues with undefined being mutable pre-ES5
*/
;(function ( namespace, undefined ) {
    // private props
    var foo = 'foo',
        bar = 'bar';
    
    // public methods and props
    namespace.foobar = 'foobar';
    
    namespace.say = function ( msg ) {
      speak( msg );
    };
    
    namespace.sayHello = function() {
      namespace.say( 'hello world!' );
    }
    
    // private method
    function speak(msg) {
      console.log( 'Your msg: ' + msg );
    };
    
    /* check whether "namespace" exists in the global namespace. if not, assign window.namespace an object literal
    */
})( window.namespace = window.namespace || {} );
  
// testing out our properties and methods:
// public
console.log( namespace.foobar ); // 'foobar'
namespace.sayHello(); // 'You said: hello world!'
// assigning new props
namespace.foobarTwobar = 'twobar';
console.log( namespace.foobarTwobar ); // 'twobar'

```

### jQuery examples
