---
layout: post
title: CocoaConf 2015
date: 2015-9-22
categories: objective-c general
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
* (_Dear reader: do you have any unasked questions? Please [let me know](http://twitter.com/hazelynut) so I can collect them, too._)

I have always hoped to ask these questions, and more, without appearing too unknowledgeable. I didn't want to sound like an amateur in front of a bunch of iOS professionals -- especially since I had over a year of experience under my belt -- and I felt very much like an amateur in the front row, surrounded by a room of people who wore Apple watches on their wrists and spoke knowledgeably of the Apple TV dev kit.

Then the first speaker, [Marcus Zarra](https://twitter.com/mzarra), of CoreData and [CocoaIsMyGirlfriend](http://www.cimgf.com/) fame, whose history I did not know at the time, began:

# "Crave fear,"
{: style="text-align: center"}

he said.
{: style="text-align: right"}

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

I learned, as I started a conversation with the table about wanting more open source code projects like [Artsy's](https://github.com/artsy) so that we could see how to best structure an app instead of seeing a solution an isolated StackOverflow problem. [David Norcott](https://twitter.com/DavidNorcott) from [Rue-La-La](https://www.ruelala.com) replied with a link to [Cheddar](https://github.com/nothingmagical/cheddar-ios). Now I have two repositories to peruse for good practices, and I am happy that I have come out of that conversation with more knowledge to share.

I learned, as I approached a friendly face in an ESPN shirt, found out that his name was [Christian](https://twitter.com/Chris_Allgood) and that he, indeed, worked at ESPN, and then we ended up comparing custom transition implementations. 

I learned, as I spoke to others in line for dinner or coffee, that there were many different flavors of Cocoa developer at CocoaConf. You had a [CEO of a successful mobile agency](https://twitter.com/graiz), a [CEO of an independent software company](https://twitter.com/siegel), [the founder of a non-profit](https://twitter.com/macgenie), [the director of mobile engineering at a startup](https://twitter.com/designatednerd), a fledgling developer at a startup... and each came, with her own background and history, to the crossroads that was CocoaConf. 

# No programmer is an island

It's easy to put your head down and code in a bubble.

I've been doing a lot of coding and building and bubbling, and after a year and a half as a growing engineer, there is a pull to think that you should _know things_ in your seniority. As our team at [Switch](https://www.switchapp.com) continues to grow and I get the opportunity to pass on institutional and engineering knowledge to newer members, I start to look back in wonder, thinking that perhaps this is what it means to be a more senior engineer, and then wondering when that transition even happened. Then the wonder fades into fear as you continue to code but feel obligated to _know_ things, and this is where you begin to feel like an imposter.

CocoaConf ended on [Jaimee Newberry](https://twitter.com/jaimeejaimee)'s keynote on #noexcuses:

# "Be an amateur",
{: style="text-align: center"}

she said.
{: style="text-align: right"}

At CocoaConf, I learned, as I spoke to the other developers around me, that I was an amateur, and that was okay.

I could be a better me.

You can be a better you.

We're all amateurs, looking to be professionals. Some of us are there, and maybe even then, they feel like they're not. 

The rest of us? 

We'll all get there, someday, and the community, rich in its people and minds, is a precious repository of what others have learned on their own journies. Mine began a year and a half ago, and Boston was a beautiful stop on the way.

![]({{ post.root }}/assets/cocoaconf.jpg){: style="width: 300px" .center-image}

_Thanks for reading! (eski)Mona has been heavy in the iOS world, and still learning loads. Spot something you have an answer for (how does one refresh data on soft closes)? Would love to hear your thoughts @ [@hazelynutter](http://www.twitter.com/hazelynutter). Until next time._

