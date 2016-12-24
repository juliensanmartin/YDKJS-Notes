#Scope And Closure

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program state

`Tokenizing/Lexing` : Breaking up a string of characters into meaningful (to the language) chunks, called tokens

The difference between `tokenizing` and `lexing` is subtle and academic, but it centers on whether or not these tokens are identified in a `stateless` or `stateful` way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether a should be considered a distinct token or just part of another token, that would be lexing.

Parsing taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an “AST” (abstract syntax tree). The tree for `var a = 2;` might start with a top-level node called `VariableDeclaration`, with a child node called `Identifier` (whose value is a), and another child called `AssignmentExpression`, which itself has a child called `NumericLiteral` (whose value is 2).

`Code-Generation` : The process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it’s targeting, and so on. So, rather than get mired in details, we’ll just handwave and say that there’s a way to take our previously described AST for `var a = 2;` and turn it into a set of machine instructions to actually create 2.

The `JavaScript engine` is vastly more complex than just those three steps, as are most other language compilers. For instance, in the process of parsing and code-generation, there are certainly steps to optimize the performance of the execution, including collapsing redundant elements, etc. So, I’m painting only with broad strokes here. But I think you’ll see shortly why these details we do cover, even at a high level, are relevant. For one thing, JavaScript engines don’t get the luxury (like other language compilers) of `having plenty of time to optimize`, because JavaScript compilation doesn’t happen in a build step ahead of time, as with other languages. For JavaScript, the compilation that occurs happens, in many cases, mere microseconds (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like `JITs`, which lazy compile and even hot recompile, etc.) that are well beyond the “scope” of our discussion here. Let’s just say, for simplicity sake, that any snippet of JavaScript has to be compiled before (usually right before!) it’s executed. So, the JS compiler will take the program `var a = 2;` and compile it first, and then be ready to execute it, usually right away.

### Understanding Scope
The way we will approach learning about `scope` is to think of the process in terms of a conversation. But, who is having the conversation? Let’s meet the cast of characters that interact to process the program `var a = 2;`, so we understand their conversations that we’ll listen in on shortly.

`Scope` Another friend of `Engine`; collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code

`Engine` sees two distinct statements, one that `Compiler` will handle during compilation, and one that `Engine` will handle during execution. So, let’s break down how `Engine` and friends will approach the program `var a = 2;`. The first thing `Compiler` will do with this program is perform `lexing` to break it down into `tokens`, which it will then `parse` into a `tree`. But when `Compiler` gets to code generation, it will treat this program somewhat differently than perhaps assumed. A reasonable assumption would be that `Compiler` will produce code that could be summed up by this pseudocode: `Allocate memory for a variable, label it a, then stick the value 2 into that variable.` Unfortunately, that’s not quite accurate. `Compiler` will instead proceed as:
1. Encountering var a, `Compiler` asks `Scope` to see if a variable a already exists for that particular scope collection. If so, `Compiler` ignores this declaration and moves on. Otherwise, `Compiler` asks `Scope` to declare a new variable called `a` for that scope collection.
2. `Compiler` then produces code for `Engine` to later execute, to handle the `a = 2` assignment.

To summarize: two distinct actions are taken for a `variable assignment`: First, `Compiler` declares a variable (if not previously declared) in the current `Scope`, and second, when executing, `Engine` looks up the variable in `Scope` and assigns to it, if found.

In our case, it is said that `Engine` would be performing an `LHS lookup` for the variable `a`. The other type of look-up is called `RHS`
In other words, an `LHS` look-up is done when a variable appears on the `lefthand side of an assignment operation`, and an `RHS` look-up is done when a variable appears on the `righthand side of an assignment operation`

Being slightly glib for a moment, you could think RHS instead means "retrieve his/her source (value)," implying that RHS means "go get the value of…"

When I say: `console.log( a );` The reference to `a` is an RHS reference, because nothing is being assigned to `a` here. Instead, we’re looking up to retrieve the value of `a`, so that the value can be passed to `console.log(..)`. By contrast: `a = 2;` The reference to `a` here is an LHS reference, because we don’t actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.
"Who’s the target of the assignment (LHS)?"" and "Who’s the source of the assignment (RHS)?"

#### Scope
Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, Engine consults the next outer containing scope, continuing until is found or until the outermost (a.k.a., global) scope has been reached.

The simple rules for traversing nested scope: Engine starts at the currently executing scope, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

`Strict Mode` which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global scoped variable to hand back from an LHS look-up, and Engine would throw a `ReferenceError` similarly to the RHS case

There are two predominant models for how scope works. The first of these is by far the most common, used by the vast majority of programming languages. It’s called `lexical scope`, and we will examine it in depth. The other model, which is still used by some languages (such as Bash scripting, some modes in Perl, etc) is called dynamic scope.

To define it somewhat circularly, `lexical scope is scope that is defined at lexing time`. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code

