+++
date = "2017-02-02T11:39:57-05:00"
title = "JavaScript Design Patterns: Constructor"
keywords = [ "jpiedra", "javascript", "design", "webdev", "es5", "part 1" ]
tags = [ "javascript", "design", "webdev", "es5", "part 1", "constructor" ]
+++

This first installment in a series discussing various JavaScript design patterns covers one of the most basic patterns available, the <i>constructor pattern.</i>

<!--more-->

<h2>Object Literals</h2>

Those of us who write JavaScript are familiar with the fastest way of creating an object, assinging an <i>object literal</i> to a variable:

<pre>
var person = {
	name: "Jonathan Piedra",
	greeting: function() {
		console.log("Hi, I'm " + this.name);
	};
};
</pre>

While simple to grasp and convenient enough for a small application or script, this won't do as soon as we have a need for a programmatic method of creating similar objects with different information.

<h2>The Constructor</h2>

Fortunately, in JavaScript, we do have a method of object construction that hews closer to what one would encounter in C++ or Python. It involves defining a special type of function that follows these conventions:

<ul>
	<li>Name of the function corresponds with the capitalized name of the object (examples: Animal, Person, Car) of which we want to create unique instances</li>
	<li>We specify parameters to this function, generally used to set the properties that an instance created using the function should have</li>
	<li>The <i>this</i> keyword, similar to how we use it in an object literal, is used to refer to properties that will be unique to each instance</li>
	<li>Finally, methods are defined on the <i>prototype</i> level! (more on this later)</li>
</ul>

Let's put this all together with an example that will create Person objects:

<pre>
// a
function Person(name) {
	this.name = name;	
};

// b
Person.prototype.greeting = function() {
	console.log("Hi, I'm " + this.name);	
};

// c
var jon = new Person("Jonathan Piedra");
jon.greeting();
// Hi, I'm Jonathan Piedra
</pre>

Segment A consists of the function that manages object creation. We name it Person, as each instance created using this function should be a Person object. A single parameter, <i>name</i>, is specified; thus, when we create instances using this function (Segment C) we have to pass in one argument, that's used to assign the instance's name property, marked using the <i>this</i> keyword.

Segment B defines a method for <i>all Person objects</i>. The importance of defining object methods in this way is explained in detail below; for now, all we need to know is that using this approach is considered a best practice. We ensure a single definition for the method <i>greeting()</i> exists.

Finally, in Segment C, we use the constructor method defined in Segment A. Again, similar to the process used in C++ or other C-like languages, the <i>new</i> keyword is used, and the function is called with a single string literal passed in. This argument, as we see in the first/only line of the <i>Person</i> method, is used to set the <i>name</i> property that belongs to our new instance, <i>jon</i>.

<h2>Why Not Inline Methods?</h2>

Let's return our attention to Segment B. Though we decided to define <i>greeting()</i> outside of the function body for the Person constructor, there's nothing stopping us from refactoring the constructor like so: 

<pre>
function Person(name) {
	this.name = name;
	this.greeting = function() {
		console.log("Hi, I'm " + this.name);
	};
};
</pre>

However, it's important to understand why this is discouraged. When creating a constructor, anything defined <i>inline</i>, or inside the function body, using the <i>this</i> keyword will become a property or method that each instance has a unique copy of. In effect, this results in every instance of Person having their "own" version of <i>greeting</i>. 

For properties that describe unique attributes, like a name or ID, this is what we want! However, for a method that provides functionality that all instances of an object have in common, this presents problems. The <i>greeting()</i> method should do the same thing regardless of <i>which Person object</i> it is being called from; additionally, if we ever want to change what that method does, then doing that when every object instance has it's own copy of the method will prove unnecessarily difficult.

<h2>Prototypal Inheritance</h2>

The best practice for defining methods used by constructed objects, involves defining those methods on the Prototype level. In JavaScript, objects are defined along a prototypal chain. What this means, is that all objects - objects we define as literals, through constructors, as well as primitive types like Object and even Array - reside somewhere on a prototype "chain." At the top of this chain would be Object, the primitive JavaScript type; at the bottom, custom objects like Person which a programmer defines.

