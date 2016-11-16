---
layout: post
title: Missing Optionals in C#  
description: Optionals are a marvelous concept, known primary from functional languages like F# - how about using them in object-oriented languages like C#?  
keywords: Optionals, Mabye, Option, donNet, F#, C#, OOP, Exceptions
---

This post is about my desire for F\#-optionals in C\#, and the reason why you
should use them wisely in object-oriented languages. 

**Let's start** it with something simple like this:

```csharp
Image LoadImage(Url imageUrl)
``` 

This is a basic LoadImage method, which takes a Url and returns an Image - quite
**readable and easy to understand**. We don’t need any implementation or
documentation to understand what this method does. Isn’t that what we expect
from **clean code**?

One reason for the superior readability is that method signatures define clearly
what they expect and what they return. We can think of this as a **promise**:
"If you pass some specified values to me, I will return some defined type to
you." We can trust in these promises - or perhaps not?

```csharp
Image LoadImage(Url imageUrl){ 
    If(!webLoader.IsImageAvailable(imageUrl)){
    Throw new ImageNotFoundException(imageUrl)
}

return webLoader.GetImage(imageUrl); 
```

Incredible, but the method signature is a liar - not only does it return an
Image, it can **additionally throw an Exception**. Of course, You may argue that
throwing an exception here is legit, but if you only look at the method
signature, you won’t be able to recognize this.

Why I don’t like exceptions
===========================

There are some reasons why I use exceptions very sparingly. The first one can be
described by a look at the purpose of exceptions. They can scream "Help me!"
(aka `throw`), and your program will jump to someone else who says "I’m the
Saviour" (aka `catch`). The problem I have with this behavior is that the catch
functionality can be completely dissociated from in code, than the throw
capability. This circumstance makes your code **hard to read, follow, or
understand**. If you think about this in detail, it even smells a bit like the
greatly feared `GOTO`.

