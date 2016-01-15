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
- [Mixins/decorators](#mixins/decorators)
- [Observer](#observer)
- [IIFEs](#iifes)
- [jQuery examples](#jquery examples)


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
```
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
 ```
// Increment our counter
testModule.incrementCounter();
 
// Check the counter value and reset
// Outputs: counter value prior to reset: 1
testModule.resetCounter();
```

### Revealing module pattern

- Define our vars and functions in the private scope and return an anonymous object with pointers to the private functionality we want to reveal as public

Example:
```
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
```
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
- Functions that return objects that have methods (esp objects w/ varying prototypes)
- Good for deferred object construction
- Good for creating complex objects

Examples:
```
function makeThing(name, status) {
  function ready() {
    return "Thing is ready";
  }
  function unready() {
    return "Thing ain't ready";
  }
  
  return {
    name: name,
    isReady: isReady,
    aboutThing: function() {
      return this.isReady ? ready() : unready();
    };
  }
}

var myThing = createThing("Bob", true);
```

### Mixins / decorators(OO)
- Functions taking an object and adding aditional behaviour

### Observer
- Observables, event emitters, PubSub
- Communicate over system boundaries

### IIFEs

A popular approach to encapsulation from the global namespace

Basic IIFE example:
```
// anon
( function() { // code })();

// named
(function foo() { // code })();
```
Expanding on the first example:

```
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
```
/*
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
    
    /* check wwhether "namespace" exists in the global namespace. if not, assign window.namespace an object literal
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
