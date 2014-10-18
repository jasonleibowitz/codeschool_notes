# Level 7 - Persisting Data

## Recent Messages

* One of the limitations of last chat app, when a client connected, we didn't show them the recent messages that were sent in that chat room
* We need to add code that will emit recent messages when a new client connects

1. Create a memory array to store recent messages
2. Define a storeMessage function that takes the name and data of the message and pushes that on to the messages array
	* But if the array is longer than 10 elements, we're going to remove the oldest element
3. Then in the messages event handler, call the storeMessage function to insert our message into the array

app.js

```
var messages = []; // store messages in array
var storeMessage = function(name, data) {
	messages.push( {name: name, data: data} ); // add message to end of array
	if (messages.length > 10) {
		messages.shift(); // if more than 10 messages long, remove the last one
	}
}

io.sockets.on('connection', function(client) {
	client.on('messages', function(message) {
		client.get('nickname', function(error, name) {
			storeMessage(name, message); // when client sends a message call storeMessage
		});
	});
});
```

## Emitting Messages

* Now, when our client joins the chat we're going to use the forEach function to iterate over the messages array and emit a new message for each item in that array

app.js

```
io.sockets.on('connection', function(client) {
	...
	client.on('join', function(name) {
		messages.forEach(function(message) {
			client.emit('messages', message.name + ": " + message.data);
		}); // iterate through the message array and emit a message on the connecting client for each one
	});
});
```

## Persisting Stores (with Redis)

* Persist messages outside of Node process
	* to filesyste or dash system
* Node works really great with a bunch of dash stores
	* MongoDB
	* CouchDB
	* PostgreSQL
	* Memcached
	* Riak
	* redis
		* key-value store
* All of these dash stores can be used in a non-blocking way
	* You can send a query and tell it to callback whenever the results come in, rather than waiting for the results to come in

#### Redis Data Structures

| data structur  |  commands |
|---|---|
| Strings  | SET, GET, APPEND, DECR, INCR...  |
| Hashes  | HSET, HGET, HDEL, HGETALL...  |
| Lists  | LPUSH, LREM, LTRIM, RPOP, LINSERT...  |
| Sets  | SADD, SREM, SMOVE, SMEMBERS...  |
| Sorted Sets  | ZADD, ZREM, ZSCORE, ZRANK  |

* Great documentation for Redi at [redis.io/commands](http://redis.io/commands)

#### Third-Party Node Library for Working with Redis

* [Node Redis](https://github.com/mranney/node_redis) is a third-party library for working with Redis within Node
	* Completely non-blocking

1. Install with ```npm install redis```
2. Require redis module
3. Create Redit client using the *createClient* function
4. You can then set values to the client using the *set* function
5. To get values from Redis, use the get command with the key as the first argument and a callback that will be called once the result is ready.

```
var redis = require('redis');
var client = redis.createClient();

client.set('message1', 'hello, yes this is dog');
client.set('message2', 'hello, no this is spider');

client.get('message1', function(err, reply) {
	console.log(reply);
});
```

#### Redis Lists: Pushing

* Add a string to the messages list
	* The calback that gets called will reply with the length of the list

```
var message = "Hello, this is dog";
client.lpush("messages", message, function(err, reply) {
	console.log(reply); // replies with length of list
});
```

#### Redis Lists: Retrieving

* Common use of Redis is to use LPUSH together with LTRIM
	* After you push an item to the list, you can trim the number of items in the list, allowing you to cap the number of items in the list

```
var message = "Oh sorry, wrong number";
client.lpush("messages", message, function(err, reply) {
	// trim keeps the first two strings and removes the rest
	client.ltrim("messages", 0, 1);
}); 

```

* Retrieving from List
	* Use the LRANGE command and pass in the key, the starting index and the end index
		* Using -1 as the end index just means "give me all of the messages in the list"

```
client.lrange("messages", 0, -1, function(err, messages) {
	// replies with all strings in the list
	console.log(messages);
})

-->
["Hello, no this is spider", "Oh sorry, wrong number"]
```

## Converting Messages to Redis

* Use storeMessage function to use Redis Lists
	* Instead of just pushing messages into array, we'll push them into the Redis list
* We use JSON.stringify because we need to use JSON to turn the object into a string in order to store it into REdis

app.js

```
var redisClient = redis.createClient();

var storeMessage = function(name, data) {
	var message = JSON.stringify( {name: name, data: data} );
	
	redisClient.lpush("messages", message, functiON(err, message) {
		// keeps only the newest 10 items
		redisClient.ltrim("message", 0, 10);	});
}

```

#### Output From List

* Reverse messages from list so they come out in the right order
	* call reverse function on the messages object
* Then loop through each message, but before we emit the message we have to use JSON.parse to emit a JSON object

app.js

```
client.on('join', function(name) {
	redisClient.lrange("messages", 0, -1, function(err, message) {
		// reverse so they're emitted in correct order
		messages = messages.reverse();
		message.forEach(function(message) {
			// parse into JSON object
			message = JSON.parse(message);
			client.emit("messages" message.name + ": " + message.data);
		}):
	});
});
```

## Current Chatter List

* Show who is currently connected to the chat room
	* Instead of list, we're going to use **Redis Sets**
* Sets are similar to lists, but instead of having anything in there, it can only be *unique* data

#### Add & Remove members of the names set

```
client.sadd('names', 'Dog');
client.sadd('names', 'Spider');
client.sadd('names', 'Gregg');

client.srem('names', 'Spider');
```

#### Reply with all members of a set

```
client.smembers('names', function(err, names) {
	console.log(names);
});

-->
["Dog", "Gregg"]
```

#### Adding Chatters When Client Joins

* When a client joins:
	* Notify all other clients that a new chatter just joined
	* Also use Redis client library to add user's name to chatter set

app.js

```
client.on('join', function(name) {
	// notify other clients a chatter has joined
	client.broadcast.emit('add chatter', name);
	
	// add name to chatters set
	redisClient.sadd('chatters', name);
```

* On client, add new listeners to add chatter event, and have that call the insertChatter function
	* inserChatter function just creates a new list item with a name and appends it to the chatters unordered list in the DOM
	
index.html

```
server.on('add chatter', insertChatter);

var insertChatter = function(name) {
	var chatter = $('<li>' + name + '</li>').data('name', name);
	$('#chatters').append(chatter);
}
```

* Also when client joins server, use smembers command to get back all members currently in chat room and emit those names on the add chatter event

app.js

```
client.on('join', function(name) {
	// notify other clients a chatter has joined
	client.broadcast.emit('add chatter', name);
	
	redisClient.smembers('names', function(err, names) {
		names.forEach(function(name) {
			// emit all the currently logged in chatters to the newly connected client
			client.emit('add chatter', name);
		});
	});
```

#### Removing Chatters

* When a chatter leaves the room, we want to remove them from the chat list
* On app, listen to a new event on app from client called disconnect
	* When client disconnects, we're going to grab their name off the socket, broadcast out to all clients they left, then remove them from the chatter set

app.js

```
client.on('disconnect', function(name) {
	client.get('nickname', function(err, name) {
		client.broadcast.emit('remove chatter', name);
		redisClient.srem('chatters', name);
	});
});
```
	
* On client, we're going to listen to remove chatter event, when that's fired we're going to find the li in the list and remove it from the DOM

index.html

```
server.on('remove chatter', removeChatter);

var removeChatter = function(name) {
	$('#chatters li[data-name=' + name + ']').remove();
}
```