The second reason is that they are **easy to overlook**. As mentioned above, the
method **signature won’t tell** you about the exception, and there is no way to
declare this behavior (besides comments, of course). There are mechanisms for
more explicit declarations in other languages, like
[checked-exceptions](https://stackoverflow.com/questions/613954/the-case-against-checked-exceptions)
in Java. Even if they aren’t that popular, they provide a possibility of telling
the caller of a method, that this method has some edge cases which won’t return
an image. In .Net there is no language-level support for a similar concept.

Another reason is their very **cumbersome syntax**. Your catch-blocks should be
as small as possible, but this leads to the clamor of declaring all your
variables outside the catch scope. Besides, the additional scope makes reading
your code difficult.

And finally, I need to highlight that this LoadImage method is a perfect example
for many similar use cases out there, where **exceptions are used for something
they aren’t made for**, to my understanding. By our given example, we must
examine two possible scenarios. 

* The first one: There should always be an
image, and if there is no image something definitely went wrong - you can stay
with an exception. 
* The second one, and in my experience the more frequent
one: It happen rarely that an image is not available, but it can be missing and
this is known – you should not go with exceptions here, because you already know
at development time, that maybe there’s no image. Some people also call this
defensive programming.

Surprise! There is even a third scenario. **A mix of the above, and this is
where it gets tricky:** Think of the LoadImage method as a functionality that is
**used by different parts** within your application. Maybe some of them are
asking for URLS, which will always have to provide an image - but on the other
hand, some parts are asking for URLs that won’t point to an image in some
individual cases.

**Throwing an exception** and handling this exception in the different
application parts seems to be a legit solution? But what if those parts of our
application, which can’t always expect an image, would instead always work with
a Placeholder image? Wouldn’t it be nice to **return a Placeholder** image from
the LoadImage method if no image was found?

Maybe; but in fact, **the LoadImage method is not the right place to decide how
to react, if an image isn’t available**. That is why exceptions are a great tool
to silence one’s conscience; because we return an image, and if no image exists
then we throw an exception that can be handled by the caller. Unfortunately,
**no one** who doesn’t look at our code will ever **recognize that an exception
gets thrown**.

That’s why we need **some concept to explicitly state**, that our LoadImage
method will return something that probably contains an image or not.

What are the alternatives?
==========================

If you’re using **value types**, .Net provides an excellent alternative, through
the introduction of [Nullable
Types](https://msdn.microsoft.com/en-us/library/1t3y8s4s.aspx). Let us imagine
that the example’s Image is a struct. Then we can change the signature to
something like this:

```csharp
Image? LoadImage(Url imageUrl) 
```

This tells the method-caller explicitly that `null` can be returned, and we can
infer, implicitly, that some cases don't return an image. I think that’s nice!  
But be careful if you are looking for a solution that fits **reference types**:
You will find some advice, to not be that meticulous and just return `null` for
reference types too.

>   In C\# you can just return `null`  
>   **Stack Overflow at its best.**

**No one will ever check your LoadImage method for returning** `null`**.** (Yes,
I think that’s even less likely than checking for an exception.) And that is
probably the reason why the whole thing you are working on, will crash one day.
And you don’t want to be the one responsible for a crashing software.

Except in the case of Nullables as mentioned above, I don’t know any good reason
to explicitly return `null` anywhere, and I think I’m in good company. It gets
even worse If you stay with this pattern in other languages. In this case, I
promise, you will someday wake up with something like this:

>   Cannot read property `'undefined'` of undefined  
>   **JavaScript at its best.**

There’s nothing you can’t wrap
==============================

A better alternative is to wrap your image into something that gives the user of
your method (maybe you, two weeks after writing it) the **required hint, that
this method won’t return an image in some cases**. For example, an ImageResult
that looks something like this:

```csharp
ImageResult LoadImage(Url imageUrl) 

// Image-Wrapper: 
Public class ImageResult{
    Bool HasImage { get; }    
    Image Image { get; }
}
```

I think this is a good way to express that it is the task of the LoadImage
method user to decide what to do if no image exists. Of course, the Image
property of ImageResult can still be called without checking the HasIamge
Boolean, but that’s the caller's fault.

So, my advice is to wrap all your return objects into some wrapper objects? Yes,
**if there is only a minimal chance that something can go wrong - and you know
about it – you should think of using this instead of exceptions**. If you are
looking for a generic way to achieve this, now, you should try out Optionals.

Optionals
=========

The concept of *optionals, options, maybes*, or whatever you call them, provides a
great solution for our problem. They are excellent in pointing out that **you
supply something - but maybe that won’t even exist**.

If you change the LoadImage method to use
[FluentOptionals](https://github.com/duffleit/fluentOptionals) (a lightweight
Optionals implementation for .Net, which I've written), it looks like this:

```csharp
Optional<Image> LoadImage(Url imageUrl) 

// implementation: 
Optional<Image> LoadImage(Url imageUrl){
    return (_webLoader.IsImageAvailable(imageUrl))
        ? Optional.From(webLoader.GetImage(imageUrl))
        : Optional.None<Image>()
}
```

The **method signature now tells explicitly that we optionally return an
image**. It is the caller’s decision how to react if the image doesn't exist.
Maybe the decision will be delegated to the call stack downwards again, but
that’s not the point. **The improvement is that our LoadImage method doesn’t
have to decide about something that should be decided by someone else.**

At the point of the application, where the knowledge about this decision exists,
we must decide how to handle none-existing optionals (we call it a None). If you
want to get the value of an Optional, it always forces you to provide a way, to
handle Nones. (at least the
[FluentOptionals](https://github.com/duffleit/fluentOptionals) implementation
does this consequently.)

```csharp
// get value of optional or if none return a DefaultImage 
var image = optionalImage.ValueOr(DefaultImage); 

// like above but lazy
var image = optioanlImage.ValueOr(() => 
_imageProvider.GetPlaceHolderImage());

// a functional flavored matching way to do it
var resizedImage = optionalImage.Match(
    some: i  => i.ResiveToProfileImage(), 
    none: () => Image.FromText("profileImageNotFound").ResizeToProfileimage())    

// a custom extension method for optional Images
var image = optionalImage.ValueOrPlaceholder();

// and you can even thow an exception if no value is available
Var image = optionalImage.ValueOrThrow(new Exception("the image with the url … must exist"));
```

There is also a way to handle scenarios, in which you don’t even need an instance of the image, you just want to do something if it exists.

```csharp
opationalImage.MatchSome(i => i.UpdateTimeStamp(DateTime.Now))

// or
opationalImage.Match(
    some: i  => i.UpdateTimeStamp(DateTime.Now),
    none: () => _logger.Log("could not … ")
)
```

*This is a short extract of the fluentOptionals-API; the full documentation can be
found on* [github](https://github.com/duffleit/fluentOptionals)*.*

As already mentioned, the concept of Optionals provides **two significant
benefits**: 

* **explicitly** tell the caller of a function that his request
probably can’t be fulfilled 
* **handle the decision**, on how to manage this
Optionals value, **at the right part of your application**.

Optionals vs. classic OOP
=========================

Maybe OOP-hardliners will now question the idea of introducing **the additional
concept of Optionals**. (thanks to
[DanielTheCoder](https://twitter.com/DanielTheCoder) for having a great
discussion about this). I support this view, as a solution for the problem **can
be achieved quite simply, with meaningful POCOs**, like the above-mentioned
ImageResult. But I think we must distinguish between different use cases.

Think of an application which has some concept of a ClientRegister. You can
straightaway use a framework-provided *List* to represent these clients - but if
they are an **important part of your domain**, you should think of creating an
own class to express this concept in your code explicitly. Maybe internally this
ClientRegister class even inherits or delegates to the framework-provided
*List*, but there was a need to create a distinct concept in your codebase. It
is the same with using Optionals.

On a **technical level**, where we often need this concept to express that
something can be available or not, I think using Optionals is an excellent way
to **trim boilerplate** and reduce complexity.

For an important **domain-specific concept,** you should be creating some
separate class to express this concept explicitly in your code, and should avoid
using a generic optional implementation. Additionally, those domain objects tend
to grow during development, and at least if you need some information to know
why the customer is not available (blocked, inactivated, deleted), you can't
stay with generic optionals, because you won’t be able to gather **more
information than available/not available** from an optional.
