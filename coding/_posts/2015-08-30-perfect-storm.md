---
layout: post
title:  The Perfect Storm - A Preventative Case-Study of A Vanishing View
date:   2015-08-30
categories: coding
tags: objective-c
root: "../"
---

One day, while testing our iOS app against 7.1, there appeared to be a mysterious, sometimes disappearance of a UICollectionView.

It was a normal view.

It vanished (sometimes).

Why did it vanish, you ask?

It was the perfect storm.

<!--more-->

<hr><br />

This storm of ours was one of those oddities that happens because of **a confluence of preventable decisions**. Nothing seemed overtly wrong at the time that we were setting up the view in question, but eventually, it all converged into a perfect storm. 

Before I begin, be warned that this blog post is not a prescriptive "how-to" that explains that you should do Y if you want to build X. As much as I would like the mastery to write that kind of blog, another kind of blog calls out to me today.

This blog will be a preventative case study, and a three-part story, of "how something went perfectly wrong", "how we uncovered the wrongness", and "what we should never have done, and never should do".

While I do not think it likely for a programmer with this exact problem to stumble upon this blog with a well-phrased Google search, I do think this blog will be **a preventative lesson for beginner iOS programmers**. Perhaps you will spot the warning signs that we did not.

## Part 1: The Circumstances

The vanishing collection view seemed to be a very normal collection view. On `viewDidLoad`, we added it as a subview to our view controller's view. Then we set the frame to the view controller's view's frame.

In every other operating system, the collection view was fine.

It was only in 7.1, and on a launch after a hard close of the app, that it was, well, missing. Sometimes. Other times, it was fine.

## Part 2: Investigation

### Clue 1: A tale of two subviews

After some investigation, we discovered that our lazy instantiated property, `collectionView`, was being called twice, and while one of them was being added as a subview, the other was not added as a subview, and was receiving the message to `reloadData`. We wondered: 

* _Isn't lazy instantiation a guarantee that only one instance of a view will be created?_

Lazy instantiation is pretty awesome if you only want to configure your view if its getter is called somewhere, and looks like this:

{% highlight objc %}
- (UIView *)aView {
    if (!_aView) {
        _aView = [[UIView alloc] initWithFrame:self.view.frame];
        _aView.backgroundColor = [UIColor redColor];
        // Here, we might configure a lot of expensive things.
    }
    return _aView;
}
{% endhighlight %}

The lazily written accessor acts as getter and setter. If there is a reference to `_aView`, it will return the instance variable that backs the property. If not, it will allocate, initialize, and save its pointer.

It wasn't working here. We tried to throw a few @synchronized locks on it, to no avail.

* _Could multiple threads be getting into that block, and how? We only use one thread!_ (this was before we realized that AFNetworking calls are of course, asynchronous)

This was our first clue. Somehow, the following two code paths were getting executed on different threads, and different instances of collection views:

{% highlight objc %}
// View controller lifecycle
[self viewDidLoad]; // which then called...
[self.view addSubview:self.collectionView];
[self refreshData];
[self.collectionView reloadData];

// UIApplicationDidBecomeActiveNotification
[self refreshData] // which then called...
[self.collectionView reloadData];
{% endhighlight %}

### Clue 2: Hard Closes

This behavior could only be reproduced on a `hard close` (our name for the _double-tap-on-home-and-swipe-up_ iPhone mechanism).

What could that mean?

Well, our view controller does register for a `UIApplicationDidBecomeActiveNotification` in its designated initializer to call `reloadData` on our collection view. 

{% highlight objc %}
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(refreshData) name:UIApplicationDidBecomeActiveNotification object:nil];
    }
    return self;
}

- (void)refreshData {
	// Backend service subclasses AFNetworkSessionManager
	[[self.backendService sharedService] getMatchesWithSuccess:(void(^)(NSArray *matches)) {
		self.matches = matches;
		self.collectionView reloadData];
	}
}
{% endhighlight %}

It's always been a little heavy-handed to fetch and reload using every time the application becomes active, but so far it's been our best weapon against the `soft close` (single-tap-on-home). (Side note, and question: how else do people make sure the data is refreshed on soft closes?).

That means that while on `soft closes`, only `UIApplicationDidBecomeActiveNotification` gets called, on hard closes, both `UIApplicationDidBecomeActiveNotification` and `viewDidLoad` might get called.

{% highlight objc %}

- (void)viewDidLoad {
	[super viewDidLoad];
	[self.view addSubview:self.collectionView];
	[self refreshData];
}

{% endhighlight %}

Two subviews, two references to `self.collectionView` in potentially two threads -- this is starting to make sense.

### Clue 3: Wait, this still doesn't make any sense.

We could see that multiple threads were accessing `self.collectionView`, but it didn't really make sense that `refreshData` was getting called before our view loaded and called `[self.view addSubview:self.collectionView]`. 

