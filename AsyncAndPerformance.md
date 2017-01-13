The JS engine doesn't run in isolation. It runs inside a hosting environment, which is for most developers the typical web browser.

But the one common "thread" (that's a not-so-subtle asynchronous joke, for what it's worth) of all these environments is that they have a mechanism in them that handles executing multiple chunks of your program over time, at each moment invoking the JS engine, called the "event loop."

In other words, the JS engine has had no innate sense of time, but has instead been an on-demand execution environment for any arbitrary snippet of JS. It's the surrounding environment that has always scheduled "events" (JS code executions).

So, for example, when your JS program makes an Ajax request to fetch some data from a server, you set up the "response" code in a function (commonly called a "callback"), and the JS engine tells the hosting environment, "Hey, I'm going to suspend execution for now, but whenever you finish with that network request, and you have some data, please call this function back."

As you can see, there's a continuously running loop represented by the while loop, and each iteration of this loop is called a "tick."

For each tick, if an event is waiting on the queue, it's taken off and executed. These events are your function callbacks. It's important to note that setTimeout(..) doesn't put your callback on the event loop queue. What it does is set up a timer; when the timer expires, the environment places your callback into the event loop, such that some future tick will pick it up and execute it.

It's very common to conflate the terms "async" and "parallel" but they are actually quite different. Remember, async is about the gap between now and later. But parallel is about things being able to occur simultaneously.

The most common tools for parallel computing are processes and threads. Processes and threads execute independently and may execute simultaneously: on separate processors, or even separate computers, but multiple threads can share the memory of a single process.

An event loop, by contrast, breaks its work into tasks and executes them in serial, disallowing parallel access and changes to shared memory. Parallelism and "serialism" can coexist in the form of cooperating event loops in separate threads.

Because of JavaScript's single-threading, the code inside of foo() (and bar()) is atomic, which means that once foo() starts running, the entirety of its code will finish before any of the code in bar() can run, or vice versa. This is called "run-to-completion" behavior.

Because foo() can't be interrupted by bar(), and bar() can't be interrupted by foo(), this program only has two possible outcomes depending on which starts running

Chunk 1 is synchronous (happens now), but chunks 2 and 3 are asynchronous (happen later), which means their execution will be separated by a gap of time.

Two outcomes from the same code means we still have nondeterminism!

In other words, it's more deterministic than threads would have been.

But it's at the function (event) ordering level, rather than at the statement ordering level

As applied to JavaScript's behavior, this function-ordering nondeterminism is the common term "race condition,"

Concurrency is when two or more "processes" are executing simultaneously over the same period, regardless of whether their individual constituent operations happen in parallel (at the same instant on separate processors or cores) or not. You can think of concurrency then as "process"-level (or task-level) parallelism, as opposed to operation-level parallelism (separate-processor threads).

If they don't interact, nondeterminism is perfectly acceptable.

More commonly, concurrent "processes" will by necessity interact, indirectly through scope and/or the DOM.

When such interaction will occur, you need to coordinate these interactions to prevent "race conditions," as described earlier.

Here's a simple example of two concurrent "processes" that interact because of implied ordering, which is only sometimes broken:
```JavaScript
var res = [];
function response(data) {    
  res.push( data );
}
// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

The concurrent "processes" are the two response() calls that will be made to handle the Ajax responses. They can happen in either-first order.

The same reasoning from this scenario would apply if multiple concurrent function calls were interacting with each other through the shared DOM, like one updating the contents of a <div> and the other updating the style or attributes of the <div> (e.g., to make the DOM element visible once it has content). You probably wouldn't want to show the DOM element before it had content, so the coordination must ensure proper ordering interaction.

Another expression of concurrency coordination is called "cooperative concurrency." Here, the focus isn't so much on interacting via value sharing in scopes

The goal is to take a long-running "process" and break it up into steps or batches so that other concurrent "processes" have a chance to interleave their operations into the event loop queue.
```JavaScript
var res = [];
// `response(..)` receives array of results from the Ajax call
function response(data) {    
  // add onto existing `res` array    
  res = res.concat(        
    // make a new transformed array with all `data` values doubled        
    data.map( function(val){            
      return val * 2;        
    } )    
  );
}

// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

If "http://some.url.1" gets its results back first, the entire list will be mapped into res all at once.

While such a "process" is running, nothing else in the page can happen, including no other response(..) calls, no UI updates, not even user events like scrolling, typing, button clicking, and the like. That's pretty painful.

So, to make a more cooperatively concurrent system, one that's friendlier and doesn't hog the event loop queue, you can process these results in asynchronous batches, after each one "yielding" back to the event loop to let other waiting events happen.

```JavaScript
var res = [];
// `response(..)` receives array of results from the Ajax call
function response(data) {    
  // let's just do 1000 at a time    
  var chunk = data.splice( 0, 1000 );    
  // add onto existing `res` array    
  res = res.concat(        
    // make a new transformed array with all `chunk` values doubled        
    chunk.map( function(val){            
      return val * 2;        
    } )    
  );    
  // anything left to process?    
  if (data.length > 0) {        
    // async schedule next batch        
    setTimeout( function(){            
      response( data );        
    }, 0 );    
  }
}
// ajax(..) is some arbitrary Ajax function given by a library
ajax( "http://some.url.1", response );
ajax( "http://some.url.2", response );
```

We process the data set in maximum-sized chunks of 1,000 items.

By doing so, we ensure a short-running "process," even if that means many more subsequent "processes," as the interleaving onto the event loop queue will give us a much more responsive (performant) site/app.

Of course, we're not interaction-coordinating the ordering of any of these "processes," so the order of results in res won't be predictable. If ordering was required, you'd need to use interaction techniques like those we discussed earlier, or ones we will cover in later chapters of this book.

We use the setTimeout(..0) (hack) for async scheduling, which basically just means "stick this function at the end of the current event loop queue."

So, the best way to think about this that I've found is that the "Job queue" is a queue hanging off the end of every tick in the event loop queue. Certain async-implied actions that may occur during a tick of the event loop will not cause a whole new event to be added to the event loop queue, but will instead add an item (aka Job) to the end of the current tick's Job queue. It's kinda like saying, "oh, here's this other thing I need to do later, but make sure it happens right away before anything else can happen."

Or, to use a metaphor: the event loop queue is like an amusement park ride, where once you finish the ride, you have to go to the back of the line to ride again. But the Job queue is like finishing the ride, but then cutting in line and getting right back on.

A Job can also cause more Jobs to be added to the end of the same queue. So, it's theoretically possible that a Job "loop" (a Job that keeps adding another Job, etc.) could spin indefinitely, thus starving the program of the ability to move on to the next event loop tick. This would conceptually be almost the same as just expressing a long-running or infinite loop (like while (true) ..) in your code.

Jobs are kind of like the spirit of the setTimeout(..0) hack, but implemented in such a way as to have a much more well-defined and guaranteed ordering: later, but as soon as possible.

the callback is the most fundamental async pattern in the language.

The reason it's difficult for us as developers to write async evented code, especially when all we have is the callback to do it, is that stream of consciousness thinking/planning is unnatural for most of us.

We think in step-by-step terms, but the tools (callbacks) available

to us in code are not expressed in a step-by-step fashion once we move from synchronous to asynchronous.

And that is why it's so hard to accurately author and reason about async JS code with callbacks: because it's not how our brain planning works.

That is what "callback hell" is all about! The nesting/indentation are basically a side show, a red herring.

Are you catching the notion here that our sequential, blocking brain planning behaviors just don't map well onto callback-oriented async code? That's the first major deficiency to articulate about callbacks: they express asynchrony in code in ways our brains have to fight just to keep in sync with (pun intended!).

We call this "inversion of control," when you take part of your program and give over control of its execution to another third party. There's an unspoken "contract" that exists between your code and the third-party utility -- a set of things you expect to be maintained.

Or perhaps still safe but friendlier:
```JavaScript
function addNumbers(x,y) {    
  // ensure numerical input    
  x = Number( x );    
  y = Number( y );    
  // + will safely do numeric addition    
  return x + y;
}

addNumbers( 21, 21 );    // 42
addNumbers( 21, "21" );    // 42
```

However you go about it, these sorts of checks/normalizations are fairly common on function inputs, even with code we theoretically entirely trust. In a crude sort of way, it's like the programming equivalent of the geopolitical principle of "Trust But Verify." So, doesn't it stand to reason that we should do the same thing about composition of async function callbacks, not just with truly external code but even with code we know is generally "under our own control"? Of course we should. But callbacks don't really offer anything to assist us. We have to construct all that machinery ourselves, and it often ends up being a lot of boilerplate/overhead that we repeat for every single async callback.

The most troublesome problem with callbacks is inversion of control leading to a complete breakdown along all those trust lines. If you have code that uses callbacks, especially but not exclusively with third-party utilities, and you're not already applying some sort of mitigation logic for all these inversion of control trust issues, your code has bugs in it right now even though they may not have bitten you yet. Latent bugs are still bugs. Hell indeed.

You can see just how quickly the unpredictability of Zalgo can threaten any JS program. So the silly-sounding "never release Zalgo" is actually incredibly common and solid advice. Always be asyncing.

Note: For more information on Zalgo, see Oren Golan's "Don't Release Zalgo!" (https://github.com/oren/oren.github.io/blob/master/posts/zalgo.md) and Isaac Z. Schlueter's "Designing APIs for Asynchrony" (http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony).

Consider:
```JavaScript
function result(data) {    
  console.log( a );
}
var a = 0;
ajax( "..pre-cached-url..", result );
a++;

```

Will this code print 0 (sync callback invocation) or 1 (async callback invocation)? Depends... on the conditions.

What if you don't know whether the API in question will always execute async? You could invent a utility like this asyncify(..) proof-of-concept:

```JavaScript
function asyncify(fn) {    
  var orig_fn = fn,        
  intv = setTimeout(
    function(){            
      intv = null;            
      if (fn) fn();        
    }, 0 );    
    fn = null;    
    return function() {        
      // firing too quickly, before `intv` timer has fired to        
      // indicate async turn has passed?        
      if (intv) {            
        fn = orig_fn.bind.apply(                
          orig_fn,                
          // add the wrapper's `this` to the `bind(..)`                
          // call parameters, as well as currying any                
          // passed in parameters                
          [this].concat( [].slice.call( arguments ) )            
        );        
      }        
      // already async        
      else {            
        // invoke original function            
        orig_fn.apply( this, arguments );        
      }    
    };
  }
  ```

You use asyncify(..) like this:

```JavaScript
function result(data) {    
  console.log( a );
}
var a = 0;
ajax( "..pre-cached-url..", asyncify( result ) );
a++;
```

Whether the Ajax request is in the cache and resolves to try to call the callback right away, or must be fetched over the wire and thus complete later asynchronously, this code will always output 1 instead of 0 -- result(..) cannot help but be invoked asynchronously, which means the a++ has a chance to run before result(..) does.

Yay, another trust issued "solved"! But it's inefficient, and yet again more bloated boilerplate to weigh your project down.

That's just the story, over and over again, with callbacks. They can do pretty much anything you want, but you have to be willing to work hard to get it, and oftentimes this effort is much more than you can or should spend on such code reasoning.

We need a generalized solution to all of the trust issues, one that can be reused for as many callbacks as we create without all the extra boilerplate overhead.

we identified two major categories of deficiencies with using callbacks to express program asynchrony and manage concurrency: lack of sequentiality and lack of trustability.

inversion of control,

Recall that we wrap up the continuation of our program in a callback function, and hand that callback over to another party (potentially even external code) and just cross our fingers that it will do the right thing with the invocation of the callback.

But what if we could uninvert that inversion of control? What if instead of handing the continuation of our program to another party, we could expect it to return us a capability to know when its task finishes, and then our code could decide what to do next?
This paradigm is called Promises.

