---
layout: post
title: Things I Learned
date: 2015-11-02
categories: General
root: "../"
---

October 29 was my last day at [Switch](https://www.switchapp.com). I learned a lot as a programmer and person, and thought I would share a few lessons learned.

<!--more-->

During my time at Switch, I kept a `Journal of Fun and Learning` with sections: **blog ideas**, **what I did**, **perplexions**, **wisdoms**, **readings**, **reflections**, and **life**. Here are my favorites:

1. **blog ideas**

	> Creating a God object and discovering that gods are difficult to overthrow.

	I had no idea that a God object was a thing until I created one that tied together tableviews, their cells, and an almighty `TableViewManager` object that implemented the delegate and datasource. The intent was to design an object that could help implement form-like behavior for all tableviews.

	Some lessons are learned the hard way. This one is still sitting as an idea for a blog because it's too embarrassingly to describe this terrible, terrible abstraction.

1. **things I did**

	> Migrate existing userbase from Django user (based on unique, required username) to custom user with unique and required email, maintaining passwords

	I learned a lot about South, and spent a few days rolling back and forth between migrations until my head hurt. 

	In the end, we figured out two ways to migrate to a custom user in Django: either rename the `auth_user`, `auth_user_groups`, and `auth_user_permissions` tables, or create new tables for `switch_user`, `switch_user_groups`, and `switch_user_permissions` and perform a datamigration. The first method ended up being easier for various reasons, to be described one day in a blog post.

1. **perplexions (unanswered questions)**

	> `[self.navigationController.navigationBar setTranslucent:NO]` redefines the frame of the view controller’s view...why?

	...this has since been upgraded to a wisdom. A view controller's frame should be defined according to its coordinate plane in the navigation controller, if wrapped in one!

	A lot of my perplexions have been upgraded to wisdoms, which is a grand feeling.

1. **wisdoms (a solved perplexion or good ol' advice)**
	
	> restart XCode and rebuild

1. **good blog posts and StackOverflow answers**

	[Good, great, 10x](http://dbgrandi.github.io/good_great_10x/) - I really cannot recommend this enough for the growing programmer.

1. **reflections (of which there are many good ones)**

	1. Define the problem, not the solution you superficially think will solve the problem. When someone asks you for something, first ask, "Why do you want it?" and not "How do I do it?"
	1. Do it simple or do it complex, but do it right.
	1. It's easy to get stuck in the details. It's also hard to get all the details right. The best way to do this is to attack complexity in chunks and iterations.
	1. There's always more to learn and refactor, and that's okay.
	1. Q: How do I get heard? A: Don’t feel like you need to raise your hand.

	<br />

1. **life**

	> Fear is the mind killer.

	There were many times when I was afraid of building something because I didn't know how. That is silly talk because no one knows how to do anything the first time, and everyone learns eventually if they try. 

	Never let fear be the thing that stops you from learning.

