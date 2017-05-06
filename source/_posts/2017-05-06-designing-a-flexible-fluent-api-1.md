---
layout: post
title: "Designing a flexible Fluent API - Part 1"
description: "Sharing experience on designing a flexible fluent API in C#"
date: 2017-05-04 11:34:00
comments: true
keywords: "c#, csharp, fluent-api, fluent, design, architect"
category: general
tags:
- general
- c#
- fluent-api
---

Recently, I had to design and create a web platform for the company I work for. The platform would be used by other developers of the company to create business applications. One of the many problems I had to solve is a pretty common requirement for this kind of sofware: 

*"The end-user must be able to customize the layout of a CRUD form"*

The solution I went for is creating a custom view engine, so that the developers will describe a view by code (not razor). The view is created as a tree of component objects that get's rendered as html. The end-users are provided with a designer that allows them to modify and save the component tree.

 I won't go into more details about the view engine since it is not the focus of this post, so lets get to the "fluent API" part. 
 
 I designed the API that the developers would use like this:

{% highlight csharp %}
public class MyView : ICodeView<ViewModel>
{
    public void BuildView(ComponentBuilder<ViewModel> builder)
    {
        builder.AddTableLayout(table=> 
        {
            table.AddRow(row =>
            {
                row.AddTextBoxFor(model => model.TextProp)
                   .AddCssClass("custom-css-class")
                   .Events(e=> e.().Change("myView.textBoxOnChangeJsHandler"));

                row.AddCheckBoxFor(model => model.BoolProp)
                   .AddCssClass("custom-css-class")
                   .AddCssClass("custom-css-class2")
                   .Events(e=> e.Change("myView.checkBoxOnChangeJsHandler"));
            });

            //Code ommited
        });
    }
}
{% endhighlight %}

Any respectful fluent API should be easy to understand and not expose irrelevant methods. For example if a component does not support adding a css class to it, then the "AddCssClass" should not be exposed for that component. The same is true for the Events API, not all components support the same events.

In the next post we will see how you can design a flexible API like that.