# YDKJS-Notes
My Notes from the YDKJS collection by Kyle Simpson

## You Don't Know JS - Up & Going

The JavaScript engine actually compiles the program on the fly and then immediately runs the compiled code

JavaScript uses the latter approach, dynamic typing, meaning variables can hold values of any type without any type enforcement

Either way, you’ll note that amount holds a running value that changes over the course of the program, illustrating the primary purpose of variables: managing program state

In other words, state is tracking the changes to values as your program runs.

The specific list of “falsy” values in JavaScript is as follows:
* "" (empty string)
* 0,
* -0,
* NaN (invalid number)
* null
* undefined
* false

You use the `var` keyword to declare a variable that will belong to the current function scope, or the global scope if at the top level outside of any function

Metaphorically, this behavior is called `hoisting`, when a var declaration is conceptually “moved” to the top of its enclosing scope

This may sound like a strange concept at first, so take a moment to ponder it. Not only can you pass a value (argument) to a function, but a `function` itself can be a value that’s assigned to variables or passed to or returned from other functions. As such, a function value should be thought of as an expression, much like any other value or expression.
Consider:
```JavaScript
var foo = function() {
  // ..
};

var x = function bar(){
  // ..
};
```

The first function expression assigned to the `foo` variable is called `anonymous` because it has no name. The second function expression is named `bar`, even as a reference to it is also assigned to the `x` variable.
`Named function expressions` are generally more preferable, though `anonymous function expressions` are still extremely common.

### `Immediately Invoked Function Expressions` (`IIFE`s)
In the previous snippet, neither of the function expressions are executed.
We could if we had included `foo()` or `x()`, for instance.

There’s another way to execute a function expression, which is typically referred to as an `immediately invoked function expression` (IIFE):
```JavaScript
(function IIFE(){
  console.log( "Hello!" );
})(); // "Hello!"
```

The outer `( .. )` that surrounds the `(function IIFE(){ .. })` function expression is just a nuance of JS grammar needed to prevent it from being treated as a normal function declaration.
The final () on the end of the expression is what actually executes the function expression referenced immediately.

Because an IIFE is just a function, and functions create variable scope, using an IIFE in this fashion is often used to declare variables that won’t affect the surrounding code outside the IIFE

```JavaScript
var a = 42;

(function IIFE(){
  var a = 10;
  console.log( a ); // 10
})();

console.log( a ); //42
```

### Closure

You can think of `closure` as a way to “remember” and continue to access a function’s scope (its variables) even once the function has finished running

Consider:
```JavaScript
function makeAdder(x) {
  // parameter `x` is an inner variable
  // inner function `add()` uses `x`, so
  // it has a "closure" over it
  function add(y) {
    return y + x;
  };

  return add;
}
```

The reference to the inner `add(..)` function that gets returned with each call to the outer `makeAdder(..)` is able to remember whatever `x` value was passed in to `makeAdder(..)`. Now, let’s use

```JavaScript
makeAdder(..);
// `plusOne` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)` var plusOne = makeAdder( 1 );
// `plusTen` gets a reference to the inner `add(..)`
// function with closure over the `x` parameter of
// the outer `makeAdder(..)`

```

The most common usage of closure in JavaScript is the `module pattern`. Modules let you define private implementation details (variables, functions) that are hidden from the outside world, as well as a public API that is accessible from the outside

```JavaScript
function User(){
  var username, password;

  function doLogin(user,pw) {
    username = user;
    password = pw; // do the rest of the login work
  }

  var publicAPI = {
    login: doLogin
  };

  return publicAPI;
}

// create a `User` module instance
var fred = User();
fred.login( "fred", "12Battery34!" );

