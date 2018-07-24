+++
title = "CSS Tip: 'overflow-x' for Narrow Contexts"
date = 2018-02-14T21:01:46-05:00
draft = false
keywords = [ "jpiedra", "development", "webdev", "css", "tips", "overflow-x" ]
tags = [ "development", "webdev", "css", "tips" ]
+++

A quick tip that can help handle design cases that include child elements larger than their immediate parent.

<!--more-->

I recently came across a problem that may be familiar to anyone who has tried to display a table full of data in a responsive view. You may think a library like Bootstrap will always scale elements gracefully to make sure your tables don't break, but this isn't always the case: 

<a href="https://imgur.com/TZD8SNG"><img src="https://i.imgur.com/TZD8SNG.png" title="source: imgur.com" style="max-width: 60%;"/></a>

The problem seen here lies in the face that the table nested inside of a parent <i>div</i> element is wider than said parent. In the case of the above layout, the <i>table</i> has rows that will only get as small as 447 pixels wide, while the <i>div</i> can shrink well below that boundary. 

Depending on the needs of your user interface, it may suffice to simply have a user scroll over to see the rest of the table if the screen they view your site or app on grows so small, it can no longer contain its wider, inner element(s). With the application of the CSS property <b>overflow-x</b> this can be accomplished easily. 

<b>An Interactive Example</b>

The below jsfiddle demonstrates what this property does. We apply the property to a container element, setting the value to 'scroll'. This causes a scrollbar to appear, indicating that the inner element (along with its table) can have the rest of its overflowing content shown by scrolling to the right. 

<script async src="//jsfiddle.net/jlpiedra90/0Lhcw26k/26/embed/html,css,result/"></script>

