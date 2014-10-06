# Level 7 - Router & History

## Introduction

**The Problem**

* Next to each to do item we have a link that we want to take us to a page that shows just that one to do item
	* However, we don't want to completely refresh the page
* We can use this bit of jQuery

```
<a href="#" class="todo"></a>

$('a.todo').click(function(e){
	e.preventDefault();
	// show single todo
})
```

* This works, but if we hit the back button to go back to the full list, the site breaks
* Backbone solves this with the router & history

## The Router

* The router's job is to map URLs to actions

### Create new Router

* The routes object maps a URL to a single action

```
var router = new Backbone.Router({
	router: { "todos": "index" },
	index: function(){
	...
	}
});
```

* When the url path is */todos* or *#todos* the index action will be called

### Routes match parameter parts

```
var rotuer = Backbone.Router({
	routes: { "todos/:id": "show" },
	show: function(id){ ... }
});
```

#### More Route Matchers

|  matcher | URL  |  params  |
|---|---|---|
| search/:query  | search/ruby  | query='ruby'  |
| search/:query/p:page  | search/ruby/p2  | query='ruby', page=2  |
| folder/:name-:mode  | folder/foo-r | name='food', mode='r'  |
| file/*path  | file/hello/world.txt  | path='hello/world.txt'  |

* Wildcard (*) matches everything after file/

### Triggering Routes

* We could use the router's navigate function
	* Pass in the URL as the first argument
	* If we want to trigger the action, we pass in trigger: true.
		* If we didn't pass in trigger: true, the browser URL would update, but the action wouldn't be called

```
router.navigate("todos/1", {
	trigger: true
});
```

* Using normal links

```
<a href="#todos/1">link</a>
```

## History

### Introduction

* There are two ways of doing history in Backbone
	* Hashbangs
		* #todos
	* HTML5 push state
		* /todos

### Start Backbone History

* In order to make the router links work, we need to first use the Backbone histoyr API
* pushState is off by default, so we have to pass in an argument to turn it on 

```
// pushState on
Backbone.history.start({pushState: true});
```
### Show Action

Define router class

```
var TodoRouter = Backbone.Router.extend({
	routes: { "todos/:id": "show" },
	show: function(id){
		this.todoList.focusOnTodoItem(id);
	},
	initialize: function(options){
		this.todoList = options.todoList;
	}
});
```
* The function *focusOnToDoItem(id)* resets the collection and only renders the item with the id that matches
* The *initialize* function takes a function with an options object, and will assign the to do list to the router from the options.
	* Now, when we create our to do router instance, we pass in an object with the todoList collection.

```
// Instantiate router instance
var todoList = new TodoList();
var TodoApp = new TodoRouter({todoList: todoList});
```

### Index Action

* We need to define the root route on our router

```
var TodoRouter = Backbone.Router.extend({
	routes: { "": "index",
		       "todos/:id": "show" },
	index: function(){
		this.todoList.fetch();
	},
	show: function(id){
		this.todoList.focusOnTodoItem(id);
	},
	initialize: function(options){
		this.todoList = options.todoList;
	}
});
```

## App Organization

* The router is a great place to organize some of the code that we've been writing
	* Instead of defining a whole new router class, since we're only going to have one router instance, just create the router instance directly

```
var TodoApp = new (Backbone.Router.extend({
	routes: { "": "index", "todos/:id": "show" },
	initialize: function() {
		this.todoList = new TodoList();
		this.todosview = new TodoListView({collection: this.todoList});
		$('#app').append(this.todosView.el);
	},
	start: function(){
		Backbone.history.start({pushState: true});
	},
	index: function(){
		this.todoList.fetch();
	},
	show: function(id){
		this.todoList.focusonTodoItem(id);
	}
}));
	
```

* When we create the router we:
	* define our routes
	* when we initialize our router: 
		* we create a new todoList collection
		* then we create our todoList view and pass in that collection
		* the last thing we need to do is append that collection views element to our DOM
	* We're going to define a new function call start, and all that's going to do is call Backbone.history.start with pushState true.
	* Next, we define our index function
	* Define the show function as well
* The really neat thing about organizing our code in the router this way is that when our page first loads we can use the jQuery ready function and all we have to do is call TodoApp.start and that'll start our whole app, everything will be initialized, and we'll be good to go.

```
$(function(){ TodoApp.start() })
```