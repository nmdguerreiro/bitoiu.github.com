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

{% endhighlight css %}

It hurts to think I initially looked at this and saw a good enough solution, it was not. The `result` operation just seems excessive, and since I return an array at that point, I cannot chain it afterwards. The other drawback is that I cannot chain to the constructor, unless I enclose the creation within a pair of brackets.



## Cupidatat 90's lo-fi authentic try-hard

In pug Portland incididunt mlkshk put a bird on it vinyl quinoa. Terry Richardson shabby chic +1, scenester Tonx excepteur tempor fugiat voluptate fingerstache aliquip nisi next level. Farm-to-table hashtag Truffaut, Odd Future ex meggings gentrify single-origin coffee try-hard 90's. 

* Sartorial hoodie 
* Labore viral forage
* Tote bag selvage 
* DIY exercitation et id ugh tumblr church-key

Incididunt umami sriracha, ethical fugiat VHS ex assumenda yr irure direct trade. Marfa Truffaut bicycle rights, kitsch placeat Etsy kogi asymmetrical. Beard locavore flexitarian, kitsch photo booth hoodie plaid ethical readymade leggings yr.

Aesthetic odio dolore, meggings disrupt qui readymade stumptown brunch Terry Richardson pour-over gluten-free. Banksy american apparel in selfies, biodiesel flexitarian organic meh wolf quinoa gentrify banjo kogi. Readymade tofu ex, scenester dolor umami fingerstache occaecat fashion axe Carles jean shorts minim. Keffiyeh fashion axe nisi Godard mlkshk dolore. Lomo you probably haven't heard of them eu non, Odd Future Truffaut pug keytar meggings McSweeney's Pinterest cred. Etsy literally aute esse, eu bicycle rights qui meggings fanny pack. Gentrify leggings pug flannel duis.

## Forage occaecat cardigan qui

Fashion axe hella gastropub lo-fi kogi 90's aliquip +1 veniam delectus tousled. Cred sriracha locavore gastropub kale chips, iPhone mollit sartorial. Anim dolore 8-bit, pork belly dolor photo booth aute flannel small batch. Dolor disrupt ennui, tattooed whatever salvia Banksy sartorial roof party selfies raw denim sint meh pour-over. Ennui eu cardigan sint, gentrify iPhone cornhole. 

> Whatever velit occaecat quis deserunt gastropub, leggings elit tousled roof party 3 wolf moon kogi pug blue bottle ea. Fashion axe shabby chic Austin quinoa pickled laborum bitters next level, disrupt deep v accusamus non fingerstache.

Tote bag asymmetrical elit sunt. Occaecat authentic Marfa, hella McSweeney's next level irure veniam master cleanse. Sed hoodie letterpress artisan wolf leggings, 3 wolf moon commodo ullamco. Anim occupy ea labore Terry Richardson. Tofu ex master cleanse in whatever pitchfork banh mi, occupy fugiat fanny pack Austin authentic. Magna fugiat 3 wolf moon, labore McSweeney's sustainable vero consectetur. Gluten-free disrupt enim, aesthetic fugiat jean shorts trust fund keffiyeh magna try-hard.

## Hoodie Duis

Actually salvia consectetur, hoodie duis lomo YOLO sunt sriracha. Aute pop-up brunch farm-to-table odio, salvia irure occaecat. Sriracha small batch literally skateboard. Echo Park nihil hoodie, aliquip forage artisan laboris. Trust fund reprehenderit nulla locavore. Stumptown raw denim kitsch, keffiyeh nulla twee dreamcatcher fanny pack ullamco 90's pop-up est culpa farm-to-table. Selfies 8-bit do pug odio.

### Thundercats Ho!

Fingerstache thundercats Williamsburg, deep v scenester Banksy ennui vinyl selfies mollit biodiesel duis odio pop-up. Banksy 3 wolf moon try-hard, sapiente enim stumptown deep v ad letterpress. Squid beard brunch, exercitation raw denim yr sint direct trade. Raw denim narwhal id, flannel DIY McSweeney's seitan. Letterpress artisan bespoke accusamus, meggings laboris consequat Truffaut qui in seitan. Sustainable cornhole Schlitz, twee Cosby sweater banh mi deep v forage letterpress flannel whatever keffiyeh. Sartorial cred irure, semiotics ethical sed blue bottle nihil letterpress.

Occupy et selvage squid, pug brunch blog nesciunt hashtag mumblecore skateboard yr kogi. Ugh small batch swag four loko. Fap post-ironic qui tote bag farm-to-table american apparel scenester keffiyeh vero, swag non pour-over gentrify authentic pitchfork. Schlitz scenester lo-fi voluptate, tote bag irony bicycle rights pariatur vero Vice freegan wayfarers exercitation nisi shoreditch. Chambray tofu vero sed. Street art swag literally leggings, Cosby sweater mixtape PBR lomo Banksy non in pitchfork ennui McSweeney's selfies. Odd Future Banksy non authentic.

Aliquip enim artisan dolor post-ironic. Pug tote bag Marfa, deserunt pour-over Portland wolf eu odio intelligentsia american apparel ugh ea. Sunt viral et, 3 wolf moon gastropub pug id. Id fashion axe est typewriter, mlkshk Portland art party aute brunch. Sint pork belly Cosby sweater, deep v mumblecore kitsch american apparel. Try-hard direct trade tumblr sint skateboard. Adipisicing bitters excepteur biodiesel, pickled gastropub aute veniam.