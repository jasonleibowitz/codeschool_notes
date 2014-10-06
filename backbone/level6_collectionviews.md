# Level 6 - Collection Views

## Introduction

* Collection Views enable us to show collections to the user
	* Collection + View = Collection View

## Review on Model Views

```
var TodoView = Backbone.View.extend({
	render: function(){
		this.$el.html(this.template(this.model.toJSON()));
		return this;
	}
	...
});
var todoItem = new TodoItem();
var todoView = new TodoView({model: todoItem});
console.log(todoView.render().el);
-->
<div>
	<h3>Pick up milk</h3>
</div>
```

## How Collection Views Differ from Model Views

* Models and views have a 1-to-1 relationship
	* Models have one view, and views have one model
* Not the case with collection views
	* Collection view is 1-to-many
	* One collection view has many models
	* As it has many models, it will also have many views. That's because the collection view delegates to the model view to render the HTML
	* The collection view doesn't render any of its own HTML
	
## Creating Collection Views

###  Define and Render Collection View

**Defining**

* Create to do list and an instance of that view, but instead of passing in a model, we pass in a collection.

```
var TodoListView = Backbone.View.extend({});
var todoListView = new TodoListView({collection: todoList});
```

**Rendering**

* First crack at render function
	* Go through each model in the collection with a forEach loop
	* Inside of the forEach loop create a to do view for each model
	* Then we're going to render each to do view and take that view's element and append it to our collection view's element

```
render: function(){
	this.collection.forEach(function(todoItem){
		var todoView = new TodoView({model: todoItem});
		this.$el.append(todoView.render().el);
	});
}
```

* Problem: The context is wrong. Whenever this function is called inside a forEach, it's going to have the wrong context.
* Fix
	* Add a new function called *addOne*. This will do what our forEach loop was doing before.
	* Update the render function and pass the *addOne* function for Each. We also have to add a second argument to make sure the context of *addOne* is called on the view. 

```
render: function(){
	this.collection.forEach(this.addOne, this);
},
addOne: function(todoItem){
	var todoView = new TodoView({model: todoItem});
	this.$el.append(todoView.render().el);
}
```

```
var todoListView = new TodoListView({collection: todoList});
todoListView.render();
console.log(todoListView.el);

-->

<div>
	<h3 class='incomplete'><input type="checkbox" />
	Pick up milk
	</h3>
	
	<h3 class='complete'><input type="checkbox" />
	Learn backbone.
	</h3>
</div>
```

## Adding new Models

* Add an initialize function to the collection view
	* The initialize function listens to an add event, and whenever that add event is triggered we'll call the *addOne* function

```
initialize: function(){
	this.collection.on('add', this.addOne, this);
},
addOne: function(todoItem){
	var todoView = new TodoView({model: todoItem});
	this.$el.append(todoView.render().el);
},
render: function(){
	this.collection.forEach(this.addOne, this);
}
```

## Reset Event

* Just like adding a new item to the collection, at first it didn't update our DOM. Right now when we fetch a bunch of new items from the server our DOM won't be updated.
	* When we call fetch on the collection, it's going to fire the reset event.
* We can listen to this event in the collection view and call this new function *addAll* when this event is fired
	* *addAll* is doing pretty much what render was doing before
* Then update render to use the addAll function

```
initialize: function(){
	this.collection.on('add', this.addOne, this);
	this.collection.on('reset', this.addAll, this);
},
addOne: function(todoItem){
	var todoView = new TodoView({model: todoItem});
	this.$el.append(todoView.render().el);
},
addAll: function(){
	this.collection.forEach(this.addOne, this);
},
render: function(){
	this.addAll();
}
```

## Fixing remove with Custom Events

* To fix this, we dont' actually have to touch the collection view
	* When we remove an item from a collection, the remove event is triggered
	* When we remove an item from a collection, we don't actually destroy the item. If we do destroy it, it would have removed it from the view, but that's not what we want.
	* We need to use custom events
* Define a hideModel function that triggers a custom event called hide on that model
* On TodoItem View listen for that hide event, and whenever it is triggered, remove the view from the DOM.

```
// TodoList Collection
initialize: function(){
	this.on('remove', this.hideModel);
},
hideModel: function(model){
	model.trigger('hide');
}

// TodoItem View
initialize: function(){
	this.model.on('hide', this.remove, this);
}
```