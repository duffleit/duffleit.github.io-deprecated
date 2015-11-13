---
layout: post
title: Driver Pattern enpowers your SpecFlow step definitions
---

**tags:** SpecFlow;  Driver Pattern; Context Injection; Page Object Pattern

Up to now I've seen quite a few projects using [SpecFlow](http://www.specflow.org) for conducting automated regression tests. Even if all these projects are located in different domains and are implemented with different technologies such as Asp.net, WPF or WCF, they still have one problem in common: **the step definitions are messy and hard to maintain. **

Driver Pattern is a powerful tool to prevent this problem. It becomes even more efficient due to its combination with [SpecFlow’s Context Injection](https://github.com/techtalk/SpecFlow/wiki/Context-Injection). Therefore, it is first important to understand how context injection works. 

## Context Injection
Context Injection is a **simple dependency-injection framework** shipped with SpecFlow. However, it is unique, as it **doesn’t support [Inversion of Control](https://wikipedia.org/wiki/Inversion_of_Control)** out of the box, as one might expect. As a result, only concrete types can be injected. At first glance, this constraint causes the feeling of incompleteness, but you will see that it is brilliant for the purpose of SpecFlow. 

Basically, context Injection was designed to share data between step definitions. The two most important rules for context injection are:

* The **lifetime of an injected object is limited to the scenario execution.**
* Within one scenario execution, you can be sure that you will **always get the same instance of the object.** (It acts like a singleton)

*More rules can be found in the official [SpecFlow documentation](https://github.com/techtalk/SpecFlow/wiki/Context-Injection)*

### Too theoretical? Here is an example:
*Story behind the example: Let’s say I’m some kind of forgetful guy and always forget to water my flowers. That’s why I bought a small sprinkler system, which luckily offers a .Net Sdk. (Sad to say that only the first sentence is correct.) I have implemented an application which allows me to start watering my flowers. I've generated some gherkin steps with following step definitions:*

{% highlight C# %}
[Binding]
public sealed class WaterFlowerSteps
{
    private readonly FlowerContext _flowerContext;

    //Inject Flower Context in Custructor
    public WaterFlowerSteps(FlowerContext flowerContext)
    {
        _flowerContext = flowerContext;
    }

    [When(@"i start watering the currently selected flower")]
    public void WhenIStartWateringIt()
    {
        var currentFlower = _flowerContext.CurrentlySelectedFlower;
        FlowerSprinker.SprinkFlower(currentFlower);
    }
}

[Binding]
public sealed class FlowerSelectorSteps
{
    private readonly FlowerContext _flowerContext;

    //also inject Flower Context in Constructor
    public FlowerSelectorSteps(FlowerContext flowerContext)
    {
        _flowerContext = flowerContext;
    }

    [When(@"I have selected only the flower ""(.*)""")]
    public void WhenIHaveSelectedOnlyTheFlower(string flowerName)
    {
        _flowerContext.CurrentlySelectedFlower = flowerName;
    }
}

//Context to share data between different steps. 
public class FlowerContext
{
    public string CurrentlySelectedFlower { get; set; }
}
{% endhighlight %}


In the above example, we have two different classes with one step definition in each. These two step definitions need to share some data. That’s why the `FlowerContext` is injected twice during the execution of one scenario:

* During **the first time**, SpecFlow creates a **new instance** of `FlowerContext` and passes it to the Constructor of `FlowerSelectorSteps`.
* During **the second time**, as Specflow is smart enough to know that an instance of `FlowerContext` was already created, it **passes** this one to `WaterFlowerSteps`.

I think the really cool thing about this is, that context injection **works** just **out of the box**. There is no need for registering the `FlowerContext` to any [IOC container](https://en.wikipedia.org/wiki/Inversion_of_control) or put it into any additional configuration. 

## Ok, but where is the driver?
Basically, a driver acts like a Data Context. The main difference between the driver and a data context is that a data context, like we have seen above, is only used for holding data. In contrast, a driver is something more complex. It isolates the automation logic, combines different tasks of this logic and may even offer some kind of state.

Many step definitions I’ve seen look something like this:

{% highlight C# %}
[Binding]
public sealed class WaterFlowerSteps
{
    FlowerSprinkerMock _flowerSprinkerMock;
    private int _numberOfWateredFlowers;

    public WaterFlowerSteps()
    {
        _flowerSprinkerMock = new FlowerSprinkerMock();
    }

    [Then(@"Todays last watered flower is '(.*)'")]
    public void TodaysLastWateredFlowerIs(string expectedFlowerName)
    {
        if (_numberOfWateredFlowers < 1)
        {
            throw new Exception("no flower was watered today.");
        }

        if (!_flowerSprinkerMock.IsAwake())
        {
            _flowerSprinkerMock.WakeUp();            
        }

        var logDataXml = _flowerSprinkerMock.GetCurrentLog();
        var serializer = new XmlSerializer(typeof(LogData));
        var reader = new StringReader(logDataXml);
        var logData = (LogData)serializer.Deserialize(reader);

        var lastWaterdFlowers = logData.LastFlowers;

        Assert.IsNotNull(lastWaterdFlowers.Last());
        var actualFlowerName = lastWaterdFlowers.Last();
        Assert.AreEqual(expectedFlowerName, actualFlowerName);
    }
}
{% endhighlight %}

I guess you see the problem with this kind of test code. It’s **hard to read, understand and also to maintain.** 
The interesting thing, however, is that most of the developers I know would never write a production code like this. But if we talk about test code: “Yolo man, it’s green”. For an unknown reason, I still have the feeling that test code is too often seen as a kind of unloved duty by many developers. But hey, here is a little secret: If you do it right, writing **test code** can be quite cool. That’s why it **deserves your love**, too.

For this reason I created a `FlowerSprinkerDriver`, which holds all the **knowledge** about **how to perform** the **automation logic**:

{% highlight C# %}
public class FlowerSprinkerDriver
{
    private FlowerSprinkerMock _flowerSprinkerMock;

    public FlowerSprinkerDriver()
    {
        _flowerSprinkerMock = new FlowerSprinkerMock();
    }

    public int TodaysWateredFlowers { get; private set; }

    private FlowerSprinkerMock AwakeFlowerSprinkerMock
    {
        get
        {
            if(!_flowerSprinkerMock.IsAwake()) 
                _flowerSprinkerMock.WakeUp();
            return _flowerSprinkerMock;
        }
    }

    public LogData GetCurrentLogData()
    {
        var logDataXml = AwakeFlowerSprinkerMock.GetCurrentLog();
        var serializer = new XmlSerializer(typeof(LogData));
        var reader = new StringReader(logDataXml);
        return (LogData)serializer.Deserialize(reader);
    }

    public string GetLastWateredFlower()
    {
        if (TodaysWateredFlowers < 1) return null;
        return GetCurrentLogData().LastFlowers.LastOrDefault();
    }
}
{% endhighlight %}

This causes, that the **step definition only delegates** to the **correct driver functions** and has some assertions:

{% highlight C# %}
[Binding]
public sealed class WaterFlowerSteps
{
    FlowerSprinkerDriver _flowerSprinkerDriver;

    public WaterFlowerSteps2()
    {
        _flowerSprinkerDriver = new FlowerSprinkerDriver();
    }

    [Then(@"Todays last watered flower is '(.*)'")]
    public void TodayslastWateredFlowerIs(string expectedFlowerName)
    {
        var lastWateredFlower = _flowerSprinkerDriver.GetLastWateredFlower();

        Assert.IsNotNull(lastWateredFlower, "there is no last watered flower");
        Assert.AreEqual(expectedFlowerName, lastWateredFlower);
    }
}
{% endhighlight %}

**Beside the logic** of how something is done, like in the `GetLastWateredFlower()` function, the **driver** also provides some state information in the `TodaysWateredFlowers` property. This results, that the state can be safely used in different step definitions during one scenario execution. 

## Wrap up
The **combination of context injection and driver pattern** is a really **powerful tool for cleaning up step definitions.** Some of the main advantages are:

* achieving a **better separation of concerns** within test code: On the one hand, the step definitions concentrate on their actual task of **translating human language to automation code;** on the other hand, each **driver has his well-defined scope of automation logic.**
* There is **no problem with transferring data between the step definitions** as the relevant data is held by the driver itself; further, through context injection, it **acts as a singleton for each scenario execution.**
* **step definitions** will be **easier to read** (and **maintain**), as one to five meaningful driver-function calls are easier to understand compared to a wall of automation code.


-----

##*About the Page Object Pattern*

In this context there is also the term **Page Object Pattern** . This pattern works like the **Driver Pattern** we have seen above. With the only difference, that a Page Objects **encapsulates UI** (for example a webpage, or only some controls of it, etc.), while a Driver encapsulates some functions layered above the UI. If you use a lot of automated UI-testing, you can consider to group your test infrastructure by Page Objects. I will not go into depth on this topic, as there are many good posts about this out there. For example [Martin Fowler wrote about it](http://martinfowler.com/bliki/PageObject.html)*. (And believe me, if Martin Fowler writes about something, you shouldn't try to make it better :P )* But **if you use the Page Object Pattern in combination with SpecFlow,** don't forget the **Context Injection**.

Since this *[bullshitbingo](https://en.wikipedia.org/wiki/Buzzword_bingo)* can be a bit confusing, here is a short summary: **a context** provides common used data (state), **a driver** encapsulates some test automation logic, and **a page object** encapsulates some UI access logic. In combination with **Context Injection** they transform to **powerful Ninjas**. 

- Page Objects: 
	- [http://www.assertselenium.com/automation-design-practices/page-object-pattern/](http://www.assertselenium.com/automation-design-practices/page-object-pattern/) (short)
	- [http://martinfowler.com/bliki/PageObject.html](http://martinfowler.com/bliki/PageObject.html) (long) 
- Page Objects with Selenium: 
	- [https://code.google.com/p/selenium/wiki/PageObjects](https://code.google.com/p/selenium/wiki/PageObjects) 
	- [https://code.google.com/p/selenium/wiki/PageFactory](https://code.google.com/p/selenium/wiki/PageFactory)


##*You want more?*
If you want to deepen your knowledge about **BDD/ATDD** and **Specflow**, I recommend the [SpecFlow Course held by Gaspar Nagy](http://gasparnagy.com/trainings/specflow/). In this training concepts such as **Design Pattern** or **Page Object Pattern** are explained in detail.
