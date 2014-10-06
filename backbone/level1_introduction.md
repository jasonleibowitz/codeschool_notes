# Level 1 - Introduction

## Why Use Backbone

* If we have an application with a lot of methods and functionality, it can be disorganized. 
* We could lose the data structure inside the DOM
	* Don't lose your data inside the DOM when you should instead abstract it into a model
* Backbone provides:
	* client-side app structure for our functions, so we can keep them organized
	* provides models to representd data structures inside of our views
	* views to hook up models into the DOM
	* takes care of synchronizes data to/from server
	
## Models

### How to Create a Model Class

```
var TodoItem = Backbone.Model.extend({});
```

**Notes:** 

* Pay attention to capitalization. We are going to capitalize the first letter of each word when we’re declaring a class
* The first letter when we’re declaring an instance is lower-case

### Set Default Values on Class

```
var ToDoItem = Backbone.Model.extend({
    defaults: {
        description: 'Empty todo...',
        status: 'incomplete'
    }
});
```

### Create Model Instance from our Class

```
var todoItem = new ToDoItem(
    { description: 'Pick up milk', status: 'incomplete', id: 1 }
    );
```

### Getting and Setting Data in Model

```
// Get an attribute
todoItem.get('description'); --> 'Pick up Milk'

// Set an attribute
todoItem.set({status: 'complete'});

// Sync data to server
todoItem.save();
```

## Review: How Backbone Displays the Data

* The server sends data to our Backbone models.
* Our backbone models provide the data to views
* It’s the view’s responsibility to build the HTML, which we will then put back into the DOM

## Views

### Create a View Class

```
var ToDoView = Backbone.View.extend({});
```

### Create a View Instance

```
var todoView = new ToDoView({ model: todoItem });
```

Here we are creating a new todo view instance of the ToDoView class, and we are going to send in the todoItem model instance we previously created.

### Rendering the View

We need our view to render out some HTML for our to do item. To do that we need to define the render function on the view class.

```
var ToDoView = Backbone.View.extend({
    render: function(){
        var html = '<h3>' + this.model.get('description') + '</h3>';
        $(this.el).html(html);
    }
});
```

* Inside the render function we get our description out of the model and create some HTML. 
* We’re then going to set the HTML of that view element. 
	* Every view instance has it’s own view element (el)
	* A single instance is associated with and element, which means it’s an HTML tag of some sort. It could be a div, which it is by default, but it could also be a paragraph, li, etc. 
* When we instantiate the view, and we call the render function, we can then print out the resulting HTML to the console

```
var todoView = new ToDoView({ model: todoItem });
todoView.render();
console.log(todoView.el); -->

    // <div> 
    //     <h3>Pick up Milk</h3>
    // </div>
```