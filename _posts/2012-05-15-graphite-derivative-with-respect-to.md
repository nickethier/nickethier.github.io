---
layout: post
title: "Graphite: Derivative With Respect To"
description: ""
category: metrics
tags: [graphite]
---
{% include JB/setup %}
After reading some of [Jason Dixon's Unhelpful Graphite Tips](http://obfuscurity.com/Tags/Graphite), I wanted to share one of my own. Graphite has the derivative function, but it takes the derivative with respect to time. Sometimes thise isn't what you want... 

For example, we used to measure cpu utilization by sampling the /proc/stat file twice then taking the difference and calculating the utilization over that sample. We did it this way so we could get very fine grained details about what the cpu was doing. Just knowing a cpu is at 100% utilization doesn't mean much. 100% user time is a much different issue than %100 iowait.

To get the user utilization I had find the difference of sample one and two, then divide it by the sum of the differences of the other measured cpu operations. I knew there had to be a way to simplify this. So I started sending the raw cpu counts to graphite and opened my composer.

![cpu count](/assets/images/graphite-tip-derivative-with-respect-to/cpu-count.png)

If I take the derivative with just this I don't quite get what I want.

![cpu derivative](/assets/images/graphite-tip-derivative-with-respect-to/cpu-derivative.png)

Since I'm sampling every two seconds and theres 100 jiffies in a second on my system we see the rate of change with respect to time. And if I zoom out, the numbers get even worse.

![cpu derivative long](/assets/images/graphite-tip-derivative-with-respect-to/cpu-derivative-long.png)

But the shape is generally correct, I just need to find out a way to show the relationship against the sum of the differences instead of time.

I'm currently graphing ```dv/dt``` which is the change in value over the change in time. What I want is ```dv/da``` or the change in value over the change of all values. I can also find the change in all values with respect to time, or ```da/dt```. If I then divide those two series I can cancel out ```dt```. Graphite can do all of that!

So first I need to sum all the ```dv/dt``` values to get ```da/dt```.

    sumSeries(derivative(<host>.cpu.0.*))  ---> da/dt

Now divide that by the ```dv/dt``` I want. I'll go ahead and plot them all like before.

    divideSeries(derivative(<host>.cpu.0.*),sumSeries(derivative(<host>.cpu.0.*)))  ---> dv/da

This will give me values between 0 and 1. You can scale this by 100 if you want.

This is the result I got:

![cpu derivative final](/assets/images/graphite-tip-derivative-with-respect-to/cpu-derivative-final.png)

And without the idle (blue), I'll use the exclude() function in graphite:

    divideSeries(derivative(exclude(<host>.cpu.0.*, "idle")),sumSeries(derivative(<host>.cpu.0.*)))

![cpu derivative long without idle](/assets/images/graphite-tip-derivative-with-respect-to/cpu-derivative-long-noidle.png)

Thats it. One a side note I would like to see a way to save custom function that are made up of other function in graphite.
