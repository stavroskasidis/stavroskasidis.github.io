---
layout: post
title: "Tips & Tricks #1: Guard Clauses in C#"
description: "How to create and use guard clauses easily and effectively in C#"
date: 2017-05-04 11:34:00
comments: true
keywords: "c#, csharp, guard-clauses, tips-and-tricks "
category: tips-and-tricks
tags:
- tips-and-tricks
---


{% highlight csharp %}
public static void ArgumentIsNotNull(object argumentValue, string argumentName)
{
    if (argument == null) 
        throw new ArgumentNullException(argumentName);
}
{% endhighlight %}