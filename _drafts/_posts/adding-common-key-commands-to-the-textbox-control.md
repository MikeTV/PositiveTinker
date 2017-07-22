---
title: Adding Common Key Commands to the Textbox Control
comments: true
date: 2017-07-22 16:14
layout: post
description: ''
tags: []
---


On today's episode of "basic things .NET form controls don't support natively" we look at the humble Textbox.  Two common key commands are mysteriously missing: Control-A to select all text and Control-Backspace to delete the previous word.

Never fear, we can add both of these through a KeyDown event handler!

Control-A is super simple:

