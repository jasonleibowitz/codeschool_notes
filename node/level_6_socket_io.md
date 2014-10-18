# Level 6 - Socket.io

## Chattr

#### Use socket.io to build a chat app

* We're going to have a bunch of clients that will need to receive messages right away
	* When a user sends a message, all clients receive that message from the server
	* This is not how it works in traditional HTTP servers work, with a request/response cycle

#### Socket.io abstracts websockets with fallbacks

* Websockets are a new technology on the client side
	* we need something to abstract websockets and give us a good API to use on newer and older browser - where socket.io comes in

1. Install using npm in CLI
	* ```npm install socket.io``
2. Inside app require it
3. After we create our express server, we can pass in the application created by express to the socket listen function
	* This will return an object that we can use to listen for connections coming from the client-side
4. When a new client connects to our server, let's log out "Client connected..."
	* The client in this case is just our browser

app.js

```
var socket = require('socket.io');
var app = express.createServer();
var io = socket.listen(app);

io.sockets.on('connection', function(client) {
	console.log('Client connected...');
});	
```

1. On the client-side we also have to load the socket.io library
	* Then we can connect to the server from the client, passing in our server's URL

index.html

```
<script src="/socket.io/socket.io.js"></script>
<script>
	var server = io.connect('http://localhost:8080');
<script>
```

## Sending Messages to the Client

* Then on the server, we're very easily able to send messages to the client
	* When our client connects, we start emitting events on that client, such as a messages event and passing in this data

app.js
```
io.sockets.on('connection', function(client) {
	console.log('Client connected...');
	
	// emit the messages event on the client
	client.emit('messages', { hello: 'world' });
});
```

* To receive this even on the client-side, all we have to do is listen on the messages event and pass in a callback
	* And that callback will have the data emitted from the server

index.html

```
<script src="/socket.io/socket.io.js'></script>
<script>
	var server = io.connect('http://localhost:8080');
	
	// listen for message events
	server.on('messages', function(data) {
		alert(data.hello);
	});
</script>
```

## Sending Messages to the Server

* On our server, set up a listener on the messages event to log whenever a client sends the messages event


app.js

```
io.sockets.on('connection', function(client) {
	client.on('messages', function(data) {
		console.log(data);
	});
});
```

* On the client, use jQuery to hook into the chat form
	* Whenever a message is submitted, we're going to take the value out of the text box and emit that messages event on the server

index.html

```
<script>
	var server = io.connect('http:/localhost:8080');
	$('#chat_form').submit(function(e) {
		var message = $('#chat_input).val();
		
		// emit the messages event on the server
		socket.emit('messages', message);
	});
</script>
```

## Broadcast Messages

#### Intro

* The really powerful thing about socket.io as opposed to regular websockets is that it makes it really easy to do something else other than 1-to-1 communication
	* Say you have a client that sends a message to the server and you want to broadcast that message out to all the other clients
		* socket.io gives us the ```broadcast``` flag for that

app.js

```
socket.broadcast.emit('message', 'Hello');
```

#### Instead of logging the message, broadcast it to all Clients

app.js

```
io.sockets.on('connection', function(client) {
	client.on('messages', function(data) {
		client.broadcast.emit('messages', data);
	}); // broadcast message to all other clients connected
});
```

* The only thing we have to do on the client now is listen for messages events
	* When we get that we insert the message to our chat app

index.html

```
<script>
	...
	server.on('messages', function(data) { insertMessage(data) });
</script>
```

## Saving Data on the Socket

#### Save a name on messages so we know who's sending it

* On server, listen on a custom event called join
	* When client emits that event, function will pass in the name of the user
		* So the name is in the callback
	* All we need to do is call set on the client to set the client's nickname

app.js

```
io.sockets.on('connection, function(client) {
	client.on('join', function(name) {
		client.set('nickname', name);
	}); // set the nickname associated with this client
});
```

* On the client, add code to listen for the connect event
	* When that happens, prompt the user to enter their nickname
	* Then emit the join event on the server, passing the nickname in

index.html

```
<script>
	var server = io.connect('http://localhost:8080');
	server.on('connect', function(data) {
		$('#status').html('Connected to chattr');
		nickname = prompt("What is your nickname?");
		
		server.emit('join', nickname); // notify the server of the nickname
	});
</script>
```

#### Saving Data on the Client

* Back on the server, before we send out a new message to all the other clients, we have to use ```client.get```	
	* Pass in the property we want to get and give it a callback, which will include the name we previously set on the client
* So now when we broadcast out that message to all the other clients, we can pass the name and the message to identify the client

app.js

```
io.sockets.on('connection', function(client) {
	client.on('join', function(name) {
		client.set('nickname', name); // set the nickname associated with this client
	});
	client.on('messages', function(data) {
		// get the nickname of this client before broadcasting message
		client.get('nickname', function(err, name) {
			client.broadcast.emit('chat', name + ": " + message);
		});
	});
});
```