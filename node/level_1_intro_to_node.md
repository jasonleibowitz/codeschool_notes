# Level 1 - Intro to Node.js

## What is Node

* Allows you to build scalable network applications using JavaScript on the server-side
* Under covers of Node, you'll find the V8 JavaScript runtime
	* Same runtime used in Chrome browser
	* Node provides wrapper around this engine, providing additionally functionality for building network apps
* Really fast
	* All written in C, which is really fast
* What can you build with Node?
	* Websocket Server
		* like a chat server - lots of browsers connecting to server, and sending messages between client and server and socket stays open
	* File Upload Client
		* you can uplaod many files at the same time (and process them) that is not blocking
	* Ad Server
		* Ads get served all over the net. Need to be served fast across the newtork
	* Real Time Data Apps
		* Any sort of network servers
		
## What Isn't Node

* Not a Framework
	* Won't replace Rails or Django
	* It is low-level network communication
* Not a multi-thread application
	* when you write applications with Node, you write them as if you're dealing with a single-threaded server

## Blocking vs Non-Blocking Code

#### Blocking Code

```
Read file from Filesystem, set equal to "contents"
Print contents
Do something else
```

* If we think about how this code executes:
	* First we read the file from the filesysem
	* We can't actually print out the contents until we've read out all the contents from the file, so it's kind of blocking here
	* Then we do something else

```
var contents = fs.readFileSync('/etc/hosts');
console.log(contents);
console.log('Doing something else');
```
	
#### Non-Blocking Version (Callback)

```
Read file from Filesystem
	whenever you're complete, print the contents
Do something else
```

* Technique of 'when you're complete' is called a **callback**.

```
fs.readFile('/etc/hosts', function(err, contents){
	console.log(contents);
});
console.log('Doing something else');
```

#### Callback Alternative Syntax

```
var callback = function(err, contents){
	console.log(contents);
}
fs.readFile('etc/hosts', callback);
fs.readFile('etc/inetcfg', callback);
```

* Because this is written as "non-blocking" these two files will be read in parallel, and run much quicker.

## Hello Dog

#### hello.js

1. Require the http module, which is a library. 
	* This syntax is how we require modules.
2. Then we call the createServer function, which takes as its single parameter a callback with request and response
3. We're then going to write a 200 status code in the header
4. We're going to write out the response body
5. Finally we're going to end the response
6. We want our server to listen on port 8000
7. And to ensure our server's running, we're going to console log that it's running

```
var http = require('http');
http.createServer(function(request, response) {
	response.writeHead(200); // Status code in the header
	response.write("Hello, this is dog"); // Response dog
	response.end(); // Close the connection
}).listen(8000); // Listen for connection on this port
console.log('Listening on port 8000...');
```
* Run the server with 

```
node hellodog.js
```

#### The Event Loop

* The first time we go through this code and execute it, Node will register events.
	* In this case, we're registering the *request* event when a request comes in
* Once it gets done executing the script, Node goes into an *event loop*. 
	* It's checking for events continiously 
	* As soon as a request comes in, it's going to trigger that event and will then trigger the callback we wrote and print our hello this is dog.
* Node has events (known events) for *request*, *connection*, *close* and more.
	* once our app gets into the event loop it can start triggering and emitting events into our **event queue*.

#### The Event Queue

* If a request event comes in at the same time as a close event, events will be processed one-at-a-time in our event loop

## Why JavaScript

* JavaScript makes it really easy for us to program in an evented way - to do evented programming using that event loop and to write code that is potentially non-blocking. 

## Long Running Process

```
var http = require('http');
http.createServer(function(request, response){
	response.writeHead(200);
	response.write("Dog is running.");
	setTimeout(function(){
		response.write("Dog is done.");
		response.end();
	}, 5000)
}).listen(8000);
```

## Typical Blocking Things

* Call out to web services
* Reads/Writes to Database
* Calls to extensions