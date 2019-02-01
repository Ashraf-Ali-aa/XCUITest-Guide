---
layout: default
title: Xcode setup
parent: XCUITest
nav_order: 2
tags: 
    - testing
    - xcode
---

# Xcode
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Xcode setup for UI testing

### Create UI testing target.
If you have an existing project and would like to add automated UI tests to it, first you need to create iOS UI testing target. This is how you do it.

> 1. Open your Xcode project.
> 2. Go to: **File** -> **New** -> **Target**
> 3. From the window Choose a template for your new target: select **iOS UI Testing Bundle** and hit **Next**:
> 4. From the window **Choose options for your new target**: select your Team and Target to be tested
> 5. Select **Finish** button and new test target has been created.


### Create UI test file.

> 1. Pick the location in your Project navigator where would you like your test file to be created.
> 2. Right-click and select New File...
> 3. From the window Choose a template for your new file select UI Test Case Class and hit Next button.
> 4. In the Choose options for your new file: window provide class name and hit Next button.
> 5. Select the location where you want the file to be created and hit Create button.
> 6. You have just created your UI test class with setUp, tearDown and testExample methods.

{% highlight ruby linenos %}
def show
  ...
end
{% endhighlight %}

Though you can add as many categories as you like, I recommend not to exceed 10. Too much of anything is bad.
{: .note}


1. google
2. xcuitest
{: .source}