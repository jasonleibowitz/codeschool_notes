# Level 4 - Models & Views

## How Models and Views Work Together

### Adding a Checkbox to our View

* We would do this by changin gthe template and adding an input of type checkbox, and we’re also going to make sure if the status of our to do item is complete, we’ll print check, so our checkbox is checked out. 
* We’ll finish off with putting our description there like before.

```
var ToDoView = View.Backbone.extend({
    template = _.template('<h3>' + 
        '<input type=checkbox ' +
        '<% if (status === "complete") print ("checked") %/>' + 
                                    '<%= description %></h3>'),
    render: function(){
        this.$el.html(this.template(this.model.toJSON()));
    }
});
```

### View Events Update the Model

#### How do we update our model when the checkbox changes?

* Our models provide the views with the data, and it’s the views responsiblity to build the HTML
* We can also go the other way:
	* When a DOM event happens, it’s the view’s repsonsiblity to handle that and update the model. 
	* Now that the model has the new data, it can update the server
* So we’re going to define a new event on the ToDoView class

```
// Whenever an input inside of this view changes, we want to call the toggleStatus function
var ToDoView = Backbone.View.extend({
    events: {
        'change input': 'toggleStatus'
    },
    toggleStatus: function(){
        if(this.model.get('status') === 'incomplete'){
            this.model.set({'status': 'complete'});
        } else {
            this.model.set({'status': 'incomplete'});
        }
    }
});
```

* Note: This is not the best code because we have a lot of model logic in our view.
	* Fix with refactoring. Have the toggleStatus function call a model function of the same name
	* This is good because our model logic is in our model, and we can call the model’s toggleStatus function from other places in the app and it would do the same thing

#### Separation of Concerns

```
var ToDoView = Backbone.View.extend({
    events: {
        'change input': 'toggleStatus'
    },
    toggleStatus: function(){
        this.model.toggleStatus();
    }
});

var ToDoItem = Backbone.Model.extend({
    toggleStatus: function(){
        if(this.get('status') === 'incomplete'){
            this.set({'status': 'complete'});
        } else {
            this.set({'status': 'incomplete'});
        }
        // we can sync any changes to the server by putting a save call here
        this.save();
    }
});
```

### Sync Changes to Server

* We could sync changes to the model by putting a save call in the model, which would do a PUT request to ‘todos/:id'

### Whenever our status is complete, add strikethrough

* We want to add class .complete to any to do item that has the status complete.
* To do this we need to change our template

```
template: _.template('<h3 class="<%= status %>">' + 
                     '<% if(status === "complete") print("checked") %/>' + 
                     ' <%= description %></h3>')
```

* Here, the class of the h3 will be set to whatever the model status is
	* Now, whenever our model status is set to complete, our new model’s style will be updated correctly.
* But, how do we update the view when the model changes? 
	* We could go back into the view’s toggleStatus function and just call render() after we’ve toggled the model’s status, and that will do what we want.
		* But, what would happen if our model changed somewhere else in our app, like in a sidebar button that checked all of our items off? This render wouldn’t work. 
	* We need some way to notify our view whenever the model changes, and re-render. 
		* We can use model events to notify the view whenever the model changes and then our view can re render and update the DOM

### Using Initialize on View

```
var ToDoView = Backbone.View.extend({
    events: {
        'change input': 'toggleStatus'
    },
    initialize: function(){
        this.model.on('change', this.render, this);
    }
    toggleStatus: function() {
        this.model.toggleStatus();
    },
    render: function(){
        this.$el.html(this.template(this.model.toJSON()));
    }
});
```

* The initialize function will be called whenever a new instance of the to do view is created
	* Inside of this function we can listen for any changes on the model, and when that change event is triggered, we call the render function
* What is that 3rd argument we’re passing in, the this? 
	* this refers to the context of the function as it is called
	* In the initialize function, we’re going to want the this to point to our to do view instance
	* Usually this isn’t a problem, but because of the way we listen to the change event on the model, when that change event is triggered, the render function will be called with the incorrect context. In fact, it will be called with the window context, which is the global object.
		* So in our render function, this will equal window, which is obviously not what we want
		* When we add that 3rd argument, it tells Backbone that when it triggers the change event and calls the function it should set the context of the render function to the to do view instance.  

### Remove View on Model Destroy

* Just like we listen to the model’s change event and we re-rendered the view, we also want to listen to the model’s destroy event and remove the view from the document
* To make this work, we have to define the remove function, and all that does is remove the element from the view

```
var ToDoView = Backbone.View.extend({
    initialize: function(){
        this.model.on('change', this.render, this);
        this.model.on('destroy', this.remove, this);
    }, 
    render: function(){
        this.$el.html(this.template(this.model.toJSON()));
    },
    remove: function(){
        this.$el.remove();
    }
});
```