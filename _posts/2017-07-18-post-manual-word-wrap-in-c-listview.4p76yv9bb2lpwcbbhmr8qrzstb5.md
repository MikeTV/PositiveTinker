---
comments: true
date: 2017-07-18T22:33:00.0000000-05:00
description: C#’s *ListView* control does not support word wrap, sadly. Let's add it. The resulting extension method applies a generic pixel width to any string.
layout: post
tags:
- C#
title: Manual Word Wrap in C# ListView
permalink: manual-word-wrap-in-c
date_created: 2017-07-18T22:33:00.0000000-05:00
---
   
   
   
   
   
   

C#’s *ListView* control does not support word wrap, sadly. The [ObjectListView][1] wrapper control does, but each list item is rendered with same height. In one project of mine, the occasional list item will have 4-5 lines while the majority fit easily on a single row. How to maintain a pleasing UI but yet not hide long strings when they appear?   

One approach is to span a long message across multiple list items. This is, for instance, the technique used by TortoiseSVN for displaying [long error messages in operation lists][2]. This does means that we will have to wrap the text ourselves.   

Here’s an extension method that fits the bill:   

```csharp    
/// <summary>  
/// Brute-force word wrap.  Calls TextRenderer.MeasureText for each word in the text.  
/// Words too long to wrap will not be broken.  
/// </summary>  
/// <param name="text">Text to wrap</param>  
/// <param name="width">Width in pixels at which to wrap</param>  
/// <param name="font">Font of the control which will display the wrapped text</param>  
/// <returns>Multiple lines containint the wrapped text</returns>  
public static IEnumerable<string> WrapStringToPixelWidth(this string text, int width, System.Drawing.Font font)  
{  
    if (System.Windows.Forms.TextRenderer.MeasureText(text, font).Width <= width)  
    {  
        yield return text;  
    }  
    else  
    {  
        var tokens = text.Split(null); // null param == "split on whitespace"    
       string line = "";  
        foreach (var t in tokens)  
        {  
            string candidate = (line + " " + t).TrimStart();  
            if (System.Windows.Forms.TextRenderer.MeasureText(candidate, font).Width < width)  
            {  
                // Plenty of room for the token on this line  
                line = candidate;  
            }  
            else  
            {  
                // Line hasn't even started and already over max?  
                // The token must be longer than the max length for a line.  
                // Put the token on its own line.  
                if (line == "")  
                {  
                    yield return t;  
                }  
                else  
                {  
                    // Adding this token would put it past the bounds.  
                    // Put the token on the next line.  
                    yield return line;  
                    line = t;  
                }  
            }  
        }    
   
       if (line != "")  
        {  
            yield return line;  
        }  
    }  
}    
```   

Limitations:   

* Does not understand and correctly handle strings with newline characters. This could be resolved with an initial split on newlines and recursive calls to WrapStringToPixelWidth for each index.  
* This is not optimized for performance. For instance, it could be modified to estimate where to break each line based on the calculated length of the entire body of text, then work its way towards the correct wrap points in a binary search.    
  

[1]: http://objectlistview.sourceforge.net/cs/index.html
[2]: https://duckduckgo.com/?q=tortoisesvn+error&amp;t=hj&amp;iax=images&amp;ia=images