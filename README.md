[![Build Status](https://travis-ci.org/efacilitation/watch-network.svg?branch=master)](https://travis-ci.org/efacilitation/watch-network)

## Information

<table>
<tr>
<td>Package</td><td>watch-network</td>
</tr>
<tr>
<td>Description</td>
<td>Execute tasks based on file events received over the network - gulpfriendly</td>
</tr>
</table>

*Scenario:* You use [Vagrant](https://github.com/mitchellh/vagrant)/[VirtualBox](https://www.virtualbox.org) in your workflow to have services and configurations in an encapsulated environment. For developing purposes you now sync a local directory into the VM using vboxfs, nfs, rsync or similar. In your VM you want to use watcher facilities for developing-concerns, but for some reason triggering [inotify](http://man7.org/linux/man-pages/man7/inotify.7.html) over the network seems to be [troublesome](http://stackoverflow.com/questions/4231243/inotify-with-nfs) or [unreliable](https://www.virtualbox.org/ticket/10660).

*Solution:* Based on the [Listen](https://github.com/guard/listen) Feature "[Forwarding file events over TCP](https://github.com/guard/listen#forwarding-file-events-over-tcp)" `watch-network` will connect to a Listen broadcaster as a receiver and watch for File Events. Upon receiving a File Event it will execute `tasks` based on `patterns`. Vagrants [rsync-auto](http://docs.vagrantup.com/v2/cli/rsync-auto.html) is based on Listen TCP too.

If you're looking for an alternative Listen implementation in Go, there's [`GoListen`](https://github.com/ekino/golisten) which has also a guide on how you can use it with `watch-network`.



## Setup Listen

> Listen Version >= 2.8 required

The listen gem provides the `listen` executable. So you need to install

```
gem install listen
```

or just bundle with the provided Gemfile

```
bundle
```

To start the listen broadcast process inside your *local* project directory you then can

```
listen -v -d .
```

## Install

```
npm install watch-network
```

## Usage

```javascript
WatchNetwork = require("gulp-watch-network");

watch = WatchNetWork({
  configs: [
    {
      patterns: 'src/*.js',
      tasks: 'something:important',
      onLoad: true
    }
  ]
});

watch.task('something:important', function(callback) {
  // if you define the optional `callback` function the watcher will wait until you call it
});

watch.initialize();

```


## Usage with Gulp

```javascript
gulp = require('gulp');
gulp.task('something:important', function() {
  // ..
});

WatchNetwork = require("gulp-watch-network");

watch = WatchNetWork({
  gulp: gulp,
  configs: [
    {
      patterns: 'src/*.js',
      tasks: ['something:important', 'another:thing']
    }
  ]
});

watch.task('another:thing', function() {
  // ..
});

watch.initialize();
```

> Note: If you define a task with the same name on the watcher and on gulp - both will get executed. On the watcher defined tasks execute first.


## API

#### WatchNetwork

Params:

- `host` String Listen Host to connect to (default `'localhost'`)
- `port` String|Number Listen Port to connect to (default `4000`)
- `rootFile` String Name of the RootFile which determines the basepath (relevant for `patterns`) (default `'.root'`)
- `flushDeferredTasks` Boolean Wether to flush tasks which got deferred while other tasks are already running (default `true`)
- `gulp` Object Gulp Object with defined Tasks which will get executed with [run-sequence](https://www.npmjs.com/package/run-sequence) (default `null`)
- `configs` Array Contains Pattern/Task Configuration Object
  - `patterns` String|Array Pattern to match against FileChange based on [minimatch](https://www.npmjs.com/package/minimatch)
  - `tasks` String|Array Tasks to execute if patterns match
  - `onLoad` Boolean Wether to execute the `tasks` once while `initialize`-Phase (default `false`)


#### initialize

Initialize the Watcher.

Params:
- callback Function Callback which gets called after the Watcher initialized


#### task

Define Task Function which gets executed if patterns match

Params:
- taskName String Name of the task
- taskFunction Function Task Function


#### stop

Stops the watcher by destroying the listen socket and removing all event listeners. Be sure to cleanup the watcher instance yourself.


#### on

Register Event Listener

Params:
- eventName String Name of the event
- subscriberFn Function Function which gets called when event fires

Example:

```javascript
watch = WatchNetwork();
watch.on('initialized', function() {
  // ..
});
watch.on('changed', function(changedFiles) {
  // ..
});
watch.initialize()
```

> Note: WatchNetwork extends [EventEmitter](http://nodejs.org/api/events.html)


### Available Events

- `initialized` Watcher initialized (RootFile-Sync completed)
- `changed` Watcher detected file changes, first parameter will include the changed files as an array


## Determining Base Path

Given

- we have a local working directory: `/home/wn/work`
- we have a synced version inside the VM: `/vagrant`

Now if we initialize WatchNetwork inside the VM it does the following:

- Touch the `RootFile` (default `process.cwd()` + `rootFile`)
- Wait for FileChange which contains `.root`
- Compute RemoteRoothPath (basedir of RootFile): `/home/wn/work`
- Initialized.
- On follow-up FileChanges we will strip the RemoteRootPath
  - Changing `/home/wn/work/foo.js`
  - What gets matched against the patterns is `foo.js` then


## License
Licensed under the MIT license.