Inheritance in JavaScript involves an object we create inheriting the methods and properties we define somewhere along the chain (through a previous custom object), as well as whatever methods and properties are defined by objects that reside earlier along that chain (such as primitive types). This means that even a Person instance, like any custom type, will inherit the methods defined in Object's prototype, because it is defined before any other object (it is, after all, the base from which any JavaScript object derives). 

This <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object">Mozilla Developer Network page</a> describes the Object primitive in greater detail. We see that, for example, the property <i>constructor</i> is defined through the Object prototype - thus, referred to as <i>Object.prototype.constructor</i>. If Person resides on the prototype chain, and comes well after the Object prototype, it follows that Person objects will also have inherited <i>constructor</i> from Object's prototype (which specifies the constructor used to create the object):

<pre>
...
jon.constructor
// function Person()
</pre>

JavaScript wouldn't be as useful if we weren't able to define our own, Prototype-level methods and properties that are automatically available to objects we create. Of course, we are able to do this for custom object types, and the form for doing so is:

<pre>
MyObject.prototype.MyFunction = function() { ... };
</pre>

Where <i>MyObject</i> and <i>MyFunction</i> can be identifiers of our choosing. Defining methods for objects this way ensures that any object we create of type MyObject (or that <i>extends</i> from MyObject) will have access to MyFunction. However, this also results in <i>one definition for the function, MyFunction()</i> being created; this definition is available to all objects we create of type MyObject. Instead of having their own copies, they just point to the Prototype definition of the method they inherited.

<h2>Person Constructor, Revisited</h2>

Turning back to our custom Person constructor, the decision to define <i>greeting()</i> on the prototype level should make more sense now. To further cement this knowledge, let's see what happens when we don't follow the aforementioned practice:

<pre>
function Person(name) {
	this.name = name;
	this.greeting = function() {
		console.log("Hi, I'm " + this.name);
	};
};

var jon = new Person("Jonathan");
var rob = new Person("Rob");

jon.greeting(); // Hi, I'm Jonathan
rob.greeting(); // Hi, I'm Rob
</pre>

It's worth mentioning that, given how we've defined our constructor, the properties and methods for Person objects are mutable; we can change their values. Being able to change the name property might not be so bad, because at most it will only impact that particular Person's name. For methods, though, the implication is that now our instances of a particular type (Person) will no longer provide reliable functionality; if it's possible to change what <i>greeting()</i> does for each instance, we can't reliably expect calling that method will do what we need it to:

<pre>
jon.greeting = function() {
	console.log("I am NOT " + this.name);
};

jon.greeting(); // I am NOT Jonathan
rob.greeting(); // Hi, I'm Rob
</pre>

In contrast, Prototypal method definition ensures against such a scenario. Any changes to the Prototype's methods will be reflected in all of the object's instances:

<pre>
function Person(name) {
	this.name = name;	
};

Person.prototype.greeting = function() {
	console.log("Hi, I'm " + this.name);	
};

var jon = new Person("Jonathan");
var rob = new Person("Rob");

jon.greeting(); // Hi, I'm Jonathan
rob.greeting(); // Hi, I'm Rob

// Oh, we need to change greeting...
Person.prototype.greeting = function() {
	console.log("My name is " + this.name);
};

jon.greeting(); // My name is Jonathan
rob.greeting(); // My name is Rob
</pre>

Using the Prototypal approach to method definition, we ensure that changes made to common functionality will be reflected across all objects of a type, reducing the likelihood of errors as well as headaches.

<i>The subject matter here is loosely adapted from, and greatly informed by Addy Osmani's in-depth book, <a href="https://addyosmani.com/resources/essentialjsdesignpatterns/book/"><b>Learning JavaScript Design Patterns</b></a>. Check out the book for great explanations on this and other design patterns.</i>