Future Value Imagine this scenario: I walk up to the counter at a fast-food restaurant, and place an order for a cheeseburger. I hand the cashier $1.47. By placing my order and paying for it, I've made a request for a value back (the cheeseburger). I've started a transaction. But often, the chesseburger is not immediately available for me. The cashier hands me something in place of my cheeseburger: a receipt with an order number on it. This order number is an IOU ("I owe you") promise that ensures that eventually, I should receive my cheeseburger. So I hold onto my receipt and order number. I know it represents my future cheeseburger, so I don't need to worry about it anymore -- aside from being hungry! While I wait, I can do other things, like send a text message to a friend that says, "Hey, can you come join me for lunch? I'm going to eat a cheeseburger." I am reasoning about my future cheeseburger already, even though I don't have it in my hands yet. My brain is able to do this because it's treating the order number as a placeholder for the cheeseburger. The placeholder essentially makes the value time independent. It's a future value. Eventually, I hear, "Order 113!" and I gleefully walk back up to the counter with receipt in hand. I hand my receipt to the cashier, and I take my cheeseburger in return. In other words, once my future value was ready, I exchanged my value-promise for the value itself. But there's another possible outcome. They call my order number, but when I go to retrieve my cheeseburger, the cashier regretfully informs me, "I'm sorry, but we appear to be all out of cheeseburgers." Setting aside the customer frustration of this scenario for a moment, we can see an important characteristic of future values: they can either indicate a success or failure. Every time I order a cheeseburger, I know that I'll either get a cheeseburger eventually, or I'll get the sad news of the cheeseburger shortage, and I'll have to figure out something else to eat for lunch.

Values Now and Later This all might sound too mentally abstract to apply to your code. So let's be more concrete. However, before we can introduce how Promises work in this fashion, we're going to derive in code that we already understand -- callbacks! -- how to handle these future values. When you write code to reason about a value, such as performing math on a number, whether you realize it or not, you've been assuming something very fundamental about that value, which is that it's a concrete now value already:
```JavaScript
var x, y = 2;
console.log( x + y ); // NaN  <-- because `x` isn't set yet
```
The x + y operation assumes both x and y are already set. In terms we'll expound on shortly, we assume the x and y values are already resolved. It would be nonsense to expect that the + operator by itself would somehow be magically capable of detecting and waiting around until both x and y are resolved (aka ready), only then to do the operation. That would cause chaos in the program if different statements finished now and others finished later, right? How could you possibly reason about the relationships between two statements if either one (or both) of them might not be finished yet? If statement 2 relies on statement 1 being finished, there are just two outcomes: either statement 1 finished right now and everything proceeds fine, or statement 1 didn't finish yet, and thus statement 2 is going to fail.

If this sort of thing sounds familiar from Chapter 1, good! Let's go back to our x + y math operation. Imagine if there was a way to say, "Add x and y, but if either of them isn't ready yet, just wait until they are. Add them as soon as you can." Your brain might have just jumped to callbacks. OK, so...
```JavaScript
function add(getX,getY,cb) {    
  var x, y;    
  getX( function(xVal){        
    x = xVal;        // both are ready?        
    if (y != undefined) {            
      cb( x + y );    // send along sum        
    }    
  } );    
  getY( function(yVal){        
    y = yVal;        
    // both are ready?        
    if (x != undefined) {            
      cb( x + y );    // send along sum        
    }    
  } );
}
// `fetchX()` and `fetchY()` are sync or async
// functions

add( fetchX, fetchY, function(sum){    
  console.log( sum ); // that was easy, huh?
} );
```

Take just a moment to let the beauty (or lack thereof) of that snippet sink in (whistles patiently). While the ugliness is undeniable, there's something very important to notice about this async pattern. In that snippet, we treated x and y as future values, and we express an operation add(..) that (from the outside) does not care whether x or y or both are available right away or not. In other words, it normalizes the now and later, such that we can rely on a predictable outcome of the add(..) operation. By using an add(..) that is temporally consistent -- it behaves the same across now and later times -- the async code is much easier to reason about. To put it more plainly: to consistently handle both now and later, we make both of them later: all operations become async. Of course, this rough callbacks-based approach leaves much to be desired. It's just a first tiny step toward realizing the benefits of reasoning about future values without worrying about the time aspect of when it's available or not.

Moreover, once a Promise is resolved, it stays that way forever -- it becomes an immutable value at that point -- and can then be observed as many times as necessary. Note: Because a Promise is externally immutable once resolved, it's now safe to pass that value around to any party and know that it cannot be modified accidentally or maliciously. This is especially true in relation to multiple parties observing the resolution of a Promise. It is not possible for one party to affect another party's ability to observe Promise resolution. Immutability may sound like an academic topic, but it's actually one of the most fundamental and important aspects of Promise design, and shouldn't be casually passed over.

Promises are an easily repeatable mechanism for encapsulating and composing future values.

Let's imagine calling a function foo(..) to perform some task. We don't know about any of its details, nor do we care. It may complete the task right away, or it may take a while. We just simply need to know when foo(..) finishes so that we can move on to our next task. In other words, we'd like a way to be notified of foo(..)'s completion so that we can continue. In typical JavaScript fashion, if you need to listen for a notification, you'd likely think of that in terms of events. So we could reframe our need for notification as a need to listen for a completion (or continuation) event emitted by foo(..).

With callbacks, the "notification" would be our callback invoked by the task (foo(..)). But with Promises, we turn the relationship around, and expect that we can listen for an event from foo(..), and when notified, proceed accordingly.

First, consider some pseudocode:
```JavaScript
foo(x) {    
  // start doing something that could take a while
}
foo( 42 ) on (foo "completion") {    
  // now we can do the next step!
}

on (foo "error") {    
  // oops, something went wrong in `foo(..)`
}
```

We call foo(..) and then we set up two event listeners, one for "completion" and one for "error" -- the two possible final outcomes of the foo(..) call. In essence, foo(..) doesn't even appear to be aware that the calling code has subscribed to these events, which makes for a very nice separation of concerns.

Here's the more natural way we could express that in JS:
```JavaScript
function foo(x) {    
  // start doing something that could take a while    
  // make a `listener` event notification    
  // capability to return    
  return listener;
}

var evt = foo( 42 );
evt.on( "completion", function(){    
  // now we can do the next step!
} );

evt.on( "failure", function(err){    
  // oops, something went wrong in `foo(..)`
} );
```

foo(..) expressly creates an event subscription capability to return back, and the calling code receives and registers the two event handlers against it. The inversion from normal callback-oriented code should be obvious, and it's intentional. Instead of passing the callbacks to foo(..), it returns an event capability we call evt, which receives the callbacks. But if you recall from Chapter 2, callbacks themselves represent an inversion of control. So inverting the callback pattern is actually an inversion of inversion, or an uninversion of control -- restoring control back to the calling code where we wanted it to be in the first place. One important benefit is that multiple separate parts of the code

Here's the more natural way we could express that in JS:
```JavaScript
function foo(x) {    
  // start doing something that could take a while    
  // make a `listener` event notification    
  // capability to return    
  return listener;
}

var evt = foo( 42 );

evt.on( "completion", function(){    
  // now we can do the next step!
});

evt.on( "failure", function(err){    
  // oops, something went wrong in `foo(..)`
});
```

foo(..) expressly creates an event subscription capability to return back, and the calling code receives and registers the two event handlers against it. The inversion from normal callback-oriented code should be obvious, and it's intentional. Instead of passing the callbacks to foo(..), it returns an event capability we call evt, which receives the callbacks. But if you recall from Chapter 2, callbacks themselves represent an inversion of control. So inverting the callback pattern is actually an inversion of inversion, or an uninversion of control -- restoring control back to the calling code where we wanted it to be in the first place. One important benefit is that multiple separate parts of the code can be given the event listening capability, and they can all independently be notified of when foo(..) completes to perform subsequent steps after its completion:
```JavaScript
var evt = foo( 42 );
// let `bar(..)` listen to `foo(..)`'s completion
bar( evt );
// also, let `baz(..)` listen to `foo(..)`'s completion
baz( evt );

```
Uninversion of control enables a nicer separation of concerns, where bar(..) and baz(..) don't need to be involved in how foo(..) is called. Similarly, foo(..) doesn't need to know or care that bar(..) and baz(..) exist or are waiting to be notified when foo(..) completes. Essentially, this evt object is a neutral third-party negotiation between the separate concerns.

As you may have guessed by now, the evt event listening capability is an analogy for a Promise.

In a Promise-based approach, the previous snippet would have foo(..) creating and returning a Promise instance, and that promise would then be passed to bar(..) and baz(..).

Note: The Promise resolution "events" we listen for aren't strictly events (though they certainly behave like events for these purposes), and they're not typically called "completion" or "error". Instead, we use then(..) to register a "then" event. Or perhaps more precisely, then(..) registers "fulfillment" and/or "rejection" event(s), though we don't see those terms used explicitly in the code.

Consider:
```JavaScript
function foo(x) {    
  // start doing something that could take a while    
  // construct and return a promise    
  return new Promise( function(resolve,reject){        
    // eventually, call `resolve(..)` or `reject(..)`,        
    // which are the resolution callbacks for        
    // the promise.    
  } );
}

var p = foo( 42 );
bar( p );
baz( p );
```
Note: The pattern shown with `new Promise( function(..){ .. } )` is generally called the "revealing constructor". The function passed in is executed immediately (not async deferred, as callbacks to then(..) are), and it's provided two parameters, which in this case we've named resolve and reject. These are the resolution functions for the promise. resolve(..) generally signals fulfillment, and reject(..) signals rejection.

```JavaScript
function bar(fooPromise) {    
  // listen for `foo(..)` to complete    
  fooPromise.then(        
    function(){            
      // `foo(..)` has now finished, so            
      // do `bar(..)`'s task        
    },        
    function(){            
      // oops, something went wrong in `foo(..)`        
    }    
  );
}
```

Another way to approach this is:
```JavaScript
function bar() {    
  // `foo(..)` has definitely finished, so    
  // do `bar(..)`'s task
}
function oopsBar() {    
  // oops, something went wrong in `foo(..)`,    
  // so `bar(..)` didn't run
}
// ditto for `baz()` and `oopsBaz()`
var p = foo( 42 );
p.then( bar, oopsBar );
```

As such, it was decided that the way to recognize a Promise (or something that behaves like a Promise) would be to define something called a "thenable" as any object or function which has a then(..) method on it. It is assumed that any such value is a Promise-conforming thenable.

The general term for "type checks" that make assumptions about a value's "type" based on its shape (what properties are present) is called "duck typing"

