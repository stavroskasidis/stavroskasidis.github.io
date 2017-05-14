---
layout: post
title: "Tips & Tricks #2: Smart minification with tag helpers in ASP.NET Core"
description: "How to create and use tag helper that automatically servers the appropriate js or css file (minified or unminified)"
date: 2017-05-14 10:00:00 +0300
comments: true
keywords: "c#, csharp, tag-helpers,aspnet-core,asp-core,net-core, tips-and-tricks "
category: tips-and-tricks
tags:
- tips-and-tricks
- tag-helpers
- aspnet-core
---

When creating a new ASP.NET Core web application, the current template has the following setup for your css and js:

**Views\Shared\_Layout.cshtml**
{% highlight html %}

<!-- code omitted -->
<head>
<!-- code omitted -->
    <environment names="Development">
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
        <link rel="stylesheet" href="~/css/site.css" />
    </environment>
    <environment names="Staging,Production">
        <link rel="stylesheet" href="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.7/css/bootstrap.min.css"
              asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
              asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
        <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
    </environment>
    <!-- code omitted -->
</head>
<!-- code omitted -->

{% endhighlight %}

**bundleconfig.json**
{% highlight json-doc %}
// Configure bundling and minification for the project.
// More info at https://go.microsoft.com/fwlink/?LinkId=808241
[
  {
    "outputFileName": "wwwroot/css/site.min.css",
    // An array of relative input file paths. Globbing patterns supported
    "inputFiles": [
      "wwwroot/css/site.css"
    ]
  },
  {
    "outputFileName": "wwwroot/js/site.min.js",
    "inputFiles": [
      "wwwroot/js/site.js"
    ],
    // Optionally specify minification options
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    // Optionally generate .map file
    "sourceMap": false
  }
]

{% endhighlight %}

I am not a big fan of this setup because when you create a new **.js** or **.css** file you have to add them to the **bundleconfig.json** and also add them to the layout's head in the **"Development"** conditional part. 

But then you may run into situations like *"oops I added a script to the layout but forgot to add it to the bundler"* or *"hey, I don't want all the js files to be loaded on all the pages of my website!"*

So let's see the solution that I am currently using for my ASP.NET Core projects.

#### 1. Switching to gulp
First of all, we are going to change from the default bundler to **gulp**. We add the following gulpfile.js to our project:

{% highlight javascript %}
/// <binding BeforeBuild='clean' AfterBuild='min' />
/*
This file in the main entry point for defining Gulp tasks and using Gulp plugins.
Click here to learn more. http://go.microsoft.com/fwlink/?LinkId=518007
*/

var gulp = require("gulp"),
  concat = require("gulp-concat"),
  cssmin = require("gulp-cssmin"),
  uglify = require("gulp-uglify"),
  rename = require('gulp-rename'),
  rimraf = require('rimraf');


var paths = {
    webroot: "./wwwroot/"
};

paths.js = paths.webroot + "js/**/*.js";
paths.minJs = paths.webroot + "js/**/*.min.js";
paths.css = paths.webroot + "css/**/*.css";
paths.minCss = paths.webroot + "css/**/*.min.css";


gulp.task("clean:js", function (cb) {
    rimraf(paths.minJs, cb);
});

gulp.task("clean:css", function (cb) {
    rimraf(paths.minCss, cb);
});

gulp.task("clean", ["clean:js", "clean:css"]);

gulp.task("min:js", function () {
    return gulp.src([paths.js, "!" + paths.minJs], { base: "." })
               .pipe(uglify())
               .pipe(rename({
                    suffix: '.min'
                }))
               .pipe(gulp.dest("."));
});

gulp.task("min:css", function () {
    return gulp.src([paths.css, "!" + paths.minCss], { base: "." })
      .pipe(cssmin())
      .pipe(rename({
            suffix: '.min'
        }))
      .pipe(gulp.dest("."));
});

gulp.task("min", ["min:js", "min:css"]);

{% endhighlight %}

