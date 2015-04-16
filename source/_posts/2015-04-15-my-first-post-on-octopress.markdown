---
layout: post
title: "My First Post on Octopress"
date: 2015-04-15 16:50:22 -0400
comments: true
categories: [Flatiron School, First]
---

Or-Equals (||= also known as "pipes-equals") is one of my favorite Ruby expressions I've encountered so far. It strikes me as sort of polite and considerate--not as pushy and uncompromising as other Ruby methods. Plus it comes in handy all the time. Let me explain how it works and why it can be so useful.

||= basically checks to see if a variable has something assigned to it besides nil or false. If it does, it stops right there and just leaves it be. If it doesn't, it assigns the variable a new value, whatever you supply on the right side of the expression.

So x ||= 10 checks x. If x is nil or false, it assigns x a value of 10. If x already has a value, ||= politely moves on without disturbing x at all. Here's where I imagine a sheepish Woody Allen voice... "uh um sorry to trouble you but uh, if I may, just um, if you don't have a value already I have this other value for you and well um... you know, you can have it if you don't have one already."


To explain it more precisely, x ||= 10 is basically shorthand for x || x = 10. When Ruby looks at an or statement like this, it evaluates the first expression on the left, and if that expression evaluates to True, it doesn't bother looking at the right half, because it only takes one true expression to know that an or statement is true.

There are two subtleties to note here. One, note that this is a little different from how other similar looking expressions work. 
x += 10 
is the same as 
x = (x+ 10) 
n 
but x ||= 10 

is NOT the same as

x = (x || 10)

rather x ||= 10 is more like

x || (x = 10)

The difference is..


Another caveat--technically x||= 10 is not EXACTLY like x || (x = 10). In certain very specific scenarios they behave slightly differently (basically, blah, more details here).

When is ||= useful? All the time! I find myself using it when iterating through a collection that might have repeats. It's great for setting up a counter too. For example, say we want sort through and count a bunch of pants. Each pair of pants is represented by a hash like this:

{type: jeans, color: blue}

And say we want to combine a bunch of these hashes into a more organized hash like this:

{
jeans: [{color: blue, count: 3}]

slacks: [{color: black, count: 2}, {color: khaki, count: 1}]

sweats: [color: gray, count: 1]
}


useful for aggregating--create an array or key-value pair if it doesn't exist, and add to it.

(useful case 1)

(explain memoization and show useful case 2)