Scope look-up stops once it finds the first match. The same identifier name can be specified at multiple layers of nested scope, which is called `shadowing` (the inner identifier “shadows” the outer identifier)

Global variables are automatically also properties of the global object (window in browsers, etc.), so it is possible to reference a global variable not directly by its lexical name, but instead indirectly as a property reference of the global object. window.a This technique gives access to a global variable that would otherwise be inaccessible due to it being shadowed. However, non-global shadowed variables cannot be accessed no matter where a function is invoked from, or even how it is invoked, its lexical scope is only defined by where the function was declared

The lexical scope look-up process only applies to first-class identifiers, such as the a, b, and c. If you had a reference to foo.bar.baz in a piece of code, the lexical scope look-up would apply to finding the foo identifier, but once it locates that variable, object property-access rules take over to resolve the bar and baz properties, respectively

The JavaScript engine has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and predetermine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution

But the inverse thinking is equally powerful and useful: take any arbitrary section of code you’ve written and wrap a function declaration around it, which in effect “hides” the code

There’s a variety of reasons motivating this scope-based hiding. They tend to arise from the software design principle `Principle of Least Privilege` :
This principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and “hide” everything else

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


Another option for `collision avoidance` is the more modern `module approach`, using any of various dependency managers

Using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifier(s) be explicitly imported into another specific scope through usage of the dependency manager’s various mechanisms
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

That’s what block-scoping is all about. Declaring variables as close as possible, as local as possible, to where they will be used. Another example:
```JavaScript
var foo = true;
if (foo) {
  var bar = foo * 2;
  bar = something( bar );
  console.log( bar );
}
```

someReallyBigData variable at all. That means, theoretically, after process(..) runs, the big memory-heavy data structure could be garbage collected. However, it’s quite likely (though implementation dependent) that the JS engine will still have to keep the structure around, since the click function has a closure over the entire scope. Block-scoping can address this concern, making it clearer to the engine that it does not need to keep someReallyBigData around:

```JavaScript
function process(data) {
  // do something interesting
} // anything declared inside this block can go away after!

{
  let someReallyBigData = { .. };
  process( someReallyBigData );
}

var btn = document.getElementById( "my_button");

btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase=*/false );
```

Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox

In ES6, the `let` keyword (a cousin to the var keyword) is introduced to allow declarations of variables in any arbitrary block of code. ```JavaScript
if (..) {
  let a = 2;
}
```
will declare a variable a that essentially hijacks the scope of the if’s { .. } block and attaches itself there



When you see `var a = 2;`, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: var a; and a = 2;. The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left in place for the execution phase.

So, one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are “moved” from where they appear in the flow of the code to the top of the code. This gives rise to the name hoisting

Function declarations are hoisted, as we just saw. But function expressions are not. `foo(); // not ReferenceError, but TypeError!` `var foo = function bar() { // ... };`

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

After foo() executed, normally we would expect that the entirety of the inner scope of foo() would go away, because we know that the engine employs a garbage collector that comes along and frees up memory once it’s no longer in use. Since it would appear that the con‐ tents of foo() are no longer in use, it would seem natural that they should be considered gone. But the “magic” of closures does not let this happen. That inner scope is in fact still in use, and thus does not go away. Who’s using it? The function bar() itself. By virtue of where it was declared, bar() has a lexical scope closure over that inner scope of foo(), which keeps that scope alive for bar() to reference at any later time. bar() still has a reference to that scope, and that reference is called closure

So, a few microseconds later, when the variable baz is invoked (invoking the inner function we initially labeled bar), it duly has access to author-time lexical scope, so it can access the variable a just as we’d expect. The function is being invoked well outside of its author-time lexical scope. Closure lets the function continue to access the lexical scope it was defined in at author time. Of course, any of the various ways that functions can be passed around as values, and indeed invoked in other locations, are all examples of observing/exercising closure.

Whatever facility we use to transport an inner function outside of its lexical scope, it will maintain a scope reference to where it was originally declared, and wherever we execute him, that closure will be exercised

Be that timers, event handlers, Ajax requests, crosswindow messaging, web workers, or any of the other asynchronous (or synchronous!) tasks, when you pass in a callback function, get ready to sling some closure around

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

The let keyword attaches the variable declaration to the scope of whatever block (commonly a { .. } pair) it’s contained in. In other words, let implicitly hijacks any block’s scope for its variable declaration.
```JavaScript
var foo = true;
if (foo) {
  let bar = foo * 2;
  bar = something( bar );
  console.log( bar );
}
console.log( bar ); // ReferenceError
```

Closures can trip us up, for instance with loops, if we’re not careful to recognize them and how they work. But they are also an immensely powerful tool, enabling patterns like modules in their various forms

Modules require two key characteristics:
1) an outer wrapping function being invoked, to create the enclosing scope
2) the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper

The short explanation is that arrow functions do not behave at all like normal functions when it comes to their this binding. They discard all the normal rules for this binding, and instead take on the this value of their immediate lexical enclosing scope, whatever it is
