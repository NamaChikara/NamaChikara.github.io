---
layout: post
title: println("Hello, world!")
date: 2019-10-27 18:25:00
description: The start of my Scala journey.
---

My team does most of our work - from data cleaning to model deployment - in R. I've gotten pretty comfortable with the language over this past
year, and I wanted to learn another language that would be useful in my career. Data science is most interesting to me when it's done at scale; the process of data wrangling and model selection takes on a whole other dimension when you're working with terabytes of data. Since some of the most popular big data tools (Spark, Flink, Akka) are implemented in Scala, I decided to move in that direction over another language (such as Python or Julia).

To start my journey, I've been working through [Programming in Scala](https://www.artima.com/shop/programming_in_scala_3ed)
and the [Scala Cookbook](http://shop.oreilly.com/product/0636920026914.do).
The former is coauthored by one of the developers of Scala, Martin Odersky, and the latter was recommended on multiple Reddit/Quora threads as an excellent resource. At the same time, I've been working through Exercism's Scala Track. They have 91 exercises ranked in order of difficulty and give you the ability to browse community solutions after you've submitted your own.  In the absence of a teacher to provide feedback on my solutions, I'm planning on comparing my work to that of others.

An essential part of my learning process is explaining the concepts I've learned to others.  That's where this blog comes in.  My plan is to write one blog post a week covering something new that I learned.  So without further ado,

{% highlight scala %}

object HelloWorld {
  def say_hello(): Unit = println("Hello, World!")
}

{% endhighlight %}