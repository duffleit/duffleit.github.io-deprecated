---
layout: post
title: The Refactoring Tunnel  
description: The Refactoring Tunnel is a simple but brilliant metaphor for one of the biggest dangers refactoring comes with. 
keywords: Refactoring, Babysteps, Divide and Conquer
---

Refactoring is an important discipline in a software engineer’s range of skillset. A lot of solid knowledge has been shared about this topic, so I won’t go into detail about refactoring itself, but I will focus on one of the biggest dangers it comes with. I will explain this by using a metaphor, which I first heard of from [Harry Roberts](https://csswizardry.com/), which is simple and brilliant at the same time, worthy of having some lines spent on it.

Think of entering a tunnel. When you do this, you will probably not see the light at the end of it yet, but the light of the entry is right behind you. This light provides a solid foundation and acts as a source of good feeling to keep you going. Only for a moment. Just another hour. Just another day. But, at some point, you will realize that there is no light behind you anymore and in most cases, you still won’t see the end either. Without any light to lead you on, __you will lose your direction__, and your only but innermost wish would be to find a way out.

Of course, this story could have been better, but I think you get the idea behind it. In real life, we would probably never enter a tunnel without knowing where it ends. So why do we do it during refactoring? Probably because of having a good tool support. As a developer, you have the vast advantage of not having to live the rest of your life in a dark tunnel; you will just hit something like `git reset --hard`. This hurts too, but is obviously not enough. That’s why I like this metaphor – it always reminds me to check if there is a light at the end of this tunnel (refactoring) before I enter it. If not, I have to pass the mountain of legacy code through another path, consisting of shorter tunnels. 

As so often, this story can be summed up with other concepts like [Divide and Conquer](https://effectivesoftwaredesign.com/2011/06/06/divide-and-conquer-coping-with-complexity/) or [Taking Babysteps](https://dzone.com/articles/baby-steps-reverse-refactoring). But I like metaphors. And for no reason, I also like tunnels. 

{% include image.html url="/images/refactoring-tunnel.jpg" description="In the middle of a Refactoring Tunnel, surrounded by legacy code." by="Adam.J.W.C." by-url="https://en.wikipedia.org/wiki/Tunnel#/media/File:Hill_60_illowra_battery_port_kembla.jpg" licence="CC BY 3.0" licence-url="https://creativecommons.org/licenses/by/3.0/" %}