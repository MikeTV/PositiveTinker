---
comments: true
date: 2017-07-15 15:07
description: |-
  Symptom: Thunderbird sometimes hangs when saving an email draft (including auto-save) or sending an email.
  Culprit: Windows 10 Indexing Service
  Solutions and side-effects follow.
layout: post
title: 'Debugged: Thunderbird Hangs when Saving Draft'
---


Been having an issue lately with Thunderbird hanging, sometimes several times in an hour.  Always it's while I'm writing an email.  Always the latest (or nearly-latest) version of the email is saved in the Drafts folder.  This led me to believe that it had something to do with the process of saving a draft.

Following [Mozilla's testing guidelines](https://wiki.mozilla.org/Thunderbird:Testing:Memory_Usage_Problems), first test was using Thunderbird in Safe Mode.  Same problem, so it wasn't the fault of any extensions.  Also excluded Windows Defender from scanning the Thunderbird profile folder, to no avail.

Number 6 on the list mentions the Windows Indexer may be causing the problem, and the [linked Bugzilla entry](https://bugzilla.mozilla.org/show_bug.cgi?id=1262517) describes the problem as occurring while writing emails, sending emails, or saving drafts.  Also, this is apparently particularly a problem in Windows 10.  That checks all the boxes!

### Solution 1: Hit It with a Very Large Hammer.

Since I use [Everything](https://www.voidtools.com/) for file search almost exclusively, and Indexing provides questionable benefit on SSDs anyways, I elected to completely disable the Windows Search service.

1. Win+R, type "services.msc"

1. Right click "Windows Search", click "Properties"

1. Click "Stop"

1. Set "Startup Type" to "Disabled"

<img src="/uploads/2017/07/15/Thunderbird_Services1.png" alt="" class=" forestry--none" style="float: none;">

Problem solved!

...with one gotcha.  Apparently Microsoft OneNote relies on Indexing for its search, and I rely on OneNote. What used to be instant is now 10-15 seconds of agony watching matching notes slowly trickle in.

### Solution 2: Tap It with a Small Hammer.

Ok, re-enabled indexing. Now to chain it to OneNote.

1. Start -> Indexing Options

1. Modify

1. Uncheck everything except the line starting with "oneindex"

1. Click OK

1. Repeat steps 2-4 until everything is *actually *checked and the main window shows zero included locations

Assuming that "oneindex16" is the location for OneNote/OneDrive, since it is prefixed with the word "one".

![](/uploads/2017/07/15/Thunderbird_Services2.png)

If you wanted to be truly precise about this, and wanted to keep Indexing active, you could check everything except for the Thunderbird profile directory. That'd probably still solve the original issue.

After a short while, the number of "items indexed" displayed on the main screen dropped to a relatively small number.  As a final test I opened OneNote and searched for a random string - bam, instant response.

![](/uploads/2017/07/15/Thunderbird_Services3.png)
