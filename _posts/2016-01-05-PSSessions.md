---
layout: post
title: PSSessions & Invoke-Command
---

Earlier today I saw a post on StackOverflow regarding PSSessions and when & how the various PSSession cmdlets should be used. Rather than duplicate efforts, please refer to the conceptual "about" topic [about_PSSessions](https://technet.microsoft.com/en-us/library/hh847839.aspx) for more information.

![About_PSSessions](/images/2016-01-05-220000-AboutPSSessionsHelp.jpg)

All caught up? Good. Now let me tell you how I use the PSSessions cmdlets.

Without a doubt I easily use the [``Invoke-Command``](https://technet.microsoft.com/en-us/library/hh849719.aspx) cmdlet more than any other PSSession cmdlet. This is primarily because I *rarely* require persistent connections to any set of computers. 

The notable exceptions are Exchange and Lync. To remotely manage Exchange or Lync via PowerShell one needs to make use of what's referred to as [Implcit Remoting](http://blogs.technet.com/b/heyscriptingguy/archive/2013/09/08/remoting-the-implicit-way.aspx). The concept basically boils down to creating a persistent PSSession to a computer and then importing a module from the remote session into your local session.

Of course there are quite a few other ways to use PSSessions other than the two ways I listed above. But perhaps I'll save those for future blog posts. ;)

If you're curious how I use PSSessions, feel free to look at [my GitHub repository](https://github.com/psrdrgz/powershell). In general you should see that most of the [advanced functions](https://technet.microsoft.com/en-us/library/hh847806.aspx) I've created are simply wrapper functions which use ``Invoke-Command``.

