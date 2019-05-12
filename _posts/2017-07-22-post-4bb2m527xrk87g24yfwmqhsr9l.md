---
comments: true
date: 2017-07-22T18:51:00.0000000-05:00
description: 'Once in a while you get blindsided by a feature that*seems*  ubiquitous, but is somehow missing from standard libraries. Two of those are keyboard shortcuts found on the the WinForms Textbox: CTRL+A (to select all text) and CTRL+Backspace (to delete the previous word). Never fear, we can add both of these through a KeyDown event handler!'
layout: post
tags:
- C#
title: Adding CTRL-A and CTRL-Backspace Support to the WinForms Textbox Control
permalink: adding-common-key-commands-to-the-textbox-control
date_created: 2017-07-22T18:51:00.0000000-05:00
---
  
  
  
  
  
  

Once in a while you get blindsided by a feature that* seems* ubiquitous, but is somehow missing from standard libraries. Two of those are keyboard shortcuts found on the the WinForms Textbox[^1]: CTRL+A (to select all text) and CTRL+Backspace[^2] (to delete the previous word). Never fear, we can add both of these through a KeyDown event handler!  

CTRL+A is super simple to implement:  
```csharp  
if (e.Control && e.KeyCode == Keys.A)  
{  
    textbox.SelectAll();  
}  
```  

CTRL+Backspace behavior is deceptively complex. Implementations appear to follow a particular set of rules: Start removing characters to the left from the caret until reaching a breaking character (whitespace, punctuation, or separator). Unless the character immediately preceding the carat is a breaking character, in which case remove breaking characters to the left until reaching a non-breaking character. Unless the breaking character preceding the carat is a space or tab, in which case remove breaking characters to the left until reaching a non-breaking character, and then *continue* removing characters to the left until reaching a breaking character again.  

After the third rewrite to handle a new branch in logic, I stopped trying to reimplement CTRL+Backspace from scratch.  

Approaching the problem from a different angle… what’s something that’s supported by Textbox that follows almost the same rules? CTRL+Left Arrow! To simulate a CTRL+Backspace, then, we need only highlight text using CTRL+SHIFT+Left, then delete it.  

First attempt:  
```csharp  
if (e.Control && e.KeyCode == Keys.Back)  
{  
    e.SuppressKeyPress = true;  
    SendKeys.Send("^+{LEFT}{BKSP}");  
}  
```  

In the Send string, “^” means CTRL and “+” means SHIFT. Basically, we’re telling the textbox to simulate CTRL+SHIFT+Left, then Backspace. This looks decent at first blush, but if the user *holds down*  CTRL+Backspace…  

[![image001.gif][1]][1]  

The first backspace works as expected, but the repetition afterward only deletes a single character at a time. This is because the “^” command character doesn’t only press CTRL, it also *releases*  it. The textbox no longer recognizes that the user is holding down the control key.  

Since the user is conveniently holding down the physical control key, we don’t need to send that command:  

```csharp  
if (e.Control && e.KeyCode == Keys.Back)  
{  
    e.SuppressKeyPress = true;  
    SendKeys.Send("+{LEFT}{BKSP}");  
}  
```  
[![image002.gif][2]][2]  

Hmm. The SendKeys command’s use of Backspace is itself triggering the KeyDown event, causing a near-infinite loop. Switching to using the Delete key instead:  
```csharp  
if (e.Control && e.KeyCode == Keys.Back)  
{  
    e.SuppressKeyPress = true;  
    SendKeys.Send("+{LEFT}{DEL}");  
}  
```  
[![image003.gif][3]][3]  

Better! Once the preceding text is all gone, however, there’s nothing left to highlight. In the absence of highlighted text, Delete starts removing text to the right of the caret. One small condition will take care of this special case:  
```csharp  
if (e.Control && e.KeyCode == Keys.Back)  
{  
    e.SuppressKeyPress = true;  
if (textbox.SelectionStart > 0)  
    {  
        SendKeys.Send("+{LEFT}{DEL}");  
    }  
}  
```  
[![image004.gif][4]][4]  

And there you have it! Altogether, this is the code I’m now adding to every new TextBox:   
```csharp  
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
```  

Further Reading:  
 
* [Stack Overflow: Winforms Textbox - Using Ctrl-Backspace to Delete Whole Word ][5] 
* [MSDN Blog “Old New Thing”: Whose idea was it to make Ctrl+Backspace delete the previous word? ][6]  
  

****  

Footnotes:  

[^1]: Yes, RichTextBox supports these key commands intrinsically, but is needlessly complex for many applications and comes with its own set of problems.  
[^2]: Actually, Control-Backspace *is*  supported when [auto-complete is enabled][7], but only if the textbox is not multiline. So close!  

[1]: /uploads/2017/07/22/Textbox-Take1.gif "image001.gif"
[2]: /uploads/2017/07/22/Textbox-Take2.gif "image002.gif"
[3]: /uploads/2017/07/22/Textbox-Take3.gif "image003.gif"
[4]: /uploads/2017/07/22/Textbox-Take4.gif "image004.gif"
[5]: https://stackoverflow.com/questions/1124639/winforms-textbox-using-ctrl-backspace-to-delete-whole-word/1197339#1197339
[6]: https://blogs.msdn.microsoft.com/oldnewthing/20071011-00/?p=24823
[7]: https://stackoverflow.com/a/30269663/3320402