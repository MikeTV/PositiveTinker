---
comments: true
date: 2017-07-04T22:24:00.0000000-05:00
description: 'Symptom: Thunderbird sometimes hangs when saving an email draft (including auto-save) or sending an email. Culprit: Windows 10 Indexing Service Solutions.'
layout: post
tags:
- debugging
title: Thunderbird Hangs When Saving Draft
permalink: thunderbird-hangs-when-saving-draft
date_created: 2017-07-04T22:24:00.0000000-05:00
---
   
   
   
   
   
   
   
&nbsp;   
Been having an issue lately with Thunderbird hanging, sometimes several times an hour. Always it’s while I’m writing an email. Always the latest (or nearly-latest) version of the email is saved in the Drafts folder, which led me to believe that it had something to do with the process of saving a draft.   
&nbsp;   
Following [Mozilla's testing guidelines][1], first test was using Thunderbird in Safe Mode. Same problem, so it wasn’t the fault of any extensions. Also excluded Windows Defender from scanning the Thunderbird profile folder, to no avail.   
&nbsp;   
Number 6 on the list mentions the Windows Indexer may be causing the problem, and the [linked Bugzilla entry][2] describes the problem as occurring while writing emails, sending emails, or saving drafts. Also, this is apparently particularly a problem in Windows 10. That checks all the boxes!   
&nbsp;  
# Solution 1: Hit it with a very large hammer.   
Since I use [Everything][3] for file search almost exclusively, and Indexing provides questionable benefit on SSDs anyways, I elected to completely disable the Windows Search service.   
&nbsp;  
1. Win+R, type “services.msc”  
2. Right click “Windows Search”, click “Properties”  
3. Click “Stop”  
4. Set “Startup Type” to “Disabled”    
   
&nbsp;   
[![image001.png][4]][4]   
&nbsp;   
Problem solved!   
&nbsp;   
…with one gotcha. Apparently Microsoft OneNote relies on Indexing for its search, and I rely on OneNote. What used to be instant is now 10-15 seconds of matching notes slowly trickling in.   
&nbsp;  
# Solution 2: Tap It with a Small Hammer.   
&nbsp;   
Ok, re-enabled indexing. Now to chain it to OneNote.  
1. Start -> Indexing Options    
  
2. Modify  
3. Uncheck everything except the line starting with “oneindex”  
4. Click OK  
5. Repeat steps 2-4 until everything is *actually&nbsp;* checked and the main window shows zero included locations    
   
&nbsp;   
Assuming that “oneindex16” is the location for OneNote/OneDrive, since it is prefixed with the word “one”.   
&nbsp;   
[![image002.png][5]][5]   
&nbsp;   
If you wanted to be truly precise about this, and wanted to keep Indexing active, you could check everything except for the Thunderbird profile directory. That’d probably still solve the original issue.   
&nbsp;   
After a short while, the number of “items indexed” displayed on the main screen dropped to a relatively small number. As a final test I opened OneNote and searched for a random string - bam, instant response.   
&nbsp;   
[![image003.png][6]][6]   
&nbsp;   
[![image004.png][7]][7]&nbsp;Has Thunderbird hung since then? This was 2017-07-12. Still running smoothly now that the Users directory is excluded? 2019-05-11 Yep.   
   

[1]: https://wiki.mozilla.org/Thunderbird:Testing:Memory_Usage_Problems
[2]: https://bugzilla.mozilla.org/show_bug.cgi?id=1262517
[3]: https://www.voidtools.com/
[4]: /uploads/2017/07/04/image001.png "image001.png"
[5]: /uploads/2017/07/04/image002.png "image002.png"
[6]: /uploads/2017/07/04/image003.png "image003.png"
[7]: /uploads/2017/07/04/image004.png "image004.png"