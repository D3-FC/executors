# asva-executors

Helper classes for your async flow control.

## Install

* **npm**: `npm install asva-executors` 
* **yarn**: `yarn add asva-executors`

## Intent

Conceptually, executors are just wrappers for asynchronous commands (functions). Yet they possess the following benefits:

* They encapsulate the command, allowing it to be safely passed around without exposing its internals.
* Provide utility methods for state monitoring. For instance, `executor.isRunning` tells whether executor runs command or not. Vanilla JS solutions often involve flags and are much clunkier.
* Enforce specific logic for command execution. You can cache results, limit concurrent runs or even encapsulate into executor the logic for your lazy loaded list.

Executors are low-level concept and might require a bit of time to wrap your head around. After that they're intuitive and fun to use.

This library was not born on the spot and classes were used in various applications big and small by multiple developers for more than a year. Expect it to be reasonably refined and well thought.

Library is armed with TypeScript declarations and tests.

## Classes

Available classes are:

* **Executor** - basic executor (is extended by all other executors).
* **CacheExecutor** - caches first result.
* **LadderExecutor** - runs subsequent request only after previous one is finished.
* **RepeatExecutor** - is just `setInterval` wrapped in class.
* **InfiniteLoader** - encapsulates lazy loaded list logic.

Here are some possible use cases:

* **Executor** - loaders (spinners), passing control through nested component structure.
* **CacheExecutor** - one time requests (languages, configs, currencies), deep nested data aggregation.
* **LadderExecutor** - live search.
* **RepeatExecutor** - timed operations, websocket simulation and other hacks : 3.
* **InfiniteLoader** - lazy loaded list.

## Code and examples

### Executor

```javascript
import { Executor } from 'asva-executors'

// This command is just example. Yours should still return promise but hopefully be more useful : 3.
const command = response => Promise.resolve(response)

// Instantiate executor
const executor = new Executor(command)

// Run command and process results
executor.run('data').then(result => console.log(result)) // Outputs 'data' to console

// Do some checks
executor.isRunning // Tells if executor currently runs command.
executor.runCount  // Show the number of currently running commands. There could be more than one, yes.
executor.wasRun // Executor was run at least once
executor.wasRunFine // Executor was run without throwing error at least once
executor.wasRunBad // Executor was run with thrown error at least once
executor.wasLastRunFine // Last executor run happened without an error
``` 

### CacheExecutor

```javascript
// We intend to make an expensive ajax call from several places.
import { CacheExecutor } from 'asva-executors'
const executor = new Executor(ajaxExpensiveCall)

// Run the same executor in a number of places simultaneously or not. 
// Command will be executed only once.
const result = await executor.run()

// If you have to load anew.
executor.runFresh()
``` 

### LadderExecutor

```javascript
// This example is live search.
import { LadderExecutor } from 'asva-executors'
const executor = new Executor(liveSearchCall)

// Imagine the case when user takes a nap on his keyboard.
executor.run('a') // This request will be run
executor.run('aa') // This request won't be run
executor.run('aaa') // This request won't be run
executor.run('aaaa') // This request will be run only after first one resolves.
// So, in total you have 2 requests instead of 4.
``` 

### DebounceExecutor

```javascript
// We want to save the form if user was inactive for 3 seconds.
import { DebounceExecutor } from 'asva-executors'
const executor = new DebounceExecutor(saveTheForm, 3000)

// User starts editing the form
executor.run()
// And does that a couple of times in quick succession.
executor.run()
executor.run()
// Then he stops. 3 seconds pass. And only then `saveTheForm` command is called.
``` 

DebounceExecutor has several public properties:
```javascript
executor.isRunning // Means command is currently executing.
executor.isWaiting // Executor is waiting the period of inactivity to finish.
executor.isActive // Executor is running or is waiting. 
```

You have to stop `RepeatExecutor` if you don't need it anymore. Similar to `setInterval` command it won't be garbage collected until then.


### RepeatExecutor

```javascript
// Being too lazy to implement websockets we decide to check notifications every ten seconds.
import { RepeatExecutor } from 'asva-executors'
const executor = new RepeatExecutor(checkNotificationsCall, 10000)

// Start checking notifications
executor.start()

// Stop checking notifications
executor.stop()
``` 

You have to stop `RepeatExecutor` if you don't need it anymore. Similar to `setInterval` command it won't be garbage collected until then.

### InfiniteLoader

```javascript
// This example is lazy loaded list of items.
import { InfiniteLoader } from 'asva-executors'

/**
* Constructor takes in two arguments:
* 
* * command, which is a function that takes in 
*   - `pointer` (position in list, OFFSET in sql),
*   - `perStep` (number of items per request, LIMIT in sql);
*   and returns promise with a specified number of items.
*   If length of items loaded is less than `perStep`, loader
*   would consider itself finished and won't trigger anymore. 
*   
* * perStep - number of items per request (defaults to 20)
*/
const infiniteLoader = new InfiniteLoader(
  async (pointer, perStep) =>  await loadListItems(pointer, perStep), 
  10
)
infiniteLoader.next() // Initial run. Let's load 10 items.
infiniteLoader.next() // User scrolls to bottom.
infiniteLoader.refresh() // User applies filter. List will be refreshed.

// To get items call
infiniteLoader.items

// You can perform various checks on `infiniteLoader`:
infiniteLoader.isRunning // Loader is running
infiniteLoader.isEmpty // We tried to load, but list is empty
infiniteLoader.ifFull // We tried to load, and list is not empty
infiniteLoader.isRefreshing // Is loading anew (refreshing or loading for first time).
```

-----------------------

Feel free to check [source code][source-code-link] or tests to get better understanding. Library exposes typings so your IDE should comfortably provide type-hinting. 

## Fun facts

* Name "executor" was taken from [Java][java-executor-url] and is quite similar conceptually.

## Issues

* Bugs will be addressed in timely manner.
* More-less library is *done*. But feel free to share your idea of improvement.

## Licence
MIT

[java-executor-url]: https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Executor.html
[source-code-link]: src/modules