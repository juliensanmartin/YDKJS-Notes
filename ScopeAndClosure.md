#Scope And Closure

## Chapter 1: What is Scope?

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program state

`Tokenizing/Lexing` : Breaking up a string of characters into meaningful (to the language) chunks, called tokens

The difference between `tokenizing` and `lexing` is subtle and academic, but it centers on whether or not these tokens are identified in a `stateless` or `stateful` way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether a should be considered a distinct token or just part of another token, that would be lexing.

Parsing taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an “AST” (abstract syntax tree). The tree for `var a = 2;` might start with a top-level node called `VariableDeclaration`, with a child node called `Identifier` (whose value is a), and another child called `AssignmentExpression`, which itself has a child called `NumericLiteral` (whose value is 2).

`Code-Generation` : The process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it’s targeting, and so on. So, rather than get mired in details, we’ll just handwave and say that there’s a way to take our previously described AST for `var a = 2;` and turn it into a set of machine instructions to actually create 2.

The `JavaScript engine` is vastly more complex than just those three steps, as are most other language compilers. For instance, in the process of parsing and code-generation, there are certainly steps to optimize the performance of the execution, including collapsing redundant elements, etc. So, I’m painting only with broad strokes here. But I think you’ll see shortly why these details we do cover, even at a high level, are relevant. For one thing, JavaScript engines don’t get the luxury (like other language compilers) of `having plenty of time to optimize`, because JavaScript compilation doesn’t happen in a build step ahead of time, as with other languages. For JavaScript, the compilation that occurs happens, in many cases, mere microseconds (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like `JITs`, which lazy compile and even hot recompile, etc.) that are well beyond the “scope” of our discussion here. Let’s just say, for simplicity sake, that any snippet of JavaScript has to be compiled before (usually right before!) it’s executed. So, the JS compiler will take the program `var a = 2;` and compile it first, and then be ready to execute it, usually right away.

### Understanding Scope
The way we will approach learning about `scope` is to think of the process in terms of a conversation. But, who is having the conversation? Let’s meet the cast of characters that interact to process the program `var a = 2;`, so we understand their conversations that we’ll listen in on shortly.

1. `Engine`: responsible for start-to-finish compilation and execution of our JavaScript program.

2. `Compiler`: one of Engine's friends; handles all the dirty work of parsing and code-generation (see previous section).

3. `Scope` Another friend of `Engine`; collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code

`Engine` sees two distinct statements, one that `Compiler` will handle during compilation, and one that `Engine` will handle during execution. So, let’s break down how `Engine` and friends will approach the program `var a = 2;`. The first thing `Compiler` will do with this program is perform `lexing` to break it down into `tokens`, which it will then `parse` into a `tree`. But when `Compiler` gets to code generation, it will treat this program somewhat differently than perhaps assumed. A reasonable assumption would be that `Compiler` will produce code that could be summed up by this pseudocode: `Allocate memory for a variable, label it a, then stick the value 2 into that variable.` Unfortunately, that’s not quite accurate. `Compiler` will instead proceed as:
1. Encountering var a, `Compiler` asks `Scope` to see if a variable a already exists for that particular scope collection. If so, `Compiler` ignores this declaration and moves on. Otherwise, `Compiler` asks `Scope` to declare a new variable called `a` for that scope collection.
2. `Compiler` then produces code for `Engine` to later execute, to handle the `a = 2` assignment.

To summarize: two distinct actions are taken for a `variable assignment`: First, `Compiler` declares a variable (if not previously declared) in the current `Scope`, and second, when executing, `Engine` looks up the variable in `Scope` and assigns to it, if found.

In our case, it is said that `Engine` would be performing an `LHS lookup` for the variable `a`. The other type of look-up is called `RHS`
In other words, an `LHS` look-up is done when a variable appears on the `lefthand side of an assignment operation`, and an `RHS` look-up is done when a variable appears on the `righthand side of an assignment operation`

Being slightly glib for a moment, you could think RHS instead means "retrieve his/her source (value)," implying that RHS means "go get the value of…"

When I say: `console.log( a );` The reference to `a` is an RHS reference, because nothing is being assigned to `a` here. Instead, we’re looking up to retrieve the value of `a`, so that the value can be passed to `console.log(..)`. By contrast: `a = 2;` The reference to `a` here is an LHS reference, because we don’t actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.
"Who’s the target of the assignment (LHS)?"" and "Who’s the source of the assignment (RHS)?"

#### Nested Scope
Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, Engine consults the next outer containing scope, continuing until is found or until the outermost (a.k.a., global) scope has been reached.

