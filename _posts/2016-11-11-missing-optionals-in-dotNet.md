---
layout: post
title: Missing Optionals in .Net  
description: Optionals are a great concept, known primary from functional languages like F# - is there a need to use them in OOP languages like C#?
keywords: Optionals, Mabye, Option, donNet, F#, C#, OOP, Exceptions
---

Let's start with something like: 

```csharp
Image LoadImage(Url imageUrl)
```

This is a simple LoadImage-method, which takes an Url and returns an Image - quite __readable and easy to understand__. We don’t need any implementation or documentation to understand what this method does. Isn’t that what we expect from __clean code__?

One reason for the superior readability is that method signatures define obvious what they expect and what they return. We can think of this like a __promise__: “If you pass some specified values to me, I will return some defined type to you.” We can trust in these promises - or perhaps not?  

```csharp
Image LoadImage(Url imageUrl){ 
    If(!webLoader.IsImageAvailable(imageUrl)){
    Throw new ImageNotFoundException(imageUrl)
}

return webLoader.GetImage(imageUrl); 
```

Unbelievable, the method signature is a liar - it does not only return an Image it can __additionally throw an Exception__. Of course, You can argue that throwing an exception here is legit, but if you only look at the method signature, you won’t be able to recognize this. 

# Why I don’t like exceptions

There are some reasons I use exceptions very sparingly. The first one can be described by a look at the purpose of exceptions. They can scream “Help me!” (aka throw), and your program will jump to someone else who says “I’m the Saviour” (aka catch). The problem I have with this behavior is that the catch functionality can be completely elsewhere in code then the throw capability. This circumstance makes your code __hard to read, follow and understand__. If you think about this in detail, it even smells a little after the greatly feared GOTO. 

The second reason is that they are __simple to overlook__. As mentioned above, the method __signature won’t tell__ you about the exception, and there is no way to declare this behavior (besides comments of course). There are mechanisms for more explicit declarations in other languages, like checked-exceptions in Java. Even if they aren’t that popular at all, they provide a possibility to tell the caller of a method, that this method has some edge cases which won’t return an image. In .Net there is no language-level support for a similar concept.

Another reason is their very __cumbersome syntax__. Your catch-blocks should be as small as possible, but this leads to the noise of declaring all your variables outside the catch scope.  Besides this, the additional scope makes reading your code difficult.

And finally, I need to highlight that this LoadImage-method is a perfect example for many similar use cases out there, where __exceptions are used for something they aren’t made for__, with my understanding. By our given example, we must divide into two possible scenarios.
* The first one: There should always be an image, and if there is no image something went definitively wrong - you can stay with an exception. 
* The second one and in my experience the more frequent one: It’s happening quite rarely that an image is not available, but it can be missing and this is known – you should not go with exceptions here – because you already know at development time, that there’s maybe no image. Some people also call this defensive programming.  

Surprise, there is even a third scenario. __A mix of the above ones, where it gets tricky:__ Think of the LoadImage-method as a functionality that is __used by different parts__ within your application. Maybe some of them asking for URLS, which will always have to provide an image - but on the other hand, some parts are asking for URLs that won’t point to a image in some individual cases. 

__Throwing an exception__ and handling this exception in the different application parts seems to be a legit solution? But what if those parts of our application, who can’t always expect an image, would instead always work with a Placeholder image? Wouldn’t it be nice to __return a Placeholder__ image from the LoadImage-method if no image was found?

Maybe, but in fact, __the LoadImage-method is not the right place to decide, how to react, if an image isn’t available__. That’s why exceptions are a great tool to silence one’s conscience. Because we return an image, and if no image exists we throw an exception that can be handled by the caller. Unfortunately, too bad, that __no one__ ever who doesn’t look at our code will __recognize that an exception gets thrown__. 

That’s why we need __some concept to explicitly tell__, that our LoadImage-method will return something that probably contains an image or not.

# What are the alternatives?

