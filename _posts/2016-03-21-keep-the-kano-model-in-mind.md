---
layout: post
title: Keep the Kano model in mind
description: The Kano model of customer satisfaction is a powerful tool, you should be aware of to understand customers’ needs’. It helps you to realize basic needs and to come up with real game changing ideas.
keywords: Kano model, customer satisfaction, customers, needs, requirements engineering, performance features, excitement features, basic features, mockups, wireframes, lateral thinking, brainstorming, Noriaki Kano, 
---

There are two kinds of learning: the kind where you understand an idea instantly, and the kind where you realize its meaning weeks, months or even years later. The __Kano model  of customer satisfaction__ absolutely fell in the second category for me. When I first heard of this model half a year ago in a lecture about requirement engineering, I didn’t really give it a thought, ticking it off as ‘well, only the hundredth slide about how hard it is to understand customers’ needs’. 

This attitude changed some weeks ago during a __meeting with a customer__. The target was to figure out his needs, but it was hard for me to follow my client and __I just didn’t get the big picture__. We moved to the whiteboard and after a couple of ‘oh, I thought this was clear’  exchanges, my understanding got a lot better. Suddenly the Kano model popped up in my head again. I had this feeling of epiphany you get when you suddenly realize the essence of a concept, and everything seems super clear and easy at once. This ‘how could I ever write PHP code?’ feeling. It’s a great feeling. 


## The Kano model

The [Kano model]("https://en.wikipedia.org/wiki/Kano_model") is a theory developed in the 1980s by [Noriaki Kano]("https://en.wikipedia.org/wiki/Noriaki_Kano"). It’s visualized as a simple two-axis grid, measuring investment against customer satisfaction. Basically, it describes three distinct categories. As we are software developers, let’s call them features: 

- ### Performance features
You would expect your product to earn 100% customer satisfaction if you deliver 100% implementation. This is true for performance features. These are the features users normally describe to you when they think about the desired software. The better your product performs in delivering these features, the more satisfaction they bring – and conversely, the worse these functions are performed, the more dissatisfaction they bring. For example, software often has the goal of making internal workflows faster. The requirement is clear, and the challenge is to fulfil this request as well as possible. The faster a workflow can be finished, the more customer satisfaction you earn. That’s easy! 

- ### Basic features  
Basic features or __must-haves__, or as I like to call them, the _I-thought-that-was-obvious-features_ are different. A good example I’ve come across in my daily work is the undo and redo feature. In these times of WYSIWYG editors and MS Office, this functionality has become a basic feature for most of us. But when they demand a software, your customers won’t tell you that they expect this feature to be included. This gets __really difficult for inexperienced participants working in unfamiliar domains__. But the really frustrating thing about basic features is that __customers see them as a matter of course__. Customers feel just neutral if these functions are performed well, but they are dissatisfied when basic features are done poorly.

- ### Excitement features  
These features are also called game changers, or as I like to call them, _the-OMG-that’s-great-features_. They __delight customers in an unexpected way__, but would have zero negative impact if they were absent. These are often details that __make a product unique among its competitors__, and they contribute 100% to positive customer satisfaction. Remember smartphones before multitouch? Laptops before solid state drives? The sad thing about these features is that after some time, they will turn into performance or even basic features and customers will expect them. This is the way the world is going.

{% include image.html url="/images/kano-model.jpg" description="the kano model classifies customer preferences into three main categories." %}

## The salient point

As you are familiar with these three categories, you may be asking yourself, ‘Nice, but where’s the excitement feature in this post?’ Well, the __real essence__ of the Kano model is that __customers will usually tell you about only the performance features__. Sound familiar? Well, I don’t have the statistics, but I would bet 80% of customer requirements consist of performance features. That’s why the __supreme challange is to figure out those basic and excitement features__ that customers want, because these features can make the difference between good software and the Windows metro calculator. 

## Choose the right technique

There are __many good techniques__ out there to help you figure out which basic features will satisfy your customers, and it’s up to you to find your preferred method. In addition to __agile development__, I’ve had good experiences with using __mockups__ or __wireframes__ to figure out basic features. 
Excitement features are __harder to find__. They can be like small gold nuggets in the huge blue ocean. Of course, these features can also be discovered through the approaches mentioned above for finding basic features, but in my opinion, only __creativity techniques__ which lead you to think outside the box, like __brainstorming__ or __[lateral thinking]("https://en.wikipedia.org/wiki/Lateral_thinking")__, will help you come up with __real game changing ideas__.
