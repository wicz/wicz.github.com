---
layout: post
title: HAI SPARROW, CAN I HAS FIXED WIDTH FONTZ? KTHXBYE
published: false
---

__UPDATE 2011-12-15:__ I'm no longer using the trial version, like said in the comments months ago. I'm using this with Sparrow Version 1.5 (1043) and it's working flawless. No code signatures problems either.

Gotta say, I just love the [Sparrow Mail App](http://www.sparrowmailapp.com/)! It's interface is simple and clean. It just does everything you need, and nothing you don't.

Only one thing I can complain: Why in hell I can't set a fixed width font for plain text messages? I don't like HTML emails and I never use that fancy editor. I like plain old fixed fonts! Just like my terminal.

If you're like me, dark days are over! All you need is tweak some CSS and voila! Inside `/Applications/Sparrow.app/Contents/Resources/`

{% highlight css linenos=table linenospecial=1 %}
// message-editing.css
body { font-family: Monospace; }

// conversation.css
div.-sparrow-messageBody { font-family: Monospace; }
div.-sparrow-quickReplyTextContents { font-family: Monospace; }
{% endhighlight %}
