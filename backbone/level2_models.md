# Level 2 - Models

## Fetching Data from the Server

### Intro 

* When our To Do page loads, the first thing we're going to want to do is fetch the JSON data from the server

```
// URL to get JSON data for model
todoItem.url = '/todo';

// Populate model from server
todoItem.fetch();
```

* fetch goes to the URL we set, pull down the JSON, load that into our model, and then we can call get to get that data out of our model.

### Fetching with a RESTful Route

* When we define our model class we can specify a URL root. By default it will use that route to get data from our server and it’s going to interact with our server in a RESTful way using the Rails flavored version of REST

```
var ToDoItem = Backbone.Model.extend({ urlRoot: '/todos' });
```

### Fetch Specific To Do 

* To fetch a specific instance of an object, just pass in the id of the object when we define the new model instance

```
var todoItem = new ToDoItem({id: 1});
```

* To fetch the data for this to do, we just call fetch, which will do a GET request to ‘/todos/1’ url, it will return the JSON and then populate the instance with that data

```
todoItem.fetch(); --> { id: 1, description: 'Pick up milk', status: 'incomplete' }
```

### Update To Do Item

* To update this model instance we call the set command. 
* Then if we want to synchronize this with our server, we call the save command.
	* This does a PUT request to url ‘todos/1’ with JSON params

```
todoItem.set({ description: 'Pick up cookies' });
todoItem.save();
```

## Creating & Destroying Model Instances

### Creating a new to do item

```
var todoItem = new ToDoItem();
todoItem.set({ description: 'Fill prescription.' });
todoItem.save();
```

* This does a POST request to ‘/todos’ with the JSON parameters. 
* We can see that this to do item was properly created on the server because when we call get id, we get back an id number

```
todoItem.get('id') --> 2
```

### Destroy a to do item

* To destroy a model instance, we just call destroy, which sends a DELETE request to ‘/todos/2’

```
todoItem.destroy();
```

## Get JSON From Model

* If we ever want to get the JSON from our model, for instance to render it on to the view, we just call:

```
todoItem.toJSON(); --> { id: 2, description: 'Fill prescription', status: 'incomplete' }
```

## Models Can Have Events

* One of the unifying concepts of Backbone is the use of events. The first place we see this is at the model layer.

### Listen for Event on Model

```
todoItem.on('event-name', function(){
    alert('event-name happened!');
})
```

### Special Backbone Model Events

* Backbone models already come with special events that will get triggered on their own. 
* The most common one is change, which listens for changes to the model. When that happens, call the function

```
todoItem.on('change', doThing);
```

* If we called set on our to do item, and we changed an attribute, the doThing event will be triggered.

### Set Attribute on Model without Triggering Event

```
todoItem.set({ description: 'Fill prescription'}, 
             {silent: true}
            );
```

### Remove Event Listener

* You can remove an event listener by calling off, instead of on.

```
todoItem.off('change', doThing);
```

### List of Built-In Backbone Events on Models

* change - When an attribute is modified
* change: <attr> - When <attr> is modified (change on specific attribute)
* destroy - When a model is destroyed
* sync - Whenever model is successfully synchronized
* error - When model save or validation fails
* all - When any event is triggered