Questions:
Promises vs Continuations
Why callbacks in nodejs vs promises

What’s the difference between async and multithreaded?
http://stackoverflow.com/questions/21122842/whats-the-difference-between-synchronous-and-asynchronous-calls-in-objective-c
When you invoke something synchronously, it means that the thread that initiated that operation will wait for the task to finish before continuing. Asynchronous means that it will not wait.
Having said that, when people suggest that you perform some slow or expensive process asynchronously, they are implicitly suggesting not only that you should run it asynchronously, but that you should do that on a background thread. The goal is to free the main thread so that it can continue to respond to the user interface (rather than freezing), so you are dispatching tasks to a background thread asynchronously.
ie, Async means - don’t lock the foreground thread

So, there are two parts to that. First, using GCD as an example, you grab a background queue (either grab one of the global background queues, or create your own):


http://www.i-programmer.info/programming/theory/6040-what-is-asynchronous-programming.html
UI is original async
Run loop
Event Handlers
Threads

http://cs.brown.edu/courses/cs168/f12/handouts/async.pdf
Sync Model: Simplest program runs in order
      - One finishes before the next one starts
Multithreaded - all at once

Async model - interleaved tasks (NB IO)
      - async a LOT faster when sync has to block and wait for things (IO/remote requests/cpu tasks/etc)
      - less time waiting
     Compared to the synchronous model, the asynchronous model performs best when:
• There are a large number of tasks so there is likely always at least one task that can make progress.
• The tasks perform lots of I/O, causing a synchronous program to waste lots of time blocking when
other tasks could be running.
• The tasks are largely independent from one another so there is little need for inter-task communication
(and thus for one task to wait upon another).
Why not just threads?
     - lots of cost for context switching and saving state
     - performance problems with lots of threads


http://stackoverflow.com/questions/16336367/what-is-the-difference-between-synchronous-and-asynchronous-programming-in-node


Promises Patent
https://www.google.com/patents/US20140282625


https://egghead.io/series/mastering-asynchronous-programming-the-end-of-the-loop

http://www.infoq.com/articles/surviving-asynchronous-programming-in-javascript
lib overview: step, flow-js, node-promise, async, async.js
     - Dojo Promises: http://www.sitepen.com/blog/2010/05/03/robust-promises-with-dojo-deferred-1-5/
          http://www.sitepen.com/blog/2010/09/20/promised-io/

     - Async - uses callbacks instead of promises/continuations because that’s what nodejs adopted

http://www.html5rocks.com/en/tutorials/async/deferred/
     - Most basic asynchronous program is waiting for a button click
     - try/catch breaks - bubbles up to new place

http://know.cujojs.com/tutorials/async/async-programming-is-messy.html.md#Exceptions-and-try-catch
Try Catches
     - have to happen with callbacks
     - try/catch will just catch errors that happen during the immediate call


Callback hell:
http://stackoverflow.com/questions/19874967/flattening-a-very-nesting-intensive-function-how-to/19883141#19883141
 - Promises Solution
 - throw custom errors so you can pull them out later

Promises : catches exceptions, and can catch custom exceptions.
Async: all about contained functions that are composed
     - here’s why: https://github.com/caolan/async/pull/396
          - belief that each is responsible for errors

http://www.joyent.com/developers/node/design/errors
     - return an event emitter in complicated scenarios, or where a bunch of different events can occur
     - "For a given function, if any operational error can be delivered asynchronously, then all operational errors should be delivered asynchronously. "

https://www.joyent.com/developers/node/design
     - async with ability to debug where things get stuck (https://github.com/davepacheco/node-vasync)

EMCA6 has promises built in.
  RSVP.js, Q.js, and the $q
http://www.html5rocks.com/en/tutorials/es6/promises/
     Good description of “why” events vs promises

NOTE: JQuery Deferreds are NOT exactly like promises (slightly different)


…thankfully this is usually what you want, or at least gives you access to what you want. Also, be aware that jQuery doesn't follow the convention of passing Error objects into rejections.

A “stored” promise is memoized (resolves immediately with the value)

Sequences - for doing a for loop of downloads:


Tracer await/yield:
http://jakearchibald.com/2014/es7-async-functions/



http://tech.pro/blog/1402/five-patterns-to-help-you-tame-asynchronous-javascript
Callbacks:
Pros
Simplicity - Callbacks are simple! (The caveat being that you are responsible for maintaining the correct context, if needed.) You pass a function, it gets invoked at some point in the future.
Lightweight - No extra libs required. Functions-as-first-class-citizens is built into the language. No need for additional code to make it work.
Cons
Can Be An Insufficient Abstraction - Sometimes you need additional 'sugar'. This is where patterns 2-5 will come into play, each build upon the foundation of callbacks in different ways.
Complex When Nested - Callbacks are difficult to read, debug and maintain when deeply nested. The goto-example these days to describe this problem is "the pyramid of doom".

2.) Observer Pattern (a.k.a. Events)
"You have a subject that emits events (often referred to as an "event emitter"), and observers, which register callbacks with the subject to be notified as to when those events occur.” - i.e., jquery/backbone events
Pros
Encourages Good De-coupling - Using the Observer pattern will help you tease apart behaviors in your application since it's forcing you to think through the interactions that can occur. This leads to well-encapsulated components, and typically results in a design more resistant to the traps of deeply nested callbacks.
Lends Itself to Testability - Due to the bias towards de-coupling that comes with this approach, you end up with components that are testable in isolation and easy to mock in most cases.
Cons
Direct Reference Required - This is the Achilles' heel for event-emitters. If you have a subject that's very interesting to most of your application, you will end up passing a reference to it everywhere. This begins to undermine any 'loose coupling' gains you've racked up, creating a kind of tight coupling that can be harder to detect until you have to change the subject's API and break most of your app in the process.
Elephant, Meet Room
So, let's tackle what I've avoided until now:

