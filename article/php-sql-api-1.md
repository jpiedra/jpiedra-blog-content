+++
date = "2017-01-15T13:20:54-05:00"
title = "Web API, written using PHP and MySQL - Part 1"
keywords = [ "jpiedra", "php", "scripting", "mysql", "api", "part 1" ]
tags = [ "php", "scripting", "mysql", "api", "part 1" ]
+++

In this first part of a series on implementing a PHP/MySQL-based API endpoint for our data, we go over some basic principles, as well as stated goals, that will guide the work we'll be undertaking later.

<!--more-->
<h2>API, defined</h2>

An <i>application programming interface</i>, or API, can be generally defined as a set of routines/sub-routines made available to us. We use these routines in our own applications as defined by the API's documentation, with little concern about <i>how</i> the routines do what they do for us. The motivation, then, behind using an API can be any of the following:

<ul>
	<li>We need a well-defined and well-tested handle to some functionality (creating/closing a file stream, for example)</li>
	<li>We need to write a script/program that performs some simple task, without completely reinventing the wheel (use somebody else's library)</li>
	<li>We need access to data, and have <i>no other way</i> to access said data than through the set of routines made available to us</li>
</ul>

<h2>Web API</h2>

That last bullet point will be the most relevant motivation behind API construction/use in our case. We'll be developing a <a href="https://en.wikipedia.org/wiki/Web_API"><i>web API</i></a>. In many cases where we develop a front-end application on the web, the first step in planning this application will involve developing, or deciding on an existing web API to use. 

As the name suggests, a web API lives on the web. Thus, the context for such an API would be web servers, and the exposed routines will be accessible as universal resource indicators (URI's) that can - sometimes - be accessed directly via a web browser. In a hypothetical API that gives us access to a company's employee data, the URI for getting the ten latest employees might be <b>api.acme.org/employees/10</b>. (such URI's are often referred to as <i>endpoints</i>)

Compare the previous explanation with a tradtional, software API, such as the <a href="https://en.wikipedia.org/wiki/Berkeley_sockets">Berkeley sockets API</a> used in *nix operating systems. Simply put, a socket is a data structure that is used to manage and establish TCP/UDP connections and communication between networked devices. An example of a routine exposed through the sockets API is <b>gethostbyname()</b>, which "resolves host names and addresses" based on the information provided to this routine. Just as we get the latest ten employees from the above URI example, using gethostbyname() would return something we expect based on documenation - a NULL pointer in case of an error, or a valid structure corresponding to the detected host.

Even though the contexts for each are markedly different (web server hosting some application vs. low-level operating system handles), the general principles can be observed: we use a handle associated with some routine or process, and get something back if all goes well.

<h2>An Example To Follow</h2>
An example of a web API that I've used before is the <a href="http://docs.brightcove.com/en/video-cloud/media/guides/search_videos-guide.html">Brightcove Media API</a>. Though currently deprecated, it provides a clear example of a well-defined API we will use to guide the creation of a new one.

<ul>
	<li>An endpoint we can access via JavaScript AJAX calls</li>
	<li><a href="https://en.wikipedia.org/wiki/Query_string">Query strings</a> are appended to the endpoint, and allow us to make granular requests for data</li>
	<li>The requested data is returned to us in JavaScript Object Notation (JSON), a widely-supported format easily adapted to web applications</li>
</ul>

Putting this all together, we would make a request such as 
<pre>api.brightcove.com/services/library?command=search_videos&page_size=3&video_fields=id,name</pre> 
and get back a JSON object, with a bunch of nested objects for each single video that matches the request. The start of our query string is the ? mark, and what follows is a query string where each field/value pair is separated by the & symbol. A field/value pair consists of the left-hand value (an option that impacts the request we're making), an = sign, and the right-hand value that defines the option preceding it in some way.

Which video fields do we want for each video object we're returned? Well, note the last field/value set, where <b>video_fields</b> is followed by a comma-separated list of fields - <b>id,name</b>. This type of form for writing requests, gives us a compact way of expressing granular requests in a single statement. We can imagine the series of comma-separated fields as corresponding to the columns of a database table. The JSON video objects we get back, then, would correspond to rows that reside in that table.

<h2>Summary</h2>

With this explanation of <i>what</i> a web API does and looks like, we'll move on to our next article in the series by describing an example API we'll create using PHP and MySQL for the programming language that powers the API, and the database system that stores our information, respectively. We'll also go over some general features or details of PHP/MySQL that we'll need to know in order to write the API.