```

The `User()` function serves as an `outer scope` that holds the variables `username` and `password`, as well as the inner `doLogin()` function; these are all private inner details of this `User` module that cannot be accessed from the outside world. We are not calling new `User()` here, on purpose, despite the fact that probably seems more common to most readers. `User()` is just a function, not a class to be instantiated, so it’s just called normally. Using `new` would be inappropriate and actually waste resources. Executing `User()` creates an instance of the User module a whole new `scope` is created, and thus a whole new copy of each of these inner variables/functions. We assign this instance to `fred`. If we run `User()` again, we’d get a new instance entirely separate from `fred`. The inner `doLogin()` function has a closure over `username` and `password`, meaning it will retain its access to them even after the `User()` function finishes running. `publicAPI` is an object with one property/method on it, `login`, which is a reference to the inner `doLogin()` function. When we return `publicAPI` from `User()`, it becomes the instance we call `fred`. At this point, the outer `User()` function has finished executing. Normally, you’d think the inner variables like `username` and `password` have gone away. But here they have not, because there’s a closure in the `login()` function keeping them alive.

That’s why we can call `fred`. `login(..)` — the same as calling the inner `doLogin(..)` — and it can still access `username` and `password` inner variables. There’s a good chance that with just this brief glimpse at `closure` and the `module pattern`, some of it is still a bit confusing. That’s OK! It takes some work to wrap your brain around it.

If a function has a `this` reference inside it, that `this` reference usually points to an object. But which object it points to depends on how the function was called

Bottom line: to understand what `this` points to, you have to examine how the function in question was called. It will be one of those four ways just shown.

The `a` property doesn’t actually exist on the `bar` object, but because `bar` is `prototype-linked` to `foo`, JavaScript automatically falls back to looking for a on the `foo` object, where it’s found. This linkage may seem like a strange feature of the language. The most common way this feature is used - and I would argue, abused — is to try to emulate/fake a “class” mechanism with “inheritance.” But a more natural way of applying prototypes is a pattern called `behavior delegation`, where you intentionally design your linked objects to be able to delegate from one to the other for parts of the needed behavior.

The check we do here takes advantage of a quirk with `NaN` values, which is that they’re the only value in the whole language that is not equal to itself. So the `NaN` value is the only one that would make `x !== x` be `true`

So the better option is to use a tool that converts your newer code into older code equivalents. This process is commonly called `tran‐ spiling` a term for transforming + compiling.

#Scope And Closure

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program state
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 2-2 | Added on Tuesday, November 22, 2016 7:54:39 AM

Tokenizing/Lexing Breaking up a string of characters into meaningful (to the lan‐ guage) chunks, called tokens
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 2-2 | Added on Tuesday, November 22, 2016 7:55:08 AM

The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a stateless or stateful way. Put simply, if the tokenizer were to invoke stateful parsing rules to fig‐ ure out whether a should be considered a distinct token or just part of another token, that would be lexing
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 2-2 | Added on Tuesday, November 22, 2016 7:55:26 AM

Parsing taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an “AST” (abstract syntax tree). The tree for var a = 2; might start with a top-level node called VariableDeclaration, with a child node called Identifier (whose value is a), and another child called AssignmentExpres sion, which itself has a child called NumericLiteral (whose value is 2). Code-Generation The process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it’s targeting, and so on. So, rather than get mired in details, we’ll just handwave and say that there’s a way to take our previously described AST for var a = 2; and turn it into a set of machine instructions to actually create 2 | Chapter 1: What Is Scope?
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 3-3 | Added on Tuesday, November 22, 2016 7:56:35 AM

The JavaScript engine is vastly more complex than just those three steps, as are most other language compilers. For instance, in the process of parsing and code-generation, there are certainly steps to optimize the performance of the execution, including collapsing re‐ dundant elements, etc. So, I’m painting only with broad strokes here. But I think you’ll see shortly why these details we do cover, even at a high level, are relevant. For one thing, JavaScript engines don’t get the luxury (like other lan‐ guage compilers) of having plenty of time to optimize, because Java‐ Script compilation doesn’t happen in a build step ahead of time, as with other languages. For JavaScript, the compilation that occurs happens, in many cases, mere microseconds (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like JITs, which lazy compile and even hot recompile, etc.) that are well beyond the “scope” of our discussion here. Let’s just say, for simplicity sake, that any snippet of JavaScript has to be compiled before (usually right before!) it’s executed. So, the JS com‐ piler will take the program var a = 2; and compile it first, and then be ready to execute it, usually right away. Understanding Scope The way we will approach learning about scope is to think of the pro‐ cess in terms of a conversation. But, who is having the conversation? The Cast Let’s meet the cast of characters that interact to process the program var a = 2;, so we understand their conversations that we’ll listen in on shortly: Understanding Scope | 3
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 4-4 | Added on Tuesday, November 22, 2016 7:58:26 AM

Scope Another friend of Engine; collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 4-4 | Added on Tuesday, November 22, 2016 7:59:08 AM

Engine sees two distinct statements, one that Compiler will handle during compilation, and one that Engine will handle during execution. So, let’s break down how Engine and friends will approach the program var a = 2;. The first thing Compiler will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree. But when Compiler gets to code generation, it will treat this program somewhat differently than perhaps assumed. A reasonable assumption would be that Compiler will produce code that could be summed up by this pseudocode: “Allocate memory for a variable, label it a, then stick the value 2 into that variable.” Unfortu‐ nately, that’s not quite accurate. Compiler will instead proceed as: 1. Encountering var a, Compiler asks Scope to see if a variable a already exists for that particular scope collection. If so, Compiler ignores this declaration and moves on. Otherwise, Compiler asks Scope to declare a new variable called a for that scope collection. 2. Compiler then produces code for Engine to later execute, to han‐ dle the a = 2 assignment. The code Engine runs will first ask Scope 4 | Chapter 1: What Is Scope?
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 5-5 | Added on Tuesday, November 22, 2016 8:00:34 AM

To summarize: two distinct actions are taken for a variable assignment: First, Compiler declares a variable (if not previously declared) in the current Scope, and second, when executing, Engine looks up the vari‐ able in Scope and assigns to it, if found
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 5-5 | Added on Tuesday, November 22, 2016 8:01:12 AM

In our case, it is said that Engine would be performing an LHS lookup for the variable a. The other type of look-up is called RHS
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 5-5 | Added on Tuesday, November 22, 2016 8:01:33 AM

In other words, an LHS look-up is done when a variable appears on the lefthand side of an assignment operation, and an RHS look-up is done when a variable appears on the righthand side of an assignment operation
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 5-5 | Added on Tuesday, November 22, 2016 8:02:25 AM

Being slightly glib for a moment, you could think RHS instead means “retrieve his/her source (value),” implying that RHS means “go get the value of…
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 6-6 | Added on Tuesday, November 22, 2016 8:02:55 AM

When I say: console.log( a ); The reference to a is an RHS reference, because nothing is being as‐ signed to a here. Instead, we’re looking up to retrieve the value of a, so that the value can be passed to console.log(..). By contrast: a = 2; The reference to a here is an LHS reference, because we don’t actually care what the current value is, we simply want to find the variable as a target for the = 2 assignment operation
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 6-6 | Added on Tuesday, November 22, 2016 8:03:20 AM

Who’s the target of the assignment (LHS)?” and “Who’s the source of the assignment (RHS)?”
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 8-8 | Added on Tuesday, November 22, 2016 8:08:37 AM

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, Engine consults the next outercontaining
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 9-9 | Added on Tuesday, November 22, 2016 8:08:53 AM

scope, continuing until is found or until the outermost (a.k.a., global) scope has been reached
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 9-9 | Added on Wednesday, November 23, 2016 7:33:20 AM

The simple rules for traversing nested scope: Engine starts at the cur‐ rently executing scope, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 11-11 | Added on Wednesday, November 23, 2016 7:35:51 AM

Strict Mode,” which was added in ES5, has a number of different be‐ haviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global scoped variable to hand back from an LHS look-up, and Engine would throw a ReferenceError similarly to the RHS case
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 13-13 | Added on Wednesday, November 23, 2016 7:38:55 AM

There are two predominant models for how scope works. The first of these is by far the most common, used by the vast majority of pro‐ gramming languages. It’s called lexical scope, and we will examine it in depth. The other model, which is still used by some languages (such as Bash scripting, some modes in Perl, etc) is called dynamic scope.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 13-13 | Added on Wednesday, November 23, 2016 7:39:53 AM

To define it somewhat circularly, lexical scope is scope that is defined at lexing time. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 15-15 | Added on Wednesday, November 23, 2016 7:43:11 AM

Scope look-up stops once it finds the first match. The same identifier name can be specified at multiple layers of nested scope, which is called “shadowing” (the inner identifer “shadows” the outer identifier
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 16-16 | Added on Wednesday, November 23, 2016 7:43:55 AM

Global variables are automatically also properties of the glob‐ al object (window in browsers, etc.), so it is possible to refer‐ ence a global variable not directly by its lexical name, but in‐ stead indirectly as a property reference of the global object. window.a This technique gives access to a global variable that would otherwise be inaccessible due to it being shadowed. However, non-global shadowed variables cannot be accessed
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 16-16 | Added on Wednesday, November 23, 2016 7:44:15 AM

No matter where a function is invoked from, or even how it is invoked, its lexical scope is only defined by where the function was declared
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 16-16 | Added on Wednesday, November 23, 2016 7:44:57 AM

The lexical scope look-up process only applies to first-class identifiers, such as the a, b, and c. If you had a reference to foo.bar.baz in a piece
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 16-16 | Added on Wednesday, November 23, 2016 7:45:07 AM

of code, the lexical scope look-up would apply to finding the foo identifier, but once it locates that variable, object property-access rules take over to resolve the bar and baz properties, respectively
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 21-21 | Added on Wednesday, November 23, 2016 7:50:22 AM

The JavaScript engine has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and pre‐ determine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 24-24 | Added on Wednesday, November 23, 2016 8:00:38 AM

But the inverse thinking is equally powerful and useful: take any arbitrary section of code you’ve written and wrap a function declaration around it, which in effect “hides” the code
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 25-25 | Added on Wednesday, November 23, 2016 8:01:10 AM

There’s a variety of reasons motivating this scope-based hiding. They tend to arise from the software design principle Principle of Least Privilege
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 25-25 | Added on Wednesday, November 23, 2016 8:02:11 AM

This principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and “hide” everything else
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 27-27 | Added on Wednesday, November 23, 2016 8:06:11 AM

Such libraries typically will create a single variable declaration, often an object, with a sufficiently unique name, in the global scope. This
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 27-27 | Added on Wednesday, November 23, 2016 8:06:23 AM

object is then used as a namespace for that library, where all specific exposures of functionality are made as properties off that object (namespace), rather than as top-level lexically scoped identifiers them‐ selves. For example: var MyReallyCoolLibrary = { awesome: "stuff", doSomething: function() { // ... }, doAnotherThing: function() { // ... } };
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 27-27 | Added on Wednesday, November 23, 2016 8:07:02 AM

Another option for collision avoidance is the more modern module approach, using any of various dependency managers
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 27-27 | Added on Wednesday, November 23, 2016 8:07:34 AM

Using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifier(s) be explicitly imported into another specific scope through usage of the dependency manager’s various mechanisms
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 28-28 | Added on Wednesday, November 23, 2016 9:37:16 PM

var a = 2; function foo() { // <-- insert this var a = 3; console.log( a ); // 3 } // <-- and this foo(); // <-- and this console.log( a ); // 2 While this technique works, it is not necessarily very ideal. There are a few problems it introduces. The first is that we have to declare a named-function foo(), which means that the identifier name foo itself “pollutes” the enclosing scope (global, in this case). We also have to explicitly call the function by name (foo()) so that the wrapped code actually executes.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 28-28 | Added on Wednesday, November 23, 2016 9:38:37 PM

var a = 2; (function foo(){ // <-- insert this var a = 3; console.log( a ); /
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 29-29 | Added on Wednesday, November 23, 2016 9:38:46 PM

})(); // <-- and this console.log( a ); //
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 29-29 | Added on Wednesday, November 23, 2016 9:39:13 PM

Instead of treating the function as a standard declaration, the function is treated as a functionexpression
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 29-29 | Added on Wednesday, November 23, 2016 9:39:30 PM

The easiest way to distinguish declaration vs. expression is the position of the word function in the statement (not just a line, but a distinct statement). If function is the very first thing in the statement, then it’s a function declaration. Otherwise, it’s a function expression. The key difference we can observe here between a function declaration and a function expression relates to where its name is bound as an identifier. Compare the previous two snippets. In the first snippet, the name foo is bound in the enclosing scope, and we call it directly with foo(). In the second snippet, the name foo is not bound in the enclosing scope, but instead is bound only inside of its own function. In other words, (function foo(){ .. }) as an expression means the identifier foo is found only in the scope where the .. indicates, not in the outer scope. Hiding the name foo inside itself means it does not pollute the enclosing scope unnecessarily. Anonymous Versus Named You are probably most familiar with function expressions as callback parameters, such as: setTimeout( function(){ console.log("I waited 1 second!"); }, 1000 ); This is called an anonymous function expression, because function() … has no name identifier on it. Function expressions can be anony‐ mous, but function declarations cannot omit the name—that would be illegal JS grammar. Functions as Scopes | 29
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 30-30 | Added on Wednesday, November 23, 2016 9:43:27 PM

The best practice is to always name your function expressions: setTimeout( function timeoutHandler(){ // <-- Look, I have a // name! console.log( "I waited 1 second!" ); }, 1000 );
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 30-30 | Added on Wednesday, November 23, 2016 9:44:11 PM

Now that we have a function as an expression by virtue of wrapping it in a ( ) pair, we can execute that function by adding another () on the end, like (function foo(){ .. })(). The first enclosing ( ) pair makes the function an expression, and the second () executes the function.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 33-33 | Added on Wednesday, November 23, 2016 9:51:35 PM

That’s what block-scoping is all about. Declaring variables as close as possible, as local as possible, to where they will be used. Another ex‐ ample: var foo = true; if (foo) { var bar = foo * 2; bar = something( bar ); console.log( bar ); }
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 37-37 | Added on Wednesday, November 23, 2016 10:01:19 PM

lyBigData variable at all. That means, theoretically, after pro cess(..) runs, the big memory-heavy data structure could be garbage collected. However, it’s quite likely (though implementation depen‐ dent) that the JS engine will still have to keep the structure around, since the click function has a closure over the entire scope. Block-scoping can address this concern, making it clearer to the en‐ gine that it does not need to keep someReallyBigData around: function process(data) { // do something interesting } // anything declared inside this block can go away after! { let someReallyBigData = { .. }; process( someReallyBigData ); } var btn = document.getElementById( "my_button"
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 37-37 | Added on Wednesday, November 23, 2016 10:01:33 PM

btn.addEventListener( "click", function click(evt){ console.log("button clicked"); }, /*capturingPhase=*/false ); Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 40-40 | Added on Wednesday, November 23, 2016 10:05:36 PM

In ES6, the let keyword (a cousin to the var keyword) is introduced to allow declarations of variables in any arbitrary block of code. if (..) { let a = 2; } will declare a variable a that essentially hijacks the scope of the if’s { .. } block and attaches itself there
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 42-42 | Added on Thursday, November 24, 2016 7:27:07 AM

When you see var a = 2;, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: var a; and a = 2;. The first statement, the declaration, is processed during the com
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 42-42 | Added on Thursday, November 24, 2016 7:27:16 AM

pilation phase. The second statement, the assignment, is left in place for the execution phase.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 43-43 | Added on Thursday, November 24, 2016 7:28:15 AM

So, one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are “moved” from where they appear in the flow of the code to the top of the code. This gives rise to the name hoisting
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 44-44 | Added on Thursday, November 24, 2016 7:32:29 AM

Function declarations are hoisted, as we just saw. But function ex‐ pressions are not. foo(); // not ReferenceError, but TypeError! var foo = function bar() { // ... };
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 44-44 | Added on Thursday, November 24, 2016 7:38:02 AM

recall that even though it’s a named function expression, the name identifier is not available in the enclosing scope: foo(); // TypeError bar(); // ReferenceError var foo = function bar() { // ... }; This snippet is more accurately interpreted (with hoisting) as: var foo; foo(); // TypeError bar(); // ReferenceError foo = function() { var bar = ...self... // ...
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 48-48 | Added on Thursday, November 24, 2016 7:43:19 AM

Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 49-49 | Added on Thursday, November 24, 2016 7:46:17 AM

function foo() { var a = 2; function bar() { console.log( a ); } return bar; } var baz = foo(); baz(); // 2 -- Whoa, closure was just observed, man. The function bar() has lexical scope access to the inner scope of foo(). But then, we take bar(), the function itself, and pass it as a value. In this case, we return the function object itself that bar refer‐ ences
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 49-49 | Added on Thursday, November 24, 2016 7:47:02 AM

After we execute foo(), we assign the value it returned (our inner bar() function) to a variable called baz, and then we actually invoke baz(), which of course is invoking our inner function bar(), just by a different identifier reference
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 49-49 | Added on Thursday, November 24, 2016 7:47:15 AM

bar() is executed, for sure. But in this case, it’s executed outside of its declared lexical scope
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 49-49 | Added on Thursday, November 24, 2016 7:47:29 AM

After foo() executed, normally we would expect that the entirety of the inner scope of foo() would go away, because we know that the
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 50-50 | Added on Thursday, November 24, 2016 7:48:12 AM

engine employs a garbage collector that comes along and frees up memory once it’s no longer in use. Since it would appear that the con‐ tents of foo() are no longer in use, it would seem natural that they should be considered gone. But the “magic” of closures does not let this happen. That inner scope is in fact still in use, and thus does not go away. Who’s using it? The function bar() itself. By virtue of where it was declared, bar() has a lexical scope closure over that inner scope of foo(), which keeps that scope alive for bar() to reference at any later time. bar() still has a reference to that scope, and that reference is called closure
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 50-50 | Added on Thursday, November 24, 2016 7:49:08 AM

So, a few microseconds later, when the variable baz is invoked (in‐ voking the inner function we initially labeled bar), it duly has access to author-time lexical scope, so it can access the variable a just as we’d expect. The function is being invoked well outside of its author-time lexical scope. Closure lets the function continue to access the lexical scope it was defined in at author time. Of course, any of the various ways that functions can be passed around as values, and indeed invoked in other locations, are all ex‐ amples of observing/exercising closure.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 51-51 | Added on Thursday, November 24, 2016 7:51:13 AM

Whatever facility we use to transport an inner function outside of its lexical scope, it will maintain a scope reference to where it was origi‐ nally declared, and wherever we execute him, that closure will be ex‐ ercised
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 52-52 | Added on Thursday, November 24, 2016 7:55:38 AM

Be that timers, event handlers, Ajax requests, crosswindow messaging, web workers, or any of the other asynchronous (or synchronous!) tasks, when you pass in a callback function, get ready to sling some closure around
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 55-55 | Added on Thursday, November 24, 2016 10:07:07 PM

Look carefully at our analysis of the previous solution. We used an IIFE to create new scope per-iteration. In other words, we actually needed a per-iteration block scope.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 55-55 | Added on Thursday, November 24, 2016 10:07:55 PM

Chapter 3 showed us the let
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 56-56 | Added on Thursday, November 24, 2016 10:08:35 PM

declaration, which hijacks a block and declares a variable right there in the block. It essentially turns a block into a scope that we can close over. So, the following awesome code just works: for (var i=1; i<=5; i++) { let j = i; // yay, block-scope for closure! setTimeout( function timer(){ console.log( j ); }, j*
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 56-56 | Added on Thursday, November 24, 2016 10:09:46 PM

How cool is that? Block scoping and closure working hand-in-hand, solving all the world’s problems. I don’t know about you, but that makes me a happy JavaScripter
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 35-35 | Added on Thursday, November 24, 2016 10:17:25 PM

The let keyword attaches the variable declaration to the scope of whatever block (commonly a { .. } pair) it’s contained in. In other words, let implicitly hijacks any block’s scope for its variable decla‐ ration. var foo = true; if (foo) { let bar = foo * 2; bar = something
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 35-35 | Added on Thursday, November 24, 2016 10:17:38 PM

The let keyword attaches the variable declaration to the scope of whatever block (commonly a { .. } pair) it’s contained in. In other words, let implicitly hijacks any block’s scope for its variable decla‐ ration. var foo = true; if (foo) { let bar = foo * 2; bar = something( bar ); console.log( bar ); } console.log( bar ); // ReferenceError
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 48-48 | Added on Friday, November 25, 2016 7:32:09 AM

Let’s jump into some code to illustrate that definition. function foo() { var a = 2; function bar() { console.log( a ); // 2 } bar();
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 48-48 | Added on Friday, November 25, 2016 7:32:18 AM

} foo(); This code should look familiar from our discussions of nested scope. Function bar() has access to the variable a in the outer enclosing scope because of lexical scope look-up rules (in this case, it’s an RHS refer‐ ence look-up). Is this closure?
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 49-49 | Added on Friday, November 25, 2016 7:32:32 AM

Well, technically…perhaps. But by our what-you-need-to-know defi‐ nition above…not exactly. I think the most accurate way to explain bar() referencing a is via lexical scope look-up rules, and those rules are only (an important!) part of what closure is. From a purely academic perspective, what is said of the above snippet is that the function bar() has a closure over the scope of foo() (and indeed, even over the rest of the scopes it has access to, such as the global scope in our case). Put slightly differently, it’s said that bar() closes over the scope of foo(). Why? Because bar() appears nested inside of foo(). Plain and simple. But, closure defined in this way is not directly observable, nor do we see closure exercised in that snippet. We clearly see lexical scope, but closure remains sort of a mysterious shifting shadow behind the code. Let us then consider code that brings closure into full light: function foo() { var a = 2; function bar() { console.log( a ); } return bar; } var baz = foo(); baz(); // 2 -- Whoa, closure was just observed, man. The function bar() has lexical scope access to the inner scope of foo(). But then, we take bar(), the function itself, and pass it as a value. In this case, we return the function object itself that bar refer‐ ences. After we execute foo(), we assign the value it returned (our inner bar() function) to a variable called baz, and then we actually invoke baz(), which of course is invoking our inner function bar(), just by a different identifier reference. bar() is executed, for sure. But in this case, it’s executed outside of its declared lexical scope. After foo() executed, normally we would expect that the entirety of the inner scope of foo() would go away, because we know that the Nitty Gritty | 49
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 52-52 | Added on Friday, November 25, 2016 7:50:09 AM

essentially whenever and wherever you treat func‐ tions (that access their own respective lexical scopes) as first-class val‐ ues and pass them around, you are likely to see those functions exer‐ cising closure.
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 56-56 | Added on Friday, November 25, 2016 7:54:11 AM

declaration, which hijacks a block and declares a variable right there in the block. It essentially turns a block into a scope that we can close over. So, the following awesome code just works: for (var i=1; i<=5; i++) { let j = i; // yay, block-scope for closure! setTimeout( function timer(){ console.log( j ); }, j*1000 )
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 63-63 | Added on Friday, November 25, 2016 8:08:21 AM

Closures can trip us up, for instance with loops, if we’re not careful to recognize them and how they work. But they are also an immensely powerful tool, enabling patterns like modules in their various forms
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 64-64 | Added on Friday, November 25, 2016 8:08:29 AM

Modules require two key characteristics: 1) an outer wrapping func‐ tion being invoked, to create the enclosing scope 2) the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper
==========
You Don't Know JS - Scope _ Closures  
- Your Highlight on page 77-77 | Added on Friday, November 25, 2016 10:42:40 PM

The short explanation is that arrow functions do not behave at all like normal functions when it comes to their this binding. They discard all the normal rules for this binding, and instead take on the this value of their immediate lexical enclosing scope, whatever it is
