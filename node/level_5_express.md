# Level 5 - Express

## Intro

#### What is Express

* It's a sinatra-inspired web app framework for Node.js
* It gives you:
	* Easy route URLs to callbacks
	* Middleware (from Connect)
	* Environment based configuration
	* Redirection helpers
	* File Uploads

## Hello World App

1. Require Express after we install it with ```npm install express```
2. Then call createServer on it, which will return an application
	* Using the application we can setup a root route, that responds to the GET request, and whenever we get a request to that route, it's going to call this callback
		* The call back is going to send the index.html file from our current directory
			* ```__dirname``` is current directory
3. Finally, just listen on port 8080

```
var express = require('express');

var app = express.createServer();
```

```
app.get('/', function(request) {
	response.sendfile(__dirname + "/index.html");
}
app.listen(8080);
```

Note: the above is deprecated

```
var express = require('express');
var http = require('http');
var app = express();
var server = http.createServer(app);

app.get('/', function(request, response){
	response.sendfile(__dirname + "/index.html");
});
app.listen(8080);
```

## Express Routes

* Make a request with the Twitter API and respond with the JSON

1. Require request and url modules
2. Define a route for tweets/username
3. Define callback
	* Inside callback we can get access to username param by calling ```req.params.username```
	* Set up options object to build URL for API request
3. Use url.format to turn the options object into a URL string
4. Put URL string into the request function and pipe back the request to the response

app.js

```
var request = require('request');
var url = require('url');

app.get('/tweets/:username', function(req, response){
	var username = req.params.username;
	
	options = {
		protocol: 'http',
		host: 'api.twitter.com',
		pathname: '/1/statuses/user_timeline.json',
		query: { screen_name: username, count: 10 }
	} // gets last 10 tweets for screen_name
	
	var twitterUrl = url.format(options);
	request(twitterUrl).pipe(response);
})
```

## Express + HTML

* Express has really good support for using template libraries and layouts to build webpages

#### Express Templates

* Instead of pipeing the results directly to the response, we're going to pass in a callback.
	* Inside of the callback we're going to parse the body of the response using JSON
	* Then we're going to call ```response.render```, which is provided to us by Express
		* We pass in the name of the template file as the first argument, and the object as the second argument

app.js

```
app.get('/tweets/:username', function(req, response) {
	...
	request(url, function(err, res, body) {
		var tweets = JSON.parse(body);
		response.render('tweets.ejs', {tweets: tweets, name: username});
	});
}
```

* In the template file we have access to the object we pass in using render.

tweet.ejs

```
<h1>Tweets for @<%= name %></h1>
<ul>
	<% tweets.forEach(function(tweet){ %>
		<li><%= tweet.text %></li>
	<% }); %>
</ul>
```

#### Template Layouts

* Like in Rails with an application.layout file, Express injects templates into a layout template
* We therefore need to define a ```layout.ejs``` file

layout.ejs

```
<!DOCTYPE html>
<html>
	<head>
		<title>Tweets</title>
	</head>
	<body>
		<%- body %>
	</body>
</html>
```