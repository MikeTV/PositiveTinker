---
title: Adding Common Key Commands to the Textbox Control
comments: true
date: 2017-07-22 16:14
layout: post
description: ''
tags: []
---
On today's episode of "basic things .NET form controls don't support natively" we look at the humble Textbox[^1].  Two common key commands are mysteriously missing: CTRL+A to select all text and CTRL+Backspace[^2] to delete the previous word.

Never fear, we can add both of these through a KeyDown event handler!

CTRL+A is super simple:
{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.A)
{
    textbox.SelectAll();
}
{% endhighlight %}

CTRL+Backspace is deceptively complex.  It appears to follow a somewhat convoluted set of rules: Start removing characters to the left from the caret until reaching a breaking character (whitespace, punctuation, or separator).  Unless the character immediately preceding the carat is a breaking character, in which case remove breaking characters to the left until reaching a non-breaking character.  Unless the breaking character preceding the carat is a space or tab, in which case remove breaking characters to the left until reaching a non-breaking character, and then *continue* removing characters to the left until reaching a breaking character again.

That's the point at which I stopped trying to reverse-engineer and reimplement CTRL+Backspace from scratch.

Approaching the problem from a different angle, what's something that's supported by Textbox that follows almost the same rules?  CTRL+Left (left arrow)!  To simulate a CTRL+Backspace, then, we need only highlight text using CTRL+Left, then delete it.

First attempt:
{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.Back)
{
    e.SuppressKeyPress = true;

    SendKeys.Send("^+{LEFT}{BKSP}");
}
{% endhighlight %}

This looks 


***
[^1]: Yes, RichTextBox supports these key commands intrinsically, but is needlessly complex for many applications and comes with its own set of problems.

[^2]: Actually, Control-Backspace *is* supported when [auto-complete is enabled](https://stackoverflow.com/a/30269663/3320402), but only if the textbox is not multiline. So close!