The simple rules for traversing nested scope: Engine starts at the currently executing scope, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

`Strict Mode` which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global scoped variable to hand back from an LHS look-up, and Engine would throw a `ReferenceError` similarly to the RHS case

## Chapter 2: Lexical Scope

There are two predominant models for how scope works. The first of these is by far the most common, used by the vast majority of programming languages. It’s called `lexical scope`, and we will examine it in depth. The other model, which is still used by some languages (such as Bash scripting, some modes in Perl, etc) is called dynamic scope.

To define it somewhat circularly, `lexical scope is scope that is defined at lexing time`. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code

Scope look-up stops once it finds the first match. The same identifier name can be specified at multiple layers of nested scope, which is called `shadowing` (the inner identifier “shadows” the outer identifier)

Global variables are automatically also properties of the global object (window in browsers, etc.), so it is possible to reference a global variable not directly by its lexical name, but instead indirectly as a property reference of the global object. window.a This technique gives access to a global variable that would otherwise be inaccessible due to it being shadowed. However, non-global shadowed variables cannot be accessed no matter where a function is invoked from, or even how it is invoked, its lexical scope is only defined by where the function was declared

### Performance

The lexical scope look-up process only applies to first-class identifiers, such as the a, b, and c. If you had a reference to foo.bar.baz in a piece of code, the lexical scope look-up would apply to finding the foo identifier, but once it locates that variable, object property-access rules take over to resolve the bar and baz properties, respectively

The JavaScript engine has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and predetermine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution

## Chapter 3: Function vs. Block Scope

As we explored in Chapter 2, scope consists of a series of "bubbles" that each act as a container or bucket, in which identifiers (variables, functions) are declared. These bubbles nest neatly inside each other, and this nesting is defined at author-time.

### Scope From Functions

The most common answer to those questions is that JavaScript has function-based scope. That is, each function you declare creates a bubble for itself, but no other structures create their own scope bubbles. As we'll see in just a little bit, this is not quite true.

Consider this code:
```JavaScript
function foo(a) {
    var b = 2;

    // some code

    function bar() {
        // ...
    }

    // more code

    var c = 3;
}
```
In this snippet, the scope bubble for foo(..) includes identifiers a, b, c and bar. It doesn't matter where in the scope a declaration appears, the variable or function belongs to the containing scope bubble, regardless. We'll explore how exactly that works in the next chapter.

bar(..) has its own scope bubble. So does the global scope, which has just one identifier attached to it: foo.

Because a, b, c, and bar all belong to the scope bubble of foo(..), they are not accessible outside of foo(..). That is, the following code would all result in ReferenceError errors, as the identifiers are not available to the global scope:
```JavaScript
bar(); // fails

console.log( a, b, c ); // all 3 fail
```
However, all these identifiers (a, b, c, foo, and bar) are accessible inside of foo(..), and indeed also available inside of bar(..) (assuming there are no shadow identifier declarations inside bar(..)).

Function scope encourages the idea that all variables belong to the function, and can be used and reused throughout the entirety of the function (and indeed, accessible even to nested scopes). This design approach can be quite useful, and certainly can make full use of the "dynamic" nature of JavaScript variables to take on values of different types as needed.

On the other hand, if you don't take careful precautions, variables existing across the entirety of a scope can lead to some unexpected pitfalls.

### Hiding In Plain Scope

The traditional way of thinking about functions is that you declare a function, and then add code inside it. But the inverse thinking is equally powerful and useful: take any arbitrary section of code you’ve written and wrap a function declaration around it, which in effect “hides” the code

The practical result is to create a scope bubble around the code in question, which means that any declarations (variable or function) in that code will now be tied to the scope of the new wrapping function, rather than the previously enclosing scope. In other words, you can "hide" variables and functions by enclosing them in the scope of a function.

There’s a variety of reasons motivating this scope-based hiding. They tend to arise from the software design principle `Principle of Least Privilege` :
This principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and “hide” everything else

This principle extends to the choice of which scope to contain variables and functions. If all variables and functions were in the global scope, they would of course be accessible to any nested scope. But this would violate the "Least..." principle in that you are (likely) exposing many variables or functions which you should otherwise keep private, as proper use of the code would discourage access to those variables/functions.

For example:
```JavaScript
function doSomething(a) {
    b = a + doSomethingElse( a * 2 );

    console.log( b * 3 );
}

function doSomethingElse(a) {
    return a - 1;
}

var b;

doSomething( 2 ); // 15
```

