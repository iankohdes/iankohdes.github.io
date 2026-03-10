---
title: "How I got into programming"
date: 2999-12-31
draft: false
tags: ["python", "reflections"]
---

{{<lead>}}
I wrote my first line of code while doing my undergraduate studies. It wasn’t my favourite thing to do.
{{</lead>}}

It was early 2012 in Singapore, and I found myself enrolled in a computational linguistics module called _Language and the Computer_. We were taught by a stout Australian professor who allowed us to eat during lectures and lab sessions as long as we shared our food with him. (I don’t recall anyone taking him up on that offer, and thus no one ate in class. Effective classroom management, this.)

_Language and the Computer_ was a peculiar module. It was taught in the linguistics department, whose students weren’t normally expected to think algorithmically. Most of us had never even programmed a computer before, with two exceptions: a final-year computer science major who’d enrolled for a free A (understandable, since my university graded on a curve), and a fellow third-year whose dad sent him to C classes when he was a teenager.

The module was taught in Python and introduced us to working with [text corpora](https://en.wikipedia.org/wiki/Text_corpus). We started with a few weeks of an introduction to Python. Imagine a whirlwind introduction to the language and general programming constructs, followed by a swift transition to the [Natural Language Toolkit (NLTK)](https://www.nltk.org/), to be used alongside those barely-existing programming fundamentals.

Now imagine writing code in Notepad++, running it in [IDLE](https://docs.python.org/3/library/idle.html), not using Git (or even knowing about its existence), being thoroughly confused by loops, and not understanding the big deal about regular expressions even though the module dedicated an entire week to that concept. That was me.

{{< alert "comment" >}}
All I remember about regular expressions back then is a quote advising the judicious use of regexes, lest one end up with two problems instead of the original one.
{{< /alert >}}

I can’t even remember if we wrote our code in Python 2 or Python 3, since at that point Python 3 was fairly new and most Python code was still of the older variety. I get the feeling that we used Python 2.

Eventually, it was time for us to work on our final project, which was to be individually done and submitted. The details are vague, but we had to use the NLTK to parse and tokenise and tag some text corpus and then do something with the refined data. (Can’t quite remember.) I was hoping that I still had my code somewhere so that I could read it and cringe / laugh at my noobness.

Alas, I couldn’t find it in my hard drive, even though my other bachelor’s module materials were there. The code must have been stored on a USB stick, since I remember using the university computers to work on my project. It has most certainly been misplaced by now. We can add an inability to back up my code as one of my early failings back then, too.

If it weren’t for that fourth-year CS student who kindly helped me, I would have failed the module. He was Vietnamese and had a younger brother whom he banned from using the calculator while the latter was learning maths at school. He once told me his dad fought in the Vietnam War on the side of the North Vietnamese army (could’ve also been the Vietcong). When I heard that, I recalled that one scene from _Top Gear_’s Vietnam special, where Jeremy Clarkson spoke to an older Vietnamese man who was writing in the sand to describe his experiences in the war. (I don’t know why I remember these random facts but didn’t have it in me to back up my bloody work.)

By the end of the semester, I had got to know three fellow classmates quite well. One of them was the one whose dad made him learn C. As we finished the first draft of our final project’s code and ran it on the corpus, one thing became painfully clear: our code was _slow_. This was partly because Python wasn’t particularly fast at the time, and mostly because we were simply writing highly inefficient programs.

Let me give you some numbers to illustrate just how bad we were. My code took three to four hours to do what it needed to do with the corpus data. Two of the abovementioned classmates had similar running times. The one who learnt C had code that ran quicker, I think, but it still took over an hour.

The Vietnamese CS student who was in our module for a free A, however, wrote code that finished executing _within ten seconds_. He ran his code in front of me, which is when I realised that I was thoroughly cooked. It was one of those times where the disparity in skill was so vast that all I could do was laugh.

Regardless, I passed _Language and the Computer_ with a highly undeserved B. To Nguyen Duc: I don’t know where in the world you are today, but thanks a million for helping me write parts of my code. You didn’t have to as it was your last semester before graduation, but you did so anyway, and that kept me from swearing off programming forever (in addition to helping me pass).

This is how I got started with programming. I’ve got better these days: I now know how to push poorly-written code to a remote Git repository for permanent display and archival. {{< icon "github" >}}