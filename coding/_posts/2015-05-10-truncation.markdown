---
layout: post
title:  Text Truncation in a UILabel
date:   2015-05-10 18:57:49
tags: objective-c
categories: coding
root: "../"
---

_When `NSLineBreakByTruncatingTail` and the default ellipsis doesn't quite cut it for us, here are three methods to truncate a string to a constraining size and append custom text to the truncated string:_

* subtraction until it fits
* addition until it doesn't
* binary search, because who doesn't like log(N)

<br>
_Read on, or go ahead and grab them as a [category](https://gist.github.com/mona-zsh/3cc23f2efb828197c456), and enjoy!_
{: .caption}

<!--more-->  
~
{: .caption}

Let's say you have a lot of text with variable length and only a 320x100 frame to display it.  You don't want to leave the user with a crude cut in characters without any indication that there's more to read. What do you do?  

iOS offers an easy fix:

{% highlight objective-c %}
label.numberOfLines = 0;
{% endhighlight %}

The `UILabel`'s' `lineBreakMode` property is set by default to `NSLineBreakByTruncatingTail`, resulting in a simple but powerful `...` to signal the end:

![]({{ post.root }}/assets/default-label.png){: .center-image} 

The ellipsis is the universal solution to this problem, and a fine one at that for signaling the intentional omission of a word. If you want to also signal that there is some way to interact with your truncated text -- for example, let's say you want to make a truncated `UILabel` tappable for an expanded view -- you might want to add a custom call to action: "see more", "expand", or even an ellipsis with a different color.

![]({{ post.root }}/assets/expand.gif){: .center-image}
_Adding some <span style="color:#ddaa77">color</span> does the trick_
{: .caption}

Here's the rub: **as far as I can tell, there is no way to reach under the hood and override iOS truncation to append your own version of an ellipsis.** You can, however, write a decent workaround with a fit function using `NSString`'s `[boundingRectWithSize:options:attributes:context]` and mutate a string until it does fit. With a little [research](http://iosdevelopertips.com/cocoa/truncate-an-nsstring-and-append-an-ellipsis-respecting-the-font-size.html), I wrote something to the effect. First, the fit function, as a category on `NSString`:

{% highlight objc %}
@implementation NSString (Truncate)

/*
 Returns whether string appended with custom truncation string will fit in given size. 
 Uses `boundingRectWithSize:options:attributes:context` to check if the max height of 
 the string given a constraining width is greater than the height of the given 
 size parameter.
 */
- (BOOL)willFitToSize:(CGSize)size 
       trailingString:(NSString *)trailingString 
           attributes:(NSDictionary *)attributes {
    NSString *fullString = [NSString stringWithFormat:@"%@%@", self, trailingString];
    return [fullString boundingRectWithSize:CGSizeMake(size.width, CGFLOAT_MAX)
                                    options:(NSStringDrawingUsesLineFragmentOrigin|NSStringDrawingUsesFontLeading)
                                 attributes:attributes
                                    context:nil].size.height <= size.height;


}
...
@end

{% endhighlight %}

Using `[boundingRectWithSize:options:attributes:context:]`, we can check if the **maximum height that a string would take up in a constraining width** is less than or equal to the **constraining height**. Our fit function takes a parameter, `trailingString`, so that we can call the fit function on the final form of the text of the label: one part truncated string, one part custom string to indicate omission. 

Now we can implement our first solution.

# Truncation Method: Subtraction

Mutate the string, subtracting characters until `[willFitToSize:trailingString:attributes:]` returns `YES`.

{% highlight objc %}

- (NSAttributedString *)stringUsingSubtractionToTruncateToSize:(CGSize)size
                                                    attributes:(NSDictionary *)attributes
                                                trailingString:(NSString *)trailingString
                                                         color:(UIColor *)color {
    
    if (![self willFitToSize:size trailingString:@"" attributes:attributes]) {
        
        NSMutableString *string = [self mutableCopy];
        NSRange rangeOfLastCharacter = {string.length - 1, 1};
        
        while (![string willFitToSize:size trailingString:trailingString attributes:attributes]) {
            [string deleteCharactersInRange:rangeOfLastCharacter];
            rangeOfLastCharacter.location--;
        }
        
        NSInteger indexOfLastCharacter = rangeOfLastCharacter.location + rangeOfLastCharacter.length;
        
        NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:[string substringToIndex:indexOfLastCharacter] attributes:attributes];
        [attributedString appendAttributedString:[[NSAttributedString alloc] initWithString:trailingString 
                                                                                 attributes:@{
                                                                                              NSForegroundColorAttributeName: color,
                                                                                              NSFontAttributeName: attributes[NSFontAttributeName]
                                                                                              }]];
        return attributedString;
    } else {
        return [[NSAttributedString alloc] initWithString:self attributes:attributes];
    }
}

{% endhighlight %}

We can create a mutable copy of the string and subtract one character at a time, stopping when height of bounds fits the constraining height. We can use that index to reconstruct string that fits.

Performance is O(N), where N is the length of the string. This naive implementation isn't so bad if you know that your strings are limited to a certain character length, but if your string is a million characters and your frame fits 100, you might want to look at the next implementation.

# Truncation Method: Addition

{% highlight objc %}
- (NSAttributedString *)stringUsingAdditionToTruncateToSize:(CGSize)size
                                                 attributes:(NSDictionary *)attributes
                                             trailingString:(NSString *)trailingString
                                                      color:(UIColor *)color {
    
    if (![self willFitToSize:size trailingString:@"" attributes:attributes]) {

        NSMutableString *stringThatFits = [@"" mutableCopy];
        NSRange range = {0, 1};
        
        while ([stringThatFits willFitToSize:size trailingString:trailingString attributes:attributes]) {
            [stringThatFits insertString:[self substringWithRange:range] atIndex:range.location];
            range.location++;
        }
        
        NSInteger indexOfLastCharacterThatFits = range.location - range.length;
        
        NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:[stringThatFits substringToIndex:indexOfLastCharacterThatFits] attributes:attributes];
        [attributedString appendAttributedString:[[NSAttributedString alloc] initWithString:trailingString 
                                                                                 attributes:@{
                                                                                              NSForegroundColorAttributeName: color,
                                                                                              NSFontAttributeName: attributes[NSFontAttributeName]
                                                                                              }]];
        return attributedString;
    } else {
        return [[NSAttributedString alloc] initWithString:self attributes:attributes];
    }
}
{% endhighlight %}