In this snippet, the b variable and the doSomethingElse(..) function are likely "private" details of how doSomething(..) does its job. Giving the enclosing scope "access" to b and doSomethingElse(..) is not only unnecessary but also possibly "dangerous", in that they may be used in unexpected ways, intentionally or not, and this may violate pre-condition assumptions of doSomething(..).

A more "proper" design would hide these private details inside the scope of doSomething(..), such as:

```JavaScript
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }

    var b;

    b = a + doSomethingElse( a * 2 );

    console.log( b * 3 );
}

doSomething( 2 ); // 15
```

Now, b and doSomethingElse(..) are not accessible to any outside influence, instead controlled only by doSomething(..). The functionality and end-result has not been affected, but the design keeps private details private, which is usually considered better software.


Such libraries typically will create a single variable declaration, often an object, with a sufficiently unique name, in the global scope. This object is then used as a namespace for that library, where all specific exposures of functionality are made as properties off that object (namespace), rather than as top-level lexically scoped identifiers themselves. For example:

```JavaScript
var MyReallyCoolLibrary = {
  awesome: "stuff",
  doSomething: function() {
    // ...
  },
  doAnotherThing: function() {
    // ...
  }
};
```


#### Module Management

Another option for collision avoidance is the more modern "module" approach, using any of various dependency managers. Using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifier(s) be explicitly imported into another specific scope through usage of the dependency manager's various mechanisms.

It should be observed that these tools do not possess "magic" functionality that is exempt from lexical scoping rules. They simply use the rules of scoping as explained here to enforce that no identifiers are injected into any shared scope, and are instead kept in private, non-collision-susceptible scopes, which prevents any accidental scope collisions.

As such, you can code defensively and achieve the same results as the dependency managers do without actually needing to use them, if you so choose. See the Chapter 5 for more information about the module pattern.

### Functions As Scopes

We've seen that we can take any snippet of code and wrap a function around it, and that effectively "hides" any enclosed variable or function declarations from the outside scope inside that function's inner scope.

For example:

```JavaScript
var a = 2;
function foo() {
  // <-- insert this
  var a = 3;
  console.log( a ); // 3
} // <-- and this
foo(); // <-- and this
console.log( a ); // 2
```

While this technique works, it is not necessarily very ideal. There are a few problems it introduces. The first is that we have to declare a named-function foo(), which means that the identifier name foo itself “pollutes” the enclosing scope (global, in this case). We also have to explicitly call the function by name (foo()) so that the wrapped code actually executes.

```JavaScript
var a = 2;
(function foo(){ // <-- insert this
  var a = 3;
  console.log( a ); /
})(); // <-- and this
console.log( a ); //
```
Instead of treating the function as a standard declaration, the function is treated as a `function expression`

The easiest way to distinguish `declaration` vs. `expression` is the position of the word function in the statement (not just a line, but a distinct statement). If function is the very first thing in the statement, then it’s a function declaration. Otherwise, it’s a function expression. The key difference we can observe here between a function declaration and a function expression relates to where its name is bound as an identifier. Compare the previous two snippets. In the first snippet, the name foo is bound in the enclosing scope, and we call it directly with foo(). In the second snippet, the name foo is not bound in the enclosing scope, but instead is bound only inside of its own function. In other words, `(function foo(){ .. })` as an expression means the identifier foo is found only in the scope where the .. indicates, not in the outer scope. Hiding the name foo inside itself means it does not pollute the enclosing scope unnecessarily. Anonymous Versus Named You are probably most familiar with function expressions as callback parameters, such as:
```JavaScript
setTimeout( function(){
  console.log("I waited 1 second!");
}, 1000 );
```
This is called an `anonymous function expression`, because function() has no name identifier on it. Function expressions can be anonymous, but function declarations cannot omit the name—that would be illegal JS grammar.

The best practice is to `always name your function expressions`:
```JavaScript
setTimeout( function timeoutHandler(){ // <-- Look, I have a // name!
  console.log( "I waited 1 second!" );
}, 1000 );
```

Now that we have a function as an expression by virtue of wrapping it in a ( ) pair, we can execute that function by adding another () on the end, like `(function foo(){ .. })()`. The first enclosing ( ) pair makes the function an expression, and the second () executes the function.

### Blocks As Scopes

While functions are the most common unit of scope, and certainly the most wide-spread of the design approaches in the majority of JS in circulation, other units of scope are possible, and the usage of these other scope units can lead to even better, cleaner to maintain code.

Many languages other than JavaScript support Block Scope, and so developers from those languages are accustomed to the mindset, whereas those who've primarily only worked in JavaScript may find the concept slightly foreign.

