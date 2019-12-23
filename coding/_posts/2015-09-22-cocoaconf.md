---
layout: post
title: CocoaConf 2015
date: 2015-9-22
tags: objective-c general
categories: coding
root: "../"
---

I attended my first conference as an iOS engineer this past weekend: [CocoaConf Boston, 2015](http://cocoaconf.com/boston-2015/home) in Peabody, Massachusetts.

<!--more-->

I was excited and apprehensive. I worried about things that I haven't worried about since my first day at school -- _will the other kids like me? what's for lunch? what if the other kids are smarter than me?_ -- and I thought of the many unasked questions that I left unasked, out of fear that I would be laughed at. 

Over the course of my brief iOS career, I have collected and put aside many unasked questions as not appropriate to the occasion, like: 

* Do you use storyboards? (The startup I work at, [Switch](http://switchapp.com), deleted our storyboard file a year and a half ago and has been flying with autolayout since then. Our code looks something like this: `@"H:|-defaultMargin-[_view]-defaultMargin-|"` with nary a visual representation)
* How do you all get that parallax animation? None of you _actually_ use `UINavigationBar`, right?
* How do you refresh your data when a user minimizes and then opens your app? We observe `UIApplicationDidBecomeActiveNotification`, but it gets pretty messy when our data fetch happens in `viewDidLoad` and in the selector that listens to `UIApplicationDidBecomeActiveNotification`.
* What tools do you use to make things better in XCode?
* Do things ever get better with XCode?

I have always hoped to ask these questions, and more, without appearing too unknowledgeable. I didn't want to sound like an amateur in front of a bunch of iOS professionals -- especially since I had over a year of experience under my belt, and since this was a professional _conference_ -- and I felt very much like an amateur in the front row, surrounded by a room full of people who wore Apple watches on their wrists and spoke knowledgeably of the Apple TV dev kit.

Then the first speaker, [Marcus Zarra](https://twitter.com/mzarra), of CoreData and [CocoaIsMyGirlfriend](http://www.cimgf.com/) fame, whose history I did not know at the time, began:

> "Crave fear,"
he said.

Do what makes you afraid, because it means there is a part of you that wants to.

<br>

### _What was I afraid of, I wondered?_
{: style="text-align: left"}

<br>

#### _Talking to people. Finding out, and showing, how little I knew._
{: style="text-align: right"}

<br>

### _What was it that I wanted to do?_
{: style="text-align: left"}

<br>

#### _I wanted to talk to people, and find out how much I could learn._
{: style="text-align: right"}

<br>

CocoaConf started on Friday morning at 8am, and went until 8pm that night. Saturday's sessions ran from 8am to 5pm. Some were inspirational (the keynote, in particular, was an inspirational start to the weekend, and lamented by Zarra himself as well outside of his comfort zone); some were highly technical; some drew analogies between coding and writing.

In between each session, we had about fifteen minutes to get up, stretch, and chat with our neighbors. 

I learned, in starting a conversation with the table on using open source code to discover best practices for application architecture, that I could take a look at [Cheddar](https://github.com/nothingmagical/cheddar-ios) in addition to [Artsy's open source projects](https://github.com/artsy).<sup>[1]</sup> I had been lamenting that our standard StackOverflow-to-squash-a-bug procedure was only a surface treatment of what could be larger problems that were solved by larger solutions, and now I had two repositories to peruse for structural wisdoms.

I learned, from a friendly face in an ESPN shirt, that his name was [Christian](https://twitter.com/Chris_Allgood), that he, indeed, worked at ESPN, and that he was a veritable source of example code for a custom transition that I had been wrestling with in my last sprint.

I learned, as I spoke to others in line for dinner or coffee, that there were many different flavors of Cocoa developer at CocoaConf. You had a [CEO of a successful mobile agency](https://twitter.com/graiz), a [CEO of an independent software company](https://twitter.com/siegel), [the founder of a non-profit](https://twitter.com/macgenie), [the director of mobile engineering at a startup](https://twitter.com/designatednerd), [a fledgling developer at a startup][2]... and each came, with her own background and history, to the crossroads that was CocoaConf. 

# No programmer is an island

It's easy to put your head down and code in a bubble.

I've been doing a lot of coding and building and bubbling, and after a year and a half as a growing engineer, there is a pull to think that you should _know things_ in your seniority. As our team at [Switch](https://www.switchapp.com) continues to grow and I get the opportunity to share institutional and engineering knowledge with newer members, I start to look back in wonder, thinking that perhaps this is what it means to be a more senior engineer. 

Then the wonder fades into fear as you continue to code but feel obligated to _know_ things, and this is where you begin to feel like an imposter and occupy your bubble out of fear of being discovered.

We don't need to feel like imposters, and we don't need to stay in that bubble.

We can ask questions, and we can carry conversations with people who know infinitely more than us without fear of being labeled ignorant, amateurish.

CocoaConf ended on [Jaimee Newberry](https://twitter.com/jaimeejaimee)'s keynote on #noexcuses:

>"Be an amateur",

she said.

At CocoaConf, I learned, as I spoke to the other developers around me, that I was an amateur.

All it meant was that I could be a better me.

(_and you can be a better you, and we can be a better we_)
{: style="text-align: right"}

We're all amateurs, looking to be professionals. Some already seem to be there, and maybe even then, they feel like they're not. 

The rest of us? 

We'll all get there, someday, and the community, rich in its people and minds, is a precious repository of what others have learned on their own journies. Mine began a year and a half ago, and Boston was a beautiful stop on the way.

![]({{ post.root }}/assets/cocoaconf.jpg){: style="width: 300px" .center-image}

_Thanks for reading! (eski)Mona kindly asks you for stupid questions and answers in iOS @ [@hazelynutter](http://www.twitter.com/hazelynutter). Until next time._

[1]: Credit to [David Norcott](https://twitter.com/DavidNorcott) from [Rue-La-La](https://www.ruelala.com)
[2]: http://www.eskimona.com/about "Wait! That's me."
