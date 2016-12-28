```JavaScript
function identify() {
  return this.name.toUpperCase();
}

function speak() {
  var greeting = "Hello, I'm " + identify.call( this );
  console.log( greeting );
}

var me = {
  name: "Kyle"
};

var you = {
  name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER
speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

If the how of this snippet confuses you, don’t worry! We’ll get to that shortly. Just set those questions aside briefly so we can look into the why more clearly.
This code snippet allows the identify() and speak() functions to be reused against multiple context objects (me and you), rather than needing a separate version of the function for each object. Instead of relying on this, you could have explicitly passed in a context object to both identify() and speak():

```JavaScript
function identify(context) {
  return context.name.toUpperCase();
}

function speak(context) {
  var greeting = "Hello, I'm " + identify( context );
  console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

The this mechanism provides a more elegant way of implicitly “passing along” an object reference, leading to cleaner API design and easier reuse

```JavaScript
function foo(num) {
  console.log( "foo: " + num ); // keep track of how many times `foo` is called
  this.count++;
}

foo.count = 0;
var i;

for (i=0; i<10; i++) {
  if (i > 5) {
    foo( i );
  }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9
// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

 foo.count is still 0, even though the four console.log statements clearly indicate foo(..) was in fact called four times. The frustration stems from a too literal interpretation of what this (in this.count++) means. When the code executes foo.count = 0, indeed it’s adding a property count to the function object foo. But for the this.count reference inside of the function, this is not in fact pointing at all to that function object, and so even though the property names are the same, the root objects are different, and confusion ensues.

Instead of stopping at this point and digging into why the this reference doesn’t seem to be behaving as expected, and answering those tough but important questions, many developers simply avoid the issue altogether, and hack toward some other solution, such as creating another object to hold the count property:
```JavaScript
function foo(num) {
  console.log( "foo: " + num ); // keep track of how many times `foo` is called
  data.count++;
}
var data = { count: 0 };
var i;
for (i=0; i<10; i++) {
  if (i > 5) {
    foo( i );
  }
} // foo: 6 // foo: 7 // foo: 8 // foo: 9
// how many times was `foo` called?
console.log( data.count ); // 4
```

While it is true that this approach “solves” the problem, unfortunately it simply ignores the real problem—lack of understanding what this means and how it works—and instead falls back to the comfort zone of a more familiar mechanism: lexical scope. Lexical scope is a perfectly fine and useful mechanism; I am not belittling the use of it, by any means (see the Scope & Closures title of this book series). But constantly guessing at how to use this, and usually being wrong, is not a good reason to retreat back to lexical scope and never learn why this eludes you. To reference a function object from inside itself, this by itself will typically be insufficient. You generally need a reference to the function object via a lexical identifier (variable) that points at it.

```JavaScript
function foo() {
  foo.count = 4; // `foo` refers to itself
}
setTimeout( function(){
  // anonymous function (no name), cannot
  // refer to itself
}, 10 );
```

In the first function, called a “named function,” foo is a reference that can be used to refer to the function from inside itself. But in the second example, the function callback passed to setTimeout(..) has no name identifier (called an “anonymous function”), so there’s no proper way to refer to the function object itself

```JavaScript
function foo() {
  var a = 2;
  this.bar();
}

function bar() {
  console.log( this.a );
}

foo(); //ReferenceError: a is not defined

```

There’s more than one mistake in this snippet. While it may seem contrived, the code you see is a distillation of actual real-world code that has been exchanged in public community help forums. It’s a wonderful (if not sad) illustration of just how misguided this assumptions can be. First, an attempt is made to reference the bar() function via this.bar(). It is almost certainly an accident that it works, but we’ll explain the how of that shortly. The most natural way to have invoked bar() would have been to omit the leading this. and just make a lexical reference to the identifier. However, the developer who writes such code is attempting to use this to create a bridge between the lexical scopes of foo() and bar(), so that bar() has access to the variable a in the inner scope of foo(). No such bridge is possible. You cannot use a this reference to look some‐ thing up in a lexical scope. It is not possible. Every time you feel yourself trying to mix lexical scope look-ups with this, remind yourself: there is no bridge. 8 | Chapter 1: this or That?

We said earlier that this is not an author-time binding but a runtime binding

When a function is invoked, an activation record, otherwise known as an execution context, is created

This record contains information about where the function was called from (the call-stack), how the function was invoked, what parameters were passed, etc

One of the properties of this record is the this reference, which will be used for the duration of that function’s execution

In Chapter 1, we discarded various misconceptions about this and learned instead that this is a binding made for each function invocation, based entirely on its call-site (how the function is called)

To understand this binding, we have to understand the call-site: the location in code where a function is called (not where it’s declared). We must inspect the call-site to answer the question: what is this this a reference to?

A subtle but important detail is that though the overall this binding rules are entirely based on the call-site, the global object is only eligible for the default binding if the contents of foo() are not running in

strict mode; the strict mode state of the call-site of foo() is irrelevant

Default Binding
Implicit Binding

Another rule to consider is whether the call-site has a context object, also referred to as an owning or containing object, though these alternate terms could be slightly misleading

```JavaScript
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2,
  foo: foo
};