But even if you've never written a single line of code in block-scoped fashion, you are still probably familiar with this extremely common idiom in JavaScript:
```JavaScript
for (var i=0; i<10; i++) {
    console.log( i );
}
```
We declare the variable i directly inside the for-loop head, most likely because our intent is to use i only within the context of that for-loop, and essentially ignore the fact that the variable actually scopes itself to the enclosing scope (function or global).

That's what block-scoping is all about. Declaring variables as close as possible, as local as possible, to where they will be used. Another example:
```JavaScript
var foo = true;

if (foo) {
    var bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}
```
We are using a bar variable only in the context of the if-statement, so it makes a kind of sense that we would declare it inside the if-block. However, where we declare variables is not relevant when using var, because they will always belong to the enclosing scope. This snippet is essentially "fake" block-scoping, for stylistic reasons, and relying on self-enforcement not to accidentally use bar in another place in that scope.

Block scope is a tool to extend the earlier "Principle of Least Privilege Exposure" [^note-leastprivilege] from hiding information in functions to hiding information in blocks of our code.

Consider the for-loop example again:
```JavaScript
for (var i=0; i<10; i++) {
    console.log( i );
}
```
Why pollute the entire scope of a function with the i variable that is only going to be (or only should be, at least) used for the for-loop?

But more importantly, developers may prefer to check themselves against accidentally (re)using variables outside of their intended purpose, such as being issued an error about an unknown variable if you try to use it in the wrong place. Block-scoping (if it were possible) for the i variable would make i available only for the for-loop, causing an error if i is accessed elsewhere in the function. This helps ensure variables are not re-used in confusing or hard-to-maintain ways.

But, the sad reality is that, on the surface, JavaScript has no facility for block scope.

That is, until you dig a little further.

#### try/catch

It's a very little known fact that JavaScript in ES3 specified the variable declaration in the catch clause of a try/catch to be block-scoped to the catch block.

#### let

Thus far, we've seen that JavaScript only has some strange niche behaviors which expose block scope functionality. If that were all we had, and it was for many, many years, then block scoping would not be terribly useful to the JavaScript developer.

Fortunately, ES6 changes that, and introduces a new keyword let which sits alongside var as another way to declare variables.

The let keyword attaches the variable declaration to the scope of whatever block (commonly a { .. } pair) it's contained in. In other words, let implicitly hijacks any block's scope for its variable declaration.

```JavaScript
var foo = true;

if (foo) {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}


console.log( bar ); // ReferenceError
```
Using let to attach a variable to an existing block is somewhat implicit. It can confuse you if you're not paying close attention to which blocks have variables scoped to them, and are in the habit of moving blocks around, wrapping them in other blocks, etc., as you develop and evolve code.

Creating explicit blocks for block-scoping can address some of these concerns, making it more obvious where variables are attached and not. Usually, explicit code is preferable over implicit or subtle code. This explicit block-scoping style is easy to achieve, and fits more naturally with how block-scoping works in other languages:
```JavaScript
var foo = true;

if (foo) {
    { // <-- explicit block
        let bar = foo * 2;
        bar = something( bar );
        console.log( bar );
    }
}

console.log( bar ); // ReferenceError
```
We can create an arbitrary block for let to bind to by simply including a { .. } pair anywhere a statement is valid grammar. In this case, we've made an explicit block inside the if-statement, which may be easier as a whole block to move around later in refactoring, without affecting the position and semantics of the enclosing if-statement.

Note: For another way to express explicit block scopes, see Appendix B.

In Chapter 4, we will address hoisting, which talks about declarations being taken as existing for the entire scope in which they occur.

