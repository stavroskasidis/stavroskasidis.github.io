---
layout: post
title: "Blazor state management without using any libraries"
description: "How to manage your blazor state without depending on 3rd party libraries"
date: 2019-08-01 10:00:00
comments: true
keywords: "c#, csharp, blazor, blazor-state, statemanagement, state-management, blazorstatemanagement, blazor-state-management"
category: blazor
tags:
- blazor
---

> **Disclaimer**: This post assumes that you know what [Blazor](https://blazor.net) is and that you know some of its basic concepts. For more info and documentation on Blazor please visit [https://blazor.net](https://blazor.net).

> **TL;DR**: All the code in this post can be found in this repo (the different iterations contained in branches) [https://github.com/stavroskasidis/BlazorSimpleState.BlogPostSample](https://github.com/stavroskasidis/BlazorSimpleState.BlogPostSample)

### Intro
When you start creating complex applications using Blazor, you will soon realize that you need a way to manage your 
application's state. If you don't, your code will start to fill with events being passed down to child components, so that parent components can stay up-to-date. 

Let's use an example to see how that looks like. 

We will create a page that shows a "counter", having two buttons: an "increment counter" button that increases the counter and a "reset counter" button that sets the counter back to zero.

> **Note:** The example is quite simple, all of it could be a single component but for demo purposes we will break it up into multiple small components.

### Iteration 1 - Using events (no "state management")

Let's how we could approach this using events. We will create 4 components:

* **Counter.razor** - This will be our "page" component, in other words, the component that represents the whole page.
* **CounterDisplay.razor** - This will be the component that shows the current count and hosts the two buttons.
* **CounterIncrementButton.razor** - This is the button that increments the counter.
* **CounterResetButton.razor** - This is the button that resets the counter.

Here's what they look like:

**Counter.razor**
{% highlight xml %}
@page "/counter"
<h1>Counter</h1>
<CounterDisplay CurrentCount="currentCount" OnIncrement="OnIncrement" OnReset="OnReset"/>
@code{
    private int currentCount = 0;
    private void OnIncrement()
    {
        currentCount++;
    }
    private void OnReset()
    {
        currentCount = 0;
    }
}
{% endhighlight %}

**CounterDisplay.razor**
{% highlight xml %}
<p>Current count: @CurrentCount</p>
<CounterIncrementButton OnIncrement="@OnIncrement" />
<CounterResetButton OnReset="@OnReset" />
@code {
    [Parameter] protected int CurrentCount { get; set; }
    [Parameter] protected EventCallback<UIMouseEventArgs> OnIncrement { get; set; }
    [Parameter] protected EventCallback<UIMouseEventArgs> OnReset { get; set; }
}
{% endhighlight %}

**CounterIncrementButton.razor**
{% highlight xml %}
<button class="btn btn-primary" @onclick="OnIncrement">Click me</button>
@code {
    [Parameter] protected EventCallback<UIMouseEventArgs> OnIncrement { get; set; }
}
{% endhighlight %}

**CounterResetButton.razor**
{% highlight xml %}
<button class="btn btn-primary" @onclick="OnReset">Reset Counter</button>
@code {
    [Parameter] protected EventCallback<UIMouseEventArgs> OnReset { get; set; }
}
{% endhighlight %}

As you can see, we are passing down events to the component tree so that the "root" component (Counter.razor) stays updated. This is required because 
Blazor components will re-render only if:
* Their [parameters](https://docs.microsoft.com/en-us/aspnet/core/blazor/components?view=aspnetcore-3.0#component-parameters) have changed.
* After an [EventCallback](https://docs.microsoft.com/en-us/aspnet/core/blazor/components?view=aspnetcore-3.0#eventcallback) is invoked.
* You invoke the `StateHasChanged()` method inside the component.

### Well... this kinda sucks !!

We are passing down events everywhere, plus the counter's state logic is included in the component's code itself. 
This is where "state management" as a concept comes to help us. 

So what is "state management" ?

No matter what pattern/library you decide to use, it all boils down to a couple simple principles: 
1. You need a **State Container**: a read-only object that will hold the state of your page.
2. You also need a way to update the state throught a controlled API.

There are many state management libraries that you can use that have been ported from javascript, like **Redux**, **Fluxor** etc. You can find a good collection
of libraries in this awesome repo here: [https://github.com/AdrienTorris/awesome-blazor](https://github.com/AdrienTorris/awesome-blazor).

These libraries are supposed to be super optimal in re-rendering, that is why they usually bring quite a bit of boilerplate code in my opinion.

However, if you want something simple and you want to avoid 3rd party dependencies, you can create your own state management following the principles above.

### Iteration 2 - Using a "State" object

So, we need a read-only object that will hold the state of the counter. The object will be passed down to all child components as a 
[**CascadingParameter**](https://docs.microsoft.com/en-us/aspnet/core/blazor/components?view=aspnetcore-3.0#cascading-values-and-parameters) and it will contain public methods to allow state to be changed. 

It will also contain an **OnCountChanged** event so that the "page" component (Counter.razor) will subscribe to, 
listening for changes to re-render itself.

**CounterState.cs**
{% highlight csharp %}
public class CounterState
{
    public int CurrentCount { get; private set; }
    public void IncrementCounter()
    {
        CurrentCount++;
        OnCountChanged?.Invoke();
    }
    public void ResetCounter()
    {
        CurrentCount = 0;
        OnCountChanged?.Invoke();
    }
    public event Action OnCountChanged;
}
{% endhighlight %}

This state object will be created in the "page" component (Counter.razor) and the child components will receive state info through **parameters**.

⚠️ It is important that the child components use **parameters** to read any data they need and **NOT** read them directly of the state, otherwise the UI may not update correctly in some cases.

Here is how the components will look like with the `CounterState` class:

**Counter.razor**
{% highlight xml %}
@page "/counter"
<h1>Counter</h1>
<CascadingValue Value="state">
    <CounterDisplay />
</CascadingValue>
@code{
    private CounterState state;
    protected override void OnInit()
    {
        state = new CounterState();
        // When counter changes, trigger the component's StateHasChanged 
        // method to notify it to rerender
        state.OnCountChanged += StateHasChanged;
    }
}
{% endhighlight %}

**CounterDisplay.razor**
{% highlight xml %}
<p>Current count: @CurrentCount</p>
<CounterIncrementButton  />
<CounterResetButton />
@code {
    [Parameter] protected int CurrentCount { get; set; }
}
{% endhighlight %}

**CounterIncrementButton.razor**
{% highlight xml %}
<button class="btn btn-primary" @onclick="State.IncrementCounter">Click me</button>
@code {
    [CascadingParameter] protected CounterState State { get; set; }
}
{% endhighlight %}

**CounterResetButton.razor**
{% highlight xml %}
<button class="btn btn-primary" @onclick="State.ResetCounter">Reset Counter</button>
@code {
    [CascadingParameter] protected CounterState State { get; set; }
}
{% endhighlight %}

This is an improvement over the the first iteration, however this solution is still mediocre in my opinion. The problem is that you have to pass down the state object and it may be "misused" by the child components if they read data directly from the state.

To improve on this concept, we need another class that will "manage" the state without exposing the state itself down the tree, while keeping it read-only.

This is where C# 's **nested** classes come into play, because a nested class can access the private members of the parent class.


### Iteration 3 - Using a "StateManager" nested class

So, we will create a **"StateManager"** class, nested in the **"State"** class, that
will contain all the logic for altering the state. This new class will then be passed down the tree for the child components to consume.

**CounterState.cs**
{% highlight csharp %}
/* Optional: Created as partial class so that the Manager class can be in a different file. */
public partial class CounterState
{
    public int CurrentCount { get; private set; }
}
{% endhighlight %}

**CounterStateManager.cs**

{% highlight csharp %}
public partial class CounterState
{
    // This is nested inside the CounterState class
    // Bonus: We can reuse a generic name like "Manager" because to reference 
    //        this class, you have to do it through the CounterState class:
    //        e.g. var manager = new CounterState.Manager(state);
    public class Manager
    {
        private readonly CounterState state;
        public Manager(CounterState state)
        {
            this.state = state;
        }
        public void IncrementCounter()
        {
            //We can access private setter of CurrentCount, because this class is nested inside CounterState
            this.state.CurrentCount++;
            OnCountChanged?.Invoke();
        }
        public void ResetCounter()
        {
            //We can access private setter of CurrentCount, because this class is nested inside CounterState
            this.state.CurrentCount = 0;
            OnCountChanged?.Invoke();
        }
        public event Action OnCountChanged;
    }
}
{% endhighlight %}

**Counter.razor**
{% highlight xml %}
<h1>Counter</h1>
<CascadingValue Value="stateManager">
    <CounterDisplay CurrentCount="state.CurrentCount" />
</CascadingValue>
@code{
    private CounterState state;
    private CounterState.Manager stateManager;
    protected override void OnInit()
    {
        state = new CounterState();
        stateManager = new CounterState.Manager(state);
        stateManager.OnCountChanged += StateHasChanged;
    }
}
{% endhighlight %}

**CounterDisplay.razor**
{% highlight xml %}
<p>Current count: @CurrentCount</p>
<CounterIncrementButton  />
<CounterResetButton />
@code {
    [Parameter] protected int CurrentCount { get; set; }
}
{% endhighlight %}

**CounterIncrementButton.razor**
{% highlight xml %}
<button class="btn btn-primary" @onclick="StateManager.IncrementCounter">Click me</button>
@code {
    [CascadingParameter] protected CounterState.Manager StateManager { get; set; }
}
{% endhighlight %}

**CounterResetButton.razor**
{% highlight xml %}
<button class="btn btn-primary" @onclick="StateManager.ResetCounter">Reset Counter</button>
@code {
    [CascadingParameter] protected CounterState.Manager StateManager { get; set; }
}
{% endhighlight %}

In my opinion, this is a much cleaner approach. We've seperated the **state** from the logic that updates it and as a bonus 
the child components cannot misuse the state object.

### Conclusion

I like this approach and this is what I am currently using in my projects. Maybe this is not as optimal as a dedicated state management library theoretically is, 
but this way you are avoiding boilerplate code and extra dependencies.

What do you think? Would you do something like this or use a library? Feel free to comment your own ideas and suggestions.

Thank you for reading.