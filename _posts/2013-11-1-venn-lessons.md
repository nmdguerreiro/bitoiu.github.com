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

// show the sub class here
}

{% endhighlight javascript %}

Awesome right? Imagine a simple array `[1,2,3]`, I pass it to my `venn.create` and this simple array now has an additional method named `union`. I can still use the array as I would do normally, but I can also call my own operations on it.

So I run the tests, and this is what I get:

{% highlight javascript %}

expected = [1,2,3]
returned = [1,2,3, function]

{% endhighlight javascript %}

When adding the `union` function as a simple property, the array treats it just as another element. No big deal, properties in javascript can be defined with a bit more context, so we just need to `defineProperty` instead:

{% highlight javascript %}

// Venn properties
Object.defineProperty(venn_prototype, "union", {
    value : _union,
    writable : false,
    enumerable : false
});

{% endhighlight javascript %}

The key here is to set `enumerable` to false and job done. Soon after this change I had the intersection and union functions ready. I went on a break from venn and I only returned to it a couple of months ago when I noticed I had a few downloads on npm. At that point I felt the urge to take another look at the code and finish it by implementing custom key functions.

### keeping state

As you can imagine implementing the union and intersection was pretty straightforward. I didn't go nuts on the implementation, I've sacrificed memory for performance, and I was quite happy with the end result.

At this point not providing access to a custom key function renderer the library useless. For simple objects the default behaviour (string hash) can be used, but even so I don't recommend it and in the future I might even remove it. If you want to test something very quickly it does the job, but I wouldn't ship anything using venn without a custom key function.

Again, I thought implementing the `keyFunction` property would be a five minute job, but again, it wasn't, at least for me. Why was that?

Take a look at the `union` implementation before the implementation of `keyFunction` :

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

If you see the code for `intersection` the two last lines are the same. So why do I call `arraySubClass` before returning? I've used `array.reduce` and `array.concat`. These methods obvisouly return a pure array and not a venn array, so before I return the result I re-decorate the array with the venn methods. I didn't mind this trade off because the code reads so much better using native array methods and come on, I'm adding 2 properties for each operation, it's nothing.

Again, I wrote my tests for the custom key function and the API was (and still is) something like this:

{% highlight javascript %}

    venn.create(["i","n","p","u","t"], myKeyFunction)

{% endhighlight javascript %}

At creation time the developer has the option to pass his own custom key function, which is saved for the lifetime of this specific venn object, in theory at least...

Oh no! I loose the `keyFunction` as soon as a `union` or `intersection` is executed. The `arraySubClass` only decorates the array with the methods, I don't save the function anywhere before I start calling native array methods. A couple of solutions came into mind

**Backup and restore**

In retrospective I could have choose this path. Why didn't I? Because I wanted to know how would I fix this problem if I was adding properties from time to time. If this was the scenario I really don't want to keep a list of properties I backup/restore. I could also try to make this a bit more dynamic by iterating over the object to look for special annotated properties, but then I would probably be where I started, when the venn array was outputting the methods as part of the contents.

**Keep the poor object**

If the original problem was caused by creating new array instances every time I used a native array method, why not just avoid that? The last version of venn doesn't use a single native array method, not a big deal but it forced me to write a few more lines of code. More than that, it uses in-place algorithms to keep the instance alive, and thus, not destroying any previous set state. Take a look at the new `union` implementation:

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

My approach to this was to *fool* the venn object setting it's `length` to zero, and adding back elements according to the operation being executed. This way I don't need to worry about backups, restores and I also don't need to do the `arraySubClass` more than once per venn instance.

## tl;dr... the whole article

After the custom `keyFunction` I just implemented a `not` and added a bit of task automation with [grunt](http://gruntjs.com/). Venn is a very humble library, I used it to keep me distracted and at the same time to develop something I plan to use in other projects. I invite everyone interested to take a look at the [source code](https://github.com/bitoiu/venn) and provide any kind of feedback. I bet there are a lot of different options I didn't explore for some of the problems and who knows, some of them might be improvements I end up making ;)
