+++
date = "2017-09-10T17:48:30-04:00"
title = "JavaScript Design Patterns: Module"
keywords = [ "jpiedra", "javascript", "design", "webdev", "es5", "part 1" ]
tags = [ "javascript", "design", "webdev", "es5", "part 2", "module" ]
+++

In the second installation of this series on JavaScript design patterns and their implementation, we take a look at the <i>module pattern.</i> By combining two commonly used features in the language, we can create robust interfaces that hide member variables and methods as we see fit.

<!--more-->

<h2>The Object Literal, Revisited</h2>

In the previous article for this series, we introduced the object literal. This is a simple container that we use in JavaScript to create an object, with member variables and methods, which is then assigned to a variable and available like so:

<pre>
var person = {
	name: "Jonathan Piedra",
	greeting: function() {
		console.log("Hi, I'm " + this.name);
	};
};

console.log(person.name); 	// "Jonathan Piedra"
person.greeting();			// "Hi, I'm Jonathan Piedra"
</pre>

The object literal helps us organize our code. Member variables and functions are stored and accessed in the variable named <i>person</i>. We can think of <i>person</i> as a rudimentary type of module, as it keeps all pertinent member data available through a name we assign it (through the variable). This is preferable to having the same variables and functions scattered throughout our source code. 

<h2>I Want My Space: Privacy in JavaScript</h2>

For people with a background in other programming languages, such as C++ or Java, something may have jumped out by now in this example. When we talk about member data in most languages, we're also provided with access modifiers through keywords like <i>public</i> and <i>private</i> that specify whether or not we want such member data to be accessed from whatever the "global" context is (for example, in "main" if we're writing C++). 

We don't have such keywords available to us in ES5, the version available on most browsers at the moment. However, data privacy is possible in JavaScript and involves the use of a feature of the language that can take some getting used to at first, but becomes very easy after repeated use. Before we explain that feature, we should understand an important property of functions in JavaScript.

<h2>Functions And Scope</h2>

While the sort of block that gets created when we create object literals (everything between curly brackets) cannot render certain data private, functions in JavaScript do create a scope that - by default - prevents anything declared within the function's body from being accessible anywhere else, as long as variables are declared using the <i>var</i> keyword. Functions within another function's scope are also private:

<pre>
function writeMessage() {
	var secret = "I'm not available.";
	console.log(secret);
};

writeMessage(); 		// "I'm not available"
console.log(secret);	// undefined
</pre>

We don't have a way of accessing the variable <i>secret</i> that was created in the function's body, because it's only accessible from within the scope that was created when we declared this function. However, anything <i>else that was created or included in the function declaration</i> will have access to the variables within.

We can get a little closer to privacy in an object. Writing a function that returns an object seems possible. The object, because it was declared inside of a function, will continue to have access to whatever other variables were set inside the function. 

<pre>
function buildModule() {
	var privateName = "James Bond";

	function privateMethod() {
		console.log(privateName);
	};

	return {
		publicMethod: privateMethod
	};
};

var obj = buildModule();	// buildModule returns an object stored in obj.
obj.publicMethod();			// its only member is a reference to private, function-scoped 'privateMethod'.

console.log(privateName);	// still can't access this (it only exists in the function's scope)

</pre>

This gives us part of the solution. While we now know of a way to achieve data privacy, we need a way of combining this with object creation in order to make it really useful. Enter the IIFE.

<h2>Immediately Invoked Function Expression (IIFE)</h2>

Let's return for a second to the concept of a module. We need a way of declaring an object that provides access to member variables and data as desired, once we define the module. We would prefer to create, then store this object in a variable, like we did with <i>person</i>, for later use in our source code.

The most common way to immediately create and store such an object, store it into a variable, is through <a href="http://benalman.com/news/2010/11/immediately-invoked-function-expression/">the IIFE.</a> An IIFE is a type of function that is written using a specific form to both declare, then immediately call it, after declaration. This is in contrast with the typical way of creating a function, which we saw above with <i>writeMessage</i>. 

<pre>
// declared, then immediately called
var obj = (function(){
	var privateName = "James Bond";

	function privateMethod() {
		console.log(privateName);
	};

	return {
		publicMethod: privateMethod
	};
})();

obj.publicMethod();
</pre>

Instead of declaring our function, and then calling it - returning an object and storing it in <i>obj</i> - we write an IIFE. Everything wrapped inside the first pair of parentheses is called immediately. This results in everything declared inside the anonymous function happening right away, with the resulting object returned and stored in <i>obj</i>. Afterwards, we use the the object as our module, calling its methods as in the previous example.

Other than the fact that we use an IIFE to immediately call the logic contained in the function, nothing else has changed with respect to the module object. It still has a single method, a reference to <i>privateMethod</i> which is otherwise not accessible from the global context, and that single method lets us print out the contents of <i>privateName</i>, another variable that is only available inside the function's (IIFE's) scope. 

<h2>Tying It Together: A Trivial UI</h2>

So we now know an IIFE can be used in JavaScript to setup data privacy inside of a module. What generally happens, is that we set up an IIFE that returns an object, storing that object in a variable in the global context. The object returned is our module, and it can have tightly controlled access to "private" vars declared inside of the IIFE's function body, as well as functions. These vars and functions end up being the private members of the module: we make them public by adding a reference to them in the returned object. We did just that in the previous example, by creating a key called <i>publicMethod</i> which is a reference to the IIFE-scoped function <i>privateMethod</i>.

Let's wrap this article up by providing an example somewhat closer to reality. We'll show a module that sets up a very simple UI based on whether the user of our module provides valid parameters. If so, then the method <i>setup</i> will successfully add an event listener to the element we provided the module. When clicked, the element we passed in will be set to execute a function that is "private," otherwise inaccessible beyond the IIFE's scope: 

<script async src="//jsfiddle.net/aqd3c8o7/3/embed/js,html,result/"></script>

In a future post, I'll demonstrate a more complete version of a UI module that I developed. It's very similar to what's demonstrated here, with the IIFE containing workhorse functions that are either exposed through the returned object, or called conditionally based on whether the user initialized the module with valid values.  

Combining object literals with the privacy introduced by functions, and IIFE's, is a valuable way to create modules with public and private member variables and methods. These same techniques will be useful in a later post, where the singleton pattern is explained as a way of ensuring only one instance of an object exists for use at a given time.

<i>The subject matter here is loosely adapted from, and greatly informed by Addy Osmani's in-depth book, <a href="https://addyosmani.com/resources/essentialjsdesignpatterns/book/"><b>Learning JavaScript Design Patterns</b></a>. Check out the book for great explanations on this and other design patterns.</i>
