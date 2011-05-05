---
layout: post
title: HAI SPARROW, CAN I HAS FIXED WIDTH FONTZ? KTHXBYE
---

Gotta say, I just love the [Sparrow Mail App](http://www.sparrowmailapp.com/)! It's interface is simple and clean. It just does everything you need, and nothing you don't.

Only one thing I can complain: Why in hell I can't set a fixed width font for plain text messages? I don't like HTML emails and I never user that fancy editor. I like plain old fixed fonts! Just like my terminal.

If you're like me, dark days are over! All you need is tweak some CSS and voila! Inside `/Applications/Sparrow.app/Contents/Resources/`

    #!css
    // message-editing.css
    body { font-family: Monospace; }

    #!css
    // conversation.css
    div.messageBody { font-family: Monospace; }
    div.quickReplyTextContents { font-family: Monospace; }
