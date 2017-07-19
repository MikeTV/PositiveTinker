---
title: Manual Word Wrap in C#
comments: true
date: 2017-07-18 21:48
layout: post
description: ''
tags: []
---
C#'s *ListView* control does not support word wrap, sadly.  The [ObjectListView](http://objectlistview.sourceforge.net/cs/index.html) wrapper control does, but each list item has to be the same height.

Another approach is to span a long message across multiple list items.  This is, for instance, the approach used by [TortoiseSVN for displaying long error messages](https://www.google.com/search?tbm=isch&q=tortoisesvn+error).  This means we have to calculate the points at which to wrap.

Here's a good-enough approach:

