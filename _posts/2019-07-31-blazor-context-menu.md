---
layout: post
title: "Introducing Blazor Context Menu"
description: "A context menu component for Blazor"
date: 2019-07-31 10:00:00
comments: true
keywords: "c#, csharp, blazor, context-menu, blazor-context-menu, blazorcontextmenu, blazor-contextmenu "
category: blazor
tags:
- blazor
- components
---

With the arrival of [Blazor](https://blazor.net) just around the corner, I would like to introduce you to a component I have created for blazor: [BlazorContextMenu](https://github.com/stavroskasidis/BlazorContextMenu)

![](https://raw.githubusercontent.com/stavroskasidis/BlazorContextMenu/master/ReadmeResources/blazor-context-menu-demo-2.gif)

This project started as a personal need of mine for another project, so I thought: "This could be a nice reusable component. Why not make it public?". So, it started quite simple, but over time more and more features got added.


### Basic usage

The most simple and basic usage is something like the following:

{% highlight xml %}
<ContextMenu Id="myMenu">
    <Item OnClick="@OnClick">Item 1</Item>
    <Item OnClick="@OnClick">Item 2</Item>
    <Item OnClick="@OnClick" Enabled="false">Item 3 (disabled)</Item>
    <Seperator />
    <Item>Submenu
        <SubMenu>
            <Item OnClick="@OnClick">Submenu Item 1</Item>
            <Item OnClick="@OnClick">Submenu Item 2</Item>
        </SubMenu>
    </Item>
</ContextMenu>

<ContextMenuTrigger MenuId="myMenu">
    <p>Right-click on me to show the context menu !!</p>
</ContextMenuTrigger>

@code{
    void OnClick(ItemClickEventArgs e)
    {
        Console.WriteLine($"Item Clicked => Menu: {e.ContextMenuId}, MenuTarget: {e.ContextMenuTargetId}, IsCanceled: {e.IsCanceled}, MenuItem: {e.MenuItemElement}, MouseEvent: {e.MouseEvent}");
    }
}
{% endhighlight %}

### More Info

For more info like code samples and how to use it in your application, please visit the github repository: [https://github.com/stavroskasidis/BlazorContextMenu](https://github.com/stavroskasidis/BlazorContextMenu). 

And if you like it and end up using it in your project(s), consider giving it a ⭐!!

😀😀😀