However, declarations made with let will not hoist to the entire scope of the block they appear in. Such declarations will not observably "exist" in the block until the declaration statement.
```JavaScript
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

### Garbage Collection

Another reason block-scoping is useful relates to closures and garbage collection to reclaim memory. We'll briefly illustrate here, but the closure mechanism is explained in detail in Chapter 5.

Consider:
```JavaScript
function process(data) {
    // do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturingPhase=*/false );
```

The click function click handler callback doesn't need the someReallyBigData variable at all. That means, theoretically, after process(..) runs, the big memory-heavy data structure could be garbage collected. However, it's quite likely (though implementation dependent) that the JS engine will still have to keep the structure around, since the click function has a closure over the entire scope.

Block-scoping can address this concern, making it clearer to the engine that it does not need to keep someReallyBigData around:

```JavaScript
function process(data) {
    // do something interesting
}

// anything declared inside this block can go away after!
{
    let someReallyBigData = { .. };

    process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturingPhase=*/false );
```
Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox.


#### let Loops

A particular case where let shines is in the for-loop case as we discussed previously.
```JavaScript
for (let i=0; i<10; i++) {
    console.log( i );
}

console.log( i ); // ReferenceError
```
Not only does let in the for-loop header bind the i to the for-loop body, but in fact, it re-binds it to each iteration of the loop, making sure to re-assign it the value from the end of the previous loop iteration.

Here's another way of illustrating the per-iteration binding behavior that occurs:

```JavaScript
{
    let j;
    for (j=0; j<10; j++) {
        let i = j; // re-bound for each iteration!
        console.log( i );
    }
}
```
The reason why this per-iteration binding is interesting will become clear in Chapter 5 when we discuss closures.

Because let declarations attach to arbitrary blocks rather than to the enclosing function's scope (or global), there can be gotchas where existing code has a hidden reliance on function-scoped var declarations, and replacing the var with let may require additional care when refactoring code.

Consider:

```JavaScript
var foo = true, baz = 10;

if (foo) {
    var bar = 3;

    if (baz > bar) {
        console.log( baz );
    }

    // ...
}
```
This code is fairly easily re-factored as:

```JavaScript
var foo = true, baz = 10;

if (foo) {
    var bar = 3;

    // ...
}

if (baz > bar) {
    console.log( baz );
}
```
But, be careful of such changes when using block-scoped variables:

```JavaScript
var foo = true, baz = 10;

if (foo) {
    let bar = 3;

    if (baz > bar) { // <-- don't forget `bar` when moving!
        console.log( baz );
    }
}
```
See Appendix B for an alternate (more explicit) style of block-scoping which may provide easier to maintain/refactor code that's more robust to these scenarios.

### const

In addition to let, ES6 introduces const, which also creates a block-scoped variable, but whose value is fixed (constant). Any attempt to change that value at a later time results in an error.

```JavaScript
var foo = true;

if (foo) {
    var a = 2;
    const b = 3; // block-scoped to the containing `if`

    a = 3; // just fine!
    b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## Chapter 4: Hoisting

Consider another piece of code:
```JavaScript
console.log( a );
var a = 2;
```

When you see `var a = 2;`, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: var a; and a = 2;. The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left in place for the execution phase.

So, one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are “moved” from where they appear in the flow of the code to the top of the code. This gives rise to the name hoisting

Function declarations are hoisted, as we just saw. But function expressions are not.
```JavaScript
foo(); // not ReferenceError, but TypeError!
var foo = function bar() {
  // ...
};
```

recall that even though it’s a named function expression, the name identifier is not available in the enclosing scope:
```JavaScript
foo(); // TypeError
bar(); // ReferenceError
var foo = function bar() {
  // ...
};
```

This snippet is more accurately interpreted (with hoisting) as:
```JavaScript
var foo;
foo(); // TypeError
bar(); // ReferenceError
foo = function() {
  var bar = ...self...
  // ...
}
```

## Chapter 5: Scope Closure

`Closure` is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope

Let’s jump into some code to illustrate that definition.

```JavaScript
function foo() {
  var a = 2;
  function bar() {
    console.log( a ); // 2
  }
  bar();
}
foo();

```

This code should look familiar from our discussions of nested scope. Function bar() has access to the variable a in the outer enclosing scope because of lexical scope look-up rules (in this case, it’s an RHS reference look-up). Is this closure?

Well, technically…perhaps. But by our what-you-need-to-know definition above…not exactly. I think the most accurate way to explain bar() referencing a is via lexical scope look-up rules, and those rules are only (an important!) part of what closure is. From a purely academic perspective, what is said of the above snippet is that the function bar() has a closure over the scope of foo() (and indeed, even over the rest of the scopes it has access to, such as the global scope in our case). Put slightly differently, it’s said that bar() closes over the scope of foo(). Why? Because bar() appears nested inside of foo(). Plain and simple. But, closure defined in this way is not directly observable, nor do we see closure exercised in that snippet. We clearly see lexical scope, but closure remains sort of a mysterious shifting shadow behind the code. Let us then consider code that brings closure into full light:

```JavaScript
function foo() {
  var a = 2;

  function bar() {
    console.log( a );
  }

  return bar;
}

var baz = foo();
baz(); // 2 -- Whoa, closure was just observed, man.

```
The function bar() has lexical scope access to the inner scope of foo(). But then, we take bar(), the function itself, and pass it as a value. In this case, we return the function object itself that bar references.
After we execute foo(), we assign the value it returned (our inner bar() function) to a variable called baz, and then we actually invoke baz(), which of course is invoking our inner function bar(), just by a different identifier reference

bar() is executed, for sure. But in this case, it’s executed outside of its declared lexical scope

After foo() executed, normally we would expect that the entirety of the inner scope of foo() would go away, because we know that the engine employs a garbage collector that comes along and frees up memory once it’s no longer in use. Since it would appear that the con‐ tents of foo() are no longer in use, it would seem natural that they should be considered gone. But the “magic” of closures does not let this happen. That inner scope is in fact still in use, and thus does not go away. Who’s using it? The function bar() itself. By virtue of where it was declared, bar() has a lexical scope closure over that inner scope of foo(), which keeps that scope alive for bar() to reference at any later time. `bar() still has a reference to that scope, and that reference is called closure`

So, a few microseconds later, when the variable baz is invoked (invoking the inner function we initially labeled bar), it duly has access to author-time lexical scope, so it can access the variable a just as we’d expect. The function is being invoked well outside of its author-time lexical scope. Closure lets the function continue to access the lexical scope it was defined in at author time. Of course, any of the various ways that functions can be passed around as values, and indeed invoked in other locations, are all examples of observing/exercising closure.

Whatever facility we use to transport an inner function outside of its lexical scope, it will maintain a scope reference to where it was originally declared, and wherever we execute him, that closure will be exercised

Essentially whenever and wherever you treat functions (which access their own respective lexical scopes) as first-class values and pass them around, you are likely to see those functions exercising closure.
Be that timers, event handlers, Ajax requests, crosswindow messaging, web workers, or any of the other asynchronous (or synchronous!) tasks, when you pass in a callback function, get ready to sling some closure around

Note: Chapter 3 introduced the IIFE pattern. While it is often said that IIFE (alone) is an example of observed closure, I would somewhat disagree, by our definition above.
```JavaScript
var a = 2;

(function IIFE(){
    console.log( a );
})();
```
This code "works", but it's not strictly an observation of closure. Why? Because the function (which we named "IIFE" here) is not executed outside its lexical scope. It's still invoked right there in the same scope as it was declared (the enclosing/global scope that also holds a). a is found via normal lexical scope look-up, not really via closure.

While closure might technically be happening at declaration time, it is not strictly observable, and so, as they say, it's a tree falling in the forest with no one around to hear it.

Though an IIFE is not itself an example of closure, it absolutely creates scope, and it's one of the most common tools we use to create scope which can be closed over. So IIFEs are indeed heavily related to closure, even if not exercising closure themselves.


### Loops + Closure

The most common canonical example used to illustrate closure involves the humble for-loop.

```JavaScript
for (var i=1; i<=5; i++) {
    setTimeout( function timer(){
        console.log( i );
    }, i*1000 );
}
```

The spirit of this code snippet is that we would normally expect for the behavior to be that the numbers "1", "2", .. "5" would be printed out, one at a time, one per second, respectively.

In fact, if you run this code, you get "6" printed out 5 times, at the one-second intervals.

Huh?

Firstly, let's explain where 6 comes from. The terminating condition of the loop is when i is not <=5. The first time that's the case is when i is 6. So, the output is reflecting the final value of the i after the loop terminates.

This actually seems obvious on second glance. The timeout function callbacks are all running well after the completion of the loop. In fact, as timers go, even if it was setTimeout(.., 0) on each iteration, all those function callbacks would still run strictly after the completion of the loop, and thus print 6 each time.

But there's a deeper question at play here. What's missing from our code to actually have it behave as we semantically have implied?

What's missing is that we are trying to imply that each iteration of the loop "captures" its own copy of i, at the time of the iteration. But, the way scope works, all 5 of those functions, though they are defined separately in each loop iteration, all are closed over the same shared global scope, which has, in fact, only one i in it.

Put that way, of course all functions share a reference to the same i. Something about the loop structure tends to confuse us into thinking there's something else more sophisticated at work. There is not. There's no difference than if each of the 5 timeout callbacks were just declared one right after the other, with no loop at all.

OK, so, back to our burning question. What's missing? We need more cowbell closured scope. Specifically, we need a new closured scope for each iteration of the loop.

We learned in Chapter 3 that the IIFE creates scope by declaring a function and immediately executing it.

Let's try:
```JavaScript
for (var i=1; i<=5; i++) {
    (function(){
        setTimeout( function timer(){
            console.log( i );
        }, i*1000 );
    })();
}
```
Does that work? Try it. Again, I'll wait.

I'll end the suspense for you. Nope. But why? We now obviously have more lexical scope. Each timeout function callback is indeed closing over its own per-iteration scope created respectively by each IIFE.

It's not enough to have a scope to close over if that scope is empty. Look closely. Our IIFE is just an empty do-nothing scope. It needs something in it to be useful to us.

It needs its own variable, with a copy of the i value at each iteration.

```JavaScript
for (var i=1; i<=5; i++) {
    (function(){
        var j = i;
        setTimeout( function timer(){
            console.log( j );
        }, j*1000 );
    })();
}
```
Eureka! It works!

A slight variation some prefer is:
```JavaScript
for (var i=1; i<=5; i++) {
    (function(j){
        setTimeout( function timer(){
            console.log( j );
        }, j*1000 );
    })( i );
}
```
Of course, since these IIFEs are just functions, we can pass in i, and we can call it j if we prefer, or we can even call it i again. Either way, the code works now.

The use of an IIFE inside each iteration created a new scope for each iteration, which gave our timeout function callbacks the opportunity to close over a new scope for each iteration, one which had a variable with the right per-iteration value in it for us to access.

Problem solved!

### Block Scoping Revisited

Look carefully at our analysis of the previous solution. We used an IIFE to create new scope per-iteration. In other words, we actually needed a per-iteration block scope.

Chapter 3 showed us the let declaration, which hijacks a block and declares a variable right there in the block. It essentially turns a block into a scope that we can close over. So, the following awesome code just works:

```JavaScript
for (var i=1; i<=5; i++) {
  let j = i; // yay, block-scope for closure!
  setTimeout( function timer(){
    console.log( j );
  }, j*1000)
}
```

How cool is that? Block scoping and closure working hand-in-hand, solving all the world’s problems. I don’t know about you, but that makes me a happy JavaScripter

### modules

There are other code patterns which leverage the power of closure but which do not on the surface appear to be about callbacks. Let's examine the most powerful of them: the module.
```JavaScript
function foo() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }
}
```
As this code stands right now, there's no observable closure going on. We simply have some private data variables something and another, and a couple of inner functions doSomething() and doAnother(), which both have lexical scope (and thus closure!) over the inner scope of foo().

But now consider:
```JavaScript
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }

    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
This is the pattern in JavaScript we call module. The most common way of implementing the module pattern is often called "Revealing Module", and it's the variation we present here.

Let's examine some things about this code.

Firstly, CoolModule() is just a function, but it has to be invoked for there to be a module instance created. Without the execution of the outer function, the creation of the inner scope and the closures would not occur.

Secondly, the CoolModule() function returns an object, denoted by the object-literal syntax { key: value, ... }. The object we return has references on it to our inner functions, but not to our inner data variables. We keep those hidden and private. It's appropriate to think of this object return value as essentially a public API for our module.

This object return value is ultimately assigned to the outer variable foo, and then we can access those property methods on the API, like foo.doSomething().

The doSomething() and doAnother() functions have closure over the inner scope of the module "instance" (arrived at by actually invoking CoolModule()). When we transport those functions outside of the lexical scope, by way of property references on the object we return, we have now set up a condition by which closure can be observed and exercised.

To state it more simply, there are two "requirements" for the module pattern to be exercised:

There must be an outer enclosing function, and it must be invoked at least once (each time creates a new module instance).

The enclosing function must return back at least one inner function, so that this inner function has closure over the private scope, and can access and/or modify that private state.

An object with a function property on it alone is not really a module. An object which is returned from a function invocation which only has data properties on it and no closured functions is not really a module, in the observable sense.

The code snippet above shows a standalone module creator called CoolModule() which can be invoked any number of times, each time creating a new module instance. A slight variation on this pattern is when you only care to have one instance, a "singleton" of sorts:

```JavaScript
var foo = (function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }

    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
Here, we turned our module function into an IIFE (see Chapter 3), and we immediately invoked it and assigned its return value directly to our single module instance identifier foo.

Modules are just functions, so they can receive parameters:
```JavaScript
function CoolModule(id) {
    function identify() {
        console.log( id );
    }

    return {
        identify: identify
    };
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```
Another slight but powerful variation on the module pattern is to name the object you are returning as your public API:
```JavaScript
var foo = (function CoolModule(id) {
    function change() {
        // modifying the public API
        publicAPI.identify = identify2;
    }

    function identify1() {
        console.log( id );
    }

    function identify2() {
        console.log( id.toUpperCase() );
    }

    var publicAPI = {
        change: change,
        identify: identify1
    };

    return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```
By retaining an inner reference to the public API object inside your module instance, you can modify that module instance from the inside, including adding and removing methods, properties, and changing their values.

### Modern Modules

Various module dependency loaders/managers essentially wrap up this pattern of module definition into a friendly API. Rather than examine any one particular library, let me present a very simple proof of concept for illustration purposes (only):
```JavaScript
var MyModules = (function Manager() {
    var modules = {};

    function define(name, deps, impl) {
        for (var i=0; i<deps.length; i++) {
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply( impl, deps );
    }

    function get(name) {
        return modules[name];
    }

    return {
        define: define,
        get: get
    };
})();
```
The key part of this code is modules[name] = impl.apply(impl, deps). This is invoking the definition wrapper function for a module (passing in any dependencies), and storing the return value, the module's API, into an internal list of modules tracked by name.

And here's how I might use it to define some modules:
```JavaScript
MyModules.define( "bar", [], function(){
    function hello(who) {
        return "Let me introduce: " + who;
    }

    return {
        hello: hello
    };
} );

MyModules.define( "foo", ["bar"], function(bar){
    var hungry = "hippo";

    function awesome() {
        console.log( bar.hello( hungry ).toUpperCase() );
    }

    return {
        awesome: awesome
    };
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
    bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```
Both the "foo" and "bar" modules are defined with a function that returns a public API. "foo" even receives the instance of "bar" as a dependency parameter, and can use it accordingly.

Spend some time examining these code snippets to fully understand the power of closures put to use for our own good purposes. The key take-away is that there's not really any particular "magic" to module managers. They fulfill both characteristics of the module pattern I listed above: invoking a function definition wrapper, and keeping its return value as the API for that module.

In other words, modules are just modules, even if you put a friendly wrapper tool on top of them.

### Review

Closures can trip us up, for instance with loops, if we’re not careful to recognize them and how they work. But they are also an immensely powerful tool, enabling patterns like modules in their various forms

Modules require two key characteristics:
1) an outer wrapping function being invoked, to create the enclosing scope
2) the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper

