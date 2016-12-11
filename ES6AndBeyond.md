You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 5-5 | Added on Sunday, November 6, 2016 10:37:34 AM

https://github.com/paul millr/es6-
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 8-8 | Added on Sunday, November 6, 2016 10:41:33 AM

block scoping will
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 38-38 | Added on Wednesday, November 9, 2016 9:37:53 PM

object literal definition
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 38-38 | Added on Wednesday, November 9, 2016 9:40:37 PM

Computed property names
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 38-38 | Added on Wednesday, November 9, 2016 9:41:14 PM

concise method
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 39-39 | Added on Wednesday, November 9, 2016 9:42:36 PM

object literal. The
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 41-41 | Added on Thursday, November 10, 2016 8:11:48 AM

string literals
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 41-41 | Added on Thursday, November 10, 2016 8:12:24 AM

interpolation expressions
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 49-49 | Added on Friday, November 11, 2016 8:32:23 AM

We used the var self = this hack, and then referenced self.makeRequest(..), which inside the callback function we’re passing to addEventListener(..), the this binding will not be the same as it is in makeRequest(..) itself. In other words, because this bindings are dynamic, we fall back to the predictability of lexical scope via the self variable. Herein we finally can see the primary design characteristic of => arrow functions. Inside arrow functions, the this binding is not dynamic, but is instead lexical. In the previous snippet, if we used an arrow function for the callback, this will be predicta‐ bly what we wanted it to be. Consider: var controller = { makeRequest: function(..) { btn.addEventListener( "click", () => { // .. Arrow Functions | 49
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 50-50 | Added on Friday, November 11, 2016 8:35:42 AM

If you have a short, single-statement inline function expression, where the only statement is a return of some computed value, and that function doesn’t already make a this reference inside it, and there’s no self-reference (recursion, event 50 | Chapter 2: Syntax
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 51-51 | Added on Friday, November 11, 2016 8:38:57 AM

binding/unbinding), and you don’t reasonably expect the function to ever be that way, you can probably safely refactor it to be an => arrow function. • If you have an inner
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 51-51 | Added on Friday, November 11, 2016 8:39:11 AM

binding/unbinding), and you don’t reasonably expect the function to ever be that way, you can probably safely refactor it to be an => arrow function. • If you have an inner function expression that’s relying on a var self = this hack or a .bind(this) call on it in the enclosing function to ensure proper this binding, that inner function expression can probably safely become an => arrow function. • If you have an inner function expression that’s relying on something like var args = Array.prototype.slice.call(arguments) in the enclosing function to make a lexical copy of arguments, that inner function expression can probably safely become an => arrow function. • For everything else — normal function declarations, longer multi-statment func‐ tion expressions, functions which need a lexical name identifier self-reference (recursion, etc.), and any other function which doesn’t fit the previous character‐ istics — you should probably avoid => function syntax. Bottom line: => is about lexical binding of this, arguments, and super. These are intentional features designed to fix some common problems, not bugs, quirks, or mistakes in ES6. Don’t believe any hype that => is primarily, or even mostly, about fewer keystrokes. Whether you save keystrokes or waste them, you should know exactly what you are intentionally doing with every character typed. If you have a function that for any of these articulated reasons is not a good match for an => arrow function, but it’s being declared as part of an object literal, recall from “Concise Methods” earlier in this chapter that there’s another option for shorter function syntax. If you prefer a visual decision chart for how/why to pick an arrow function: for..of Loops Joining the for and for..in loops from the JavaScript we’re all familiar with, ES6 adds a for..of loop, which loops over the set of values produced by an iterator. The value you loop over with for..of must be an iterable, or it must be a value which can be coerced/boxed to an object (see the Types & Grammar title of this series) that is an iterable. An iterable is simply an object that is able to produce an iterator, which the loop then uses. Let’s compare for..of to for..in to illustrate the difference: for..of Loops | 51
==========
The Miracle Morning: The Not-So-Obvious Secret Guaranteed to Transform Your Life (Before 8AM) (Hal Elrod)
- Your Highlight on Location 528-530 | Added on Friday, November 11, 2016 7:49:33 PM

“On the one hand, we all want to be happy. On the other hand, we all know the things that make us happy. But we don’t do those things. Why? Simple. We are too busy. Too busy doing what? Too busy trying to be happy.”
==========
The Miracle Morning: The Not-So-Obvious Secret Guaranteed to Transform Your Life (Before 8AM) (Hal Elrod)
- Your Highlight on Location 553-554 | Added on Friday, November 11, 2016 7:51:27 PM

“Hal, if you want your life to be different, you have to be willing to do something different first!”
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 84-84 | Added on Sunday, November 13, 2016 7:57:41 AM

Consistency eases understanding and learning
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 98-98 | Added on Sunday, November 13, 2016 8:05:35 AM

don’t think it’s an exaggeration to suggest that the single most important code orga‐ nization pattern in all of JavaScript is, and always has been, the module. For myself, and I think for a large cross-section of the community, the module pattern drives the vast majority of code. 98 | Chapter 3: Organization
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 99-99 | Added on Sunday, November 13, 2016 8:08:04 AM

The Old Way The traditional module pattern is based on an outer function with inner variables and functions, and a returned “public API” with methods that have closure over the inner data and capabilities. It’s often expressed like this: function Hello(name) { function greeting() { console.log( "Hello " + name + "!" ); } // public API return { greeting: greeting }; } var me = Hello( "Kyle" ); me.greeting(); // Hello Kyle! This Hello(..) module can produce multiple instances by being called subsequent times. Sometimes, a module is only called for as a singleton — just needs one instance — in which case a slight variation on the previous snippet, using an IIFE, is common: var me = (function Hello(name){ function greeting() { console.log( "Hello " + name + "!" ); } // public API return { greeting: greeting }; })( "Kyle" ); me.greeting(); // Hello Kyle! This pattern is tried and tested. It’s also flexible enough to have a wide assortment of variations for a number of different scenarios. One of the most common is the Asynchronous Module Definition (AMD), and another is the Universal Module Definition (UMD). We won’t cover the particulars of these patterns and techniques here, but they’re explained extensively in many places online. Moving Forward As of ES6, we no longer need to rely on the enclosing function and closure to provide us with module support. ES6 modules have first class syntactic and functional sup‐ port. Modules | 99
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 100-100 | Added on Sunday, November 13, 2016 8:12:08 AM

Before we get into the specific syntax, it’s important to understand some fairly signifi‐ cant conceptual differences with ES6 modules compared to how you may have dealt with modules in the past: • ES6 modules are file-based, meaning one module per file. At this time, there is no standardized way of combining multiple modules into a single file. That means that if you are going to load ES6 modules directly into a browser web application, you will be loading them individually, not as a large bundle in a single file as has been common in performance optimization efforts. It’s expected that the contemporaneous advent of HTTP/2 will significantly mitigate any such performance concerns, as it operates on a persistent socket connection and thus can very efficiently load many smaller files in parallel and interleaved with each other. * The API of an ES6 module is static. That is, you define statically what all the top-level exports are on your module’s public API, and those cannot be amended later. Some uses are accustomed to being able to provide dynamic API definitions, where methods can be added/removed/replaced in response to run-time conditions. Either these uses will have to change to fit with ES6 static APIs, or they will have to restrain the dynamic changes to properties/methods of a second-level object. * ES6 modules are singletons. That is, there’s only one instance of the module, which maintains its state. Every time you import that module into another module, you get a reference to the one centralized instance. If you want to be able to produce multiple module instances, your module will need to provide some sort of factory to do it. * The prop‐ erties and methods you expose on a module’s public API are not just normal assign‐ ments of values or references. They are actual bindings (almost like pointers) to the identifiers in your inner module definition. In pre-ES6 modules, if you put a property on your public API that holds a primitive value like a number or string, that property assignment was by value-copy, and any internal update of a corresponding variable would be separate and not affect the pub‐ lic copy on the API object. With ES6, exporting a local private variable, even if it currently holds a primitive string/number/etc, exports a binding to to the variable. If the module changes the variable’s value, the external import binding now resolves to that new value. * Import‐ ing a module is the same thing as statically requesting it to load (if it hasn’t already). If you’re in a browser, that implies a blocking load over the network. If you’re on a server (i.e., Node.js), it’s a blocking load from the filesystem. However, don’t panic about the performance implications. Because ES6 modules have static definitions, the import requirements can be statically scanned, and loads will happen preemptively, even before you’ve used the module. 100 | Chapter 3: Organization
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 101-101 | Added on Sunday, November 13, 2016 8:13:37 AM

ES6 doesn’t actually specify or handle the mechanics of how these load requests work. There’s a separate notion of a Module Loader, where each hosting environment (browser, Node.js, etc.) provides a default Loader appropriate to the environment. The importing of a module uses a string value to represent where to get the module (URL, file path, etc), but this value is opaque in your program and only meaningful to the Loader itself. You can define your own custom Loader if you want more fine-grained control than the default Loader affords — which is basically none, since it’s totally hidden from your program’s code. As you can see, ES6 modules will serve the overall use-case of organizing code with encapsulation, controlling public APIs, and referencing dependency imports. But they have a very particular way of doing so, and that may or may not fit very closely with how you’ve already been doing modules for years. CommonJS There’s a similar, but not fully compatible, module syntax called CommonJS, which is familiar to those in the Node.js ecosystem. For lack of a more tactful way to say this, in the long run, ES6 modules essentially are bound to supercede all previous formats and standards for modules, even Com‐ monJS, as they are built on syntactic support in the language. This will, in time, inevi‐ tably win out as the superior approach, if for no other reason than ubiquity. We face a fairly long road to get to that point, though. There are literally hundreds of thousands of CommonJS style modules in the server-side JavaScript world, and ten times that many modules of varying format standards (UMD, AMD, ad hoc) in the browser world. It will take many years for the transitions to make any significant pro‐ gress. In the interim, module transpilers/converters will be an absolute necessity. You might as well just get used to that new reality. Whether you author in regular modules, AMD, UMD, CommonJS, or ES6, these tools will have to parse and convert to a for‐ mat that is suitable for whatever environment your code will run in. For Node.js, that probably means (for now) that the target is CommonJS. For the browser, it’s probably UMD or AMD. Expect lots of flux on this over the next few years as these tools mature and best practices emerge. From here on out, my best advice on modules is this: whatever format you’ve been religiously attached to with strong affinity, also develop an appreciation for and understanding of ES6 modules, such as they are, and let your other module tenden‐ cies fade. They are the future of modules in JS, even if that reality is a bit of a ways off. Modules | 101
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 102-102 | Added on Sunday, November 13, 2016 8:17:24 AM

named exports
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 102-102 | Added on Sunday, November 13, 2016 8:18:05 AM

Anything you don’t label with export stays private inside the scope of the module. That is, even though something like var bar = .. looks like it’s declaring at the toplevel global scope, the top-level scope is actually the module itself; there is no global scope in modules. 102 | Chapter 3: Organization
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 103-103 | Added on Sunday, November 13, 2016 8:21:52 AM

default export
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 113-113 | Added on Tuesday, November 15, 2016 8:02:28 AM

Module Loader
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 115-115 | Added on Tuesday, November 15, 2016 9:49:20 PM

prototype mechanism
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 116 | Added on Tuesday, November 15, 2016 9:53:13 PM

prototype
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 119-119 | Added on Wednesday, November 16, 2016 7:47:53 AM

[Proto type]] delegation
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 124-124 | Added on Thursday, November 17, 2016 7:59:35 AM

Modules allow private encapsulation of implementation details with a publicly exported API. Module definitions are file-based, singleton instances, and stati‐ cally resolved at compile
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 127-127 | Added on Thursday, November 17, 2016 8:02:05 AM

async flow control
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 128 | Added on Thursday, November 17, 2016 8:07:40 AM

order
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 128 | Added on Thursday, November 17, 2016 8:07:58 AM

predictability
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 134-134 | Added on Thursday, November 17, 2016 10:22:10 PM

The important pattern to recognize: a generator can yield a promise, and that promise can then be wired to resume the generator with its fulfillment value
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 135-135 | Added on Thursday, November 17, 2016 10:23:53 PM

Promises are a trustable system that univerts the inversion of control of normal call‐ backs or thunks (see the Async & Performance title of this series). So, combining the trustability of Promises and the synchronicity of code in generators effectively addresses all the major deficiences of callbacks
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 136-136 | Added on Friday, November 18, 2016 7:58:55 AM

trol logic in your program, you can and should use a promise-yielding generator driven by a run utility to express the flow control in a synchronous-fashion. This will make for much easier to understand and maintain code. This yield-a-promise-resume-the-generator pattern is going to be so common and so powerful, the next version of JavaScript is almost certainly going to introduce a new function type which will do it automatically without needing the run utility. We’ll cover (expected name) `async function`s in Chapter 8
==========
You Don't Know JS - ES6 _ Beyond
- Your Highlight on page 211-211 | Added on Sunday, November 20, 2016 11:09:30 AM

One of the holy grails of front-end web development is data binding — listening for updates to a data object and syncing the DOM representation of that data. Most JS frameworks provide some mechanism for these sorts of operations. It appears likely that post ES6, we’ll see support added directly to the language, via a utility called Object.observe(..). Essentially, the idea is that you can set up a lis‐ tener to observe an object’s changes, and have a callback called any time a change occurs. You can then update the DOM accordingly, for instance. There are six types of changes that you can observe: • add • update • delete • reconfigure Object.observe(..) | 211
