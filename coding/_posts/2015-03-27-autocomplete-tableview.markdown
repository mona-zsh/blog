---
layout: post
title:  "How to build an autocomplete tableview"
date:   2015-03-27 18:57:49
categories: objective-c
root: "../"
---

##### _that is, a tableview that takes keyboard input, displays a list of autocomplete values, and allows user selection of a value_

##### _alt subtitle: the unfortunate mystery of `-[UITableView reloadData]` and `resignFirstResponder`_

<!--more--> 

# Why build this at all (and what exactly is it that we’re building)?

Mobile apps are supposed to be a little like magic -- type in a letter, we’ll guess what you need, and before you know it, [there’s a tiger on your doorstep](http://techcrunch.com/2015/02/27/catch-a-tiger-by-the-toe/).

[Switch](http://www.switchapp.com) is one such app (in full disclosure, it just so happens that I am an engineer on the team behind this app). Here at Switch, we want to make switching jobs easy as swiping left or right. To do that, it has to be possible to create a mini resume with the tap of a few keys. In other words: give us a resume, and we’ll find you a job.

![]({{ post.root }}/assets/profile.gif){: .center-image}
Magic, really long scrolls... same thing, right?
{: .caption}

I'll be the first to admit that our current onboarding process is a bit of a monster and involves at least 11 tableview cells. It isn’t quite the magic we are looking for.

It is certainly admirable how anyone manages to get through this process, and it gets even more complicated when dealing with data that aren’t free form text. For example, while we would love for users to be able to tell us where they are, we really can’t accept “Mars” as a location (as much as we would love to have jobs for you in Mars). Neither can we present our users 150,000 cities and expect that to be a good experience for anyone.

Was there a way to require less of users while generating more information? Could we give users freedom in input and collect structured data at the same time? 

# Enter the autocomplete tableview.

![]({{ post.root}}/assets/autocompleteview.png){: .center-image }
Courtesy of [3drops](https://dribbble.com/3drops)
{: .caption}

Imagine the happy Switch User, who can now type in a letter at the top of a table view into a text field, get a list of results, and select a value from the given results. This works particularly well for selecting a location, of which there are many in the world, and of which we do not want to stuff into a `UIPicker`. The user is happy and our backend is happy that user input corresponds to actual rows in our database.

And of course, as the person building this in XCode, I was very happy.

You see, sometimes, good design and Apple’s UIControl gifts don’t intersect as often as we might like. I’ve received wireframes and hi-fi mocks of impossibly beautiful, impossible-to-implement UIControls -- a `UISlider` with two knobs, a `UIAlertView` with a custom background and custom button -- only to spend a day rummaging through [cocoacontrols](https://www.cocoacontrols.com/), hooking one up, and ending up with an indeterministic EXC_BAD_ACCESS.

Everything in this design, however, looked familiar and Apple-apropos. **Everything looked easy enough to build in iOS.**

And like everything in iOS that looks easy, it took the better part of a day to break down, involved a bit of wonkiness, and required an implementation less intuitive than the first one struck upon.

Without further ado, let’s break down how the magic happens (and later, see where it all breaks down).

# The first step in making magic: design to implementation

Let’s break down the anatomy of this design. Once we have all its component parts in Objective-C, we can go forth and let users select all sorts of values.

![]({{ post.root}}/assets/breakdown.png){: .center-image}

First, we have a `UITableView` with a variable number of cells. The first cell is simple: we have a `UITextField` that is empty to start. The text field is populated with free-form text if the user is typing, set with a value from the autocomplete list when the user has selected a value, and empty if the user chooses to clear the selection. 

Cells 2 through N should be populated whenever a user enters a string. All we need to do is return cells that display the values of an array, perhaps mutable, so that we can update the values as the string changes. When a user selects a cell, `[tableView didSelectRowAtIndexPath:]` updates a property on the view controller to indicate that a valid selection has been made. The `UITextField` displays the selection and we clear the autocomplete array.

Not too bad at all, right? All we need:

* 1 view controller with a property to hold the selected value
* 1 master array of all possible values
* 1 mutable array of values to select from
* 1 `UITableView`, to be reloaded whenever the mutable array changes
* 1 `UITextField`
* `UITextField` delegate methods to update the mutable array to match what the user is typing into the text field
* `UITableView` datasource and delegate methods to return the right cells and keep track of what was selected

Now that we have the outline, we can get some practice building it with more concrete example. Perhaps we would enjoy an example less mundane than selecting locations. For example, I am sure you have always wanted to type a letter and receive a list a cute animal emoji, out of only those cute animal emoji that are found in the iOS keyboard world, whose names begin with that letter. Which brings us to the world of...

# Alphabetimals

![]({{ post.root}}/assets/alphabetimals.png){: .center-image }
The complete project is [here](https://github.com/mona-zsh/alphabetimals).
{: .caption}

If you start typing in the alphabetimal text field, the table view populates its cells with animal emoji names and their emoji (so some of the name-emoji pairings are something like “Rabbit Face 🐰”, which sounds a little bit like what a bully would call you). You can select this cell. It gets displayed in your text field and then you submit it into the wild, perhaps to your servers, for adoption or whatever you would do with such a thing.

Our view controller has an array of all alphabetimals (`Alphabetimals *`), a property `selectedAlphabetimal`, and an autocomplete array for the `UITableViewCell`.

{% highlight objc %}
@interface ViewController () <UITableViewDataSource, UITableViewDelegate, UITextFieldDelegate>

// Array of all Alphabetimals
@property (nonatomic) NSArray *alphabetimalArray;              

// Array of Alphabetimals from autocomplete
@property (nonatomic) NSMutableArray *autocompleteAnimalArray; 

// Current Alphabetimal selection from autocomplete table view
@property (nonatomic) Alphabetimal *selectedAlphabetimal;      

@end
{% endhighlight %}

We [load this name-emoji data from a plist](https://github.com/mona-zsh/alphabetimals/blob/master/Alphabetimals/Alphabetimals/ViewController.m#L60-L75). If anyone is curious, I wasn’t sure how to coerce the `NSString` to display unicode 32 characters, so I converted them to [surrogate pairs](https://github.com/mona-zsh/alphabetimals/blob/master/Alphabetimals/Alphabetimals/alphabetimal.plist). 

We set up our `UITableViewCell` and `UITextField` like so:

{% highlight objc %}
- (AATableViewCell *)textFieldTableViewCell {
    if (!_textFieldTableViewCell) {
        _textFieldTableViewCell = [[AATableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:nil];
        [_textFieldTableViewCell addSubview:self.textField];
    }
    return _textFieldTableViewCell;
}

- (UITableView *)tableView {
    if (!_tableView) {
        _tableView = [[UITableView alloc] initWithFrame:CGRectZero style:UITableViewStylePlain];
        _tableView.dataSource = self;
        _tableView.delegate = self;
    }
    return _tableView;
}
{% endhighlight %}

We implement our `UITableViewDataSource` methods. We include two sections. I have an extra cell that is displayed if the autocomplete array is empty so that the user will be prompted to start typing.

{% highlight objc %}
#pragma mark - UITableViewDataSource

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    if (section == 0) {
        return 1;
    } else {
        if ([self.autocompleteAnimalArray count] == 0) {
            // User is not currently typing; no autocomplete
            // Display cell with prompt to start typing
            return 1;
        }
        // Return number of autocompleted animal names
        return [self.autocompleteAnimalArray count];
    }
}

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView {
    return 2;
}
{% endhighlight %}

The `UITableViewDelegate` method `[tableView didSelectRowAtIndexPath:]` then updates `selectedAlphabetimal` when a `UITableViewCell` containing that alphabetimal is selected. Easy!

{% highlight objc %}
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    if ([self.autocompleteAnimalArray count] > 0) {
        self.selectedAlphabetimal = self.autocompleteAnimalArray[indexPath.row];
    } else {
        // If there is no autocomplete array, selecting the first row of the second section
        // should prompt the user to enter text in the textfield.
        if (indexPath.row == 0 && indexPath.section == 1) {
            [self.textField becomeFirstResponder];
        }
    }
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
}
{% endhighlight %}

We update animations, set our selected alphabetimal, and reload the tableview.

{% highlight objc %}
- (void)setSelectedAlphabetimal:(Alphabetimal *)selectedAlphabetimal {
    _selectedAlphabetimal = selectedAlphabetimal;
    
    // Clear autocomplete array
    [_autocompleteAnimalArray removeAllObjects];
    if (selectedAlphabetimal == nil) {
        self.textFieldTableViewCell.accessoryType = UITableViewCellAccessoryNone;
    } else {
        self.textField.text = selectedAlphabetimal.displayName;
        self.textFieldTableViewCell.accessoryType = UITableViewCellAccessoryCheckmark;
    }
    
    // Reload tableview so that selected animal name and emoji are displayed
    [self.tableView reloadData];
}
{% endhighlight %}

And `[textField shouldChangeCharactersInRange:replacementString]` uses a helper method to find alphabetimals that match the current string, updates the autocomplete array, and reloads the table view datasource.

{% highlight objc %}
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
    NSString *textFieldString = [self.textField.text stringByReplacingCharactersInRange:range withString:string];
    
    // Clear selected alphabetimal if one is selected
    if (self.selectedAlphabetimal != nil) {
        self.selectedAlphabetimal = nil;
    }
    
    if (string.length == 0 && textField.text.length == 1) {
        // Clear autocomplete animal array on deletion of last character
        [self.autocompleteAnimalArray removeAllObjects];
    } else {
        // Use new text field string to reload the autocomplete animal array
        [self reloadAutoCompleteAnimalArrayFromString:textFieldString];
    }
    
    // Reload data after autocompleteAnimalArray has been updated
    [self.tableView reloadData];
    return YES;
}

- (void)reloadAutoCompleteAnimalArrayFromString:(NSString *)string {
    // Find all alphabetimals with a name that starts with the current string in text field
    [self.autocompleteAnimalArray removeAllObjects];
    for (Alphabetimal *alphabetimal in self.alphabetimalArray) {
        NSString *animalDisplayName = alphabetimal.displayName;
        NSRange substringRange = [[animalDisplayName lowercaseString] rangeOfString:string];
        if (substringRange.location == 0) {
            [self.autocompleteAnimalArray addObject:alphabetimal];
        }
    }
}
{% endhighlight %}

Here’s the problem:

**It doesn’t work.**

I encourage you to try building the [project](https://github.com/mona-zsh/alphabetimals)  and typing into the text field. It is an utterly frustrating User Experience and a good lesson in frustration. Every letter you type, your `UITextField` will resign. No letters will appear. Yet, if you chance upon the right letters, you will have the good fortune to have a loaded tableView datasource. 

![]({{ post.root}}/assets/frustration.gif){: .center-image}
You can select an alphabetimal.
{: .caption}

![Frustration]({{ post.root}}/assets/frustration2.gif){: .center-image}
But you can’t change your mind and hit backspace.
{: .caption}

Why is the keyboard disappearing when we try to enter a letter? Is it something we did in `[textField shouldChangeCharactersInRange:replacementString:]`? Are we returning NO? Is one of our text field delegate methods clearing the text field? Are we calling `resignFirstResponder`?

The keyboard dismissal looks suspiciously like what happens when a `UITextField` resigns first responder, so let’s set a breakpoint for `[UITextField resignFirstResponder]` and see if it gets called--and by what.

![]({{ post.root}}/assets/breakpoint.png){: .center-image}

`[UITableView reloadData]`, which we do indeed call when a text field character changes, is calling for our poor text field’s resignation.

This is apparently [known behavior](http://stackoverflow.com/questions/11372589/uitableview-reloaddata-causes-uitextfield-to-resignfirstresponder). 

> Your text field is resigning because reloaded cells are sent a -resignFirstResponder message due to the fact that their survival is not guaranteed after a reload. See this [related](http://stackoverflow.com/questions/6409370/uitableview-reloaddata-resigns-first-responder) question for more.

Clearly, the `UITableView` knows better than to trust us when we call `reloadData`. That could lead to some awkward situations, like a keyboard without a textfield. The table view knows better. The table view does what it wants.

That’s all well and good, but what if the text field needs to survive a reload? [StackOverflow](http://stackoverflow.com/a/18969996) suggests a promising solution:

> I solved this by subclassing UITextView, overriding -(BOOL)resignFirstResponder and by adding a BOOL canResign. this variable is set before reloading the data and unset a short time after.

So if the `UITextField` is set to return `NO` for `resignFirstResponder`, we will be able to reload our table view without worrying about the keyboard being dismissed.

Let’s try returning `NO` (we’ll worry about dismissing the keyboard when we select an alphabetimal later).

{% highlight objc %}
AATextField.m

- (BOOL)resignFirstResponder {
    return NO;
}

{% endhighlight %}

Simple enough, right?

![]({{ post.root}}/assets/noreload.gif){: .center-image}

Good news: we can type again! Bad news: our table view isn’t being reloaded when we call `[UITableView reloadData]`. We’re still calling `reloadData`, but none of the data source methods are being called.

Okay, well, let’s return `YES` in `resignFirstResponder`. If the text field resigns, the table view will be able to reload again.

![]({{ post.root}}/assets/noreload-yes.gif){: .center-image}
We said YES! What more do you want?
{: .caption}

We’re still not reloading data, even though we’ve given into the UITableView’s demands and coerced the text field to resign first responder.

So how do we get the table view to reload again? Most of the time, all it takes is to read Apple documentation more carefully:

On `resignFirstResponder`

> The default implementation returns YES, resigning first responder status. Subclasses can override this method to update state or perform some action such as unhighlighting the selection, or to return NO, refusing to relinquish first responder status. If you override this method, *you must call super (the superclass implementation) at some point in your code.*
> 
>[Source](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIResponder_Class/index.html#//apple_ref/occ/instm/UIResponder/resignFirstResponder)

This looks like a winning combination: calling the superclass implementation and then refusing the relinquish status. `[super resignFirstResponder]` should perform whatever underlying logic is necessary to reload the data source methods, and then BOOL override returned will keep the keyboard on the screen. 

Unfortunately, this leads us back to the drawing board: every letter you type, the text field will be handing in its resignation. 

**To recap: there are two user experiences now. Either your textfield will continuously resign responder while your tableview reloads data from a pitifully invisible first letter, or your textfield will gloriously fill with text while your UITableViewCells remain empty.**

What does this all mean? 

Somewhere under Apple’s covers, there’s something in the superclass implementation of `resignFirstResponder` that actually resigns the UITextField and also allows the UITableView to reload, and it seems that you can’t have one without the other. 

We have just witnessed the mysterious responder chain at work.

I’ll have to admit that I had no idea how to proceed from here. Set more breakpoints? Read more about the UIResponder chain? Figure out what this relationship was?

I’ll leave this mystery as an exercise for the reader, and solve the first one instead: how to build a darn table view with a text field in it.

# The Solution

There are a few ways to crack this egg. `[tableView reloadSectionsAtIndexPath:]` seems promising, and allows you to reload data on a particular section of the table view. If your text field is in another section, it seems to escape the first responder purge. Unfortunately, in the original project I was building, the first section dynamically adds a cell if there is a selected value (see our nifty radius slider), which makes it difficult to reload one section without raising an `NSInternalInconsistencyException` for the invalid number of rows in the other section. I’m sure this method is possible, but I was sick and tired of table views, and wanted a quicker fix.

This is all I did:

{% highlight objc %}
[self.view addSubview:self.tableView];
[self.view addSubview:self.textFieldTableViewCell];

// Set up frames
self.tableView.frame = CGRectMake(
                                  0,
                                  TEXT_FIELD_TABLEVIEW_CELL_HEIGHT,
                                  self.view.frame.size.width,
                                  self.view.frame.size.height - TEXT_FIELD_TABLEVIEW_CELL_HEIGHT
                                  );
self.textFieldTableViewCell.frame = CGRectMake(
                                               0,
                                               0,
                                               self.view.frame.size.width,
                                               TEXT_FIELD_TABLEVIEW_CELL_HEIGHT
                                               );
{% endhighlight %}

In the `UITableViewDataSource` methods, the number of sections in this table view will always be one. The number of rows will correspond to the number of autocomplete entries in the mutable array. 

Things check out, and you can now select a Front-Facing Baby Chick.

![]({{ post.root}}/assets/chick.gif){: .center-image}

## Reflection

Here’s what went wrong with what was supposed to be a clean, easy design.

### One: I was too excited by the idea of a clean, easy design. 

From the mockup, all I saw was a bunch of `UITableViewCells`. As a result, I wanted to shove everything into a `UITableView` and forget about instantiating a `UITextField` as its own field and defining the frames. I thought I was being clever by keeping the text field in the table view -- after all, that’s what the design looked like. Which led to…

### Two: I forgot to think about what I was building. 

My first instinct was to start breaking things down into `UITableViewCells` and `UITextFields`. But that’s not what I was building -- I was building something to take user input and return acceptable values that the user could choose from and submit. If I had to break this project down again, I would think less about design and more about function: we want to take user input and we want to display values that conformed to existing data.

These functions seem different enough to perhaps warrant separate views and business logic. If you find yourself wanting to display a list of changing values while an input field remains unchanged, it might be a sign that they shouldn’t live in the same view.

In fact, many wise designers and developers distinguish between input and display.

![]({{ post.root}}/assets/facebook-twitter.png){: .center-image}

Notice that Facebook and Twitter allow the user to input data in a search bar that lives on the navigation bar. These are popular apps which implement the feature we are trying to build.

### This leads me to reason 3: I didn’t do my homework.

The concept of taking user input and outputting a list of possible values is not a new one. There’s no reason not to research how others have invented their respective wheels and realize that the preferred style of input is a search bar within the navigation bar, distinct from the table view.

# Lessons Learned:

1. **Beware the responder chain.** The superclass implementation of `resignFirstResponder` both resigns the `UITextField` and allows the `UITableView` to reload… (by the way, does anyone know how to decouple the two?)
2. **Look at designs before you jump in.** There’s a reason why people have implemented designs in a certain way, due to limitations of language or because it’s the right way to do things. 
3. **Think about the function of what you’re trying to build**, not just how to make it look like what it needs to look like.
4. Front-Facing Baby Chick is what Apple calls 🐥.

*Thanks for reading! (eski)Mona Zhang is a proud English major turned software engineer at Switch in NYC. She will continue to blog about iOS wonkiness and neat Python tricks. Stay tuned!*