---
layout: post
title: Being a Programmer
date: 2015-12-31
tags: objective-c general
categories: coding
published: true
root: "../"
---

Happy New Year's Eve, readers!

I began writing this post about programming in November, when traveling. I had just moved on from being a software engineer at [Switch](http://www.switchapp.com) for two years, and I marveled at all I had learned as a programmer. 

There have been a few times in my career when I realized, _"I'm finally becoming a real software engineer, and not just someone who writes code."_

While I suspected that these experiences were common to the journeywoman programmer, as far as I knew, they have not been named. And since naming things (variables, functions, common human experiences) helps others more easily share abstractions, I had wanted to name these learning experiences and see if anyone else had a name for them, too.

But I never did finish the post in November. Now I'm back, re-reading a draft of this post after having worked on a completely new project for close to a month now. At first, I was mortified because I've experienced nothing that I had proudly described just last month. Was I becoming someone who "just writes code"?

In the past month of doing completely new things (interpolated splines, lots of polar coordinates, and netting of solids), I've been learning a new stack and encountering different technical challenges. I once had a [friend and mentor](http://www.twitter.com/brett1211) tease me for not wanting to touch RoR as a framework language, wondering if I had "settled into the calcification of one language". In a way, I had. I was very comfortable with Django and iOS, and what I was working on, and my setup just _worked_. I didn't have to struggle with console errors to get matplotlib installed in my virtual environment because Python was a "framework build"<sup>[1]</sup>. I could work on the same types of challenges and get deeper and deeper into the language, and deeper and deeper into the problem, and that was good for me. 

But recently, I've been in the beginning stages of problem-solving, where you do have to struggle to get set up, and to understand the new domain of a problem. And that's good for me too. Learning to uncalcify is part of being a programmer.

So some of the experiences below only happen if you've been working on a technical challenge for a long(er) time. And some only happen if you're encountering a problem for the first time. Both, I think, are important on this journey. Both are fun to experience.

<!--more-->

<sub>Side note: I am a huge fan of [programming jargon](http://blog.codinghorror.com/new-programming-jargon/), which does a great job of naming things that happen to all of us as programmers (e.g. Pokemon exception handling, Heisenbugs, and more), and I think there is room in this lexicon for more jargon.</sub>

# Naming Experiences of the Journeywoman or Journeyman Programmer

1. The **There and back again** experience

	_Alternative names_: Deja Vu

	You open up an old piece of code and realize it needs to be refactored. This is the point in every programmer's career when she encounters the ugly consequences of past code.

	This is a great learning experience because it hammers in what NOT to do. It's especially hard as a beginner programmer to code a solution to a problem when you don't anticipate the ways in which you might extend that feature into solving more problems in the future.

	The first time you encounter this, you will become warier of future-proofing your code. You will also realize that **you are not your code**, and that you can and should make changes to past code without fear.

1. The **It's not you, it's me** experience

	_Alternative names_: This is not the answer you are looking for

	There comes a time when your interface looks completely whacky for no discernible reason, and you __know__ you shouldn't try to Google "UIView is missing" because it won't work.

	This is that moment when you realize whether a bug is attributable to one or many causes. If it's a bug that likely has one cause, Google and Stack Overflow is your best friend. When it has more than one, you have more soul-searching to do.

	The exception to this rule is iOS programming. Sometimes it is not your code. Sometimes you just need to quit XCode, restart your Mac, and then clean and rebuild. 

1. The **Anything is possible** experience

	This is a realization that no matter what problem your PM throws at you, you know you can solve it in code.

	When an apprentice programmer begins to solve problems with code, some problems are so daunting that she questions whether it can be solved. Then, as she breaks down the problem and solves it incrementally, she realizes that any problem can be incrementally solved.

	The only question is how much it costs, and whether you solve the problem in a way that allows for future, unthought-of but related problems, to be solved through an extension of the original solution.

1. The **Gentile Force** experience

	Shortly after the "anything is possible" realization, the programmer realizes that one's brute force solution is going to work just fine, but cost more to maintain different use cases. This is a wonderful moment in which the programmer foresees future pain, and spends a little bit of time now to avoid it.

	It is a happy moment when the programmer knows she can wield raw, dark power, but chooses the light side instead.

1. The **Oh, there's a library for that** experience

	When I first wrote a Python script to make HTTP requests, I formatted the URL with some manual encoding (and typed, by hand, things like %20 and %2b for spaces and pluses in URLs). Then I realized that there's a [library for that](https://docs.python.org/3/library/urllib.html#module-urllib), and an even higher level library for [requests](http://docs.python-requests.org/en/latest/).

	There is nothing quite like building something from scratch to learn the ropes, so I don't regret it much. Sometimes you end up building your own library of a sort. Sometimes you reinvent the wheel. It's a learning experience, and can be avoided (with the help of research and a mentor) or embraced (if you have time and curiosity to build something of your own).

<hr>

Any other experiences to share, or any alternative names for the above?

Happy New Year, and see you in 2016!

[1]: http://matplotlib.org/faq/virtualenv_faq.html

_P.S. You might notice a few new sections in the blog. I've been renovating to make room for posts about ["life"]({{root}}/life), and I sometimes ["scribble"]({{root}}/scribbles/) about it while I'm at it._