> **Note:** You can go [here](https://docs.microsoft.com/en-us/aspnet/core/client-side/using-gulp) for a full guide about adding **gulp** to your project.

What the **"min"** task does is scan our **.js** and **.css** files and create their **".min"** equivalent. So for example if we have **"page1.js"** and **"page2.js"**, gulp will create (side-by-side) **"page1.min.js"** and **"page2.min.js"** respectively.

By keeping the files seperate we can load them individually in our pages when appropriate. Now we can move to the next step.

#### 2. Creating a tag helper to serve the appropriate files

So by now you may be thinking, "But dude, now we have to micromanage all of our **.js** and **.css** files!". 

We are going to solve this with a set of **tag helpers**. 

The intent is that when the application is running in **"Development"**, then the non-minified version of the file should be served for debugging purposes, but for **"Staging"** and **"Production"** we definitely want the minified version for performance. 

We will create a couple tag helpers that do that automatically. 

This is the intended markup:

{% highlight html %}
<script src="~/js/page1.js" asp-append-version="true" minified-auto-switch="true"></script>
{% endhighlight %}

Notice the <span class="highlight"><span class="na">minified-auto-switch=</span><span class="s">"true"</span></span> part ? That will be our tag helper. 

When the above example code would be run in development then **"page1.js"** would be served, but when we
are running in production then **"page1.min.js"** would be served instead.

Likewise for css files:

{% highlight html %}
<link rel="stylesheet" href="~/css/site.css" asp-append-version="true" minified-auto-switch="true" />
{% endhighlight %}

Here is the full code of the tag helpers that implement the above functionality:

{% highlight csharp %}
/// <summary>
/// Base tag helper that contains the core code for "minified-auto-switch"
/// </summary>
public abstract class MinifiedAutoSwitchTagHelper : TagHelper
{
    public const string MinifiedAutoSwitchAttributeName = "minified-auto-switch";

    private IHostingEnvironment HostingEnviroment;

    protected abstract string AffectedAttribute { get; }

    public MinifiedAutoSwitchTagHelper(IHostingEnvironment hostingEnviroment)
    {
        HostingEnviroment = hostingEnviroment;
    }

    public override int Order
    {
        get
        {
            return int.MinValue;
        }
    }

    /// <summary>
    /// Value indicating if the file path should automatically switch to its' normal 
    /// or its' minified version, depending on the enviroment (Development: normal,
    /// Staging,Production: minified)
    /// </summary>
    [HtmlAttributeName(MinifiedAutoSwitchAttributeName)]
    public bool? MinifiedAutoSwitch { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        TagHelperAttribute originalAttribute;
        if (MinifiedAutoSwitch.HasValue && 
            MinifiedAutoSwitch.Value && 
            context.AllAttributes.TryGetAttribute(AffectedAttribute, out originalAttribute) &&
            originalAttribute.Value != null)
        {
            string attributeValue = originalAttribute.Value.ToString();
            var extension = Path.GetExtension(attributeValue);
            if (HostingEnviroment.IsDevelopment())
            {
                if (attributeValue.EndsWith(".min" + extension))
                {
                    attributeValue = attributeValue.Replace(".min" + extension, extension);
                }
            }
            else
            {
                if (!attributeValue.EndsWith(".min" + extension))
                {
                    attributeValue = attributeValue.Replace(extension, ".min" + extension);
                }
            }

            output.Attributes.SetAttribute(AffectedAttribute, attributeValue);
        }
    }
}


/// <summary>
/// The javascript "minified-auto-switch" tag helper
/// </summary>
[HtmlTargetElement("script", Attributes = MinifiedAutoSwitchAttributeName)]
public class MinifiedAutoSwitchScriptTagHelper : MinifiedAutoSwitchTagHelper
{
    public MinifiedAutoSwitchScriptTagHelper(IHostingEnvironment hostingEnviroment) 
          : base(hostingEnviroment)
    {
    }

    protected override string AffectedAttribute
    {
        get
        {
            return "src";
        }
    }
}

/// <summary>
/// The css "minified-auto-switch" tag helper
/// </summary>
[HtmlTargetElement("link", Attributes = MinifiedAutoSwitchAttributeName)]
public class MinifiedAutoSwitchLinkTagHelper : MinifiedAutoSwitchTagHelper
{
    public MinifiedAutoSwitchLinkTagHelper(IHostingEnvironment hostingEnviroment) 
          : base(hostingEnviroment)
    {
    }

    protected override string AffectedAttribute
    {
        get
        {
            return "href";
        }
    }
}

{% endhighlight %}


> **Note:** To learn more about **tag helpers** and how to add them to your project, go [here](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro).


Now when we create a new js or css file, all we have to do is add our tag helper attribute at the file's reference tag, as shown above:

{% highlight html %}
<script src="~/js/page1.js" asp-append-version="true" minified-auto-switch="true"></script>
{% endhighlight %}

Awesomeness achieved !!