We add one character at a time from the string and stop when height of bounds exceeds the constraining height. We use the index to reconstruct string that fits.
 
Performance: constant in the size parameter.

While this implementation should be sufficient for most cases (and the most efficient, for super long strings), I was excited to try one more implementation: binary search. I've known in theory that binary search is a useful algorithm for sorting problems, but I've never encountered the fabled Algorithm in the wild. 

This problem presented a rare opportunity to implement an algorithm outside the laboratory of computer science assignments.

# Truncation Mode: Binary Search

{% highlight objc %}
- (NSAttributedString *)stringUsingBinarySearchToTruncateToSize:(CGSize)size
                                                    attributes:(NSDictionary *)attributes
                                                trailingString:(NSString *)trailingString
                                                         color:(UIColor *)color {

    if (![self willFitToSize:size trailingString:@"" attributes:attributes]) {

        NSInteger indexOfLastCharacterThatFits = [self binarySearchForStringIndexThatFitsSize:size
                                                                                   attributes:attributes 
                                                                                     minIndex:0 
                                                                                     maxIndex:self.length 
                                                                               trailingString:trailingString];
    
        NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:[self substringToIndex:indexOfLastCharacterThatFits] attributes:attributes];
        
        // Construct string with attributed trailing string
        [string appendAttributedString:[[NSAttributedString alloc] initWithString:trailingString 
                                                                       attributes:@{
                                                                                    NSForegroundColorAttributeName: color,
                                                                                    NSFontAttributeName: attributes[NSFontAttributeName]
                                                                                    }]];
        return string;
    } else {
        return [[NSAttributedString alloc] initWithString:self attributes:attributes];
    }

}

- (NSInteger)binarySearchForStringIndexThatFitsSize:(CGSize)size 
                                         attributes:(NSDictionary *)attributes 
                                           minIndex:(NSInteger)minIndex 
                                           maxIndex:(NSInteger)maxIndex 
                                     trailingString:(NSString *)trailingString {
    /* 
     Invariants: 
     - height at minIndex <= size.height
     - height at maxIndex > size.height
    */
    
    NSInteger midIndex = (minIndex + maxIndex) / 2;
    NSString *subString = [self substringWithRange:NSMakeRange(0, midIndex)];
    
    // Invariant assertions
    // assert([[self substringWithRange:NSMakeRange(0, minIndex)] willFitToSize:size trailingString:trailingString attributes:attributes]);
    // assert(![[self substringWithRange:NSMakeRange(0, maxIndex)] willFitToSize:size trailingString:trailingString attributes:attributes]);
    
    if (maxIndex - minIndex == 1) {
        return minIndex;
    }
    
    // String is greater than constraining size, start search with minIndex as new maximum
    // The max index will always be greater than the size
    if (![subString willFitToSize:size trailingString:trailingString attributes:attributes]) {
        return [self binarySearchForStringIndexThatFitsSize:size attributes:attributes minIndex:minIndex maxIndex:midIndex trailingString:trailingString];
    }
    // String is less than constraining size, start search with midIndex as new minimum
    // The minimum index will be less than or equal to the size
    else {
        return [self binarySearchForStringIndexThatFitsSize:size attributes:attributes minIndex:midIndex maxIndex:maxIndex trailingString:trailingString];
    }
}
{% endhighlight %}

Using 0 and N as starting indices, where N is the length of the string, we perform a binary search that maintains the invariants that:

* height at minIndex <= size.height
* height at maxIndex > size.height
 
We return minIndex when minIndex and maxIndex are adjacent. Performance: log(N).

Hopefully, this saves you some heartache the next time you want to add a fancy "more" indicator. You can check out how the truncation works [here](https://github.com/mona-zsh/EMTruncation).