Many popular libraries (like jQuery) provide what looks like an "Observer pattern" implementation, when in fact they are acting as a mediator between observer and subject yet still presenting the API as if it were being called directly on the subject. This isn't a problem, of course! It's just good to know how things work.
Many of these implementations provide event-emitting behaviors, with the intent that the API is on the prototype, or mixed-in as instance members, etc. However, many developers take these event-emitting libraries and create a singleton instance and use this instance as a generalized "emitter", so subjects no longer have the on/off/etc. calls themselves. When this happens, we are, in my opinion, leaving the observer pattern behind and we've begun to wander into the territory of messaging.

3) Messaging
Mediator - "Communication between objects is encapsulated within a mediator. Objects no longer communicate directly with each other, but instead communicate through the mediator."
Event Aggregator - "Channel events from multiple objects into a single object to simplify registration for clients."
(Client-side) Message Broker - "Receives messages from multiple destinations, determines the correct destination & routes the message."
NOT PubSub (in his opinion)
Use in “tandem” with Events/Observer based on “bounded contexts"

Pros
Promotes clean separation of concerns - you don't have to build a large app. Instead, build several smaller ones and use messaging for communications between them.
Very testable - it's super easy to fake a message or test for message output.
Complements events nicely - Observer pattern 'events' work nicely for local concerns. Messaging complements this when those events need to be promoted to app-level messages.
Cons
Can be prone to "boilerplate proliferation" - you can mitigate this by writing app-specific helpers
Can be confusing to devs who are new to the concept in general - take the proper time to explain and teach.

4) Promises
Promises provide a very powerful way to express asynchronous code. Instead of "continuation passing style" (which we see with plain callbacks), our target methods return a promise - an "eventual value".
Pros
Reduces Complexity of Nesting & Flow - Flattening pyramids of doom is nearly always a win. Being able to express certain aspects of asynchronous code in a synchronous style (with return values, etc.) empowers developers at nearly any level to quickly understand what a section of code is doing.
Results Can be "Cached" - If you need to keep a resolved promise around, any additional success handlers added to it are invoked (as soon as the runtime is free) and passed the resulting value. This can save you some boilerplate as well as prevent unnecessary operations (like a duplicate AJAX request, for example).
Cons
Results Can be "Cached" - Wait - wasn't this a PRO?! Yes - but it's a double-edged sword. If you're keeping promise instances around, avoid doing so for values that frequently change (and, for example, might require another HTTP request to fetch the latest).
Opinionated on OSS APIs - If you're an open source author writing libraries for general consumption, please use a spec-compliant library! When you don't, you force a highly opinionated dependency on consuming developers.
Many Non-Spec-Adhering Implementations Exist - Choosing one of these can get you past that:
Q
when.js
RSVP.js

http://www.2ality.com/2012/06/continuation-passing-style.html
- Continuation-passing style = normal callbacks in js

http://www.slideshare.net/domenicdenicola/callbacks-promises-and-coroutines-oh-my-the-evolution-of-asynchronicity-in-javascript
     - Presentation on Callbacks/Promises

http://www.slideshare.net/domenicdenicola/promises-promises?next_slideshow=1
     - Deep dive on promises

https://github.com/bellbind/using-promise-q
     - all about patterns to use the Q library

Delegates vs Observer
http://programmers.stackexchange.com/questions/178008/what-are-the-advantages-of-the-delegate-pattern-over-the-observer-pattern


Delegation Pattern on Mobile
http://myexperienceproject.blogspot.com/2012/07/delegation-pattern.html

Android Delegates, AKA Listeners
http://homepages.ius.edu/RWISMAN/C490/html/Android-Delegates.htm

Android Async:
Run loops
RXJava
http://www.slideshare.net/TimoTuominen1/rxjava-architectures-on-android-android-livecode-berlin?next_slideshow=1

iOS Delegation Definition
https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html
      In a managed memory environment, the delegating object maintains a weak reference to its delegate; in a garbage-collected environment, the receiver maintains a strong reference to its delegate.