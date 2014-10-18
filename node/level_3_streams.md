# Level 3 - Streams

## File Read Strem

#### What are Streams?

* In Node we're building a lot of network apps, we're sending data back and forth. We want to process the data the moment it arrives, piece by piece. 
* We do this using streams. 
* Stream can be readable, writeable, or both.
	* The server request is a readable stream
	* The server response is a writeable stream

#### How to Read from the Request?
* How do we read from the request if we send some data into the server? 
	* That request is a readable stream, which inherits from event emitter
	* The event emits a couple different events, two of which, are the **data event** and the **end event** when it's done transmitting data to the server.

* In this example we're going to print what we receive from the request. 
	* Note: we have to call *toString* because chunk is a buffer. We could be dealing with binary data.

```
http.createServer(function(request, response) {
	response.writeHead(200);
	request.on('data', function(chunk) {
		console.log(chunk.toString());
	});
	request.on('end', function() {
		response.end();
	});
}).listen(8080);
```

#### How To Create an Echo Server

* We only have to add one line of code
	* Instead of console logging the chunk, write it.

```
http.createServer(function(request, response) {
	response.writeHead(200);
	request.on('data', function(chunk) {
		response.write(chunk);
	});
	request.on('end', function() {
		response.end();
	});
}).listen(8080);
```

## Pipe 

* When all we need to do is read data from a readable stream and write that data to a writeable stream, there's a shortcut in Node that makes this easier. 
	* We simply call ```request.pipe(response);```

```
http.createServer(function(request, response){
	response.writeHead(200);
	request.pipe(response);
}).listen(8080);
```

#### Reading and Writing a File (Using Pipe)

1.  Request the file system module
2. Call the create read strem function and pass in the file we want to read
	* That returns us a read stream
3. Then create a new write stream for the file we want to write to
	* Returns a write stream
4. All we have to do to read from one and write to the other is use *pipe*

```
var fs = require('fs'); // require filesystem module
var file = fs.createReadStream('readme.md');
var newFile = fs.createWriteStream('readme_copy.md');

file.pipe(newFile);
```

## Upload a File

```
var fs = require('fs');
var http = require('http');

http.createServer(function(request, response){
	var newFile = fs.createWriteStream("readme_copy.md");
	request.pipe(newFile);
	
	request.on('end', function() {
		response.end('uploaded!');
	});
}).listen(8080);
```

## The Awesome Streaming

* We're streaming pieces of the file from the client to the server, then the server's streaming that to the hard drive. 
	* At no point in time does the server have the whole file, it's streaming. 
	* Because we're using Node, it's also non-blocking
* If we upload two files at the same time to the server, they'll be uploading and streaming together in parallel 

#### Back Pressue

* Because we're streaming piece-by-piece it could introduce another problem, which looks like: ```writable stream slower than readable stream```
	* We're reading files from the client faster than we can write them to disk and pieces of a file might get built up in memory and fill up our memory, which is bad
* Pipe takes care of this for us
	* It will properly pause the read stream, so the write stream can catch up. 

#### Pipe Solves Backpressure

* Pause when writeStream is full
	* The function ```writeStream.write(chunk)``` will return false if the kernel buffer is full
		* Therefore, is the buffer is not good, let's pause it
* Listen for the drain event to know when to resume the writeStream
	* The **drain event** means we're ready to write again
	
```
readStream.on('data', function(chunk) {
	var buffer_good = writeStream.write(chunk);
	if (!buffer_good) readStream.pause();
});

writeStream.on('drain', function(){
	readStream.resume();
});
```

* Note: All of this logic is encapsulated in the pipe command

```
readStream.pipe(writeStream);
```

## File Uploading Progress

### Intro

* We need two modules:
	* HTTP Server
	* File System

### File Upload Code

* Take the code from before, but add variables
	* fileBytes
	* uploadedByted
	* listen to data event that gets called every time we receive a new chunk from the request
		* increase number of uploaded bytes by the length of the chunk
		* calculate progress by dividing uploaded bytes by total file bytes, times 100
		* then we're going to write the progress back to the response

```
var fs = require('fs');
var http = require('http');

http.createServer(function(request, response) {
	var newFile = fs.createWriteStream("readme_copy.md");
	var fileBytes = request.headers['content-length'];
	var uploadedBytes = 0;
	
	request.pipe(newFile);
	
	request.on('data', function(chunk) {
		uploadedBytes += chunk.length;
		var progress = (uploadedBytes / fileBytes) * 100;
		response.write("progress: " + parseInt(progress, 10) + "%\n");
	});
	
	request.on('end', function() {
		response.end('uploaded!');
	});
}).listen(8080);
```

* Pipe Automatically Closes Write Stream
	* Pipe will automaticaly close the write stream. In order to do something before the pipe write stream ends, you need to pass in an object to the pipe

```
file.pipe(process.stdout, {end: false});
```