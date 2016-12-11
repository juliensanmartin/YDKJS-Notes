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
