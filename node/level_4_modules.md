# Level 4 - Modules

## Requiring Modules

* How does require work?
	* How does 'require' return the libraries?
	* How does it find these files?
	* What does require return?
* Modules
	* In Node, when you want to share code, you create modules

#### Create Custom Module

custom_hello.js
* To define what we want returned when we require this file, use exports

```
var hello = function() {
	console.log("hello!");
}
exports = hello;
```

* You can also export a function into the exports object
	* So when we return it, and object is returned, then we can call thta function on the object

```
exports.goodbyte = function(){
	console.log("bye!");
}
```

app.js

```
var hello = require('./custom_hello');
var gb = require('./custom_goodbye');
hello();
gb.goodbye();
```

* You can also call the function from a module immediately

```
require('.custom_goodbye').goodbye();
```

## Export Multiple Functions

* Define multiple functions in the my_module.js file to our exports file
	* This makes any functions that are exported available after requiring, but any other functions stay private

my_module.js

```
var foo = function() { ... }
var bar = function() { ... }
var baz = function() { ... }

exports.foo = foo
exports.bar = bar
```

app.js

```
var myMod = require('./my_module')
myMod.foo();
myMod.bar();
```

## Making HTTP Requests

#### Creating HTTP Client That Makes Requests

* Instead of making a server that responds to requests, we're going to make a client that makes requests

1. Require http module
2. Message that we want to send in the request body
3. Define options object that will eventually be turned into URL request
4. Then we'll call the request function on our http module
	* That function accepts two parameters: 
		1. options for URL that we are going to make the request to
		2. second param is a callback that passes in a response, that is a readable stream so we can read data back from the response
	* Just print out response to the console
5. Finally, we write our message to the request body
6. When we call end, it'll finish the request

```
var http = require('http');
var message = "Here's looking at you, kid";
var options = {
	host: 'localhost', post: 8080, path: '/', method: 'POST'
}
var request = http.request(options, function(response) {
	response.on('data', function(chunk) {
		console.log(data); // logs response body
	}
}
request.write(message); // begins request
request.end(); // finishes request
```

#### Encapsulating The Function

* Let's encapsulate this functionality using a function
	* Allows makeRequest to be easier, only requiring us to pass in message body

```
var http = require('http');

var makeRequest = function(message) {
	var options = { 
		host: 'localhost', port: 8080, path: '/', method: 'POST'
	}

	var request = http.request(options, function(response) {
		response.on('data', function(chunk) {
			console.log(data); // logs response body
		}
	}
	request.write(message); // begins request
	request.end(); // finishes request
}
makeRequest("Here's looking at you, kid.");
```
## Creating Modules

#### Using Modules Across The App

* Now we have a makeRequest function, but if we want to use it across our application, we'd want to create a module

1. Move the makeRequest function into a new file ```make_request.js```
2. Then all we have to do is assign the makeRequest function to the exports object

```
var http = require('http');

var makeRequest = function(message) {
	...
}

exports = makeRequest;
```

#### Using Modules In Your App

* To use a module, in your app.js, require the module
* Then we can use it just as we were before

app.js

```
var makeRequest = require('./make_request.js');

makeRequest("Here's looking at you, kid.");
makeRequest("Hello, this is dog.");
```

## Require Search

#### Where on the filesystem does require look for our modules?

* When we require with ```./make_request.js```, we're telling require to look in the same directory
* Require with ```../make_request.js``` is telling require to look one directory up
* You can also pass in an absolute path to the module
	* ```var makeRequest = require('/Users/jason/nodes/make_request')```
* *However*, when you just pass in the name of the module, node will, by default, look in the **node_modules** directory
	* It will first look in ```/Home/jason/my_app/node_modules```
	* If it doesn't find it there, it'll keep going a directly up in the node_modules directory

## NPM The Userland Sea

#### NPM (Node Package Manager)

* Easily Install Third-Party Modules From The Community
* NPM is a lot like Ruby Gems
* NPM:
	* Comes with node
	* It's a module repository 
	* Has dependency management
	* Makes it easy to publish your own modules
	* Follow a *local only* install mode

#### Install a NPM Module

/Home/my_app

```
npm install request
```

* This will install the third-party request library
	* ```http://github.com/mikeal/request/```
* Install into local node_modules directory
	* ```Home/my_app/node_modules/request```
* To require one of these modules, we don't need to define the directory
	* ```var request = require('request');```

#### Local vs Global

* Some modules need to be installed globally, like CoffeeScript
* We do this by passing a ```-g``` flag
	* Then we call the coffee executable
* The rule of thumb is that if the module has an executable, you have to use the global flag when installing it

```
npm install coffee-script -g

coffee app.coffee
```

* One thing to note, is that you can't require global modules
	* You have to install the module locally

```
npm install coffee-script

var coffee = require('coffee-script');
```

#### Finding Modules

* [npm registry](https://www.npmjs.org/)
* npm command line
	* ```npm search request```
* github search
* [toolbox.no.de](http://toolbox.no.de/)
	* they list the best modules based on their category

## Defining Your Dependencies

* After a while, when building your app, you're going to start installing more and more npm modules
	* Would be nice to have a place to define what modules your app depends on
* npm makes that easy, by allowing you to define a **package.json** in the root of your application
	* this file contains a JSON literal
	* all you have to do is:
		* define the name of your app
		* which version it currently is
		* then you can pass in a dependencies object
			* in the dependencies object, we just list the modules we depend on next to their version number

package.json

```
{
	'name': 'MyApp',
	'version': '1',
	'dependencies': { 
		'connect': '1.8.7'
	}
}	
```

* After the package.json file is defined, just run ```npm install``` and it'll install all of our dependencies for us in the node_modules directory

## Symantic Versioning

* Symantic versioning is a way to give meaning to version numbers on dependencies
	* in ```"connect": "1.8.7"```:
		* 1 is the major version
		* 8 is the minor version
		* 7 is the patch
* We don't have to define the exact version number our module needs, instead we can use **ranges**
	* Using a tilda (~) gives us a range. ~1 means less than 2 and greater than 1
		* ```"connect": "~1"``` means ```>=1.0.0 < 2.0.0```
	* However, this is considered dangerous, because any changes to the minor version could break your app
		* We can make it a little safer by doing ```"connect": "~1.8"```, which means ```>=1.8 <2.0.0```
		* Still dangerous because the API could change between 1.8 and 1.9
	* The safest way to define a range is to use ```"connect": "~1.8.7"```, which means that npm will install ```>=1.8.7 < 1.9.0```, or anything greater than 1.8.7, but less than 1.9.0
		* You'll only get patch updates when running npm install