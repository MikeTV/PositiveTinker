---
comments: true
date: 2019-05-12T16:29:00.0000000-05:00
description: This should show as the description on the article listing
layout: post
title: Testing markdown conversion issues
date_created: 2019-05-11T17:03:40.0000000-05:00
---
   
   
Footnote[^1]   

****   

----   

<i>Html in the page</i>   

Line break right under text   
----   

****   
And above text   

Footnote[^2]   

{% highlight html %}
<i>HTML in   the code block</i>
{% endhighlight %}   

Indented code block:   
{% highlight csharp %}
if   (System.Windows.Forms.TextRenderer.MeasureText(text, font).Width <=   width)  
        {  
            yield return text;  
        }
{% endhighlight %}   

Built-in syntax highlighting?   

```csharp     
    if   (System.Windows.Forms.TextRenderer.MeasureText(text, font).Width <=   width)  
        {  
            yield return text;  
        }     
```   

**Test bold**    
*Test italic*    
<u>Test underline</u>    
***<u>Test all 3</u>***    

[^1]: Footnote destination   

Footnote right under text:   
[^2]: Under text   