Shouldn't `viewDidLoad` get triggered, and shortly after that, `refreshData`, either the one in the `viewDidLoad` or the one registered for the NSNotification -- at which point there would already be an instance variable, `_collectionView`?

That would certainly hold true if we triggered `loadView` right after `init`. But if you, on app launch, allocate and initialize instances of view controllers through a factory method, so that you can present the views on these view controllers later...

Why, then, it's a race between the notification-triggered `refreshData` and `viewDidLoad`.

## Mystery Solved

![]({{ post.root }}/assets/ScoobyDoo.png){: .center-image} 

1. App launches
1. View controller is instantiated in a constructor method
1. `viewDidLoad` executes `[self.view addSubview:self.collectionView]` on the main thread
1. **A collectionView is born, and it is added to the view. It has no data. The pointer to the instance variable backing collectionView is not yet saved.**  _// In the meantime, simultaneously..._
1. `UIApplicationDidBecomeActiveNotification` fires and the view controller is instantiated and ready to listen. At this exact point in time, no view is loaded, and there is still no ivar.
1. View controller executes `refreshData` because of the `NSNotification`
1. `refreshData` calls `[self.collectionView reloadData]` on a background thread
1. **A collectionView is born, not as a subview, and it loads some data into its cells. A pointer to this object is saved in the view controller's instance variable.**
1. `viewDidLoad`'s `refreshData` is called, and by this point, `_collectionView` refers to the second collection view NOT in the view hierarchy.

{% highlight objc %}
// View controller lifecycle
[self viewDidLoad];                        // (1)
[self.view addSubview:self.collectionView] // (2) _collectionView = some instance of collectionView
[self refreshData];                        // (5)
[self.collectionView reloadData];          // (6) refers to (4) _collectionView

// UIApplicationDidBecomeActiveNotification
[self refreshData];                        // (3)
[self.collectionView reloadData];          // (4) _collectionView = new instance of collectionView
{% endhighlight %}

All you need for this storm is:

1. A constructor or factory method to instantiate a view controller, resulting in separate instantiation and presentation logic
1. Registering to the `UIApplicationDidBecomeActiveNotification` in the view controller's designated initializer (we ended up moving this to viewDidLoad as a first-defense solution)
1. Lazy instantiated properties
1. Calling a property on a background thread

## What did we learn?

A lot of this can be solved by rule #1 of iOS programming: **Perform UI updates on the Main Thread**. It's easy to forget. Don't. I was instilled this cardinal rule as a fledgling iOS programmer, at which point I didn't really understand threads or memory addresses or instantiation. Here's the rundown of what might happen with multi-threaded view code:

Remember our lazy instantiation?

{% highlight objc %}
- (UIView *)aView {
    if (!_aView) {
        _aView = [[UIView alloc] initWithFrame:self.view.frame];
        _aView.backgroundColor = [UIColor redColor];
        // Here, we might configure a lot of expensive things.
    }
    return _aView;
}
{% endhighlight %}

Thread 1 tries to add `aView` as a subview. It checks if there is a pointer to the instance variable, `_aView`. In other words, have we already allocated and initialized the object, and then saved the pointer to `_aView`?

Thread 1 says no. Thread 1 goes ahead and allocates, initializes the object to some address (let's say 123), and <del>saves the pointer.</del>

Wait! Thread 2 comes into play, and tries to send a message to `self.aView`. That requires going through lazy instantiation again, because it's really a getter AND a setter! Let's check, has `_aView` been allocated and initialized?

Thread 2 says no, too. Thread 2 goes ahead and allocates, initializes the object to some address (let's say 234) -- <sub>oh, by the way, Thread 1 finishes saving the the pointer to `_aView` but it's too late because</sub> -- Thread 2 is ready to save the pointer to `_aView` as memory address 234. All messages go to the second view.

Next time a view of yours disappears in 7.1, check that your view logic is being performed on the main thread. Enforce it with a few calls to `performSelectorOnMainThread:`.

<hr><br />

In the end, this wasn't a problem we could StackOverflow or Google, though I am sure we tried. What would we query? "UIView not showing"? Could it be the datasource not providing data, or perhaps the frame being set incorrectly? How would we inspect the view hierarchy for iOS 7.1?

This was a lesson on the many things you should not do as an iOS programmer. It was also a lesson in how to chase down the series of unfortunate events that contributed to a missing view.

While I am usually wary of learning and teaching what not to do as opposed to what you should do, and while I think learning should be a journey of creation, and doing new things, rather than remembering all the things that you should not be doing--

There can always be a lesson in other's mistakes, and I hope that one of you will have learned something valuable from mine.

_Thanks for reading! (eski)Mona has been heavy in the iOS world, and still learning loads. Spot something you have an answer for (how does one refresh data on soft closes)? Would love to hear your thoughts @ [@hazelynutter](http://www.twitter.com/hazelynutter). Until next time._
