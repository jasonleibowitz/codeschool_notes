# Level 2 - Events

## Events in the DOM

* The DOM triggers events, and you can listen for those events
	* click
	* submit
	* hover
* To attach events to the DOM, we'd use jQuery event listeners

## Events in Node

* Just like in the DOM, many objects in Node emit events
* If they emit events, they most likely inherit from the *event emitter* constructor. 
	* The *.net Server* class inherits from the *EventEmitter* and it emits the request event.
	* If we're reading a file and we call *fs.readStream* it returns a stream that inherits from *EventEmitter*, which will emit the data event as we're reading data out of the file.

## Custom Event Emitters

* Require the events class
* For our custom event emitter, we are going to create a logger event 
	* We want to emit error events, warn events and info events. And we want to be able to write listeners, so we can listen for when those events occur.

```
var EventEmitter = require('events').EventEmitter;
var logger = new EventEmitter();

// Listen for the error event

logger.on('error', function(message){
	console.log('ERR:' + message);
});

// To trigger the event

logger.emit('error', 'Spilled milk');
```

## HTTP Echo Server

#### Request event

* Our request event was an event emitter, but how did the event get attached to the emitter?

```
function(request, response){...}
```

#### Create Server Function

```
http.createServer(function(request, response){...});
```

* ```http.createServer([requestListener])``` returns a new web server object.
	* The *requestListener* is a function, which is automatically added to the 'request' event
* Class: httpServer is an *EventEmitter* with the following events:
	* Event: 'request'

	```
	function(request, response){ }
	```
	
	* Emitted each time there is a request

#### Alternate Syntax

* Instead of sending the callback *into* create server, we can instead create the server with no parameters and then tell the server that on the request event call this function.

```
http.createServer(function(request, response){ ... });
```

is the same as 

```
var server = http.createServer();
server.on('request', function(request, response){ ... });
```

* This syntax is how we typically add event listeners in Node