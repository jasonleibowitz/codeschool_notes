# Level 5 - Collections

## Creating Collections

* Collections help us handle set of models

### Define a Collection

```
var ToDoList = Backbone.Collection.extend({
	model: ToDoItem
});
```

* Here, we are defining a to do list collection class, just like we've done with models and views. 
	* The only thing we have to do is specify which model the collection is managing.
* Then we can create a new instance of the to do list collection

```
var todoList = new ToDolist();
```

* Todolist manages a set of TodoItem model instances. 

## Interacting with Collection Instances

### Add/Remove/Get

**Get Number of Models inside**

```
todoList.length();
```

**Add a model instnace**

```
todoList.add(todoItem1);
```

**Get a model instance at a certain index**

```
todoList.at(0);
```

**Get a model instance by id**

```
todoList.get(1);
```

**Remove a model instance**

```
todoList.remove(todoItem1);
```

### Bulk Population

* Collections have a reset function, where all we have to do is pass in an array of objects and each one of those objects will become a to do item.

```
var todos = [
	{description: 'Pick up milk', status: 'incomplete'},
	{description: 'Get a car wash', status: 'incomplete'},
	{description: 'Learn backbone', status: 'incomplete'}
];

todoList.reset(todos);
```

	* Now our to do list has 3 items in it

### Fetching data from the server

* Populate to do list collection with data from the server
	* First, specify a URL when creating the to do list class, so when we call the fetch function from the to do list collection backbone will make a GET to to the todos URL and populate the collection with data from the server.

```
var ToDoList = Backbone.Collection.extend({
	model: ToDoItem,
	url: '/todos'
});

var todoList = new ToDoList();

todoList.fetch();
```

## Collections Can Have Events

* Just like we can listen to events on a model, we can also listen to events on our collection instance

```
todoList.on('event-name', function(){
	alert('event-name happened');
});
```

### Special Collection Events

* Listen for Reset
	* Triggered whenever fetch *or* reset is called

```todoList.on('reset', doThing);```

* Run event without triggering event
	* Pass in silent: true
	
```todoList.fetch({silent: true});```

* Remove the event listener

```todoList.off('reset', doThing);```

### Built-In Collection Events

* add - whenever a model is added to the collection
* remove - whenever a model is removed from the collection
* reset - whenever the collection is reset or fetched

```
todoList.on('add', function(todoItem){
	...
});
```

* There is a special case with add and remove, when the function is called, it passes in the model that was either added or removed

### Model Events

* On collections, we can also listen for events triggered on models
	* For instance, we can listen to the change event on any model that is inside of the collection
* List of model events:

	* change - when an attribute is modified
	* change:<attr> - when <attr> is modified
	* destory - when a model is destroyed
	* sync - when successfully synced
	* error - when an atrtibute is modified
	* all - when an attribute is modified

* Events triggered on a model in a collection will also be triggered on the collection

## Iteration on Collection

Setup our collection 

```
todoList.reset([
	{description: 'Pick up milk', status: 'incomplete', id: 1},
	{description: 'Get a car wash', status: 'incomplete', id: 2}
]);
```

Alert each model's description
* This function will be called for each item in the collection

```
todoList.forEach(function(todoItem){
	alert(todoItem.get('description'));
});
```

* Backbone provides many different iteration functions
	* forEach - iterates over every item and runs the function
	* map - like for each, it takes a funciton and will be called with each item in the collection. However, this time it will build a new array based off the return items of that function

	```
	todoList.map(function(todoItem){
		return todoItem.get('description');
	});
	
	--> ['Pick up milk', 'Get a car wash']
	```
	* filter - create a new array with only the items that return true from the function

	```
	todoList.filter(function(todoItem){
		return todoItem.get('status') === 'incomplete';
	});
	```
* Other iteration function (provided by Underscore - [more info](http://documentcloud.github.io/backbone/#Collection))
	* forEach
	* find
	* every
	* include
	* min
	* sortedIndex
	* size
	* rest
	* indexOf
	* chain
	* reduce
	* filter
	* all
	* invoke
	* sortBy
	* shuffle
	* first
	* last
	* lastIndexOf
	* reduceRight
	* reject
	* some
	* max
	* groupBy
	* toArray
	* initial
	* without
	* isEmpty