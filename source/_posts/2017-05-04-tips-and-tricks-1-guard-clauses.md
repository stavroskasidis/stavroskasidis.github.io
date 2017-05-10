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
- guard-clauses
---

Following the guidelines of [defensive programming][defensive_programming] and [fail-fast system design][fail_fast], a method should always validate it's input.

The code that validates your method's inputs is called a **Guard Clause**. It makes your code more understandable and it protects you from bugs and unexpected behaviors.

For example, lets assume we have a simple method that validates a Url:

{% highlight csharp %}
public bool IsUrlValid(string url)
{
    return Uri.TryCreate(url, UriKind.Absolute, out Uri result) && 
           (result.Scheme == "http" || result.Scheme == "https");
}
{% endhighlight %}


If we pass <span class="inline-highlight k">null</span> as the <span class="inline-highlight n">url</span> parameter then the method will return <span class="inline-highlight k">false</span>. At first glance, this may seem ok, since <span class="inline-highlight k">null</span> is indeed an invalid url. However this wasn't our intention and we do not consider <span class="inline-highlight k">null</span> a valid input value.

So we could rewrite the method as:

{% highlight csharp %}
public bool IsUrlValid(string url)
{
    if(url == null) 
        throw new ArgumentNullException(nameof(url));
    return Uri.TryCreate(url, UriKind.Absolute, out Uri result) && 
           (result.Scheme == "http" || result.Scheme == "https");
}
{% endhighlight %}

That <span class="inline-highlight k">if</span> expression is a **Guard Clause**. However we can do better than that, since repeating this code over and over in all of our methods is a bad practice. ([DRY][DRY] principle).

So lets create a helper class:

{% highlight csharp %}
public static class GuardClauses
{
    public static void IsNotNull(object argumentValue, string argumentName)
    {
        if (argument == null) 
            throw new ArgumentNullException(argumentName);
    }
}
{% endhighlight %}

And use it like this:

{% highlight csharp %}
public bool IsUrlValid(string url)
{
    GuardClauses.IsNotNull(url,nameof(url));
    return Uri.TryCreate(url, UriKind.Absolute, out Uri result) && 
           (result.Scheme == "http" || result.Scheme == "https");
}
{% endhighlight %}

> **Note:** I am usually against the usage of <span class="inline-highlight k">static</span> methods. However in this case, I believe this a rare exception where a <span class="inline-highlight k">static</span> method can be used since it is not affecting our code's testability and it does not increase it's complexity.

We can keep adding more Guard Clauses to our helper class as needed. Here are some more:

{% highlight csharp %}
public static class GuardClauses
{
    public static void IsNotNull(object argumentValue, string argumentName)
    {
        if (argument == null) 
            throw new ArgumentNullException(argumentName);
    }

    public static void IsNotNullOrEmpty(string argumentValue, string argumentName)
    {
        if (string.IsNullOrEmpty(argument)) 
            throw new ArgumentNullException(argumentName);
    }

    public static void IsNotZero(int argumentValue, string argumentName)
    {
        if (argument == 0) 
            //C# 7.0 syntax: $"Argument '{argumentName}' cannot be zero"
            throw new ArgumentException("Argument '" + argumentName +"' cannot be zero",
                                         argumentName);
    }

   public static void IsLessThan(int maxValue, int argumentValue, string argumentName)
   {
       if (argument >= maxValue) 
            //C# 7.0 syntax: $"Argument '{argumentName}' cannot exceed '{maxValue}'"
            throw new ArgumentException("Argument '" + argumentName +"' cannot exceed " + maxValue,
                                         argumentName);
   }

   public static void IsMoreThan(int minValue, int argumentValue, string argumentName)
   {
       if (argument <= minValue) 
            //C# 7.0 syntax: $"Argument '{argumentName}' cannot be lower than '{minValue}'"
            throw new ArgumentException("Argument '" + argumentName +"'  cannot be lower than " + minValue,
                                         argumentName);
   }
}
{% endhighlight %}

> **Note:** Some may disagree with Guard Clauses like "**IsLessThan**" and "**IsMoreThan**" because they may step into business logic rules. It's up to you to decide.

[defensive_programming]: https://en.wikipedia.org/wiki/Defensive_programming
[fail_fast]: https://en.wikipedia.org/wiki/Fail-fast
[DRY]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself