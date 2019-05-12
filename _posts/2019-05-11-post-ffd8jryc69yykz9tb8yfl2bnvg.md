---
comments: true
date: 2019-05-11T17:39:42.0000000-05:00
description: This should show as the description on the article listing
layout: post
title: Testing markdown conversion issues
date_created: 2019-05-11T17:03:40.0000000-05:00
---


[^my-named-footnote] Result

----

Footnote[^1]   
&nbsp;   
&nbsp;   
Manual footnote in editor[^2]
&nbsp;   
Named footnote [^my-named-footnote]
<i>Html in the page</i>   
&nbsp;   
{% highlight html %}
<i>HTML in the code block</i>
{% endhighlight %}   
&nbsp;   

****

Indented code block:   
{% highlight csharp %}
if (System.Windows.Forms.TextRenderer.MeasureText(text, font).Width <= width)  
 {  
 yield return text;  
 }
{% endhighlight %}   
&nbsp;   

***



[^1]: Footnote destination   
[^2]: Down
