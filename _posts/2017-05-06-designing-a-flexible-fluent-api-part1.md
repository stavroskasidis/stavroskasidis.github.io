---
layout: post
title: "Designing a flexible Fluent API - Part 1"
description: "Sharing experience on designing a flexible fluent API in C# (Part 1)"
date: 2017-05-06 08:00:00
comments: true
keywords: "c#, csharp, fluent-api, fluent, design, architect"
category: general
tags:
- fluent-api
---

>Through this series we will see a real life example of how to create a fluent API that is flexible. Keep reading to see what I mean.

Recently, I had to design and create a web platform. The platform would be used by other developers to create business applications. One of the many problems I had to solve is a pretty common requirement for this kind of sofware: 

*"The end user must be able to customize the layout of a CRUD view; e.g. change the order of the fields, hide them, change their captions or size etc"*

The solution we went for is creating a custom view engine for ASP MVC, so that the developers will describe views by code (not razor). The view is created as a tree of component objects that get's rendered as html. The end users are provided with a designer that allows them to modify and save the component tree, thus achieving customization.

I won't go into more details about the view engine since it is not the focus of this post, so lets get to the "fluent API" part. 
 
I designed the API that the developers would use like this:

{% highlight csharp %}
public class MyView : ICodeView<ViewModel>
{
    public void BuildView(ContainerBuilder<ViewModel> builder)
    {
        
        builder.AddTextBoxFor(model => model.TextProp1);

        builder.AddPanel(panel=> 
        {
            panel.AddTextBoxFor(model => model.TextProp2)
                 .AddCssClass("custom-css-class")
                 .Events(e=> e.OnChange("myView.textBoxOnChangeJsHandler"));

            panel.AddCheckBoxFor(model => model.BoolProp)
                 .AddCssClass("custom-css-class")
                 .AddCssClass("custom-css-class2")
                 .Events(e=> e.OnCheck("myView.checkBoxOnCheckJsHandler"));

            //Code omitted
        });
    }
}
{% endhighlight %}

Any respectful API should be easy to understand and not expose irrelevant methods or throw "**NotSupported**" exceptions. 

For example, if a component does not support adding a css class to it, then the "AddCssClass" should not be exposed for that component. The same is true for the Events API, not all components support the same events.

In the [next post]({% post_url 2017-05-06-designing-a-flexible-fluent-api-part2 %}) we will see how you can design a flexible API like that.