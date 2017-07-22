---
title: Adding Common Key Commands to the Textbox Control
comments: true
date: 2017-07-22 16:14
layout: post
description: ''
tags: []
---
On today's episode of "basic things .NET form controls don't support natively" we look at the humble Textbox.  Two common key commands are mysteriously missing: Control-A to select all text and Control-Backspace[^1] to delete the previous word.

Never fear, we can add both of these through a KeyDown event handler!

Control-A is super simple:
{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.A)
{
textbox.SelectAll();
}
{% endhiglight %}

Control-Backspace is deceptively complex.  It appears to follow a somewhat convoluted set of rules: Erase back to the

[^1]:  Actually, Control-Backspace *is* supported when [auto-complete is enabled](https://stackoverflow.com/a/30269663/3320402), but only if the textbox is not multiline. So close!