"If it looks like a duck, and quacks like a duck, it must be a duck" (see
```JavaScript
if ( p !== null && 
  (        
    typeof p === "object" ||        
    typeof p === "function"    
  ) &&    
  typeof p.then === "function"
) {    
  // assume it's a thenable!
}else {    
  // not a thenable
}
```

Yuck!

the single most important characteristic that the Promise pattern establishes: trust.

Let's start by reviewing the trust issues with callbacks-only coding. When you pass a callback to a utility foo(..), it might: Call the callback too early Call the callback too late (or never) Call the callback too few or too many times Fail to pass along any necessary environment/parameters swallow any errors/exceptions that may happen The characteristics of Promises are intentionally designed to provide useful, repeatable answers to all these concerns.

Calling Too Early

when you call then(..) on a Promise, even if that Promise was already resolved, the callback you provide to then(..) will always be called asynchronously

Calling Too Late

a Promise's then(..) registered observation callbacks are automatically scheduled when either resolve(..) or reject(..) are called by the Promise creation capability.

For example:
```JavaScript
p.then( function(){    
  p.then( function(){        
    console.log( "C" );    
  });    
  console.log( "A" );
});

p.then( function(){    
  console.log( "B" );
} );
// A B
```
Here, "C" cannot interrupt and precede "B", by virtue of how Promises are defined to operate.

It's important to note, though, that there are lots of nuances of scheduling where the relative ordering between callbacks chained off two separate Promises is not reliably predictable. If two promises p1 and p2 are both already resolved, it should be true that p1.then(..); p2.then(..) would end up calling the callback(s) for p1 before the ones for p2.

But
```JavaScript
var p3 = new Promise( function(resolve,reject){  
    resolve( "B" );
} );

var p1 = new Promise( function(resolve,reject){    
  resolve( p3 );
} );

var p2 = new Promise( function(resolve,reject){    
  resolve( "A" );
} );

p1.then( function(v){    
  console.log( v );
} );

p2.then( function(v){    
  console.log( v );
} );
// A B  <-- not  B A  as you might expect
```

p1 is resolved not with an immediate value, but with another promise p3 which is itself resolved with the value "B". The specified behavior is to unwrap p3 into p1, but asynchronously, so p1's callback(s) are behind p2's callback(s) in the asynchronus Job queue

a good practice is not to code in such a way where the ordering of multiple callbacks matters at all. Avoid that if you can.

Promises are a pattern that augments callbacks with trustable semantics, so that the behavior is more reason-able and more reliable. By uninverting the inversion of control of callbacks, we place the control with a trustable system (Promises) that was designed specifically to bring sanity to our async.

Every time you call then(..) on a Promise, it creates and returns a new Promise, which we can chain with. Whatever value you return from the then(..) call's fulfillment callback (the first parameter) is automatically set as the fulfillment of the chained Promise (from the first point).

```JavaScript
var p = Promise.resolve( 21 );
var p2 = p.then( function(v){    
  console.log( v );    
  // 21    
  // fulfill `p2` with value `42`    r
  eturn v * 2;
} );
// chain off `p2`
p2.then( function(v){    
  console.log( v );    
  // 42
} );
```

By returning v * 2 (i.e., 42), we fulfill the p2 promise that the first then(..) call created and returned. When p2's then(..) call runs, it's receiving the fulfillment from the return v * 2 statement. Of course, p2.then(..) creates yet another promise, which we could have stored in a p3 variable. But it's a little annoying to have to create an intermediate variable p2 (or p3, etc.). Thankfully, we can easily just chain these together:
```JavaScript
var p = Promise.resolve( 21 );
p.then( function(v){    
  console.log( v );    
  // 21    
  // fulfill the chained promise with value `42`    
  return v * 2;
} )// here's the chained promise
.then( function(v){    
  console.log( v );    
  // 42
} );
```

What if we want step 2 to wait for step 1 to do something asynchronous? We're using an immediate return statement, which immediately fulfills the chained promise.

The key to making a Promise sequence truly async capable at every step is to recall how Promise.resolve(..) operates when what you pass to it is a Promise or thenable instead of a final value. Promise.resolve(..) directly returns a received genuine Promise, or it unwraps the value of a received thenable -- and keeps going recursively while it keeps unwrapping thenables.

If we introduce asynchrony to that wrapping promise, everything still nicely works the same:
```JavaScript
var p = Promise.resolve( 21 );
p.then( function(v){    
  console.log( v );    
  // 21    
  // create a promise to return    
  return new Promise( function(resolve,reject){        
    // introduce asynchrony!        
    setTimeout( function(){            
      // fulfill with value `42`            
      resolve( v * 2 );        
    }, 100 );    
  } );
} )
.then( function(v){    
  // runs after the 100ms delay in the previous step    
  console.log( v );    // 42
} );
```

That's incredibly powerful! Now we can construct a sequence of however many async steps we want, and each step can delay the next step (or not!), as necessary.

To further the chain illustration, let's generalize a delay-Promise creation (without resolution messages) into a utility we can reuse for multiple steps:
```JavaScript
function delay(time) {    
  return new Promise( function(resolve,reject){        
    setTimeout( resolve, time );    
  } );
}
delay( 100 ) // step 1
.then( function STEP2(){    
  console.log( "step 2 (after 100ms)" );    
  return delay( 200 );
} )
.then( function STEP3(){    
  console.log( "step 3 (after another 200ms)" );
} )
.then( function STEP4(){    
  console.log( "step 4 (next Job)" );    
  return delay( 50 );
} )
.then( function STEP5(){    
  console.log( "step 5 (after another 50ms)" );
} )...
```
Calling delay(200) creates a promise that will fulfill in 200ms, and then we return that from the first then(..) fulfillment callback, which causes the second then(..)'s promise to wait on that 200ms promise.

To be honest, though, sequences of delays with no message passing isn't a terribly useful example of Promise flow control. Let's look at a scenario that's a little more practical. Instead of timers, let's consider making Ajax requests:
```JavaScript
// assume an `ajax( {url}, {callback} )` utility
// Promise-aware ajax
function request(url) {    
  return new Promise( function(resolve,reject){        
    // the `ajax(..)` callback should be our        
    // promise's `resolve(..)` function        
    ajax( url, resolve );    
  } );
}
```

We first define a request(..) utility that constructs a promise

to represent the completion of the ajax(..) call:
```JavaScript
request( "http://some.url.1/" )
.then( function(response1){    
  return request( "http://some.url.2/?v=" + response1 );
} )
.then( function(response2){    
  console.log( response2 );
} );
```

Let's review briefly the intrinsic behaviors of Promises that enable chaining flow control: A then(..) call against one Promise automatically produces a new Promise to return from the call. Inside the fulfillment/rejection handlers, if you return a value or an exception is thrown, the new returned (chainable) Promise is resolved accordingly. If the fulfillment or rejection handler returns a Promise, it is unwrapped, so that whatever its resolution is will become the resolution of the chained Promise returned from the current then(..).

While chaining flow control is helpful, it's probably most accurate to think of it as a side benefit of how Promises compose (combine) together, rather than the main intent. As we've discussed in detail several times already, Promises normalize asynchrony and encapsulate time-dependent value state, and that is what lets us chain them together in this useful way.

There's some slight confusion around the terms "resolve," "fulfill," and "reject"

In callbacks, some standards have emerged for patterned error handling, most notably the "error-first callback" style:
```JavaScript
function foo(cb) {    
  setTimeout( function(){        
    try {            
      var x = baz.bar();            
      cb( null, x );
      // success!        
    }        
    catch (err) {            
      cb( err );        
    }    
  }, 100 );
}

foo( function(err,val){    
  if (err) {        
    console.error( err ); // bummer :(    
  }    
  else {        
    console.log( val );    
  }
} );
```

Note: The try..catch here works only from the perspective that the baz.bar() call will either succeed or fail immediately, synchronously.

So we come back to error handling in Promises, with the rejection handler passed to then(..). Promises don't use the popular "error-first callback" design style, but instead use "split callbacks" style; there's one callback for fulfillment and one for rejection:
```JavaScript
var p = Promise.reject( "Oops" );
p.then(    
  function fulfilled(){
     // never gets here    
  },    
  function rejected(err){        
    console.log( err ); // "Oops"    
  }
);
```
Consider:
```JavaScript
var p = Promise.resolve( 42 );
p.then(    
  function fulfilled(msg){        
    // numbers don't have string functions,        
    // so will throw an error        
    console.log( msg.toLowerCase() );    
  },    
  function rejected(err){        
    // never gets here    
  });
);
```

If the msg.toLowerCase() legitimately throws an error (it does!), why doesn't our error handler get notified? As we explained earlier, it's because that error handler is for the p promise, which has already been fulfilled with value 42. The p promise is immutable, so the only promise that can be notified of the error is the one returned from p.then(..), which in this case we don't capture.

Pit of Success The following is just theoretical, how Promises could be someday changed to behave. I believe it would be far superior to what we currently have. And I think this change would be possible even post-ES6 because I don't think it would break web compatibility with ES6 Promises. Moreover, it can be polyfilled/prollyfilled in, if you're careful. Let's take a look: Promises could default to reporting (to the developer console) any rejection, on the next Job or event loop tick, if at that exact moment no error handler has been registered for the Promise. For the cases where you want a rejected Promise to hold onto its rejected state for an indefinite amount of time before observing, you could call defer(), which suppresses automatic error reporting on that Promise. If a Promise is rejected, it defaults to noisily reporting that fact to the developer console (instead of defaulting to silence). You can opt out of that reporting either implicitly (by registering an error handler before rejection), or explicitly (with defer()). In either case, you control the false positives. Consider:
```JavaScript
var p = Promise.reject( "Oops" ).defer();
// `foo(..)` is Promise-aware
foo( 42 ).then(    
  function fulfilled(){        
    return p;    
  },    
  function rejected(err){        
    // handle `foo(..)` error    
  });...
```  

When we create p, we know we're going to wait a while to use/observe its rejection, so we call defer() -- thus no global reporting. defer() simply returns the same promise, for chaining purposes. The promise returned from foo(..) gets an error handler attached right away, so it's implicitly opted out and no global reporting for it occurs either. But the promise returned from the then(..) call has no defer() or error handler attached, so if it rejects (from inside either resolution handler), then it will be reported to the developer console as an uncaught error. This design is a pit of success. By default, all errors are either handled or reported -- what almost all developers in almost all cases would expect. You either have to register a handler or you have to intentionally opt out, and indicate you intend to defer error handling until later; you're opting for the extra responsibility in just that specific case. The only real danger in this approach is if you defer() a Promise but then fail to actually ever observe/handle its rejection. But you had to intentionally call defer() to opt into that pit of despair -- the default was the pit of success -- so there's not much else we could do to save you from your own mistakes. I think there's still hope for Promise error handling (post-ES6). I hope the powers that be will rethink the situation and consider this alternative. In the meantime, you can implement this yourself (a challenging exercise for the reader!), or use a smarter Promise library that does so for you!

```JavaScript
Promise.all([ .. ])
```

In classic programming terminology, a "gate" is a mechanism that waits on two or more parallel/concurrent tasks to complete before continuing. It doesn't matter what order they finish in, just that all of them have to complete for the gate to open and let the flow control through. In the Promise API, we call this pattern all([ .. ]).

```JavaScript
var p1 = request( "http://some.url.1/" );
var p2 = request( "http://some.url.2/" );
Promise.all( [p1,p2] ).then( function(msgs){    
  // both `p1` and `p2` fulfill and pass in    
  // their messages here
  return request(        
    "http://some.url.3/?v=" + msgs.join(",")    
  );}
 ).then(
   function(msg){    
     console.log( msg );
   }
 );
 ```

Remember to always attach a rejection/error handler to every promise, even and especially the one that comes back from Promise.all([ .. ]).

```JavaScript
Promise.race([ .. ])
```

sometimes you only want to respond to the "first Promise to cross the finish line," letting the other Promises fall away. This pattern is classically called a "latch," but in Promises it's called a "race."

```JavaScript
// `foo()` is a Promise-aware function
// `timeoutPromise(..)`, defined ealier, returns
// a Promise that rejects after a specified delay
// setup a timeout for `foo()`
Promise.race( [    
  foo(),                    
  // attempt `foo()`    
  timeoutPromise( 3000 )    
  // give it 3 seconds
] ).then(    
  function(){        
    // `foo(..)` fulfilled in time!    
  },    
  function(err){        
    // either `foo()` rejected, or it just        
    // didn't finish in time, so inspect        
    // `err` to know which    
  });
```

"Finally"

Concurrent Iterations

```JavaScript
// polyfill-safe guard check
if (!Promise.wrap) {    
  Promise.wrap = function(fn) {        
    return function() {            
      var args = [].slice.call( arguments );            
      return new Promise( function(resolve,reject){                
        fn.apply(                    
          null,                    
          args.concat( function(err,v){                        
            if (err) {                            
              reject( err );                        
            }                        
            else {                            
              resolve( v );                        
            }                    
          } )                
        );            
      } );        
    };    
  };
}
```
OK,

It takes a function that expects an error-first style callback as its last parameter, and returns a new one that automatically creates a Promise to return, and substitutes the callback for you, wired up to the Promise fulfillment/rejection.

let's just look at how we use it: var request = Promise.wrap( ajax );request( "http://some.url.1/" ).then( .. ).. Wow, that was pretty easy!

Promise.wrap(..) does not produce a Promise. It produces a function that will produce Promises. In a sense, a Promise-producing function could be seen as a "Promise factory." I propose "promisory" as the name for such a thing ("Promise" + "factory"). The act of wrapping a callback-expecting function to be a

Note: Promisory isn't a made-up term. It's a real word, and its definition means to contain or convey a promise. That's exactly what these functions are doing, so it turns out to be a pretty perfect terminology match!

Consider a callback-based scenario like the following:
```JavaScript
function foo(x,y,cb) {    
  ajax(        
    "http://some.url.1/?x=" + x + "&y=" + y,        
    cb    
  );
}

foo( 11, 31, function(err,text) {    
  if (err) {        
    console.error( err );    
  }    
  else {        
    console.log( text );    
  }
} );
```

So back to our earlier example, we need a promisory for both ajax(..) and foo(..):

```JavaScript
// make a promisory for `ajax(..)`
var request = Promise.wrap( ajax );
// refactor `foo(..)`, but keep it externally
// callback-based for compatibility with other
// parts of the code for now -- only use
// `request(..)`'s promise internally.
function foo(x,y,cb) {    
  request(        
    "http://some.url.1/?x=" + x + "&y=" + y    
  )    
  .then(        
    function fulfilled(text){            
      cb( null, text );        
    },        
    cb    
  );
}
// now, for this code's purposes, make a
// promisory for `foo(..)`
var betterFoo = Promise.wrap( foo );
// and use the promisorybetter
Foo( 11, 31 ).then(    
  function fulfilled(text){        
    console.log( text );    
  },    
  function rejected(err){        
    console.error( err );    
  });
```

Of course, while we're refactoring foo(..) to use our new request(..) promisory, we could just make foo(..) a promisory itself, instead of remaining callback-based and needing to make and use the subsequent betterFoo(..) promisory. This decision just depends on whether foo(..) needs to stay callback-based compatible with other parts of the code base or not. Consider:
```JavaScript
//`foo(..)` is now also a promisory because itdelegates to the `request(..)` promisory
function foo(x,y) {    
  return request(        
    "http://some.url.1/?x=" + x + "&y=" + y    
  );
}

foo( 11, 31 ).then( .. )..
```

While ES6 Promises don't natively ship with helpers for such promisory wrapping,

the embodiment of the "action at a distance" anti-pattern (http://en.wikipedia.org/wiki/Action_at_a_distance_%28computer_programming%29).

Note: Many Promise abstraction libraries provide facilities to cancel Promises, but this is a terrible idea!

the problem is that it would let one consumer/observer of a Promise affect some other consumer's ability to observe that same Promise. This violates the future-value's trustability (external immutability), but morever is the embodiment of the "action at a distance" anti-pattern (http://en.wikipedia.org/wiki/Action_at_a_distance_%28computer_programming%29).

A single Promise is not really a flow-control mechanism (at least not in a very meaningful sense), which is exactly what cancelation refers to; that's why Promise cancelation would feel awkward.

By contrast, a chain of Promises taken collectively together -- what I like to call a "sequence" -- is a flow control expression, and thus it's appropriate for cancelation to be defined at that level of abstraction.

No individual Promise should be cancelable, but it's sensible for a sequence to be cancelable, because you don't pass around a sequence as a single immutable value like you do with a Promise.

Promise Performance

More work to do, more guards to protect, means that Promises are slower as compared to naked, untrustable callbacks.

Review

Promises are awesome. Use them. They solve the inversion of control issues that plague us with callbacks-only code.

They don't get rid of callbacks, they just redirect the orchestration of those callbacks to a trustable intermediary mechanism that sits between us and another utility.

Chapter 04: Generators

Breaking Run-to-Completion

an expectation that JS developers almost universally rely on in their code: once a function starts executing, it runs until it completes, and no other code can interrupt and run in between.

ES6 introduces a new type of function that does not behave with the run-to-completion behavior. This new type of function is called a "generator."

To understand the implications, let's consider this example:
```JavaScript
var x = 1;function foo() {    
  x++;    
  bar(); // <-- what about this line?    
  console.log( "x:", x );
}

function bar() {    
  x++;
}

foo();  
```

In this example, we know for sure that bar() runs in between x++ and console.log(x). But what if bar() wasn't there? Obviously the result would be 2 instead of 3.

Now let's twist your brain. What if bar() wasn't present, but it could still somehow run between the x++ and console.log(x) statements? How would that be possible?

In preemptive multithreaded languages, it would essentially be possible for bar() to "interrupt" and run at exactly the right moment between those two statements. But JS is not preemptive, nor is it (currently) multithreaded. And yet, a cooperative form of this "interruption" (concurrency) is possible, if foo() itself could somehow indicate a "pause" at that part in the code.

Here's the ES6 code to accomplish such cooperative concurrency:
```JavaScript
var x = 1;
function *foo() {    
  x++;    
  yield; // pause!    
  console.log( "x:", x );
}

function bar() {    
  x++;
}
```

Now, how can we run the code in that previous snippet such that bar() executes at the point of the yield inside of `*foo()`?

```JavaScript
// construct an iterator `it` to control the generator
var it = foo(); // start `foo()` here!
it.next();
x; // 2
bar();
x; // 3
it.next(); // x: 3
```

let's walk through the behavior flow: The it = foo() operation does not execute the `*foo()` generator yet, but it merely constructs an iterator that will control its execution. More on iterators in a bit. The first it.next() starts the `*foo()` generator, and runs the x++ on the first line of `*foo()`. `*foo()` pauses at the yield statement, at which point that first it.next() call finishes. At the moment, `*foo()` is still running and active, but it's in a paused state. We inspect the value of x, and it's now 2. We call bar(), which increments x again with x++. We inspect the value of x again, and it's now 3. The final it.next() call resumes the `*foo()` generator from where it was paused, and runs the console.log(..) statement, which uses the current value of x of 3.

So, a generator is a special kind of function that can start and stop one or more times, and doesn't necessarily ever have to finish.

Consider:
```JavaScript
function *foo(x) {    
  var y = x * (yield);    
  return y;
}
var it = foo( 6 );
// start `foo(..)`
it.next();
var res = it.next( 7 );
res.value;        
// 42
```

First, we pass in 6 as the parameter x. Then we call it.next(), and it starts up `*foo(..)`. Inside `*foo(..)`, the var y = x .. statement starts to be processed, but then it runs across a yield expression. At that point, it pauses `*foo(..)` (in the middle of the assignment statement!), and essentially requests the calling code to provide a result value for the yield expression. Next, we call it.next( 7 ), which is passing the 7 value back in to be that result of the paused yield expression. So, at this point, the assignment statement is essentially var y = 6 * 7. Now, return y returns that 42 value back as the result of the it.next( 7 ) call.

###Tale of Two Questions

Consider only the generator code:
```JavaScript
var y = x * (yield);
return y;
```

This first yield is basically asking a question: "What value should I insert here?" Who's going to answer that question? Well, the first next() has already run to get the generator up to this point, so obviously it can't answer the question. So, the second next(..) call must answer the question posed by the first yield.

But let's flip our perspective. Let's look at it not from the generator's point of view, but from the iterator's point of view.

To properly illustrate this perspective, we also need to explain that messages can go in both directions -- yield .. as an expression can send out messages in response to next(..) calls, and next(..) can send values to a paused yield expression. Consider this slightly adjusted code:
```JavaScript
function *foo(x) {    
  var y = x * (yield "Hello");    // <-- yield a value!    
  return y;
}
var it = foo( 6 );
var res = it.next();    // first `next()`, don't pass anything
res.value;  // "Hello"
res = it.next( 7 ); // pass `7` to waiting `yield`
res.value; // 42
```

yield .. and next(..) pair together as a two-way message passing system during the execution of the generator.

The first next() call (with nothing passed to it) is basically asking a question: "What next value does the `*foo(..)` generator have to give me?" And who answers this question? The first yield "hello" expression.

Producers and Iterators Imagine you're producing a series of values where each value has a definable relationship to the previous value. To do this, you're going to need a stateful producer that remembers the last value it gave out. You can implement something like that straightforwardly using a function closure (see the Scope & Closures title of this series):
```JavaScript
var gimmeSomething = (function(){    
  var nextVal;    
  return function(){        
    if (nextVal === undefined) {            
      nextVal = 1;        
    }        
    else {            
      nextVal = (3 * nextVal) + 6;        
    }        
    return nextVal;    
  };
})();

gimmeSomething();        // 1
gimmeSomething();        // 9
gimmeSomething();        // 33
gimmeSomething();        // 105
```

In fact, this task is a very common design pattern, usually solved by iterators. An iterator is a well-defined interface for stepping through a series of values from a producer. The JS interface for iterators, as it is in most languages, is to call next() each time you want the next value from the producer.

We could implement the standard iterator interface for our number series producer:
```JavaScript
var something = (function(){    
  var nextVal;    
  return {        // needed for `for..of` loops        
    [Symbol.iterator]: function(){
      return this;
    },        
    // standard iterator interface method        
    next: function(){            
      if (nextVal === undefined) {                
        nextVal = 1;            
      }            
      else {                
        nextVal = (3 * nextVal) + 6;            
      }            
      return {
        done:false,
        value:nextVal
      };        
    }    
  };
})();

something.next().value;        // 1
something.next().value;        // 9
something.next().value;        // 33
something.next().value;        // 105
```
Note: We'll explain why we need the [Symbol.iterator]: .. part of this code snippet in the "Iterables" section. Syntactically though, two ES6 features are at play. First, the [ .. ] syntax is called a computed property name (see the this & Object Prototypes title of this series). It's a way in an object literal definition to specify an expression and use the result of that expression as the name for the property. Next, Symbol.iterator is one of ES6's predefined special Symbol values (see the ES6 & Beyond title of this book series).

The next() call returns an object with two properties: done is a boolean value signaling the iterator's complete status; value holds the iteration value.

ES6 also adds the for..of loop, which means that a standard iterator can automatically be consumed with native loop syntax:
```JavaScript
for (var v of something) {    
  console.log( v );    
  // don't let the loop run forever!    
  if (v > 500) {        
    break;    
  }
}// 1 9 33 105 321 969
```

The for..of loop automatically calls next() for each iteration -- it doesn't pass any values in to the next() -- and it will automatically terminate on receiving a done:true. It's quite handy for looping over a set of data.

In addition to making your own iterators, many built-in data structures in JS (as of ES6), like arrays, also have default iterators:
```JavaScript
var a = [1,3,5,7,9];
for (var v of a) {    
  console.log( v );
}// 1 3 5 7 9
```

###Iterables

The something object in our running example is called an iterator, as it has the next() method on its interface. But a closely related term is iterable, which is an object that contains an iterator that can iterate over its values.

As of ES6, the way to retrieve an iterator from an iterable is that the iterable must have a function on it, with the name being the special ES6 symbol value Symbol.iterator. When this function is called, it returns an iterator. Though not required, generally each call should return a fresh new iterator.

a in the previous snippet is an iterable. The for..of loop automatically calls its Symbol.iterator function to construct an iterator. But we could of course call the function manually, and use the iterator it returns:
```JavaScript
var a = [1,3,5,7,9];
var it = a[Symbol.iterator]();
it.next().value;    // 1
it.next().value;    // 3
it.next().value;    // 5..
```

In the previous code listing that defined something, you may have noticed this line: [Symbol.iterator]: function(){ return this; } That little bit of confusing code is making the something value -- the interface of the something iterator -- also an iterable; it's now both an iterable and an iterator. Then, we pass something to the for..of loop: for (var v of something) {..} The for..of loop expects something to be an iterable, so it looks for and calls its Symbol.iterator function. We defined that function to simply return this, so it just gives itself back, and the for..of loop is none the wiser.

### Generator Iterator

A generator can be treated as a producer of values that we extract one at a time through an iterator interface's next() calls.

So, a generator itself is not technically an iterable, though it's very similar -- when you execute the generator, you get an iterator back: function 
```JavaScript
*foo(){
  ..
}
var it = foo();
```

We can implement the something infinite number series producer from earlier with a generator, like this:
```JavaScript
 function *something() {    
   var nextVal;    
   while (true) {        
     if (nextVal === undefined) {            
       nextVal = 1;        
     }        
     else {            
       nextVal = (3 * nextVal) + 6;        
     }        
     yield nextVal;    
   }
 }
 ```

However, in a generator, such a loop is generally totally OK if it has a yield in it, as the generator will pause at each iteration, yielding back to the main program and/or to the event loop queue. To put it glibly, "generators put the while..true back in JS programming!"

That's a fair bit cleaner and simpler, right? Because the generator pauses at each yield, the state (scope) of the `function *something()` is kept around, meaning there's no need for the closure boilerplate to preserve variable state across calls.

And now we can use our shiny new `*something()` generator with a for..of loop, and you'll see it works basically identically:
```JavaScript
for (var v of something()) {    
  console.log( v );    // don't let the loop run forever!    
  if (v > 500) {        
    break;    
  }
}// 1 9 33 105 321 969
```

But don't skip over for (var v of something()) ..! We didn't just reference something as a value like in earlier examples, but instead called the `*something()` generator to get its iterator for the for..of loop to use.

Why couldn't we say for (var v of something) ..? Because something here is a generator, which is not an iterable. We have to call something() to construct a producer for the for..of loop to iterate over. The something() call produces an iterator, but the for..of loop wants an iterable, right? Yep. The generator's iterator also has a Symbol.iterator function on it, which basically does a return this, just like the something iterable we defined earlier. In other words, a generator's iterator is also an iterable!

But there's a hidden behavior that takes care of that for you. "Abnormal completion" (i.e., "early termination") of the for..of loop -- generally caused by a break, return, or an uncaught exception -- sends a signal to the generator's iterator for it to terminate.

If you specify a try..finally clause inside the generator, it will always be run even when the generator is externally completed. This is useful if you need to clean up resources (database connections, etc.):
```JavaScript
function *something() {    
  try {        
    var nextVal;        
    while (true) {            
      if (nextVal === undefined) {                
        nextVal = 1;            
      }            
      else {                
        nextVal = (3 * nextVal) + 6;            
      }            
      yield nextVal;        
    }    
  }    // cleanup clause    
  finally {        
    console.log( "cleaning up!" );    
  }
}
```

you could instead manually terminate the generator's iterator instance from the outside with return(..):
```JavaScript
var it = something();
for (var v of it) {    
  console.log( v );    
  // don't let the loop run forever!    
  if (v > 500) {        
    console.log(            
      // complete the generator's iterator            
      it.return( "Hello World" ).value        
    );        
    // no `break` needed here    
  }
}
// 1 9 33 105 321 969
// cleaning up!
// Hello World
```
When we call it.return(..), it immediately terminates the generator, which of course runs the finally clause.

### Iterating Generators Asynchronously

3. Let's recall the callback approach:
```JavaScript
function foo(x,y,cb) {    
  ajax(        
    "http://some.url.1/?x=" + x + "&y=" + y,        
    cb    
  );
}

foo( 11, 31, function(err,text) {    
  if (err) {        
    console.error( err );    
  }    
  else {        
    console.log( text );    
  }
}
);
```

If we wanted to express this same task flow control with a generator, we could do:
```JavaScript
function foo(x,y) {    
  ajax(        
    "http://some.url.1/?x=" + x + "&y=" + y,        
    function(err,data){            
      if (err) {                
        // throw an error into `*main()`                
        it.throw( err );            
      }            
      else {                
        // resume `*main()` with received `data`                
        it.next( data );            
      }        
    }    
  );
}

function *main() {    
  try {        
    var text = yield foo( 11, 31 );        
    console.log( text );    
  }    

  catch (err) {        
    console.error( err );    
  }
}

var it = main();// start it all up!

it.next();
```

First, let's look at this part of the code, which is the most important:
```JavaScript
var text = yield foo( 11, 31 );
console.log( text );
```

If you recall
```JavaScript
var data = ajax( "..url 1.." );
console.log( data );
```
And that code didn't work!

Can you spot the difference? It's the yield used in a generator.

That's the magic! That's what allows us to have what appears to be blocking, synchronous code, but it doesn't actually block the whole program; it only pauses/blocks the code in the generator itself.

In yield foo(11,31), first the foo(11,31) call is made, which returns nothing (aka undefined), so we're making a call to request data, but we're actually then doing yield undefined. That's OK, because the code is not currently relying on a yielded value to do anything interesting.

So, the generator pauses at the yield, essentially asking the question, "what value should I return to assign to the variable text?" Who's going to answer that question?

Look at foo(..). If the Ajax request is successful, we call: it.next( data );

That's resuming the generator with the response data, which means that our paused yield expression receives that value directly, and then as it restarts the generator code, that value gets assigned to the local variable text.

Take a step back and consider the implications. We have totally synchronous-looking code inside the generator (other than the yield keyword itself), but hidden behind the scenes, inside of foo(..), the operations can complete asynchronously. That's huge!

In essence, we are abstracting the asynchrony away as an implementation detail, so that we can reason synchronously/sequentially about our flow control: "Make an Ajax request, and when it finishes print out the response."

### Synchronous Error Handling

```JavaScript
try {    
  var text = yield foo( 11, 31 );    
  console.log( text );
}
catch (err) {    
  console.error( err );
}
```

The awesome part is that this yield pausing also allows the generator to catch an error. We throw that error into the generator with this part of the earlier code listing:
```JavaScript
if (err) {    
  // throw an error into `*main()`    
  it.throw( err );
}
```

The yield-pause nature of generators means that not only do we get synchronous-looking return values from async function calls, but we can also synchronously catch errors from those async function calls!

### Generators + Promises

we lost something very important: the trustability and composability of Promises

The best of all worlds in ES6 is to combine generators (synchronous-looking async code) with Promises (trustable and composable).

```JavaScript
function foo(x,y) {    
  return request(        
    "http://some.url.1/?x=" + x + "&y=" + y    
  );
}

foo( 11, 31 ).then(    
  function(text){        
    console.log( text );    
  },    
  function(err){        
    console.error( err );    
  }
);
```

In our earlier generator code for the running Ajax example, foo(..) returned nothing (undefined), and our iterator control code didn't care about that yielded value.

But here the Promise-aware foo(..) returns a promise after making the Ajax call. That suggests that we could construct a promise with foo(..) and then yield it from the generator, and then the iterator control code would receive that promise.

But what should the iterator do with the promise? It should listen for the promise to resolve (fulfillment or rejection), and then either resume the generator with the fulfillment message or throw an error into the generator with the rejection reason.

it's so important.

The natural way to get the most out of Promises and generators is to yield a Promise, and wire that Promise to control the generator's iterator.

```JavaScript
function foo(x,y) {    
  return request(        
    "http://some.url.1/?x=" + x + "&y=" + y    
  );
}
function *main() {    
  try {        
    var text = yield foo( 11, 31 );        
    console.log( text );    
  }    
  catch (err) {        
    console.error( err );    
  }
}
```

The most powerful revelation in this refactor is that the code inside `*main()` did not have to change at all! Inside the generator, whatever values are yielded out is just an opaque implementation detail,

But how are we going to run `*main()` now?
```JavaScript
var it = main();
var p = it.next().value;// wait for the `p` promise to resolve
p.then(    
  function(text){        
    it.next( text );    
  },    
  function(err){        
    it.throw( err );    
  }
);
```

### Promise-Aware Generator Runner

This is such an important pattern, and you don't want to get it wrong (or exhaust yourself repeating it over and over), so your best bet is to use a utility that is specifically designed to run Promise-yielding generators in the manner we've illustrated.

What if we wanted to be able to Promise-drive a generator no matter how many steps it has?
=
let's just define our own standalone utility that we'll call run(..): // thanks to Benjamin Gruenbaum (@benjamingr on GitHub) for// big improvements here!

```JavaScript
function run(gen) {    
  var args = [].slice.call( arguments, 1),
  it;    // initialize the generator in the current context    
  it = gen.apply( this, args );    
  // return a promise for the generator completing    
  return Promise.resolve()        
  .then( function handleNext(value){            
    // run to the next yielded value            
    var next = it.next( value );            
    return (function handleResult(next){                
      // generator has completed running?                
      if (next.done) {                    
        return next.value;                
      }                
      // otherwise keep going                
      else {                    
        return Promise.resolve( next.value )                        
        .then(                            
          // resume the async loop on                            
          // success, sending the resolved                            
          // value back into the generator                            
          handleNext,                            
          // if `value` is a rejected                            
          // promise, propagate error back                            
          // into the generator for its own                            
          // error handling                            
          function handleErr(err) {                                
            return Promise.resolve(                                    
              it.throw( err )                                
            )                                
            .then( handleResult );                            
          }                        
        );                
      }            
    })(next);        
  });
}
```

So, a utility/library helper is definitely the way to go.

How would you use run(..) with `*main()` in our running Ajax example?
```JavaScript
function *main() {    
  // ..
}

run( main );
```
That's it!

The way we wired run(..), it will automatically advance the generator you pass to it, asynchronously until completion.

Imagine a scenario where you need to fetch data from two different sources, then combine those responses to make a third request, and finally print out the last response.

```JavaScript
function *foo() {    
  var r1 = yield request( "http://some.url.1" );    
  var r2 = yield request( "http://some.url.2" );    
  var r3 = yield request(        
    "http://some.url.3/?v=" + r1 + "," + r2    
  );    

  console.log( r3 );
}
// use previously defined `run(..)` utility
run( foo );
```

This code will work, but in the specifics of our scenario, it's not optimal.

Can you spot why? Because the r1 and r2 requests can -- and for performance reasons, should -- run concurrently, but in this code they will run sequentially; the "http://some.url.2"

But how exactly would you do that with a generator and yield? We know that yield is only a single pause point in the code, so you can't really do two pauses at the same time. The most natural and effective answer is to

But how exactly would you do that with a generator and yield? We know that yield is only a single pause point in the code, so you can't really do two pauses at the same time. The most natural and effective answer is to base the async flow on Promises, specifically on their capability to manage state in a time-independent fashion (see "Future Value" in Chapter 3).

```JavaScript
function *foo() {    
  // make both requests "in parallel"    
  var p1 = request( "http://some.url.1" );    
  var p2 = request( "http://some.url.2" );    
  // wait until both promises resolve    
  var r1 = yield p1;    
  var r2 = yield p2;    
  var r3 = yield request(        
    "http://some.url.3/?v=" + r1 + "," + r2    
  );    

  console.log( r3 );
}
// use previously defined `run(..)` utility
run( foo );
```

It doesn't matter which one finishes first, because promises will hold onto their resolved state for as long as necessary.

Either way, both p1 and p2 will run concurrently, and both have to finish, in either order, before the r3 = yield request.. Ajax request will be made.

If that flow control processing model sounds familiar, it's basically the same as what we identified in Chapter 3 as the "gate" pattern, enabled by the Promise.all([ .. ]) utility. So, we could also express the flow control like this:

```JavaScript
function *foo() {    
  // make both requests "in parallel," and    
  // wait until both promises resolve    
  var results = yield Promise.all( [        
    request( "http://some.url.1" ),        
    request( "http://some.url.2" )    
  ] );    

  var r1 = results[0];    
  var r2 = results[1];    
  var r3 = yield request(        
    "http://some.url.3/?v=" + r1 + "," + r2    
  );    

  console.log( r3 );
}
// use previously defined `run(..)` utility
run( foo );
```

In other words, all of the concurrency capabilities of Promises are available to us in the generator+Promise approach. So in any place where you need more than sequential this-then-that async flow control steps, Promises are likely your best bet.

#### Promises, Hidden

As a word of stylistic caution, be careful about how much Promise logic you include inside your generators.

this might be a cleaner approach:
```JavaScript
// note: normal function, not generator
function bar(url1,url2) {    
  return Promise.all( [        
    request( url1 ),        
    request( url2 )    
  ] );
}
function *foo() {    
  // hide the Promise-based concurrency details    
  // inside `bar(..)`    
  var results = yield bar(        
    "http://some.url.1",        
    "http://some.url.2"    
  );    
  var r1 = results[0];    
  var r2 = results[1];    
  var r3 = yield request(        
    "http://some.url.3/?v=" + r1 + "," + r2    
  );    

  console.log( r3 );
}// use previously defined `run(..)` utility
run( foo );
```

Inside `*foo()`, it's cleaner and clearer that all we're doing is just asking bar(..) to get us some results, and we'll yield-wait on that to happen. We don't have to care that under the covers a Promise.all([ .. ]) Promise composition will be used to make that happen.

We treat asynchrony, and indeed Promises, as an implementation detail.

Hiding your Promise logic inside a function that you merely call from your generator is especially useful if you're going to do a sophisticated series flow-control. For example:
```JavaScript
function bar() {    
  Promise.all( [        
    baz( .. )        
    .then( .. ),        
    Promise.race( [ .. ] )    
  ] ).then( .. )
}
```

That kind of logic is sometimes required, and if you dump it directly inside your generator(s), you've defeated most of the reason why you would want to use generators in the first place. We should intentionally abstract such details away from our generator code so that they don't clutter up the higher level task expression.

Beyond creating code that is both functional and performant, you should also strive to make code that is as reason-able and maintainable as possible.

#### Generator Delegation

It may then occur to you that you might try to call one generator from another generator, using our run(..) helper, such as:
```JavaScript
function *foo() {    
  var r2 = yield request( "http://some.url.2" );    
  var r3 = yield request( "http://some.url.3/?v=" + r2 );    
  return r3;
}

function *bar() {    
  var r1 = yield request( "http://some.url.1" );    
  // "delegating" to `*foo()` via `run(..)`    
  var r3 = yield run( foo );    
  console.log( r3 );
}
run( bar );
```

We run `*foo()` inside of `*bar()` by using our run(..) utility again.

But there's an even better way to integrate calling `*foo()` into `*bar()`, and it's called yield-delegation.

The special syntax for yield-delegation is: yield * __

```JavaScript
function *foo() {    
  console.log( "`*foo()` starting" );    
  yield 3;    
  yield 4;    
  console.log( "`*foo()` finished" );
}

function *bar() {    
  yield 1;    
  yield 2;    
  yield *foo();    
  // `yield`-delegation!    
  yield 5;
}

var it = bar();
it.next().value;    // 1
it.next().value;    // 2
it.next().value;    // `*foo()` starting                    
// 3
it.next().value;    // 4
it.next().value;    // `*foo()` finished 
// 5
```

First, calling foo() creates an iterator exactly as we've already seen. Then, yield * delegates/transfers the iterator instance control (of the present `*bar()` generator) over to this other `*foo()` iterator.

So now back to the previous example with the three sequential Ajax requests:
```JavaScript
function *foo() {    
  var r2 = yield request( "http://some.url.2" );    
  var r3 = yield request( "http://some.url.3/?v=" + r2 );    
  return r3;
}
function *bar() {    
  var r1 = yield request( "http://some.url.1" );    
  // "delegating" to `*foo()` via `yield*`    
  var r3 = yield *foo();    
  console.log( r3 );
}
run( bar );
```

The only difference between this snippet and the version used earlier is the use of yield `*foo()` instead of the previous yield run(foo).

### Why Delegation?

The purpose of yield-delegation is mostly code organization, and in that way is symmetrical with normal function calling.

For all these exact same reasons, keeping generators separate aids in program readability, maintenance, and debuggability. In that respect, yield * is a syntactic shortcut for manually iterating over the steps of `*foo()` while inside of `*bar()`.

#### Delegating Asynchrony

#### Delegating "Recursion"

```JavaScript
function *foo(val) {    
  if (val > 1) {        
    // generator recursion        
    val = yield *foo( val - 1 );    
  }    
  return yield request( "http://some.url/?v=" + val );
}

function *bar() {    
  var r1 = yield *foo( 3 );    
  console.log( r1 );
}

run( bar );
```

Hang on, this is going to be quite intricate to describe in detail: run(bar) starts up the `*bar()` generator. foo(3) creates an iterator for `*foo(..)` and passes 3 as its val parameter. Because 3 > 1, foo(2) creates another iterator and passes in 2 as its val parameter. Because 2 > 1, foo(1) creates yet another iterator and passes in 1 as its val parameter. 1 > 1 is false, so we next call request(..) with the 1 value, and get a promise back for that first Ajax call. That promise is yielded out, which comes back to the `*foo(2)` generator instance. The yield * passes that promise back out to the `*foo(3)` generator instance. Another yield * passes the promise out to the `*bar()` generator instance. And yet again another yield * passes the promise out to the run(..) utility, which will wait on that promise (for the first Ajax request) to proceed. When the promise resolves, its fulfillment message is sent to resume `*bar()`, which passes through the yield * into the `*foo(3)` instance, which then passes through the yield * to the `*foo(2)` generator instance, which then passes through the yield * to the normal yield that's waiting in the `*foo(3)` generator instance. That first call's Ajax response is now immediately returned from the `*foo(3)` generator instance, which sends that value back as the result of the yield * expression in the `*foo(2)` instance, and assigned to its local val variable. Inside `*foo(2)`, a second Ajax request is made with request(..), whose promise is yielded back to the `*foo(1)` instance, and then yield * propagates all the way out to run(..) (step 7 again). When the promise resolves, the second Ajax response propagates all the way back into the `*foo(2)` generator instance, and is assigned to its local val variable. Finally, the third Ajax request is made with request(..), its promise goes out to run(..), and then its resolution value comes all the way back, which is then returned so that it comes back to the waiting yield * expression in `*bar()`.

#### Generator Concurrency
two different simultaneous Ajax response handlers needed to coordinate with each other to make sure that the data communication was not a race condition.

We slotted the responses into the res array like this:
```JavaScript
function response(data) {    
  if (data.url == "http://some.url.1") {        
    res[0] = data;    
  }    
  else if (data.url == "http://some.url.2") {        
    res[1] = data;    
  }
}
```

 But how can we use multiple generators concurrently for this scenario?
 ```JavaScript
 // `request(..)` is a Promise-aware Ajax utility
 var res = [];
 function *reqData(url) {    
   res.push(        
     yield request( url )    
   );
 }
 ```

Instead of having to manually sort out res[0] and res[1] assignments, we'll use coordinated ordering so that res.push(..) properly slots the values in the expected and predictable order. The expressed logic thus should feel a bit cleaner.

But how will we actually orchestrate this interaction? First, let's just do it manually, with Promises:
```JavaScript
var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );
var p1 = it1.next();
var p2 = it2.next();
p1.then(
  function(data){    
    it1.next( data );    
    return p2;
  } )
  .then(
    function(data){    
      it2.next( data );
    } );
    ```


`*reqData(..)`'s two instances are both started to make their Ajax requests, then paused with yield. Then we choose to resume the first instance when p1 resolves, and then p2's resolution will restart the second instance. In this way, we use Promise orchestration to ensure that res[0] will have the first response and res[1] will have the second response.

this is awfully manual,

Let's try it a different way:
```JavaScript
// `request(..)` is a Promise-aware Ajax utility
var res = [];
function *reqData(url) {    
  var data = yield request( url );    
  // transfer control    
  yield;    
  res.push( data );
}

var it1 = reqData( "http://some.url.1" );
var it2 = reqData( "http://some.url.2" );
var p1 = it.next();
var p2 = it.next();
p1.then( function(data){    
  it1.next( data );
} );
p2.then( function(data){    
  it2.next( data );
} );

Promise.all( [p1,p2] )
.then( function(){    
  it1.next();    
  it2.next();
} );
```
OK, this is a bit better

two instances of `*reqData(..)` run truly concurrently,

In the previous snippet, the second instance was not given its data until after the first instance was totally finished. But here, both instances receive their data as soon as their respective responses come back, and then each instance does another yield for control transfer purposes. We then choose what order to resume them in the Promise.all([ .. ]) handler.

this approach hints at an easier form for a reusable utility, because of the symmetry.

Let's imagine using a utility called runAll(..):
```JavaScript
// `request(..)` is a Promise-aware Ajax utility
var res = [];
runAll(    
  function*(){        
    var p1 = request( "http://some.url.1" );        
    // transfer control        
    yield;        
    res.push( yield p1 );    
  },    
  function*(){        
    var p2 = request( "http://some.url.2" );        
    // transfer control        
    yield;        
    res.push( yield p2 );    
  });
);
```

Note: We're not including a code listing for runAll(..) as it is not only long enough to bog down the text, but is an extension of the logic we've already implemented in run(..) earlier. So, as a good supplementary exercise for the reader, try your hand at evolving the code from run(..) to work like the imagined runAll(..). Also, my asynquence library provides a previously mentioned runner(..) utility with this kind of capability already built in, and will be discussed in Appendix A of this book. Here's how the processing inside runAll(..) would operate: The first generator gets a promise for the first Ajax response from "http://some.url.1", then yields control back to the runAll(..) utility. The second generator runs and does the same for "http://some.url.2", yielding control back to the runAll(..) utility. The first generator resumes, and then yields out its promise p1. The runAll(..) utility does the same in this case as our previous run(..), in that it waits on that promise to resolve, then resumes the same generator (no control transfer!). When p1 resolves, runAll(..) resumes the first generator again with that resolution value, and then res[0] is given its value. When the first generator then finishes, that's an implicit transfer of control. The second generator resumes, yields out its promise p2, and waits for it to resolve. Once it does, runAll(..) resumes the second generator with that value, and res[1] is set.

In this running example, we use an outer variable called res to store the results of the two different Ajax responses -- that's our concurrency coordination making that possible.

But it might be quite helpful to further extend runAll(..) to provide an inner variable space for the multiple generator instances to share, such as an empty object we'll call data below. Also, it could take non-Promise values that are yielded and hand them off to the next generator. Consider:
```JavaScript
// `request(..)` is a Promise-aware Ajax utility
runAll(    
  function*(data){        
    data.res = [];        
    // transfer control (and message pass)        
    var url1 = yield "http://some.url.2";        
    var p1 = request( url1 ); // "http://some.url.1"        
    // transfer control        
    yield;        
    data.res.push( yield p1 );    
  },    
  function*(data){        
    // transfer control (and message pass)        
    var url2 = yield "http://some.url.1";        
    var p2 = request( url2 ); // "http://some.url.2"        
    // transfer control        
    yield;        
    data.res.push( yield p2 );    
  });
)
  ```
In this formulation, the two generators are not just coordinating control transfer, but actually communicating with each other, both through data.res and the yielded messages that trade url1 and url2 values. That's incredibly powerful!

Such realization also serves as a conceptual base for a more sophisticated asynchrony technique called CSP (Communicating Sequential Processes),

Thunks
=
So far, we've made the assumption that yielding a Promise from a generator -- and having that Promise resume the generator via a helper utility like run(..) -- was the best possible way to manage asynchrony with generators. To be clear, it is.

In general computer science, there's an old pre-JS concept called a "thunk."

a narrow expression of a thunk in JS is a function that -- without any parameters -- is wired to call another function.

In other words, you wrap a function definition around function call -- with any parameters it needs -- to defer the execution of that call, and that wrapping function is a thunk.

When you later execute the thunk, you end up calling the original function. For example:
```JavaScript
function foo(x,y) {    
  return x + y;
}

function fooThunk() {    
  return foo( 3, 4 );
}
// later
console.log( fooThunk() );    // 7
```

But what about an async thunk?

Consider:
```JavaScript
function foo(x,y,cb) {    
  setTimeout( function(){        
    cb( x + y );    
  }, 1000 );
}
function fooThunk(cb) {    
  foo( 3, 4, cb );
}
// later
fooThunk( function(sum){    
  console.log( sum );        // 7} );
});
```

As you can see, fooThunk(..) only expects a cb(..) parameter, as it already has values 3 and 4 (for x and y, respectively) pre-specified and ready to pass to foo(..). A thunk is just waiting around patiently for the last piece it needs to do its job: the callback.

let's invent a utility that does this wrapping for us. Consider:
```JavaScript
function thunkify(fn) {    
  var args = [].slice.call( arguments, 1 );    
  return function(cb) {        
    args.push( cb );        
    return fn.apply( null, args );    
  };
}

var fooThunk = thunkify( foo, 3, 4 );
// later
fooThunk( function(sum) {    
  console.log( sum );        // 7
} );
```

The preceding formulation of thunkify(..) takes both the foo(..) function reference, and any parameters it needs, and returns back the thunk itself (fooThunk(..)). However, that's not the typical approach you'll find to thunks in JS.

Instead of thunkify(..) making the thunk itself, typically -- if not perplexingly -- the thunkify(..) utility would produce a function that produces thunks.

Consider:
```JavaScript
function thunkify(fn) {    
  return function() {        
    var args = [].slice.call( arguments );        
    return function(cb) {            
      args.push( cb );            
      return fn.apply( null, args );        
    };    
  };
}
```

The main difference here is the extra return function() { .. } layer. Here's how its usage differs:
```JavaScript
var whatIsThis = thunkify( foo );
var fooThunk = whatIsThis( 3, 4 );
// later
fooThunk( function(sum) {    
  console.log( sum );        // 7
} );
```

Obviously, the big question this snippet implies is what is whatIsThis properly called? It's not the thunk, it's the thing that will produce thunks from foo(..) calls. It's kind of like a "factory" for "thunks." There doesn't seem to be any kind of standard agreement for naming such a thing.

So, my proposal is "thunkory" ("thunk" + "factory"). So, thunkify(..) produces a thunkory, and a thunkory produces thunks. That reasoning is symmetric to my proposal for "promisory" in Chapter 3:
```JavaScript
var fooThunkory = thunkify( foo );
var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );
// later
fooThunk1( function(sum) {    
  console.log( sum );        // 7
} );

fooThunk2( function(sum) {    
  console.log( sum );        // 11
} );
```

in general, it's quite useful to make thunkories at the beginning of your program to wrap existing API methods, and then be able to pass around and call those thunkories when you need thunks.

preserve a cleaner separation of capability.

```JavaScript
// cleaner:
var fooThunkory = thunkify( foo );
var fooThunk1 = fooThunkory( 3, 4 );
var fooThunk2 = fooThunkory( 5, 6 );
// instead of:
var fooThunk1 = thunkify( foo, 3, 4 );
var fooThunk2 = thunkify( foo, 5, 6 );
```

# Chapter 5: Program Performance

why asynchrony really matters to JS. The most obvious explicit reason is performance.

Web Workers

Imagine splitting your program into two pieces, and running one of those pieces on the main UI thread, and running the other piece on an entirely separate thread. What kinds of concerns would such an architecture bring up?

For one, you'd want to know if running on a separate thread meant that it ran in parallel (on systems with multiple CPUs/cores) such that a long-running process on that second thread would not block the main program thread. Otherwise, "virtual threading" wouldn't be of much benefit over what we already have in JS with async concurrency. And you'd want to know if these two pieces of the program have access to the same shared scope/resources. If they do, then you have all the questions that multithreaded languages (Java, C++, etc.) deal with, such as needing cooperative or preemptive locking (mutexes, etc.). That's a lot of extra work, and shouldn't be undertaken lightly.

Alternatively, you'd want to know how these two pieces could "communicate" if they couldn't share scope/resources.

we explore a feature added to the web platform circa HTML5 called "Web Workers."

is a feature of the browser (aka host environment) and actually has almost nothing to do with the JS language itself. That is, JavaScript does not currently have any features that support threaded execution.

But an environment like your browser can easily provide multiple instances of the JavaScript engine, each on its own thread, and let you run a different program in each thread. Each of those separate threaded pieces of your program is called a "(Web) Worker." This type of parallelism is called "task parallelism," as the emphasis is on splitting up chunks of your program to run in parallel.

```JavaScript
var w1 = new Worker( "http://some.url.1/mycoolworker.js" );
```

The URL should point to the location of a JS file

The browser will then spin up a separate thread and let that file run as an independent program in that thread.

The w1 Worker object is an event listener and trigger, which lets you subscribe to events sent by the Worker as well as send events to the Worker.
=
Here's how to listen for events (actually, the fixed "message" event):
```JavaScript
w1.addEventListener( "message", function(evt){    
  // evt.data
} );
```

And you can send the "message" event to the Worker:
```JavaScript
w1.postMessage( "something cool to say" );
```

Inside the Worker, the messaging is totally symmetrical:
```JavaScript
// "mycoolworker.js"
addEventListener( "message", function(evt){    
  // evt.data
} );
postMessage( "a really cool reply" );
```

If you have two or more pages (or multiple tabs with the same page!) in the browser that try to create a Worker from the same file URL, those will actually end up as completely separate Workers.

Worker Environment

you cannot access any of its global variables, nor can you access the page's DOM or other resources. Remember: it's a totally separate thread.

You can,

perform network operations (Ajax, WebSockets) and set timers.

the Worker has access to its own copy of several important global variables/features, including navigator, location, JSON, and applicationCache.

What are some common uses for Web Workers? Processing intensive math calculations Sorting large data sets Data operations (compression, audio analysis, image pixel manipulations, etc.) High-traffic network communications

Data Transfer

You may notice a common characteristic of most of those uses, which is that they require a large amount of information to be transferred across the barrier between threads using the event mechanism, perhaps in both directions.

In the early days of Workers, serializing all data to a string value was the only option.

the data was being copied,

doubling of memory

If you pass an object, a so-called "Structured Cloning Algorithm" (https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm) is used to copy/duplicate the object on the other side.

An even better option, especially for larger data sets, is "Transferable Objects" (http://updates.html5rocks.com/2011/12/Transferable-Objects-Lightning-Fast).

What happens is that the object's "ownership" is transferred, but the data itself is not moved.

any data structure that implements the Transferable interface (https://developer.mozilla.org/en-US/docs/Web/API/Transferable) will automatically be transferred this way (support Firefox & Chrome).

Shared Workers

If your site or app allows for loading multiple tabs of the same page (a common feature), you may very well want to reduce the resource usage of their system by preventing duplicate dedicated Workers; the most common limited resource in this respect is a socket network connection, as browsers limit the number of simultaneous connections to a single host. Of course, limiting multiple connections from a client also eases your server resource requirements. In this case, creating a single centralized Worker that all the page instances of your site or app can share is quite useful.

That's called a SharedWorker,

```JavaScript
var w1 = new SharedWorker( "http://some.url.1/mycoolworker.js" );
```

the Worker needs a way to know which program a message comes from. This unique identification is called a "port"

```JavaScript
w1.port.addEventListener( "message", handleMessages );
// ..w1.port.postMessage( "something cool" );
```

Also, the port connection must be initialized, as: w1.port.start();

Inside the shared Worker, an extra event must be handled: "connect". This event provides the port object for that particular connection. The most convenient way to keep multiple connections separate is to use closure (see Scope & Closures title of this series) over the port, as shown next, with the event listening and transmitting for that connection defined inside the handler for the "connect" event:

```JavaScript
// inside the shared Worker
addEventListener( "connect", function(evt){    
  // the assigned port for this connection    
  var port = evt.ports[0];    
  port.addEventListener( "message", function(evt){        
    // ..        
    port.postMessage( .. );        
    // ..    
  } );    
  // initialize the port connection    
  port.start();
} );
```

### Polyfilling Web Workers

As we detailed in Chapter 1, JS's asynchronicity (not parallelism) comes from the event loop queue, so you can force faked Workers to be asynchronous using timers (setTimeout(..), etc.). Then you just need to provide a polyfill for the Worker API. There are some listed here (https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#web-workers), but frankly none of them look great.

SIMD Single instruction, multiple data (SIMD) is a form of "data parallelism," as contrasted to "task parallelism" with Web Workers, because the emphasis is not really on program logic chunks being parallelized, but rather multiple bits of data being processed in parallel.

With SIMD, threads don't provide the parallelism. Instead, modern CPUs provide SIMD capability with "vectors" of numbers -- think: type specialized arrays -- as well as instructions that can operate in parallel across all the numbers; these are low-level operations leveraging instruction-level parallelism. The effort to expose SIMD capability to JavaScript is primarily spearheaded by Intel (https://01.org/node/1495), namely by Mohammad Haghighat (at the time of this writing), in cooperation with Firefox and Chrome teams. SIMD is on an early standards track with a good chance of making it into a future revision of JavaScript, likely in the ES7 timeframe.

SIMD JavaScript proposes to expose short vector types and APIs to JS code, which on those SIMD-enabled systems would map the operations directly through to the CPU equivalents, with fallback to non-parallelized operation "shims" on non-SIMD systems.

Early proposal forms of the SIMD API at the time of this writing look like this:
```JavaScript
var v1 = SIMD.float32x4( 3.14159, 21.0, 32.3, 55.55 );
var v2 = SIMD.float32x4( 2.1, 3.2, 4.3, 5.4 );
var v3 = SIMD.int32x4( 10, 101, 1001, 10001 );
var v4 = SIMD.int32x4( 10, 20, 30, 40 );
SIMD.float32x4.mul( v1, v2 );    // [ 6.597339, 67.2, 138.89, 299.97 ]
SIMD.int32x4.add( v3, v4 );    
```

### Review

The first four chapters of this book are based on the premise that async coding patterns give you the ability to write more performant code, which is generally a very important improvement. But async behavior only gets you so far, because it's still fundamentally bound to a single event loop thread. So in this chapter we've covered several program-level mechanisms for improving performance even further. Web Workers let you run a JS file (aka program) in a separate thread using async events to message between the threads. They're wonderful for offloading long-running or resource-intensive tasks to a different thread, leaving the main UI thread more resposive. SIMD proposes to map CPU-level parallel math operations to JavaScript APIs for high-performance data-parallel operations, like number processing on large data sets. Finally, asm.js describes a small subset of JavaScript that avoids the hard-to-optimize parts of JS (like garbage collection and coercion) and lets the JS engine recognize and run such code through aggressive optimizations. asm.js could be hand authored, but that's extremely tedious and error prone, akin to hand authoring assembly language (hence the name). Instead, the main intent is that asm.js would be a good target for cross-compilation from other highly optimized program languages -- for example, Emscripten (https://github.com/kripken/emscripten/wiki) transpiling C/C++ to JavaScript. While not covered explicitly in this chapter, there are even more radical ideas under very early discussion for JavaScript, including approximations of direct threaded functionality (not just hidden behind data structure APIs). Whether that happens explicitly, or we just see more parallelism creep into JS behind the scenes, the future of more optimized program-level performance in JS looks really promising.

## Chapter 6: Benchmarking & Tuning

### Benchmarking

JS developers, if asked to benchmark the speed (execution time) of a certain operation, would initially go about it something like this:
```JavaScript
var start = (new Date()).getTime();    
// or `Date.now()`
// do some operation
var end = (new Date()).getTime();
console.log( "Duration:", (end - start) );
```

Raise your hand if that's roughly what came to your mind. Yep, I thought so. There's a lot wrong with this approach, but don't feel bad; we've all been there.

Your "benchmark" is basically useless. And worse, it's dangerous in that it implies false confidence, not just

Repetition "OK," you now say, "Just put a loop around it so the whole test takes longer." If you repeat an operation 100 times, and that whole loop reportedly takes a total of 137ms, then you can just divide by 100 and get an average duration of 1.37ms for each operation, right? Well, not exactly.

### Benchmark.js

so I'll hand wave around some terms: standard deviation, variance, margin of error.

Luckily, smart folks like John-David Dalton and Mathias Bynens do understand these concepts, and wrote a statistically sound benchmarking tool called Benchmark.js (http://benchmarkjs.com/). So I can end the suspense by simply saying: "just use that tool."

But just for quick illustration purposes, here's how you could use Benchmark.js to run a quick performance test: ```JavaScript
function foo() {    
  // operation(s) to test}
  var bench = new Benchmark(    
    "foo test",                // test name    
    foo,                    // function to test (just contents)    
    {        
      // ..                
      // optional extra options (see docs)    
    }
  );

  bench.hz;                    // number of operations per second
  bench.stats.moe;            // margin of error
  bench.stats.variance;        // variance across samples
  // ..
  ```
One largely untapped potential use-case for Benchmark.js is to use it in your Dev or QA environments to run automated performance regression tests against critical path parts of your application's JavaScript. Similar to how you might run unit test suites before deployment, you can also compare the performance against previous benchmarks to monitor if you are improving or degrading application performance.

### Writing Good Tests

Some commonly cited examples (https://github.com/petkaantonov/bluebird/wiki/Optimization-killers) for v8:

Programmers waste enormous amounts of time thinking about, or worrying about, the speed of noncritical parts of their programs, and these attempts at efficiency actually have a strong negative impact when debugging and maintenance are considered. We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%.

### Tail Call Optimization (TCO)

ES6 includes a specific requirement that ventures into the world of performance. It's related to a specific form of optimization that can occur with function calls: tail call optimization.

a "tail call" is a function call that appears at the "tail" of another function, such that after the call finishes, there's nothing left to do (except perhaps return its result value).

For example, here's a non-recursive setup with tail calls:
```JavaScript
function foo(x) {    
  return x;
}

function bar(y) {    
  return foo( y + 1 );    // tail call
}

function baz() {    
  return 1 + bar( 40 );    // not tail call
}

baz();
```
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4604-4606 | Added on Friday, December 30, 2016 7:40:58 AM

Without getting into too much nitty-gritty detail, calling a new function requires an extra amount of reserved memory to manage the call stack, called a "stack frame." So the preceding snippet would generally require a stack frame for each of baz(), bar(..), and foo(..) all at the same time.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4606-4609 | Added on Friday, December 30, 2016 7:41:18 AM

However, if a TCO-capable engine can realize that the foo(y+1) call is in tail position meaning bar(..) is basically complete, then when calling foo(..), it doesn't need to create a new stack frame, but can instead reuse the existing stack frame from bar(..). That's not only faster, but it also uses less memory.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4610-4611 | Added on Friday, December 30, 2016 7:41:36 AM

With TCO the engine can perform all those calls with a single stack frame!
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4614-4624 | Added on Friday, December 30, 2016 7:42:37 AM

Consider that recursive factorial(..) from before, but rewritten to make it TCO friendly: function factorial(n) {    function fact(n,res) {        if (n < 2) return res;        return fact( n - 1, n * res );    }    return fact( n, 1 );}factorial( 5 );        // 120 This version of factorial(..) is still recursive, but it's also optimizable with TCO, because both inner fact(..) calls are in tail position.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4633-4633 | Added on Friday, December 30, 2016 7:43:39 AM

Review
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4636-4636 | Added on Friday, December 30, 2016 7:43:50 AM

Benchmark.js
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4637-4639 | Added on Friday, December 30, 2016 7:43:56 AM

It's important to get as many test results from as many different environments as possible to eliminate hardware/device bias. jsPerf.com is a fantastic website for crowdsourcing performance benchmark test runs.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4646-4646 | Added on Friday, December 30, 2016 7:44:28 AM

Appendix A: asynquence Library
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4649-4650 | Added on Friday, December 30, 2016 7:44:54 AM

I referenced my own asynchronous library asynquence (http://github.com/getify/asynquence) -- "async" + "sequence" = "asynquence"
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4661-4662 | Added on Friday, December 30, 2016 7:46:12 AM

Sequences, Abstraction Design
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4663-4665 | Added on Friday, December 30, 2016 7:46:45 AM

fundamental abstraction: any series of steps for a task, whether they separately are synchronous or asynchronous, can be collectively thought of as a "sequence". In other words, a sequence is a container that represents a task, and is comprised of individual (potentially async) steps to complete that task.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4665-4665 | Added on Friday, December 30, 2016 7:49:52 AM

Each step in the sequence is controlled under the covers by a Promise
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4665-4666 | Added on Friday, December 30, 2016 7:50:02 AM

every step you add to a sequence implicitly creates a Promise that is wired to the previous end of the sequence.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4668-4668 | Added on Friday, December 30, 2016 7:50:15 AM

meaning that step 2 always comes after step 1 finishes,
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4668-4669 | Added on Friday, December 30, 2016 7:50:55 AM

a new sequence can be forked off an existing sequence,
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4669-4670 | Added on Friday, December 30, 2016 7:51:05 AM

Sequences can also be combined in various ways, including having one sequence subsumed by another
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4675-4675 | Added on Friday, December 30, 2016 7:52:09 AM

Promises themselves should never be able to be canceled, as this violates a fundamental design imperative: external immutability.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4676-4676 | Added on Friday, December 30, 2016 7:52:15 AM

But sequences have no such immutability design principle,
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4676-4677 | Added on Friday, December 30, 2016 7:52:25 AM

mostly because sequences are not passed around as future-value containers that need immutable value semantics.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4677-4677 | Added on Friday, December 30, 2016 7:52:35 AM

the proper level of abstraction to handle abort/cancel behavior.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4681-4682 | Added on Friday, December 30, 2016 7:53:22 AM

Abstractions are meant to reduce boilerplate and tedium,
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4682-4684 | Added on Friday, December 30, 2016 7:53:39 AM

With Promises, your focus is on the individual step, and there's little assumption that you will keep the chain going. With sequences, the opposite approach is taken, assuming the sequence will keep having more steps added indefinitely.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4689-4691 | Added on Friday, December 30, 2016 7:54:33 AM

But if you abstract your thinking to a sequence, and consider a step as a wrapper around a Promise, that step wrapper can hide such details, freeing you to think about the flow control in the most sensible way without being bothered by the details.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4691-4692 | Added on Friday, December 30, 2016 7:54:51 AM

Second, and perhaps more importantly, thinking of async flow control in terms of steps in a sequence allows you to abstract out the details of what types of asynchronicity are involved with each individual step.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4702-4704 | Added on Friday, December 30, 2016 7:56:07 AM

The takeaway is that sequences are a more powerful and sensible abstraction for complex asynchrony than just Promises (Promise chains) or just generators, and asynquence is designed to express that abstraction with just the right level of sugar to make async programming more understandable and more enjoyable.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4713-4713 | Added on Friday, December 30, 2016 9:44:32 PM

Steps
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4718-4734 | Added on Friday, December 30, 2016 9:45:45 PM

ASQ(    // step 1    function(done){        setTimeout( function(){            done( "Hello" );        }, 100 );    },    // step 2    function(done,greeting) {        setTimeout( function(){            done( greeting + " World" );        }, 100 );    })// step 3.then( function(done,msg){    setTimeout( function(){        done( msg.toUpperCase() );    }, 100 );} )// step 4.then( function(done,msg){    console.log( msg );            // HELLO WORLD} );
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4738-4738 | Added on Friday, December 30, 2016 9:46:07 PM

with asynquence, all you need to do is call the continuation callback
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4739-4753 | Added on Friday, December 30, 2016 9:46:24 PM

Each step defined by then(..) is assumed to be asynchronous. If you have a step that's synchronous, you can either just call done(..) right away, or you can use the simpler val(..) step helper: // step 1 (sync)ASQ( function(done){    done( "Hello" );    // manually synchronous} )// step 2 (sync).val( function(greeting){    return greeting + " World";} )// step 3 (async).then( function(done,msg){    setTimeout( function(){        done( msg.toUpperCase() );    }, 100 );} )// step 4 (sync).val( function(msg){    console.log( msg );} );
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4814-4814 | Added on Friday, December 30, 2016 9:49:03 PM

Parallel Steps
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4816-4816 | Added on Friday, December 30, 2016 9:49:16 PM

A step in a sequence in which multiple substeps are processing concurrently is called a gate(..)
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4817-4817 | Added on Friday, December 30, 2016 9:49:27 PM

and is directly symmetric to native Promise.all([..]).
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4704-4705 | Added on Saturday, December 31, 2016 7:16:31 AM

asynquence API
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4705-4705 | Added on Saturday, December 31, 2016 7:17:13 AM

the way you create a sequence
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4706-4706 | Added on Saturday, December 31, 2016 7:17:18 AM

is with the ASQ(..) function.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4713-4713 | Added on Saturday, December 31, 2016 7:17:56 AM

http://github.com/getify/asynquence
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4755-4756 | Added on Saturday, December 31, 2016 7:18:48 AM

Think of val(..) as representing a synchronous "value-only" step, which is useful for synchronous value operations, logging, and the like.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4756-4756 | Added on Saturday, December 31, 2016 7:18:56 AM

Errors
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4757-4759 | Added on Saturday, December 31, 2016 7:19:17 AM

With Promises, each individual Promise (step) in a chain can have its own independent error, and each subsequent step has the ability to handle the error or not. The main reason for this semantic comes (again) from the focus on individual Promises rather than on the chain (sequence) as a whole.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4772-4787 | Added on Saturday, December 31, 2016 7:21:20 AM

Just like with Promises, all JS exceptions become sequence errors, or you can programmatically signal a sequence error: var sq = ASQ( function(done){    setTimeout( function(){        // signal an error for the sequence        done.fail( "Oops" );    }, 100 );} ).then( function(done){    // will never get here} ).or( function(err){    console.log( err );            // Oops} ).then( function(done){    // won't get here either} );// latersq.or( function(err){    console.log( err );            // Oops} ); Another
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4772-4793 | Added on Saturday, December 31, 2016 7:22:19 AM

Just like with Promises, all JS exceptions become sequence errors, or you can programmatically signal a sequence error: var sq = ASQ( function(done){    setTimeout( function(){        // signal an error for the sequence        done.fail( "Oops" );    }, 100 );} ).then( function(done){    // will never get here} ).or( function(err){    console.log( err );            // Oops} ).then( function(done){    // won't get here either} );// latersq.or( function(err){    console.log( err );            // Oops} ); Another really important difference with error handling in asynquence compared to native Promises is the default behavior of "unhandled exceptions". As we discussed at length in Chapter 3, a rejected Promise without a registered rejection handler will just silently hold (aka swallow) the error; you have to remember to always end a chain with a final catch(..). In asynquence, the assumption is reversed. If an error occurs on a sequence, and it at that moment has no error handlers registered, the error is reported to the console. In other words, unhandled rejections are by default always reported so as not to be swallowed and missed.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4819-4855 | Added on Saturday, December 31, 2016 7:23:39 AM

Consider: ASQ( function(done){    setTimeout( done, 100 );} ).gate(    function(done){        setTimeout( function(){            done( "Hello" );        }, 100 );    },    function(done){        setTimeout( function(){            done( "World", "!" );        }, 100 );    }).val( function(msg1,msg2){    console.log( msg1 );    // Hello    console.log( msg2 );    // [ "World", "!" ]} ); For illustration, let's compare that example to native Promises: new Promise( function(resolve,reject){    setTimeout( resolve, 100 );} ).then( function(){    return Promise.all( [        new Promise( function(resolve,reject){            setTimeout( function(){                resolve( "Hello" );            }, 100 );        } ),        new Promise( function(resolve,reject){            setTimeout( function(){                // note: we need a [ ] array here                resolve( [ "World", "!" ] );            }, 100 );        } )    ] );} ).then( function(msgs){    console.log( msgs[0] );    // Hello    console.log( msgs[1] );    // [ "World", "!" ]} ); Yuck. Promises require a lot more boilerplate overhead to express the same asynchronous flow control.
==========
You Don't Know JS: Async & Performance (Kyle Simpson)
- Your Highlight on Location 4855-4856 | Added on Saturday, December 31, 2016 7:23:51 AM

That's a great illustration of why the asynquence API and abstraction make dealing with Promise steps a lot nicer.
==========