If your using __value types__, .Net provided an excellent alternative, by the introduction of [Nullable Types](https://msdn.microsoft.com/en-us/library/1t3y8s4s.aspx). Let’s imagine the example’s Image is a struct. Then we can change the signature to something like this: 

```csharp
Image? LoadImage(Url imageUrl) 
```

This tells the method-caller explicit that null can be returned, and we can implicit infer, that some cases don't return a image. I think that’s nice!  
But be careful if you are looking for a solution that fits __reference types__. You will find some advice, to not be that meticulous and just return a null for reference types too. 
 
> In C# you can just return `null`   
> __StackOverflow at its best.__

__No one will ever check your LoadImage-Method for returning null.__ (Yes, I think that’s even less likely, then checking for an exception.) And that’s probably the reason why the whole thing you are working on, will crash one day. And you don’t want to be the one who is responsible for a crashing software. 

Except in the case of Nullables as mentioned above, I don’t know any good reason to explicit return null anywhere, and I think I’m in good company. It even gets worse If you stay with this pattern in other languages. In this case, I promise, you’ll someday wake up with something like this: 

> Cannot read property `'undefined'` of undefined   
> __JavaScript at its best.__

# There’s nothing you can’t wrap

A better alternative is to wrap your image into something that gives the user of your method (maybe you, two weeks after writing it) the __needed hint, that this method won’t return an image in some cases__. For example, an ImageResult that looks something like this:

```csharp
ImageResult LoadImage(Url imageUrl) 

// Image-Wrapper: 
Public class ImageResult{
    Bool HasImage { get; }    
    Image Image { get; }
}
```

I think this is a good way to express that it is the task of the LoadImage-method-user to decide what to do if no image exists. Of course, the Image property of ImageResult can still be called without checking the HasIamge Boolean, but that’s the caller's fault. 

So, my advice is to wrap all your return objects into some wrapper objects? Yes, __if there is only the minimal chance that something can happen, and you know about this possibility – you should think of using this instead of exceptions__. If you start looking for a generic way to this now, you should try out Optionals.

# Optionals

The concept of Optionals, options, maybes or however you call them, provide a great solution for our problem. As they are excelt in pointing out that __you supply something- but maybe that won’t even exist__. 

If change the LoadImage-method to use [FluentOptionals](https://github.com/duffleit/fluentOptionals) (a lightweight Optionals-implementation for .Net, if written) it looks like this: 

```csharp
Optional<Image> LoadImage(Url imageUrl) 

// implementation: 
Image LoadImage(Url imageUrl){
    return (_webLoader.IsImageAvailable(imageUrl))
        ? Optional.From(webLoader.GetImage(imageUrl))
        : Optional.None<Image>()
}
```

The __method signature now tells explicitly that we optionally return an image__. It’s the caller’s decision how to react if the image doesn't. Maybe the decision will be delegated the call stack downwards again, but that’s not the point. __The improvement is that our LoadImage-method doesn’t have to decide about something, that should be decided by someone else.__

At the point of the application, where the knowledge about this decision exists, we must decide how to handle none-existing optional (we call it a None). If you want to get the value of an Optional, it always forces you to provide a way, how to handle Nones. (at least my [FluentOptionals](https://github.com/duffleit/fluentOptionals) implementation does this consequently.)

```csharp
// get value of optional or if none return a DefaultImage 
var image = optionalImage.ValueOr(DefaultImage); 

// like above but lazy
var image = optioanlImage.ValueOr(() => 
_imageProvider.GetPlaceHolderImage());

// a functional flavored matching way to do it
var resizedImage = optionalImage.Match(
    some: i => i.ResiveToProfileImage(), 
    none () => Image.FromText(“profileImageNotFound”).ResizeToProfileimage())    

// a custom extension method for optional Images
var image = optionalImage.ValueOrPlaceholder();

// and you can even thow an exception if no value is available
Var image = optionalImage.ValueOrThrow(new Exception(“the image with the url … must exist”));
```

There is also a way to handle scenarios, in which you don’t even need an instance of the image, you just want to do something if it exists.

```csharp
opationalImage.MatchSome(i => i.UpdateTimeStamp(DateTime.Now))

// or
opationalImage.Match(
    some: i  => i.UpdateTimeStamp(DateTime.Now),
    none: () => _logger.Log(“could not … “)
)
```
_This is a short extract of the fluentOpations-API, the full documentation can be found on [github](https://github.com/duffleit/fluentOptionals)._

As already mentioned the concept of Optionals provides __two significant benefits__:
* __explicitly__ tell the caller of a function that his request probably can’t be fulfilled 
* __handle the decision__, on how to manage this Optionals value, __at the right part of your application__. 

# Optionals vs. classic OOP

OOP-hardliners maybe will now question the idea of introducing __the additional concept of Optionals__. (thanks to [DanielTheCoder](https://twitter.com/DanielTheCoder) for having a great discussion about this). I support this view, as a solution for the problem __can be simple achieved with meaningful POCOs__, like the above mentioned ImageResult. But I think we must distinguish in different use cases.

Think of an application which has some concept of a ClientRegister. You can straightforward use a framework provided _List_ to represent these clients - but if they are an __important part of your domain__, you should think of creating an own class to express this concept in your code explicitly. 
Maybe internally this ClientRegister-class even inherits or delegates to the framework provide _List_, but there was a need to create a distinct concept in your codebase. It’s the same with using Optionals. 

If we are on a __technical level__, where we often need this concept to express that something can be available or not, I think using Optionals is an excellent way to __reduce boilerplate__ and reduce complexity.

For an important  __domain-specific concept__  you should creating some separate class to express this concept explicitly in your code and should avoid using a generic optional implimentation. Additionally, those domain objects, tend to grow during development, and at least if you need some information to know why the customer is not available (blocked, inactivated, deleted), you can't stay with generic optionals as you won’t be able to gather __more information than available/not available__ from an optional.