obj.foo(); // 2
```

First, notice the manner in which foo() is declared and then later added as a reference property onto obj. Regardless of whether foo() is initially declared on foo, or is added as a reference later (as this snippet shows), in neither case is the function really “owned” or “contained” by the obj object.

However, the call-site uses the obj context to reference the function, so you could say that the obj object “owns” or “contains” the function reference at the time the function is called

at the point that foo() is called, it’s preceeded by an object reference to obj. When there is a context object for a function reference, the implicit binding rule says that it’s that object that should be used for the function call’s this binding. Because obj is the this for the foo() call, this.a is synonymous with obj.a

Explicit Binding

```JavaScript
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2
};

foo.call( obj ); // 2
```

Invoking foo with explicit binding by foo.call(..) allows us to force its this to be obj

Hard binding

```JavaScript
function foo() {
  console.log( this.a );
}

var obj = {
  a: 2
};

var bar = function() {
  foo.call( obj );
};

bar(); // 2

setTimeout( bar, 100 ); // 2 // hard-bound `bar` can no longer have its `this` overridden
bar.call( window ); // 2
```

Let’s examine how this variation works. We create a function bar() which, internally, manually calls foo.call(obj), thereby forcibly invoking foo with obj binding for this. No matter how you later invoke the function bar, it will always manually invoke foo with obj. This binding is both explicit and strong, so we call it hard binding.

The most typical way to wrap a function with a hard binding creates a pass-through of any arguments passed and any return value received

```JavaScript
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = function() {
  return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Another way to express this pattern is to create a reusable helper:

```JavaScript
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}
// simple `bind` helper
function bind(fn, obj) {
  return function() {
    return fn.apply( obj, arguments );
  };
}

var obj = {
  a: 2
};

var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3 c
onsole.log( b ); // 5

```
Since hard binding is such a common pattern, it’s provided with a builtin utility as of ES5, Function.prototype.bind, and it’s used like this:

```JavaScript
function foo(something) {
  console.log( this.a, something );
  return this.a + something;
}

var obj = {
  a: 2
};

var bar = foo.bind( obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
bind(..) returns a new function that is hardcoded to call the original function with the this context set as you specified. API call “contexts”
Many libraries’ functions, and indeed many new built-in functions in the JavaScript language and host environment, provide an optional parameter, usually called “context” which is designed as a workaround for you not having to use bind(..) to ensure your callback function uses a particular this.
For instance:
```JavaScript
function foo(el) {
  console.log( el, this.id );
}

var obj = {
  id: "awesome"
}; // use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome 2 awesome 3 awesome
```
Internally, these various functions almost certainly use explicit binding via call(..) or apply(..), saving you the trouble.

new Binding

First, let’s redefine what a “constructor” in JavaScript is. In JS, constructors are just functions that happen to be called with the new operator in front of them

When a function is invoked with new in front of it, otherwise known as a constructor call, the following things are done automatically

1. A brand new object is created (aka constructed) out of thin air.
2. The newly constructed object is [[Prototype]]-linked.
3. The newly constructed object is set as the this binding for that function call
4. Unless the function returns its own alternate object, the new invoked function call will automatically return the newly constructed object

```JavaScript
function foo(a) {
  this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

Now, we can summarize the rules for determining this from a function call’s call-site, in their order of precedence. Ask these questions in this order, and stop when the first rule applies.
1. Is the function called with new (new binding)? If so, this is the newly constructed object. `var bar = new foo()`
2. Is the function called with call or apply (explicit binding), even hidden inside a bind hard binding? If so, this is the explicitly specified object. `var bar = foo.call( obj2 )`
3. Is the function called with a context (implicit binding), otherwise known as an owning or containing object? If so, this is that con‐ text object. `var bar = obj1.foo()`
4. Otherwise, default the this (default binding). If in strict mode, pick undefined, otherwise pick the global object. `var bar = foo()`

That’s it. That’s all it takes to understand the rules of this binding for normal function calls. Well…almost

It’s quite common to use apply(..) for spreading out arrays of values as parameters to a function call. Similarly, bind(..) can curry parameters(preset values), which can be very helpful

```JavaScript
function foo(a,b) {
  console.log( "a:" + a + ", b:" + b );
}
// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3
// currying with `bind(..)`
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

Perhaps a somewhat “safer” practice is to pass a specifically set up object for this that is guaranteed not to be an object that can create problematic side effects in your program. Borrowing terminology from networking (and the military), we can create a “DMZ” (demilitarized zone) object

```JavaScript
function foo(a,b) {
  console.log( "a:" + a + ", b:" + b );
}
// our DMZ empty object
var ø = Object.create( null );
// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2
// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2
```

If you find yourself writing this-style code, but most or all the time, you defeat the this mechanism with lexical self = this or arrow function “tricks,” perhaps you should either:
1. Use only lexical scope and forget the false pretense of this-style code.
2. Embrace this-style mechanisms completely, including using bind(..) where necessary, and try to avoid self = this and arrow-function “lexical this” tricks

The literal syntax for an object looks like this:
```JavaScript
var myObj = {
  key: value
  // ...
}
```

The constructed form looks like this:

```JavaScript
var myObj = new Object();
myObj.key = value;
```

It’s extremely uncommon to use the “constructed form” for creating objects as just shown. You would pretty much always want to use the literal syntax form. The same will be true of most of the built-in objects (explained later).
Type Objects are the general building block upon which much of JS is built. They are one of the six primary types (called “language types” in the specification) in JS:
* string
* number
* boolean
* null
* undefined
* object

Note that the simple primitives (string, boolean, number, null, and undefined) are not themselves objects.
null is sometimes referred to as an object type, but this misconception stems from a bug in the language that causes typeof null to return the string "object" incorrectly (and confusingly). In fact, null is its own primitive type. It’s a common misstatement that “everything in JavaScript is an object.” This is clearly not true. By contrast, there are a few special object subtypes, which we can refer to as complex primitives. function is a subtype of object (technically, a “callable object”).

Functions in JS are said to be “first class” in that they are basically just normal objects (with callable behavior semantics bolted on), and so they can be handled like any other plain object

Luckily, the language automatically coerces a string primitive to a String object when necessary, which means you almost never need to explicitly create the Object form. It is strongly preferred by the majority of the JS community to use the literal form for a value, where possible, rather than the constructed object form

What is stored in the container are these property names, which act as pointers (technically, references) to where the values are stored.

The .a syntax is usually referred to as “property access,” whereas the ["a"] syntax is usually referred to as “key access"

```JavaScript
var myObject = {
  a: 2
};

var idx;

if (wantA) {
  idx = "a";
}
// later
console.log( myObject[idx] ); // 2
```

ES6 adds computed property names, where you can specify an expression, surrounded by a [ ] pair, in the key-name position of an object literal declaration:

```JavaScript
var prefix = "foo";
var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

The most common usage of computed property names will probably be for ES6 Symbols, which we will not be covering in detail in this book. In short, they’re a new primitive data type that has an opaque unguessable value (technically a string value). You will be strongly discouraged from working with the actual value of a Symbol (which can theoretically be different between different JS engines), so the name of the Symbol, like Symbol.Something (just a made up name!), will be what you use:
```JavaScript
var myObject = {
  [Symbol.Something]: "hello world"
};
```

Even when you declare a function expression as part of the object literal, that function doesn’t magically belong more to the object there are still just multiple references to the same function

Use objects to store key/value pairs, and arrays to store values at numeric indices

A shallow copy would end up with a on the new object as a copy of the value

```JavaScript
function anotherFunction() {
/*..*/
}

var anotherObject = {
  c: true
};

var anotherArray = [];
var myObject = {
  a: 2,
  b: anotherObject, // reference, not a copy!
  c: anotherArray, // another reference!
  d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

A shallow copy would end up with a on the new object as a copy of the value 2, but the b, c, and d properties as just references to the same places as the references in the original object. A deep copy would duplicate not only myObject, but anotherObject and anotherArray. But then we have the issue that anotherArray has references to anotherObject and myObject in it, so those should also be duplicated rather than reference-preserved. Now we have an infinite circular duplication problem because of the circular reference

One subset solution is that objects that are JSON-safe (that is, can be serialized to a JSON string and then reparsed to an object with the same structure and values) can easily be duplicated with:
```JavaScript
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

At the same time, a shallow copy is fairly understandable and has far fewer issues, so ES6 has now defined Object.assign(..) for this task

```JavaScript
var newObj = Object.assign( {}, myObject )
```

all properties are described in terms of a property descriptor.
Consider this code:
```JavaScript
var myObject = {
  a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {  
// value: 2,
// writable: true,
// enumerable: true,
// configurable: true
// }

```
As you can see, the property descriptor (called a “data descriptor” since it’s only for holding a data value) for our normal object property a is much more than just its value of 2. It includes three other characteristics: writable, enumerable, and configurable.
delete is only used to remove object properties

If an object property is the last remaining reference to some object/function, and you delete it, that removes the reference and now that unreferenced object/function can be garbage-collected

By combining writable:false and configurable:false, you can essentially create a constant (cannot be changed, redefined, or deleted) as an object property

If you want to prevent an object from having new properties added to it, but otherwise leave the rest of the object’s properties alone, call Object.preventExtensions

Object.seal(..) creates a “sealed” object, which means it takes an existing object and essentially calls Object.preventExtensions(..) on it, but also marks all its existing properties as configurable:false

Object.freeze(..) creates a frozen object, which means it takes an existing object and essentially calls Object.seal(..) on it, but it also marks all “data accessor” properties as writable:false, so that their values cannot be changed. This approach is the highest level of immutability
```JavaScript
var myObject = {
  a: 2
};

myObject.a; // 2
```
The myObject.a is a property access, but it doesn’t just look in myObject for a property of the name a, as it might seem. According to the spec, the previous code actually performs a [[Get]] operation (kinda like a function call: [[Get]]()) on the myObject

But one important result of this [[Get]] operation is that if it cannot through any means come up with a value for the requested property, it instead returns the value undefined:
```JavaScript
var myObject = {
  a: 2
};

myObject.b; // undefined

var myObject = {
  a: 2
};

("a" in myObject); // true
("b" in myObject); // false
myObject.hasOwnProperty( "a" ); // true
myObject.hasOwnProperty( "b" ); // false
```

for..in loops applied to arrays can give somewhat unexpected results, in that the enumeration of an array will include not only all the numeric indices, but also any enumerable proper‐ ties. It’s a good idea to use for..in loops only on objects, and traditional for loops with numeric index iteration for arrays

The for..in loop iterates over the list of enumerable properties on an object (including its [[Prototype]] chain

With numerically indexed arrays, iterating over the values is typically done with a standard for loop

forEach(..) will iterate over all values in the array, and it ignores any callback return values. every(..) keeps going until the end or the callback returns a false (or “falsy”) value, whereas some(..) keeps going until the end or the callback returns a true (or “truthy”) value

ES6 adds a for..of loop syntax for iterating over arrays (and objects, if the object defines its own custom iterator):
```JavaScript
var myArray = [ 1, 2, 3 ];
for (var v of myArray) {
  console.log( v );
} // 1 // 2 // 3

```
The for..of loop asks for an iterator object (from a default internal function known as @@iterator in spec-speak) of the thing to be iter‐ ated, and the loop then iterates over the successive return values from calling that iterator object’s next() method, once for each loop iteration


Class/inheritance describes a certain form of code organization and architecture—a way of modeling real world problem domains in our software

OO or class-oriented programming stresses that data intrinsically has associated behavior (of course, different depending on the type and nature of the data!) that operates on it, so proper design is to package up (aka encapsulate) the data and the behavior together. This is some‐ times called data structures in formal computer

Another key concept with classes is polymorphism, which describes the idea that a general behavior from a parent class can be overridden in a child class to give it more specifics. In fact, relative polymorphism lets us reference the base behavior from the overridden behavior

Depending on your level of formal education in programming, you may have heard of procedural programming as a way of describing code that only consists of procedures (aka functions) calling other functions, without any higher abstractions. You may have been taught that classes were the proper way to transform procedural-style “spaghetti code” into well-formed, well-organized code.

While we may have a syntax that looks like classes, it’s as if JavaScript mechanics are fighting against you using the class design pattern

Don’t let polymorphism confuse you into thinking a child class is linked to its parent class. A child class instead gets a copy of what it needs from the parent class. Class inheritance implies copies

JavaScript is simpler: it does not provide a native mechanism for “multiple inheritance.” Many see this is a good thing, because the complexity savings more than make up for the “reduced” functionality. But this doesn’t stop developers from trying to fake it in various ways, as we’ll see next

JavaScript’s object mechanism does not automatically perform copy behavior when you inherit or instantiate. Plainly, there are no “classes” in JavaScript to instantiate, only objects. And objects don’t get copied to other objects, they get linked together (more on that in Chapter 5). Since observed class behaviors in other languages imply copies, let’s examine how JS developers fake the missing copy behavior of classes in JavaScript: mixins.

Objects in JavaScript have an internal property, denoted in the specification as [[Prototype]], which is simply a reference to another object
```JavaScript
var anotherObject = {
  a: 2
};
// create an object linked to `anotherObject`
var myObject = Object.create( anotherObject );
myObject.a; // 2
```

So, we have myObject that is now [[Prototype]] linked to another Object. Clearly myObject.a doesn’t actually exist, but nevertheless, the property access succeeds

The top end of every normal [[Prototype]] chain is the built-in Object.prototype

If the property name foo ends up both on myObject itself and at a higher level of the [[Prototype]] chain that starts at myObject, this is called shadowing. The foo property directly on myObject shadows any foo property that appears higher in the chain, because the myObject.foo lookup would always find the foo property that’s lowest in the chain

Shadowing methods leads to ugly explicit pseudopolymorphism so you should try to avoid it if possible

In JavaScript, classes can’t (being that they don’t exist!) describe what an object can do. The object defines its own behavior directly. There’s just the object

```JavaScript
function Foo() {
  // ...
}

var a = new Foo();
Object.getPrototypeOf( a ) === Foo.prototype; // true
```

When a is created by calling new Foo(), one of the things that happens (see Chapter 2 for all four steps) is that a gets an internal [[Prototype]] link to the object that Foo.prototype is pointing at.

new Foo() results in a new object (we called it a), and that new object a is internally [[Prototype]]-linked to the Foo.prototype object.

We end up with two objects, linked to each other. That’s it. We didn’t instantiate a class. We certainly didn’t do any copying of behavior from a “class” into a concrete object. We just caused two objects to be linked to each other

In class-oriented languages, multiple copies (aka instances) of a class can be made, like stamping something out from a mold. As we saw in Chapter 4, this happens because the process of instantiating (or inheriting from) a class means, “copy the behavior plan from that class into a physical object,” and this is done again for each new instance. But in JavaScript, there are no such copy actions performed. You don’t create multiple instances of a class. You can create multiple objects that are [[Prototype]]-linked to a common object. But by default, no copying occurs, and thus these objects don’t end up totally

In class-oriented languages, multiple copies (aka instances) of a class can be made, like stamping something out from a mold. As we saw in Chapter 4, this happens because the process of instantiating (or inheriting from) a class means, “copy the behavior plan from that class into a physical object,” and this is done again for each new instance. But in JavaScript, there are no such copy actions performed. You don’t create multiple instances of a class. You can create multiple objects that are [[Prototype]]-linked to a common object. But by default, no copying occurs, and thus these objects don’t end up totally separate and disconnected from each other, but rather, quite linked

new Foo() is an indirect, roundabout way to end up with what we want: a new object linked to another object

Can we get what we want in a more direct way? Yes! The hero is Object.create(..)

In JavaScript, we don’t make copies from one object (“class”) to another (“instance”). We make links between objects. For the [[Prototype]] mechanism, visually, the arrows move from right to left, and from bottom to top:

This mechanism is often called prototypal inheritance (we’ll explore the code in detail shortly), which is commonly said to be the dynamic language version of classical inheritance

Inheritance implies a copy operation, and JavaScript doesn’t copy object properties (natively, by default). Instead, JS creates a link between two objects, where one object can essentially delegate property/function access to another object. Delegation (see Chapter 6) is a much more accurate term for JavaScript’s object-linking mechanism.

```JavaScript
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function() {
  return this.name;
};

var a = new Foo( "a" );
var b = new Foo( "b" );
a.myName(); // "a"
b.myName(); // "b"
```

This snippet shows two additional “class orientation” tricks in play:
1. this.name = name adds the .name property onto each object (a and b, respectively; see Chapter 2 about this binding), similar to how class instances encapsulate data values.

2. Foo.prototype.myName = ... is perhaps the more interesting technique; this adds a property (function) to the Foo.prototype object. Now, a.myName() works, but perhaps surprisingly.

So, by virtue of how they are created, a and b each end up with an internal [[Prototype]] linkage to Foo.prototype. When myName is not found on a or b, respectively, it’s instead found (through delegation; see Chapter 6) on Foo.prototype

```JavaScript
function Foo() {
  /* .. */
}

Foo.prototype = { /* .. */ };
// create a new prototype object
var a1 = new Foo();
a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

Object(..) didn’t “construct” a1, did it? It sure seems like Foo() “constructed” it. Most developers think of Foo() as doing the construction, but where everything falls apart is when you think “con‐ structor” means “was constructed by,” because by that reasoning, a1.constructor should be Foo, but it isn’t.

What’s happening? a1 has no .constructor property, so it delegates up the [[Prototype]] chain to Foo.prototype. But that object doesn’t have a .constructor either (like the default Foo.prototype object would have had!), so it keeps delegating, this time up to Object.prototype, the top of the delegation chain. That object indeed has a .constructor on it, which points to the built-in Object(..) function.

```JavaScript
function Foo() {
  /* .. */
}

Foo.prototype = {
  /* .. */
};

// create a new prototype object
// Need to properly "fix" the missing `.constructor`
// property on the new object serving as `Foo.prototype
// See Chapter 3 for `defineProperty(..)`.

Object.defineProperty( Foo.prototype, "constructor" , {
  enumerable: false,
  writable: true,
  configurable: true,
  value: Foo // point `.constructor` at `Foo`
});
```
That’s a lot of manual work to fix .constructor. Moreover, all we’re really doing is perpetuating the misconception that “constructor” means “was constructed by.” That’s an expensive illusion

a1.constructor is extremely unreliable, and it’s an unsafe reference to rely upon in your code. Generally, such references should be avoided where possible

And, here’s the typical “prototype-style” code that creates such links:
```JavaScript
function Foo(name) {
  this.name = name;
}

Foo.prototype.myName = function() {
  return this.name;
};

function Bar(name,label) {
  Foo.call( this, name );
  this.label = label;
}
// here, we make a new `Bar.prototype`
// linked to `Foo.prototype

Bar.prototype = Object.create( Foo.prototype );
// Beware! Now `Bar.prototype.constructor` is gone,
// and might need to be manually "fixed" if you're
// in the habit of relying on such properties!

Bar.prototype.myLabel = function() {
  return this.label;
};

var a = new Bar( "a", "obj a" );
a.myName(); // "a"
a.myLabel(); // "obj
```

The important part is `Bar.prototype = Object.create( Foo.prototype )`. The call to Object.create(..) creates a “new” object out of thin air, and links that new object’s internal [[Prototype]] to the object you specify (Foo.prototype in this case). In other words, that line says: “make a new Bar dot prototype object that’s linked to Foo dot prototype

```JavaScript
// doesn't work like you want!
Bar.prototype = Foo.prototype;

// works kinda like you want, but with
// side effects you probably don't want :(
Bar.prototype = new Foo();
```

`Bar.prototype = Foo.prototype` doesn’t create a new object for Bar.prototype to be linked to. It just makes Bar.prototype another reference to Foo.prototype, which effectively links Bar directly to the same object to which Foo links: Foo.prototype. This means when you start assigning, like `Bar.prototype.myLabel = ...`, you’re modifying not a separate object but the shared Foo.prototype object itself, which would affect any objects linked to Foo.prototype. This is almost certainly not what you want. If it is what you want, then you likely don’t need Bar at all, and should just use only Foo and make your code simpler

`Bar.prototype = new Foo()` does in fact create a new object that is duly linked to Foo.prototype as we’d want. But, it used the Foo(..) “constructor call” to do it. If that function has any side effects (such as logging, changing state, registering against other objects, adding data properties to this, etc.), those side effects happen at the time of this linking (and likely against the wrong object!), rather than only when the eventual Bar() “descendents” are created, as would likely be expected

So, we’re left with using Object.create(..) to make a new object that’s properly linked, but without having the side effects of calling Foo(..). The slight downside is that we have to create a new object, throwing the old one away, instead of modifying the existing default object we’re provided

It would be nice if there was a standard and reliable way to modify the linkage of an existing object. Prior to ES6, there’s a nonstandard and not fully cross-browser way, via the `.__proto__` property, which is settable. ES6 adds a `Object.setPrototypeOf(..)` helper utility, which does the trick in a standard and predictable

```JavaScript
// pre-ES6
// throws away default existing `Bar.prototype`
Bar.prototype = Object.create( Foo.prototype );

// ES6+
// modifies existing `Bar.prototype`
Object.setPrototypeOf( Bar.prototype, Foo.prototype );
```

```JavaScript
// helper utility to see if `o1` is
// related to (delegates to) `o2`
function isRelatedTo(o1, o2) {
  function F(){}
  F.prototype = o2;
  return o1 instanceof F;
}

var a = {};
var b = Object.create( a );
isRelatedTo( b, a ); // true
```

we borrow a throwaway function F, reassign its .prototype to arbitrarily point to some object o2, and then ask if o1 is an “instance of” F.

The second, and much cleaner, approach to [[Prototype]] reflection is:
```JavaScript
Foo.prototype.isPrototypeOf( a ); // true
```

==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 104-104 | Added on Friday, December 2, 2016 7:39:00 AM

Notice that in this case, we don’t really care (or even need) Foo, we just need an object (in our case, arbitrarily labeled Foo.prototype
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 104-104 | Added on Friday, December 2, 2016 7:39:19 AM

Notice that in this case, we don’t really care (or even need) Foo, we just need an object (in our case, arbitrarily labeled Foo.prototype) to test
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 105-105 | Added on Friday, December 2, 2016 7:39:26 AM

against another object. The question isPrototypeOf(..) answers is: in the entire [[Prototype]] chain of a, does Foo.prototype ever appear?
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 105-105 | Added on Friday, December 2, 2016 7:40:20 AM

our isRelatedTo(..) utility is built in to the language, and it’s called isPrototypeOf
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 105-105 | Added on Friday, December 2, 2016 7:40:54 AM

Most browsers (not all!) have also long supported a nonstandard al‐ ternate way of accessing the internal [[Prototype]]: a.__proto__ === Foo.prototype; // true
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 105-105 | Added on Friday, December 2, 2016 7:41:20 AM

The strange .__proto__ (not standardized until ES6!) property “mag‐ ically” retrieves the internal [[Prototype]] of an object as a reference, which is quite helpful if you want to directly inspect (or even tra‐ verse: .__proto__.__proto__...) the chain.
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 106-106 | Added on Friday, December 2, 2016 7:42:38 AM

we could envision .__proto__ implemented (see Chapter 3 for object property definitions) like this: Object.defineProperty( Object.prototype, "__proto__", { get: function() { return Object.getPrototypeOf( this ); }, set: function(o) { // setPrototypeOf(..) as of ES6 Object.setPrototypeOf( this, o ); return o; } } );
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 107-107 | Added on Friday, December 2, 2016 7:46:39 AM

What’s the point of the [[Prototype]] mechanism? Why is it so com‐ mon for JS developers to go to so much effort (emulating classes) in their code to wire up these linkages? Remember we said much earlier in this chapter that Object.cre ate(..) would be a hero? Now, we’re ready to see how: var foo = { something: function() { console.log( "Tell me something
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 107-107 | Added on Friday, December 2, 2016 7:47:03 AM

What’s the point of the [[Prototype]] mechanism? Why is it so com‐ mon for JS developers to go to so much effort (emulating classes) in their code to wire up these linkages? Remember we said much earlier in this chapter that Object.cre ate(..) would be a hero? Now, we’re ready to see how: var foo = { something: function() { console.log( "Tell me something good..." ); } }; var bar = Object.create( foo ); bar.something(); // Tell me something good... Object.create(..) creates a new object (bar) linked to the object we specified (foo), which gives us all the power (delegation) of the [[Pro totype]] mechanism, but without any of the unnecessary complica‐ tion of new functions acting as classes and constructor calls
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 108-108 | Added on Friday, December 2, 2016 7:48:18 AM

Object.create(null) creates an object that has an empty (aka null) [[Prototype]] linkage, and thus the object can’t dele‐ gate anywhere. Since such an object has no prototype chain, the instanceof operator (explained earlier) has nothing to check, so it will always return false. These special empty- [[Prototype]] objects are often called “dictionaries,” as they are typically used purely for storing data in properties, most‐ ly because they have no possible surprise effects from any delegated properties/functions on the [[Prototype]] chain, and are thus purely flat data storage
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 108-108 | Added on Friday, December 2, 2016 7:49:42 AM

if (!Object.create) { Object.create = function(o) { function F(){} F.prototype = o; return new F(); }; } This polyfill works by using a throwaway F function, and
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 108-108 | Added on Friday, December 2, 2016 7:49:57 AM

if (!Object.create) { Object.create = function(o) { function F(){} F.prototype = o; return new F(); }; } This polyfill works by using a throwaway F function, and we override its .prototype property to point to the object we want to link to. Then we use new F() construction to make a new object that will be linked as we specified
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 110-110 | Added on Friday, December 2, 2016 7:51:29 AM

It may be tempting to think that these links between objects primari‐ ly provide a sort of fallback for “missing” properties or methods. While that may be an observed outcome, I don’t think it represents the right way of thinking about [[Prototype
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 111-111 | Added on Friday, December 2, 2016 7:52:44 AM

Don’t miss an important but nuanced point here. Designing software where you intend for a developer to, for instance, call myObject.cool() and have that work even though there is no cool() method on myObject, introduces some “magic” into your API design that can be surprising for future developers who maintain your software
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 111-111 | Added on Friday, December 2, 2016 7:53:13 AM

You can however design your API with less “magic” to it, but still take
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 111-111 | Added on Friday, December 2, 2016 7:53:38 AM

advantage of the power of [[Prototype]] linkage: var anotherObject = { cool: function() { console.log( "cool!" ); } }; var myObject = Object.create( anotherObject ); myObject.doCool = function() { this.cool(); // internal delegation! }; myObject.doCool(); // "cool!" Here, we call myObject.doCool(), which is a method that actually exists on myObject, making our API design more explicit (less “mag‐ ical”). Internally, our implementation follows the delegation design pattern (see Chapter 6), taking advantage of [[Prototype]] delega‐ tion to anotherObject.cool
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 111-111 | Added on Friday, December 2, 2016 7:54:11 AM

In other words, delegation will tend to be less surprising/confusing if it’s an internal implementation detail rather than plainly exposed in your API interface design. We will expound on delegation in great detail in the next chapter
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 113-113 | Added on Friday, December 2, 2016 10:26:32 PM

As a brief review of our conclusions from Chapter 5, the [[Proto type]] mechanism is an internal link that exists on one object that references another object
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 114-114 | Added on Friday, December 2, 2016 10:27:21 PM

In other words, the actual mechanism, the essence of what’s important to the functionality we can leverage in JavaScript, is all about objects being linked to other objects.
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 115-115 | Added on Saturday, December 3, 2016 12:29:45 PM

Importantly, the class design pattern encourages you to employ meth‐ od overriding (and polymorphism) to get the most out of inheritance, where you override the definition of some general Task method in your XYZ task
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 115-115 | Added on Saturday, December 3, 2016 12:30:53 PM

But now let’s try to think about the same problem domain, using behavior delegation instead of classes
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 115-115 | Added on Saturday, December 3, 2016 12:31:18 PM

You will first define an object (not a class, nor a function as most JSers would lead you to believe) called Task, and it will have concrete be‐ havior on it that includes utility methods that various tasks can use
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 116-116 | Added on Saturday, December 3, 2016 12:32:08 PM

read: delegate to!). Then, for each task (“XYZ,” “ABC”), you define an object to hold that task-specific data/behavior. You link your taskspecific object(s) to the Task utility object, allowing them to delegate to it when they need to
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 116-116 | Added on Saturday, December 3, 2016 12:32:33 PM

Basically, think about needing behaviors from two sibling/peer objects (XYZ and Task) to perform task “XYZ.” But rather than needing to compose them together, via class copies, we can keep them in their separate objects, and we can allow the XYZ object to delegate to Task when needed
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 116-116 | Added on Saturday, December 3, 2016 12:34:19 PM

Here’s some simple code to suggest how you accomplish that: Task = { setID: function(ID) { this.id = ID; }, outputID: function() { console.log( this.id );
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 116-116 | Added on Saturday, December 3, 2016 12:34:30 PM

}; // make `XYZ` delegate to `Task` XYZ = Object.create( Task ); XYZ.prepareTask = function(ID,Label) { this.setID( ID ); this.label = Label; }; XYZ.outputTaskDetails = function() { this.outputID(); console.log( this.label ); }; // ABC = Object.create( Task ); // ABC ... = ... In this code, Task and XYZ are not classes (or functions), they’re just objects. XYZ is set up via Object.create(..) to [[Prototype]]- delegate to the Task object
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 116-116 | Added on Saturday, December 3, 2016 12:34:57 PM

Here’s some simple code to suggest how you accomplish that: Task = { setID: function(ID) { this.id = ID; }, outputID: function() { console.log( this.id ); } }; // make `XYZ` delegate to `Task` XYZ = Object.create( Task ); XYZ.prepareTask = function(ID,Label) { this.setID( ID ); this.label = Label; }; XYZ.outputTaskDetails = function() { this.outputID(); console.log( this.label ); }; // ABC = Object.create( Task ); // ABC ... = ... In this code, Task and XYZ are not classes (or functions), they’re just objects. XYZ is set up via Object.create(..) to [[Prototype]]- delegate to the Task object (see Chapter 5). As compared to class orientation (aka object orientation), I call this style of code OLOO (objects linked to other objects). All we really care about is that the XYZ object delegates to the Task object (as does the ABC object
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 117-117 | Added on Saturday, December 3, 2016 12:36:30 PM

1. Both the id and label data members from the previous class ex‐ ample are data properties directly on XYZ (neither is on Task). In general, with [[Prototype]] delegation, you want state to be on the delegators (XYZ, ABC), not on the delegate (Task
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 117-117 | Added on Saturday, December 3, 2016 12:37:07 PM

. With the class design pattern, we intentionally named output Task the same on both parent (Task) and child (XYZ), so that we could take advantage of overriding (polymorphism). In behavior delegation, we do the opposite: we avoid if at all possible naming things the same at different levels of the [[Prototype]] chain
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 117-117 | Added on Saturday, December 3, 2016 12:37:22 PM

called shadowing—see Chapter 5), because having those name collisions creates awkward/brittle syntax to disambiguate refer‐ ences (see Chapter 4), and we want to avoid that if we can.
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 117-117 | Added on Saturday, December 3, 2016 12:37:45 PM

This design pattern calls for less use of general method names that are prone to overriding and instead more use of descriptive meth‐ od names, specific to the type of behavior each object is doing. This can actually create easier to understand/maintain code, because the names of methods (not only at the definition location but strewn throughout other code) are more obvious (selfdocumenting).
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 117-117 | Added on Saturday, December 3, 2016 12:38:20 PM

this.setID(ID); inside of a method on the XYZ object first looks on XYZ for setID(..), but since it doesn’t find a method of that name on XYZ, [[Prototype]] delegation means it can follow the link to Task to look for setID(..), which it of course finds. More‐ over, because of implicit call-site this binding rules (see Chap‐ ter 2), when setID(..) runs, even though the method was found on Task, the this binding for that function call is XYZ, exactly
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 117-117 | Added on Saturday, December 3, 2016 12:38:35 PM

In other words, the general utility methods that exist on Task are available to us while interacting with XYZ, because XYZ can delegate to Task
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 118-118 | Added on Saturday, December 3, 2016 12:39:25 PM

This is an extremely powerful design pattern, very distinct from the ideas of parent and child classes, inheritance, polymorphism, etc. Rather than organizing the objects in your mind vertically, with pa‐ rents flowing down to children, think of objects side by side, as peers, with any direction of delegation links between the objects as necessary
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 121-121 | Added on Sunday, December 4, 2016 6:20:49 PM

OO style: function Foo(who) { this.me = who; } Foo.prototype.identify = function() {
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 121-121 | Added on Sunday, December 4, 2016 6:20:59 PM

return "I am " + this.me; }; function Bar(who) { Foo.call( this, who ); } Bar.prototype = Object.create( Foo.prototype ); Bar.prototype.speak = function() { alert( "Hello, " + this.identify() + "." ); }; var b1 = new Bar( "b1" ); var b2 = new Bar( "b2" ); b1.speak(); b2.speak(
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 121-121 | Added on Sunday, December 4, 2016 6:21:13 PM

Now, let’s implement the exact same functionality using OLOO-style code: Foo = { init: function(who) { this.me = who; }, identify: function() { return "I am " + this.me; }
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 122-122 | Added on Sunday, December 4, 2016 6:21:21 PM

}; Bar = Object.create( Foo ); Bar.speak = function() { alert( "Hello, " + this.identify() + "." ); }; var b1 = Object.create( Bar ); b1.init( "b1" ); var b2 = Object.create( Bar ); b2.init( "b2" ); b1.speak(); b2.speak
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 122-122 | Added on Sunday, December 4, 2016 6:21:32 PM

We take exactly the same advantage of [[Prototype]] delegation from b1 to Bar to Foo as we did in the previous snippet between b1,
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 122-122 | Added on Sunday, December 4, 2016 6:21:39 PM

Bar.pro totype, and Foo.prototype. We still have the same three objects linked together
==========
You Don't Know JS - This _ Object Prototypes
- Your Bookmark on page 123 | Added on Sunday, December 4, 2016 6:23:39 PM


==========
You Don't Know JS - This _ Object Prototypes
- Your Bookmark on page 124 | Added on Sunday, December 4, 2016 6:25:20 PM


==========
You Don't Know JS - This _ Object Prototypes
- Your Bookmark on page 125 | Added on Sunday, December 4, 2016 6:27:13 PM


==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 125-125 | Added on Sunday, December 4, 2016 6:27:35 PM

because OLOO-style code embraces the fact that the only thing we ever really cared about was the objects linked to other objects
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 128-128 | Added on Sunday, December 4, 2016 6:33:45 PM

Whether you use the classic prototypal syntax or the new ES6 sugar, you’ve still made a choice to model the problem domain (UI widgets) with “classes.” And as the previous few chapters try to demonstrate, this choice in JavaScript is opting you into extra headaches and mental tax
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 130-130 | Added on Sunday, December 4, 2016 6:36:59 PM

With this OLOO-style approach, we don’t think of Widget as a parent and Button as a child. Rather, Widget is just an object and is sort of a utility collection that any specific type of widget might want to delegate to, and Button is also just a standalone object (with a delegation link to Widget, of course
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 130-130 | Added on Sunday, December 4, 2016 6:37:20 PM

From a design pattern perspective, we didn’t share the same method name render(..) in both objects, the way classes suggest, but instead we chose different names (insert(..) and build(..)) that were more descriptive of what task each does specifically. The initialization methods are called init(..) and setup(..), respectively, for the same reasons
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 130-130 | Added on Sunday, December 4, 2016 6:38:03 PM

Syntactically, we also don’t have any constructors, .prototype, or new present, as they are, in fact, just unnecessary cruft
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 130-130 | Added on Sunday, December 4, 2016 6:38:47 PM

Now, if you’re paying close attention, you may notice that what was previously just one call (var btn1 = new Button(..)) is now two calls (var btn1 = Object.create(Button) and btn1.setup(..)). Initially this may seem like a drawback (more code). However, even this is something that’s a pro of OLOO-style code as compared to classical prototype style code. How? With class constructors, you are forced (not really, but it is strongly suggested) to do both construction and initialization in the same step. However, there are many cases where being able to do these two steps
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 130-130 | Added on Sunday, December 4, 2016 6:38:55 PM

separately (as you do with OLOO!) is more flexible. For example, let’s say you create all your instances in a pool at the beginning of your program, but you wait to initialize them with a specific setup when they are pulled from the pool and used. We showed the two calls happening right next to each other, but of course they can happen at very different times and in very different parts of our code, as needed
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 131-131 | Added on Sunday, December 4, 2016 6:39:33 PM

OLOO better supports the principle of separation of concerns, where creation and initialization are not necessarily conflated into the same operation
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 145-145 | Added on Monday, December 5, 2016 7:35:13 AM

Chapter 4 points out that classes in traditional class-oriented languages actually pro‐ duce a copy action from parent to child to instance, whereas in [[Pro totype]], the action is not a copy, but rather the opposite—a delega
==========
You Don't Know JS - This _ Object Prototypes
- Your Highlight on page 145-145 | Added on Monday, December 5, 2016 7:35:20 AM

tion link
