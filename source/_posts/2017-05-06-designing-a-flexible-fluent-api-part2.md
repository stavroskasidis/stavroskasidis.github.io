---
layout: post
title: "Designing a flexible Fluent API - Part 2"
description: "Sharing experience on designing a flexible fluent API in C# (Part 2)"
date: 2017-05-06 18:00:00
comments: true
keywords: "c#, csharp, fluent-api, fluent, design, architect"
category: general
tags:
- general
- c#
- fluent-api
---

In the [previous post]({% post_url 2017-05-06-designing-a-flexible-fluent-api-part1 %}), we saw a real life requirement (a custom view engine) and the intended API that the developers would use.

Let's begin with some of the basic constructs used:

{% highlight csharp %}
public interface IComponent
{
     //Code omitted
}

public interface IComponentContainer : IComponent
{
     List<IComponent> ChildComponents {get; set;}
     //Code omitted
}

/// Root component of our view
public class View : IComponentContainer
{
    //Code omitted
}
{% endhighlight %}

The <span class="inline-highlight nc">View</span> class is the root component that contains all of our other components. As you can see it is an **IComponentContainer**. 
In the code displayed in the previous post, you may remember a <span class="inline-highlight nc">Panel</span> component that contained some other components. A 
<span class="inline-highlight nc">Panel</span> is also an **IComponentContainer**. 

Bear with me, there is a reason I am explaining all of this.

Lets see the <span class="inline-highlight nc">TextBox</span> component:

{% highlight csharp %}

public class TextBox: ICustomCssEnabledComponent, 
                      IEventEnabledComponent
{
    public string Name {get; set;}
    //Code omitted
}

//Not all components will expose access to custom css 
public interface ICustomCssEnabledComponent : IComponent
{
    List<string> CssClasses {get; set;}
}

//A component that supports events will implement this interface
public interface IEventEnabledComponent : IComponent
{
    List<ComponentEvent> Events {get; set;}
}

public class ComponentEvent
{
    public string EventName {get; set;}
    public string JsHandler {get; set;}
}


{% endhighlight %}

Now, let's get into the component builders that provide the fluent API we saw in the [previous post]({% post_url 2017-05-06-designing-a-flexible-fluent-api-part1 %}).

{% highlight csharp %}

//Provides the API for the TextBox component
public class TextBoxBuilder
{
    private TextBox _textBox;

    public TextBoxBuilder(TextBox textBox)
    {
        _textBox = textBox;
    }
    
    //Textbox specific option
    public TextBoxBuilder Name(string name)
    {
        _textBox.Name = name;
        return this;
    }
    //code omitted
}

//This is the builder that all component containers' builders inherit. 
//It provides the base entry point for adding components
public class ContainerBuilder<TModel>
{
    protected IComponentContainer _container;

    public TextBoxBuilder AddTextBoxFor<TProperty>
                    (Expression<Func<TModel,TProperty>> propertyExpression)
    {
        var textBox = new TextBox();
        //code omitted: Extracting info from the property expression like: 
        //              property name , caption (MVC style, reading attributes) etc
        return new TextBoxBuilder(textBox);
    }
     //code omitted
}
{% endhighlight %}

Ok, so the <span class="inline-highlight nc">TextBoxBuilder</span> is where the textbox specific options will be and each call to an option will return the builder instance so that the chain can keep going.

But what about all those properties that are inherited from the common interfaces, like **ICustomCssEnabledComponent** and **IEventEnabledComponent**.
We do not want to duplicate the code for handling these properties in every builder, right?

If we try to solve the problem using inheritance, like having <span class="inline-highlight nc">TextBoxBuilder</span> inherit from a base **ComponentBuilder** class that contains those common options, we run into a flexibility problem.

What happens if an other component builder doesn't need all of those methods? We would expose methods that are not appropriate is some cases and this is **bad**.

In the [next post]({% post_url 2017-05-07-designing-a-flexible-fluent-api-part3 %}) we will see how we are going to solve this problem, using C#'s extension methods.