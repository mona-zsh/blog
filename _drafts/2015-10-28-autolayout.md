---
layout: post
title: Autolayout
date: 2015-10-28
categories: objective-c ios
root: "../"
---

There is an age-old debate and question that will come up in every iOS meetup you ever go to:

Storyboard, nibs, or programmatic?

<!--more-->


One of the first things we did at [Switch](https://www.switchapp.com/) was to delete our storyboard file.

Even within the programmatic style of user interface layout, you have several options:

1. Resizing masks
1. Frames
1. Autolayout

With resizing masks, you can define the resizing behavior of a subview by what attributes are flexible - for example, a subview with flexible width and height will expand to fill its parent when its parent bounds change. All views have `autoresizesSubviews = YES` by default, and you can then proceed to define the `autoresizingMask` bitwise properties on your views (`myView.autoresizingMask = UIViewFlexibleWidth | UIViewFlexibleHeight`).

With setting frames, you override `layoutSubviews` and get very good at pixel math. I really like this method when I only have 3 subviews or so to calculate.

With autolayout, you define a set of relationship constraints for your views. Those constraints are used to calculate the size and position of the views in your view hierarchy.

**We chose autolayout.**

It was a decision that we made without looking back. We don't know what's on the other side, and whether storyboards are greener. 

What matters is that we learned many things about autolayout along the way. What follows is a brief primer on autolayout, and then the tips and tricks we use.

* What is the easiest way to add a bunch of subviews and align them?
* What is the easiest way to add a bunch of subviews and align most of them, and set an explicit alignment for some of them?
* How do I add equal padding to both sides of a view if I don't know how much I should add?
* How do I center multiple subviews?
* How do I lay something out that has intrinsic content size, and not define its height explicitly?
* How do I do the above when the interface I’m using requires me to define an explicit size (UICollectionView)?
* How do I align views to that pesky navigation bar?



<sub>You can follow along on this [demo](https://github.com/mona-zsh/EMAutolayout/tree/master)!</sub>

# What is Autolayout?

This is a [pretty good intro to Autolayout](http://commandshift.co.uk/blog/2013/01/31/visual-format-language-for-autolayout/). Otherwise you can read along for a brief overview.

You can define UILayoutConstraints with two methods.

1. `constraintWithItem:attribute:relatedBy:attribute:multiplier:constant:` is a super verbose method that defines a single constraint. Think of this as a linear equation with the following format:
    
    `item1.attribute1 = multiplier × item2.attribute2 + constant`

1. `constraintsWithVisualFormat:options:metrics:views:` uses visual format language and allows you to describe a set of these constraints. Visual format looks like this:
 * `visualFormat:@"H:|   [_blueView]   [_redView(==_blueView)]    |"`
 * `options:NSLayoutFormatAlignAllLeft | NSLayoutFormatAlignAllRight`
 * `metrics:@{@"margin":@50}` so you can refer to those constants in VFL: `H:|-margin-[_blueView]-margin-|`
 * `views:@{@"view": myView}` so you can refer to those views in VFL: `V:|[view]|`

You get a lot more precision with the first method, and we favored this method in the early days of our autolayout code. Later on we learned how to cheat with VFL, which allows you to define many, many, constraints at once in a single string.

You can probably accomplish anything you'd like with VFL, so I definitely recommend learning it the VFL way and falling back on a single constraint only if you need one.

# VFL Glossary

* `|` means superview
* `-` means NSSpace (8 pixels)
* `[`a subview is referenced between two square brackets`]`
* `[subview(width)]-margin-|` metrics can be referenced anywhere by its key in the dictionary
* Depending on whether you are defining V: or H: constraints, `H:[subview(this is width)]` and `V:[subview(this is height)]`


# Before we begin...

There is a very good argument against VFL: we don't really think of visual elements in terms of constraints. Only the OS can, and that's because it's smarter than us as it calculates sizes and positions based on constraints as different superviews get bigger and smaller. It is very hard to think of a mockup in terms of linear formulas between views.

Because of that, it's super important to get into the right mindset for thinking about constraints. Something that really helped me imagine constraints was realizing that you either have **views with explicit dimensions** (views that are equal to or smaller than your screen resolution) or **views that don't** in one or more dimensions (scrollviews).

Ask yourself...

1. Should the intrinsic content of the view and subviews expand to show all their content? (This is usually YES if your superview is a scroll view or collection view). If so, save the work of setting explicit dimensions, and let your views define their own content sizes.
	1. Define spacing constraints for all subviews in your view and the view’s edges (top + bottom, or left + right)
	1. Define the position of your view, but not its intrinsic dimension (this could be height or width, or both). 
	1. If the height and width are ambiguous, you can accomplish this with defining its center or two of its sides
	1. If the height is ambiguous, define any of its left/right/centerX properties
	1. If the width is ambiguous, define any of its top/bottom/centerY properties

1. Should there be an explicit dimension of the view and subviews? (This is usually YES if your superview is a view with a defined width or height!). If so, how do we handle truncated content? Do we add ellipses, shrink fonts, shrink images, etc.
	1. Set explicit height, width, and position (usually center) of the view
	1. Define how the content should be truncated or shrunk.

# What we wanted to do, and how we did it

**I want to add a bunch of views vertically and align their left and right sides.**

Options are your best friend! Apply them in the opposite plane that your constraints are defined. Here, we define that the `topLabel` and `bottomLabel` will be placed vertically in the following order:

1. top edge of superview
1. topLabel
1. bottomLabel
1. bottom edge of superview

With the following alignment:

1. Left
1. Right

{% highlight obj-c %}
[NSLayoutConstraint constraintsWithVisualFormat:@"V:|[_topLabel][_bottomLabel]|"
options:NSLayoutFormatAlignAllLeft | NSLayoutFormatAlignAllRight
metrics:nil 
views:NSDictionaryOfVariableBindings(_topLabel, _bottomLabel)];
{% endhighlight %}

Then, we define the `topLabel` in the horizontal plane. 
This set of horizontal constraints will be applied to `bottomLabel` as well because `topLabel` is aligned left and right.

1. left edge of superview
1. topLabel
1. right edge of superview

{% highlight obj-c %}
[NSLayoutConstraint constraintsWithVisualFormat:@"H:|[_topLabel]|" 
options:0 
metrics:nil 
views:NSDictionaryOfVariableBindings(_topLabel, _bottomLabel)];
{% endhighlight %}

When we define vertical constraints for subviews, we can also them by Left, Right, Center X, and any horizontally defined alignment options. The same holds true the opposite way!

It gets tricky when you only want to align certain subviews and not others.

**I want to add a bunch of views vertically and align some of left and right sides.**

Let’s say you want to align the topLabel and bottomLabel with its superview’s left and right edges, but the middleLabel should be padded.

{% highlight obj-c %}
H:|[_topLabel]|

V:|[_topLabel]->=0-[_bottomLabel]|
NSLayoutFormatAlignAllLeft | NSLayoutFormatAlignAllRight
            
V:[_topLabel][_middleLabel][_bottomLabel]

H:|-15-[_middleLabel]-15-|
{% endhighlight %}

We define the horizontal constraints of `middleLabel` and `topLabel`, `bottomLabel` separately, but we can still take a shortcut with alignment options for `topLabel` and `bottomLabel`.

**I want to add equal padding to two sides of a set of views, but I’m not sure how much.**

The lazy way:

If you set an explicit height on your view and can expect padding to be constant, you can use trial-by-error and figure out the exact padding your views need for your exact content. This assumes that your content will always be an expected height, or that you’ve handled it in some way (e.g. for labels, you have defined numberOfLines = 0 and font = someFont.

{% highlight obj-c %}
V:self(36)
V:|-3-[_topLabel][_bottomLabel(==_topLabel)]-3-|
{% endhighlight %}

The above works... but it's not very flexible. You get pretty unexpected behavior for different resolutions. The lazy way is deceptively more work because you have to build and rebuild in different simulators to make sure everything looks okay with magic numbers.

Here's the "lazier" way, because it saves you time in the end: use empty UIViews to space things evenly.

`V:|[spacerOne][_topLabel][_bottomLabel(==_topLabel)][spacerTwo(==spacerOne)]|`

Watch out! Because your spacer views are empty and have no intrinsic height, and because 0 == 0, your spacer heights will collapse to 0 padding given the opportunity. Set a minimum height if you want a minimum padding.

`V:|[spacerOne(>=5)][_topLabel][_bottomLabel(==_topLabel)][spacerTwo(==spacerOne)]|`

This does mean that you will have to alloc init empty UIViews and remember to add them as subviews, as well as add them to your views dictionary. We suggest using `NSDictionaryOfVariableBindings` to save yourself some keystrokes.

**I want to center something with multiple subviews.**

You can use the spacer strategy to center something, or you can use a container view. This is particularly useful when there is no height constraint on the content, and you want it to fill out the superview without concerns that the content will be too big.

{% highlight obj-c %}
V:|->=0-[containerView]->=0-|
H:|->=0-[containerView]->=0-|
CenterX = superView.centerX
CenterY = superView.centerY
{% endhighlight %}

**I want to know how the above works when you didn't define a size, and only the position, for your subviews.**

When your content has an intrinsic size and you don't want to explicitly define its height (or width), you can accomplish it with the following:

* the container view has no height constraint
* the top subview has a spacing constraint from the top of the container view
* the contained views all have spacing constraints between them
* the bottom view has a spacing constraint to the bottom of the container view

The above example works if all the subviews within `containerView` are defined and have intrinsic content size. These are usually UILabels with text or UIImageViews with images. Then all you need is a position for the superview of all the subviews, and its height will be calculated based on its content.

**I want my subviews to expand to show its content in its superview, but the interface I’m using requires me to define an explicit size (UICollectionView).**

Override `- (void)intrinsicContentSize` in your subview and calculate what your layout suggests. This plays nicely if you define paddings, not spacer views...because then you might need to keep a reference and do some crazy calculations.

```
V:|-padding-[_topLabel][_bottomLabel(==_topLabel)]-padding-|
height = 2 * (padding + _topLabel.intrinsicContentSize.height)
```

**I want my views to be below that status bar and above that tab bar.**

Go ahead and use your view controller’s top layout and bottom layout guide properties to help you along! You can add them to your views dictionary.

`viewController.topLayoutGuide, viewController.bottomLayoutGuide`

```
format:V:|[topLayoutGuide][_topView][_bottomView(200)]| 
options:NSLayoutFormatAlignAllLeft | NSLayoutFormatAlignAllRight 
metrics:nil views:NSDictionaryOfVariableBindings(_topView, _bottomView, topLayoutGuide)]]
```

These are but a few ways to address a few use cases, but they have saved me many hours.

# Troubleshooting

If things go wrong, ask:

1. Is the view’s frame CGRectZero? If so, you did not define enough constraints to allow Apple’s Autolayout system to do its work. You need either of the following.
    * Height, width, centerX, centerY
    * Top, bottom, left, right

    Either of these sets of constraints will allow Apple to lay out your view. Height and width, if defined by your constraint (it’s an image or label), don’t need to be explicitly set in your constraints. Be careful, this means your subviews may escape their bounds!

1. Did you set translatesAutoresizingMaskIntoConstraints = NO?

# Gotchas

* Exceptions: Most times, VFL will throw an exception if you do something wrong. This mostly comprises bad syntax and not adding a view to its superview before defining a constraint relationship.
* Do not be fancy with your linear formula if you expect half pixels and animations. Your views will jiggle.
* If you call removeFromSuperview, you will ask the parent to layout all its views. If one of its views is auto laid out, you might have some UI layout problems. We can avoid this by giving our swappable subview a container parent view, so that layoutSubviews doesn’t get called for all the view’s subviews.

Conclusion
===========

Autolayout is very powerful, and I've enjoyed learning how to think in it. There are still many concepts in autolayout that I haven't addressed (partially because I don't use them enough to know how they're useful! Looking at you, `UILayoutPriority`), and I look forward to figuring out what they actually do.

Although, after months of typing constraints, I am getting tired enough to eye xibs/nibs with some curiosity. Those are coming up next :)

[Let me know if you have any other tips and tricks to add!](https://www.twitter.com/hazelynutter)