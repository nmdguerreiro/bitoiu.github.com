---
layout: post
title: "Venn lessons"
description: "What I've learned, remembered and forgot again while writting venn"
category: tech
tags: [tech, js, venn]
image:
  feature: tech-image.jpg
  credit: Vitor Monteiro
  creditlink: http://bitoiu.github.io/
comments: true  
---

One of these days I was writing a node command tool, the 10th or so mini-project I wouldn't finish. This pet-project, which I still plan to finish but not as a node command tool, is a a tool to quickly delete local and remote git branches. Although in my cloud projects this is not an issue, especially since after GitHub added *delete your branch* after a pull request. At work though, branches tend to linger around, and it's understandable for a number a reasons which I'm bored to enumerate. Whatever the view here, I still think it's much better to have a lot of zombie branches, than deleting a branch you assumed it was ok to get rid of... please don't do that. Anyway, more of this clean-up tool another day.

While writing the first prototypes, I came upon a pattern where I would have list of branches that I needed to merge or intersect. For example, if I wanted to delete only merged local branches, that's an intersection between all local branches with all local merged branches. I though I would give this approach a try, knowing from the start that in terms of performance I could eventually do better, but in terms of code simplicity I assumed this would be a better choice. When I goggled for set operation libraries, a lot of the usual suspects came in the results, most notably [underscore](http://underscorejs.org). That was it, if I wanted I could just add this library to my dependencies and go ahead, but the reason I go home and code is to be able to work on whatever I feel like, and that particular night, I felt like I really wanted to use a fluent API instead of underscore's good, community approved and tested but imperative solution. Really, I was making excuses to do my own standalone library.

##[venn](http://bitoiu.github.io/venn/)

So I've committed myself to do a library that supported set operations, union and intersections to start off.

I named it [venn](http://bitoiu.github.io/venn/).

## class approach

Because I still have a bit of OO in my blood and because the more I code in Javascript the less I feel the need to implement classes, I thought I had upon me a very good chance to do exactly that: encapsulate all these nice set operations into a neatly little class called venn.

As soon as I started writing the tests, something didn't feel quite right, but I couldn't put a finger on it. I can't remember precisely but I had something which should be very similar to this:

{% highlight javascript %}

var set = new Venn([])

// so here I have my fluent API
set.union([1,2,3])
   .union([2,3,4])

// and now to get the current array
set.result()

{% endhighlight javascript %}

It hurts to think I initially looked at this and saw a good enough solution, it was not. The `result` operation just seems excessive, and since I return an array at that point, I cannot chain it afterwards. The other drawback is that I cannot chain to the constructor, unless I enclose the creation within a pair of brackets.

## array approach

Gladly I know a guy or two that do amazing code reviews and one of them, [@sammyt](https://github.com/sammyt) asked me if I though about just making it a plain array, much like [d3](http://d3js.org/). Yes sir, I want it to be just a plain array.

Before I go into the code aspect of it, this is not a lesson learned, but something I recommend everyone to do as much as they can. Get someone else's opinion, discuss it, post it at [codereview](http://codereview.stackexchange.com/). It's the best way to learn.

So after this chat [@sammyt](https://github.com/sammyt) pointed me to the piece of code that creates an array sub-class. By array sub-class I'm referring to adding methods to an array object which is looks like this:

{% highlight javascript %}

var arraySubClass = [].__proto__
    ? function(array, prototype) {
        array.__proto__ = prototype
    }
    : function(array, prototype) {
        for (var property in prototype) array[property] = prototype[property]
    }

{% endhighlight javascript %}

The else function is to support IE6, 7, 8 and Opera, which don't implement `__proto__`. The second argument `prototype` is just an object that lists a set of methods to decorate the first array with.

Suddenly I don't need a `result` operation, because the variable is already an array. I can also chain everything and get a result back in one line, making this API miles away from the previous one:

{% highlight javascript %}

var set = venn.create([1,2,3])
    .union([2,3,4])
    .union([1,2,3]

console.log(set) // [2,3,4,1]

{% endhighlight javascript %}


## gotchas

There were a couple of things that although simple to solve were not part of my plans initially.

### iterable properties

So I re-wrote my tests with the new API based on the design showed above. When these were all failing it was time for the implementation. My first approach to define the `union` method was something like this:

{% highlight javascript %}

// could as well be an object
var venn_prototype = []

venn_prototype.union = function () {
    // do pretty things
}

{% endhighlight javascript %}

Awesome right? Imagine a simple array `[1,2,3]`, I pass it to my `venn.create` and this simple array now has an additional method named `union`. I can still use the array as I would do normally, but I can also call my own operations on it.

So I run the tests, and what comes out immediately as a problem?

{% highlight javascript %}

expected = [1,2,3]
returned = [1,2,3, function]

{% endhighlight javascript %}

Oops, since javascript