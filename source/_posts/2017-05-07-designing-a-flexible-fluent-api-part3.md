---
layout: post
title: "Designing a flexible Fluent API - Part 3"
description: "Sharing experience on designing a flexible fluent API in C# (Part 3)"
date: 2017-05-07 12:00:00
comments: true
keywords: "c#, csharp, fluent-api, fluent, design, architect"
category: general
tags:
- fluent-api
---

In the [previous post]({% post_url 2017-05-06-designing-a-flexible-fluent-api-part2 %}), while designing our API we stumbled on a flexibility problem.
**How do we add common methods to our API builders without compromising on flexibility?**

We are going to use C#'s extension methods. Here is how:

{% highlight csharp %}

public interface ICustomCssEnabledBuilder<TBuilder>
{
    TBuilder GetBuilderInstance();
    ICustomCssEnabledComponent GetComponent();
}

public static class CustomCssEnabledBuilderExtensions
{
    public static TBuilder AddCssClass<TBuilder>
            (this ICustomCssEnabledBuilder<TBuilder> builder, string cssClass)
    {
        builder.GetComponent().CssClasses.Add(cssClass);
        return builder.GetBuilderInstance();
    }
}

{% endhighlight %}

Now when a builder needs to support adding a css class to a css enabled component, 
it just has to implement the **ICustomCssEnabledBuilder** interface.

{% highlight csharp %}
public class TextBoxBuilder : ICustomCssEnabledBuilder<TextBoxBuilder>
{
    TextBox _textBox;

    public TextBoxBuilder(TextBox textBox)
    {
        _textBox = textBox;
    }

    [EditorBrowsable(EditorBrowsableState.Never)]
    public TextBoxBuilder GetBuilderInstance()
    {
        return this;
    }

    [EditorBrowsable(EditorBrowsableState.Never)]
    public ICustomCssEnabledComponent GetComponent()
    {
        return _textBox;
    }

    //Textbox specific option
    public TextBoxBuilder Name(string name)
    {
        _textBox.Name = name;
        return this;
    }

    //code omitted
}
{% endhighlight %}

Now our builder can be used like this

{% highlight csharp %}
panel.AddTextBoxFor(model => model.TextProp2)
                 .TextBoxSpecificOption(...)
                 .AddCssClass("custom-css-class")
                 .OtherTextBoxSpecificOption(...);
{% endhighlight %}

The same technique can be used for the events API.

{% highlight csharp %}
public interface IEventEnabledBuilder<TBuilder, TEventBuilder>
{
    TBuilder GetBuilderInstance();
    TEventBuilder GetEventBuilder();
}

public static class EventEnabledBuilderExtensions
{
    public static TBuilder Events<TBuilder, TEventBuilder>(this IEventEnabledBuilder<TBuilder, TEventBuilder> builder, 
                                                           Action<TEventBuilder> eventBuilderExpression)
    {
        var eventBuilder = builder.GetEventBuilder();
        eventBuilderExpression(eventBuilder);
        return builder.GetBuilderInstance();
    }
}

public class TextBoxEventBuilder
{
    private TextBox _textBox;

    public TextBoxEventBuilder(TextBox textBox)
    {
        _textBox = textBox;
    }

    public TextBoxEventBuilder OnChange(string jsHandle)
    {
        _textBox.Events.Add(new ComponentEvent()
        {
            EventName = "change",
            JsHandler = jsHandle
        });
        return this;
    }
}
{% endhighlight %}

So our <span class="inline-highlight nc">TextBoxBuilder</span> is changed to this.

{% highlight csharp %}
public class TextBoxBuilder : ICustomCssEnabledBuilder<TextBoxBuilder>,
                              IEventEnabledBuilder<TextBoxBuilder, TextBoxEventBuilder>
{
    private TextBox _textBox;

    public TextBoxBuilder(TextBox textBox)
    {
        _textBox = textBox;
    }

    [EditorBrowsable(EditorBrowsableState.Never)]
    public TextBoxBuilder GetBuilderInstance()
    {
        return this;
    }

    [EditorBrowsable(EditorBrowsableState.Never)]
    public ICustomCssEnabledComponent GetComponent()
    {
        return _textBox;
    }

    [EditorBrowsable(EditorBrowsableState.Never)]
    public TextBoxEventBuilder GetEventBuilder()
    {
        return new TextBoxEventBuilder(_textBox);
    }

    //Textbox specific option
    public TextBoxBuilder Name(string name)
    {
        _textBox.Name = name;
        return this;
    }

    //code omitted
}
{% endhighlight %}

And now it can be used like this.

{% highlight csharp %}
panel.AddTextBoxFor(model => model.TextProp2)
                 .TextBoxSpecificOption(...)
                 .AddCssClass("custom-css-class")
                 .OtherTextBoxSpecificOption(...)
                 .Events(e=> e.OnChange("jsHandlerName"));
{% endhighlight %}