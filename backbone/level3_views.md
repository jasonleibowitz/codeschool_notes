# Level 3 - Views

## More on the View Element

### Default top-level element

* The default top level element is div

```
// Define simple View class
var SimpleView = Backbone.View.extend({});

// Define instance of this class
var simpleView = new SimpleView();

// If we print out the view's el, we'll just get a blank div
console.log(simpleView.el); --> <div></div>
```

### Change the view's top-level element

* If we want to change a view's top level element, we can specify the tag name when we define the class. 
* We can change the tag name to any HTML element.

```
var SimpleView = Backbone.extend({tagName: 'li'});
var simpleView = new SimpleView();

console.log(simpleView.el) --> <li></li>
```

### Back to To Do

```
var ToDoView = Backbone.View.extend({
    tagName: 'article',
    id: 'todo-view',
    className: 'todo'
});

var todoView = new ToDoView();

console.log(todoView.el) --> <article id="todo-view" class="todo"></article>
```

### Get HTML from To Do View

* Using jQuery we could do

```
$('#todo-view').html();
```

* However, there’s a better way to do it in Backbone. 
* That el is a DOM element, so we can wrap that in a jQuery function and simply call .html

```
$(todoView.el).html();
```

* Backbone even gives us a shortcut on top of this

```
todoView.$el.html();
```

* This is good because we don’t know what the id of the el will be. The construct out of our view could be dynamic, so it’s much better to simply use this way.

## Templating

```
var ToDoView = Backbone.View.extend({
    tagName: 'article',
    id: 'todo-view',
    class: 'todo',
    render: function(){
        var html = '<h3>' + this.model.get('description') + '</h3>';
        this.$el.html(html);
    }
});

var todoView = new ToDoView();
todoView.render();
console.log(todoView.el); -- > 

// <article id="todo-view" class="todo">
//      <h3>Pick up milk</h3>
// </article>
```

### Templating with Underscore

* In the above example, we’re generating our view HTML by appending it all together. If we had a lot of HTML and a lot of attributes that we were printing out inside of it, it would get really messy.
* We can organize that by using some sort of client-side templating framework
	* Backbone comes with the underscore library for doing this.
* We set the template with _.template and inside of our render function we’re going to create the attributes variable, setting that equal to the JSON from our model
* To render the template, we just call the template function and send in those attributes.
	* If we render the following into the console, like before, we’ll get the same result, this time using a template

```
var ToDoView = Backbone.View.extend({
    ...
    template: _.template('<h3><%= description %></h3>'),
    render: function(){
        var attributes = this.model.toJSON();
        this.$el.html(this.template(attributes));
    }
});
```

### Alternatives to Backbone Templating

* Event though Backbone comes with Underscore, we have other options for templating

```
// Underscore
<h3><%= description %></h3>

// Mustache
<h3>{{description}}</h3>

// Haml-js
%h3= description

// Eco
<h3><%= description %></h3>
```

## Event Listeners on Views

### Add Events to Views

* We want to add a click event, so that whenever someone clicks on the h3, it pops up an alert. 
* If we were to do this in jQuery, it might look like this:

```
$("h3").click(alertStatus);
function alertStatus(){
    alert('Hey you clicked the h3!');
}
```

* However, this is NOT how we would do DOM events in Backbone.
* In Backbone, views are responsible for responding to any user interaction.
	* So these events are defined inside of our view

```
var ToDoView = Backbone.View.extend({
    events: {
        "click h3": "alertStatus"
    },
    alertStatus: function(e){
        alert('Hey you clicked the h3!');
    }
});
```

* This alert will not pop up on any h3 on the page. Backbone is smart enough to make the h3 event scoped to inside of this particular element. 
	* It’s actually calling the jQuery delegate method. 
	* So that means inside of this element, if there’s any h3s, when they’re clicked, call the alertStatus
* Here’s what it looks like:

```
// Selector is scoped to the el
this.$el.delegate('h3', 'click', alertStatus);
```

### Views Can Have Many Events

```
var DocumentView = Backbone.View.extend({
    events: {
        "dblclick": "open", 
        "click .icon.doc": "select",
        "click .show_notes": "toggleNotes",
        "click .title .lock": "editAccessLevel",
        "mouseover .title .date": "showTooltip"
    },
});
```

* With the dblclick event we haven’t defined a selector, so if we double click anywhere within the element, it will call the open function

### View Events We Can Listen To (jQuery)

* change
* click
* dblclick 
* focus 
* focusin
* focusout
* hover
* keydown
* keypress
* load
* mousedown
* mouseenter
* mouseleave
* mosuemove
* mouseout
* mouseover
* mouseup
* ready
* resize
* scroll
* select
* unload

* You can always create custom events on the model or view level