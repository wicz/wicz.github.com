---
layout: post
title: Hello Alfred App, how can I completely disable Spotlight?
published: false
---

__UPDATE 2011-05-10:__ STOP RIGHT NOW! After a few days I wrote this, I realized that Alfred __uses__ the metadata services, which means if you disable it, Alfred will stop working. Why I couldn't find anything about that in their docs? I'm still using it, though I'm not sure if I'll keep it.

So, finally I'm giving [Alfred App](http://www.alfredapp.com) a try. It reminds me a lot the (discontinued?) [Enso](http://www.humanized.com/enso/) from Humanized.

Since I'll be using Alfred in favor of Spotlight, I want to disable the latter. Googling a bit gives us a lot of ways to do it, but all looks a bit intrusive and destructive to me. I don't want to delete Spotlight, I just want to disable it!

Spotlight consists of a set backend and frontend tools. Metadata server (mds), Metadata worker (mdworker), etc. do the hard work of indexing the volumes and monitoring file changes. The frontend is basically the small loupe that appears in the menu bar (that's enough, we don't need all the details.)

The first thing is to disable all backend. You can stop the mds from indexing your volumes:

{% highlight bash %}
$ sudo mdutil -a -v -i off
{% endhighlight %}

OK, but it won't stop Spotlight from initiating after a system restart. According to other solutions, you could delete files from `/System/Library/Frameworks/CoreServices.framework/Frameworks/Metadata.framework/Support`, but since I don't want to destroy anything, I chose to reset the permissions:

{% highlight bash %}
$ sudo chmod 000 /System/Library/Frameworks/CoreServices.framework/Frameworks/Metadata.framework/Support/*
{% endhighlight %}

Great! `ps aux | grep md` shows no metadata processes running. `dmesg` shows no error. But looking at Console.app (/var/log/system.log) I see that launchd is trying to start `mds` every 10 seconds!

{% highlight console %}
May 3 11:41:11 wicz com.apple.launchd[1] (com.apple.metadata.mds[879]): posix_spawn("/System/Library/Frameworks/CoreServices.framework/Frameworks/Metadata.framework/Support/mds", ...): Permission denied
May 3 11:41:11 wicz com.apple.launchd[1] (com.apple.metadata.mds[879]): Exited with exit code: 1
May 3 11:41:11 wicz com.apple.launchd[1] (com.apple.metadata.mds): Throttling respawn: Will start in 10 seconds
{% endhighlight %}

Well, I don't wanna lose those CPU cycles. So `chmod` isn't a acceptable solution and neither is deleting those files. Probably the error would be `No such file or directory`.

The acceptable solution is to disable `mds` in `launchd`. So, RTFM! `manpage launchctl`* has the answer we're looking for:

{% highlight bash %}
$ sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.metadata.mds.plist
{% endhighlight %}

Now to the frontend. I didn't find a way to disable this nicely. It seems to be somehow hard coded in the SystemUIServer.app. So here we have to use the `chmod` approach. At least it is just a one time failure.

{% highlight console %}
5/3/11 11:31:46 AM SystemUIServer[256] Error loading /System/Library/CoreServices/Search.bundle/Contents/MacOS/Search: dlopen(/System/Library/CoreServices/Search.bundle/Contents/MacOS/Search, 265): no suitable image found. Did find: /System/Library/CoreServices/Search.bundle/Contents/MacOS/Search: open() failed with errno=13
{% endhighlight %}

OK, errno=13 is an EACCES Permission denied. No loupe appears on menu bar. But I wonder if it will crash future system updates. So I ended up just renaming the damn thing.

{% highlight bash %}
$ cd /System/Library/CoreServices/Search.bundle/Contents/MacOS
$ sudo mv Search _Search
{% endhighlight %}

It seems everything is fine till now. If something stop working I'll update this post. And if you know something I'm missing, please let me know in the comments.

*Just a few notes. Reading the launchctl manpage it says:

{% highlight console %}
-w Overrides the Disabled key and sets it to true. In previous versions, this option would modify the configuration file. Now the state of the Disabled key is stored elsewhere on-disk.
{% endhighlight %}

WTF!? "elsewhere on-disk"? FWIW, the "elsewhere" is `/private/var/db/launchd.db/com.apple.launchd.peruser.<userid>`.

And launchctl looks at many different places for .plist files. So if you're getting errors from apps that you've already uninstalled but are still trying to start, you might want take a look into:


{% highlight console %}
~/Library/LaunchAgents -- Per-user agents provided by the user.
/Library/LaunchAgents -- Per-user agents provided by the administrator.
/Library/LaunchDaemons -- System wide daemons provided by the administrator.
/System/Library/LaunchAgents -- Mac OS X Per-user
/System/Library/LaunchDaemons -- Mac OS X System wide daemons.
{% endhighlight %}

