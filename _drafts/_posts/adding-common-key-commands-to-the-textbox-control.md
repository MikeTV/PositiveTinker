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

```
SendKeys.Send("^+{LEFT}{BKSP}");

```

}
{% endhighlight %}

In the Send string, "^" means CTRL and "+" means SHIFT.  Basically, we're telling the textbox to simulate CTRL+SHIFT+Left, then Backspace.  This looks decent at first blush, but if the user *holds down* CTRL+Backspace...

{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.Back)
{
    e.SuppressKeyPress = true;
    SendKeys.Send("^+{LEFT}{BKSP}");
}
{% endhighlight %}

![](/uploads/2017/07/22/Textbox-Take1.gif)

The first backspace works as expected, but the repitition afterwards only deletes a single character as a time.  This is because the "^" command character doesn't only press CTRL, it also *releases* it.  The textbox no longer recognizes that the user is holding down the control key.

Since the user is actually holding down the control key, we don't need to send that command:

{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.Back)
{
    e.SuppressKeyPress = true;
    SendKeys.Send("+{LEFT}{BKSP}");
}
{% endhighlight %}

![](/uploads/2017/07/22/Textbox-Take2.gif)

Hmm.  The SendKeys command's use of Backspace is itself triggering the KeyDown event, causing a near-infinite loop.  Switching to using the Delete key instead:

{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.Back)
{
    e.SuppressKeyPress = true;
    SendKeys.Send("+{LEFT}{DEL}");
}
{% endhighlight %}

![](/uploads/2017/07/22/Textbox-Take3.gif)

Better!  Once the preceding text is all gone, however, there's nothing left to highlight.  In the absence of highlighted text, Delete starts removing text to the right of the caret.  One small condition will take care of this special case:

{% highlight csharp %}
if (e.Control && e.KeyCode == Keys.Back)
{
    e.SuppressKeyPress = true;

    if (textbox.SelectionStart > 0)
    {
        SendKeys.Send("+{LEFT}{DEL}");
    }
}
{% endhighlight %}

![](/uploads/2017/07/22/Textbox-Take4.gif)

And there you have it!  Altogether, this is the code I'm now adding to every new TextBox:
{% highlight csharp %}
// On form init
this.txtMyTextbox.KeyDown += new System.Windows.Forms.KeyEventHandler(TextBox_KeyDown_CommonKeyCommands);

/// <summary>
/// Adds support for the following TextBox key commands:
/// CTRL+A
/// CTRL+Backspace
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
public static void TextBox_KeyDown_CommonKeyCommands(object sender, KeyEventArgs e)
{
    var textbox = (sender as TextBox);
    if (textbox == null)
    {
        return;
    }

    // Add support for CTRL+A
    if (e.Control && e.KeyCode == Keys.A)
    {
        textbox.SelectAll();
    }

    // Add support for CTRL+Backspace
    if (e.Control && e.KeyCode == Keys.Back)
    {
        e.SuppressKeyPress = true;

        if (textbox.SelectionStart > 0)
        {
            /*
                * Piggyback off of the supported "CTRL + Left Cursor" feature.
                * Does not need to send {CTRL}, because the user is currently holding {CTRL}.
                * Uses {DEL} rather than {BKSP} in order to avoid creating an infinite loop.
                * NOTE: {DEL} has the side effect of deleting text to the right if the cursor is
                *       already as far left as it can go, since no text will be selected by {LEFT}.
                *       The .SelectionStart > 0 condition prevents this side effect.
                */
            SendKeys.Send("+{LEFT}{DEL}");
        }
    }
}
{% endhighlight %}

<hr>

[^1]: Yes, RichTextBox supports these key commands intrinsically, but is needlessly complex for many applications and comes with its own set of problems.

[^2]: Actually, Control-Backspace *is* supported when [auto-complete is enabled](https://stackoverflow.com/a/30269663/3320402), but only if the textbox is not multiline. So close!