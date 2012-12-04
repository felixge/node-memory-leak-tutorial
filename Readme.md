# Debugging memory leaks in node.js

This is a quick tutorial for debugging memory leaks in node.js.

## Step 1: Install the debugging tools

First of all, you need to install the `v8-profiler` module. Without that
required from within your app, you will not be able to take heap snapshots
in the debugger.

```
mkdir node_modules
npm install v8-profiler
```

Once you have that installed, you need to install the node inspector utility
by doing:

```
npm install -g node-inspector
```

## Step 2: Create some leaking code

In order to debug a memory leak, we first need to create one. This is as easy
as adding a bunch of objects to an array every 1s inside a file called `leak.js`
(also included in this repo).

``` javascript
require('v8-profiler');

// It is important to use named constructors (like the one below), otherwise
// the heap snapshots will not produce useful outputs for you.
function LeakingClass() {
}

var leaks = [];
setInterval(function() {
  for (var i = 0; i < 100; i++) {
    leaks.push(new LeakingClass);
  }

  console.error('Leaks: %d', leaks.length);
}, 1000);
```

## Step 3: Debug the memory leak

With all the above in place, you can now debug your memory leak by running the
`leak.js` file in debug mode:

```
node --debug leak.js
```

**Note:** You can also send the SIGUSR1 signal to an existing node process to
enable the debug mode as you need it.

Now that your app is running (and leaking memory), start the node inspector in
a new shell/tab to debug the memory leak:

```
node-inspector
```

In order to use the inspector, use Google Chrome to open up the url of the
debugger which should be: [http://0.0.0.0:8080/debug?port=5858](http://0.0.0.0:8080/debug?port=5858).

You should see a typical Chrome Developer Tools window:

* Select "Profiles" from the tabs on top
* Hit the "Enable Profiling" button
* Now click the eye icon on the bottom left to take a heap snapshot
* Wait a few seconds
* Click the eye icon again to take another heapsnapshot
* Click the "Count +/-" column on top to sort by it.

You should now see something like this:

![Screenshot](https://github.com/felixge/node-memory-leak-tutorial/raw/master/screenshot.png)

In our case this lets you see that there have been 2200 new LeakingClass
objects created since the last snapshot, and that there are 29000 of them
total in the heap at this point.

**Note:** In order to find out which objects prevent your LeakingClass instances from
being garbage collected, you can use this [v8-profiler fork](https://github.com/fgnass/v8-profiler)
to explore the graph of _retainers_ ([screenshot](http://fgnass.posterous.com/finding-memory-leaks-in-nodejs-applications)).


**Update:** You can also try out [Nodetime](http://nodetime.com)'s memory profiler. It uses V8's heap profiler to take snapshots and show the heap as seen from the retainers, e.g. variables and properties. Installing Nodetime is just two simple steps: 1) `npm install nodetime` and 2) `require('nodetime').profile()` (before other require statements). It will then print out a link for accessing the profiler's web console at nodetime.com. More on memory profiling with Nodetime in the blog post [
Detecting Memory Leaks in Node.js Applications](http://nodetime.com/blog/detecting-memory-leaks-in-nodejs-apps) (with screenshot). 


That's it, you have learned the basics of tracking down memory leaks in
node.js.
