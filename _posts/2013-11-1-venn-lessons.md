---
layout: post
title: "meet venn"
description: "lessons learned and re-learned when writting venn"
category: tech
tags: [tech, js, venn]
image:
  feature: tech-image.jpg
  credit: Vitor Monteiro
  creditlink: http://bitoiu.github.io/
comments: true  
---

One of these days, I was writing a node command tool, the 10th or so mini-project I wouldn't finish. This pet-project, which I still plan to finish is a a tool to quickly delete git branches.  While writing the first prototypes, I stumbled upon a pattern where I would have list of branches that I needed to merge or intersect. For example, if I wanted to delete only merged local branches, that would be an intersection between all local branches with all local merged branches. I thought I would give this approach a try, knowing from the start that in terms of performance I could eventually do better, but in terms of code simplicity I assumed this would be a better choice.

When I goggled for set operation libraries, a lot of the usual suspects came back in the results, most notably [underscore](http://underscorejs.org). That was it, if I wanted I could just add this library to my dependencies but the reason I go home and code is to be able to work on whatever I feel like, and that particular night, I felt like I really wanted to use a fluent API instead of underscore's good, community approved and tested but imperative solution. I was making excuses to do my own standalone library.

##[venn](http://bitoiu.github.io/venn/)

So then there was it, I named it [venn](http://bitoiu.github.io/venn/), a fluent API for set operations. And this post is about some of the design options I did, along with unexpected problems and the solutions employed. If you're looking for excitement, you're in the wrong website.

##the beginning - class approach

Because I still have a bit of OO in my blood and because the more I code in Javascript the less I feel the need to implement classes, I thought I had upon me a very good chance to do exactly that: encapsulate all these nice set operations into a neatly little class called venn.

As soon as I started writing the tests, something didn't feel quite right, but I couldn't put a finger to it. I can't remember precisely but I had something which should be very similar to this:

{% highlight javascript %}

var set = new Venn([])

// so here I have my fluent API
set.union([1,2,3])
   .union([2,3,4])

// and I use result to get the current state
set.result()

{% endhighlight javascript %}

It hurts to think I initially looked at this and saw a good enough solution: it was not. The `result` operation just seems excessive, and since I return a pure array at that point, I cannot chain it. The other drawback is that I cannot chain the constructor, unless I enclose the creation within a pair of brackets.

##array approach

Gladly I know a guy or two that do amazing code reviews and one of them, [@sammyt](https://github.com/sammyt) asked me if I had though about making venn just a plain array, much like [d3](http://d3js.org/). Yes sir, I want it to be just a plain array.

So after this chat, [@sammyt](https://github.com/sammyt) pointed me to the piece of code that creates an array sub-class in the d3 codebase. By array sub-class we're referring to the process of adding properties to an array object:

{% highlight javascript %}

var arraySubClass = [].__proto__
    ? function(array, prototype) {
        array.__proto__ = prototype
    }
    : function(array, prototype) {
        for (var property in prototype) array[property] = prototype[property]
    }

{% endhighlight javascript %}

The else function is there to support IE6, 7, 8 and Opera, which don't implement `__proto__`. The second argument `prototype` is just an object that lists a set of properties to decorate the first argument with, in my case methods like `union` and `intersection`.

Suddenly I don't need a `result` operation, because the variable is already an array. I can also chain everything and get a result back in one line, making this API much more elegant than the previous one:

{% highlight javascript %}

var set = venn.create([1,2,3])
    .union([2,3,4])
    .union([1,2,3]

console.log(set) // [2,3,4,1]

{% endhighlight javascript %}

##gotchas

There were a couple of things that, although simple to solve, were not initially part of my plans.

###iterable properties

So I re-wrote my tests with the new API based on the design shown above. My first approach to define the `union` method was something like this:

{% highlight javascript %}

// could as well be an object
var venn_prototype = []

venn_prototype.union = function () {
    // do pretty things
}

{% endhighlight javascript %}

Simple right? Now Imagine an array `[1,2,3]`, I pass it to my `venn.create` which in turn calls `arraySubClass`. Now the original array gets all the methods that were part of the prototype. So I run the tests, and this is what I get:

{% highlight javascript %}

expected = [1,2,3]
returned = [1,2,3, function]

{% endhighlight javascript %}

When adding the `union` function as a simple property the array treats it just as another element. Not a big deal, properties in javascript can be defined with a bit more context, so we just need to `defineProperty` instead:

{% highlight javascript %}

// Venn properties
Object.defineProperty(venn_prototype, "union", {
    value : _union,
    writable : false,
    enumerable : false
});

{% endhighlight javascript %}

The key here is to set `enumerable` to false and job done. Soon after this change, I had the intersection and union functions ready.


After this, I took a break from venn, and it was until I noticed I had a few downloads on npm that I thought of finishing it. At that point I felt the urge to take another look at the code and complete it by implementing custom key functions and the `not` operation.

###keeping state

At this point not providing access to a custom key function renderer the library almost useless. For simple objects the default behaviour (object to string hash) could be used, but I would never ship production code with it.

By the time I was about to start implementing the `keyFunction`, this how the `union` implementation looked like:

{% highlight javascript %}

var _union = function(set) {

    var map = this.concat(set)
      .reduce(function(curr, next) {
        curr[uid(next)] = next
        return curr
      }, {})

    var result = []
    for (var key in map) {
      result.push(map[key])
    }

    arraySubclass(result, venn_prototype)

    return result
  }

{% endhighlight javascript %}

If you see the code for `intersection` the two last lines are the same. So why do I always call `arraySubClass` before returning? I've used `array.reduce` and `array.concat`. These methods return a pure array and not a venn array, so before I return the result I re-decorate the array with the venn methods. I didn't mind this trade off because the code reads better using native array methods and I'm only adding a couple of properties for each operation.

I wrote my tests for the custom key function and the API was (and still is) something like this:

{% highlight javascript %}

    venn.create(["i","n","p","u","t"], myKeyFunction)

{% endhighlight javascript %}

At creation time, the developer has the option to pass his own custom key function, which is saved for the lifetime of this specific venn object, or that was my plan at least.

The truth is, I was loosing the `keyFunction` value as soon as a `union` or `intersection` was executed. The `arraySubClass` works fine to add constant properties, in this case method definitions. When we want to track state, in this case a `keyFunction` value, this approach by itself needs to be adapted.

**Backup and restore**

One of the first options that came into mind was to backup the properties before calling the native methods and restore them after `arraySubClass`. In retrospective, if it was today, I would have adopted this solution, but at the time, I was more focused on finding a pattern that would not require maintenance if more properties were added. I discarded a solution that would involve iterating through the venn array and looking for special named properties since I removed made them non-enumerable in the first place.

**Avoid destroying the original object**

If the original problem was caused by creating new array instances every time I used a native array method, why not just avoid that? The last version of venn doesn't use a single native array method, not a big deal but it forced me to write a few more lines of code. It uses in-place algorithms to keep the instance alive, preserving the current state:

{% highlight javascript %}

  var _intersection = function(set) {

    var that = this

    if(!set || set.length == 0 || !this || this.length == 0) {
      this.length = 0
    } else {

      var copiedVenn = [].concat(this)
        , visited = {}
        , key

      this.length = 0

      set.forEach(function(element) {
        visited[getKey.call(that,element)] = true
      })

      copiedVenn.forEach(function(element) {

        key = getKey.call(that,element)
        if( visited[key] ) {
          that.push(element)
          delete visited[key]
        }
      })
    }

    return this
  }

{% endhighlight javascript %}

My approach to this was to *fool* the object's length by setting it to zero, and adding back elements according to the operation being executed. This way I don't need to worry about backups, restores and I also don't need to do the `arraySubClass` more than once per venn instance.

##tl;dr... the whole article

After the custom `keyFunction` I just implemented a `not` and added a bit of task automation with [grunt](http://gruntjs.com/). Venn is a very humble library, I used it to keep me distracted and at the same time to develop something I plan to use in other projects. I invite everyone interested to take a look at the [source code](https://github.com/bitoiu/venn) and provide any kind of feedback. I bet there are a lot of different options for some of the problems and I would be glad to hear about them.