The short explanation is that arrow functions do not behave at all like normal functions when it comes to their this binding. They discard all the normal rules for this binding, and instead take on the this value of their immediate lexical enclosing scope, whatever it is

## Appendix A: Dynamic Scope

In Chapter 2, we talked about "Dynamic Scope" as a contrast to the "Lexical Scope" model, which is how scope works in JavaScript (and in fact, most other languages).

We will briefly examine dynamic scope, to hammer home the contrast. But, more importantly, dynamic scope actually is a near cousin to another mechanism (this) in JavaScript, which we covered in the "this & Object Prototypes" title of this book series.

As we saw in Chapter 2, lexical scope is the set of rules about how the Engine can look-up a variable and where it will find it. The key characteristic of lexical scope is that it is defined at author-time, when the code is written (assuming you don't cheat with eval() or with).

Dynamic scope seems to imply, and for good reason, that there's a model whereby scope can be determined dynamically at runtime, rather than statically at author-time. That is in fact the case. Let's illustrate via code:
```JavaScript
function foo() {
    console.log( a ); // 2
}

function bar() {
    var a = 3;
    foo();
}

var a = 2;

bar();
```
Lexical scope holds that the RHS reference to a in foo() will be resolved to the global variable a, which will result in value 2 being output.

Dynamic scope, by contrast, doesn't concern itself with how and where functions and scopes are declared, but rather where they are called from. In other words, the scope chain is based on the call-stack, not the nesting of scopes in code.

So, if JavaScript had dynamic scope, when foo() is executed, theoretically the code below would instead result in 3 as the output.
```JavaScript
function foo() {
    console.log( a ); // 3  (not 2!)
}

function bar() {
    var a = 3;
    foo();
}

var a = 2;

bar();
```
How can this be? Because when foo() cannot resolve the variable reference for a, instead of stepping up the nested (lexical) scope chain, it walks up the call-stack, to find where foo() was called from. Since foo() was called from bar(), it checks the variables in scope for bar(), and finds an a there with value 3.

Strange? You're probably thinking so, at the moment.

But that's just because you've probably only ever worked on (or at least deeply considered) code which is lexically scoped. So dynamic scoping seems foreign. If you had only ever written code in a dynamically scoped language, it would seem natural, and lexical scope would be the odd-ball.

To be clear, JavaScript does not, in fact, have dynamic scope. It has lexical scope. Plain and simple. But the this mechanism is kind of like dynamic scope.

The key contrast: lexical scope is write-time, whereas dynamic scope (and this!) are runtime. Lexical scope cares where a function was declared, but dynamic scope cares where a function was called from.

Finally: this cares how a function was called, which shows how closely related the this mechanism is to the idea of dynamic scoping. To dig more into this, read the title "this & Object Prototypes".
