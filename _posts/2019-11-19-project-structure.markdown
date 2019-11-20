---
layout: post
title: Project Structure (SBT)
date: 2019-11-19 18:25:00
description: My First Scala Project
---
I've been spending more time reading [Programming in Scala](https://www.oreilly.com/library/view/programming-in-scala/9780981531687/) than working through Exercism problems recently. I realized that to properly practice and understand the concepts, I needed to leave the command line behind and start a (small) project.  

The most common Scala project structure is [SBT](https://en.wikipedia.org/wiki/Sbt_(software)) or "Simple Build Tool."  I've been using the IntelliJ IDE which has an SBT plugin that allows user to quickly and easily create a project that follows SBT conventions.  Assuming you have the Scala and SBT plugins installed, it's as simple as opening Intellij, selecting "Create New Project" > "Scala" > "sbt".

### My Project Structure

The project I'll be working on is an implementation of various [finite difference](https://en.wikipedia.org/wiki/Finite_difference_method) numerical integration schemes for simple functions.  So my starting SBT structure looks like

{% highlight scala %}
- FiniteDifferences
  - project
    - build.properties
  - src
    - main
      - scala 
    - test
      - scala
  - build.sbt 
{% endhighlight %}

The entry point to my program, Main.scala, belongs in src/main/scala.  I'll need to create a package with several subpackages and/or classes (e.g. Polynomials, TrapezoidMethod, etc.) to support the program.  Where should this code go?

According to the [docs](https://docs.scala-lang.org/tour/packages-and-imports.html) the base package for an organization such as Google would follow the structure

<pre>GoogleProject/src/main/scala/com/google/googleproject/package1/class1.scala</pre>

I am merely an individual so I will be adopting the structure this [stackoverflow post](https://stackoverflow.com/a/5914129/11407644) describes:

<pre>FiniteDifferences/src/main/scala/org/ztbarry/finitedifferences/nicefunctions/polynomials.scala</pre>

### Writing A Package

I want to make a package of "nice" functions that my finite difference methods will accept as input.  One class of these functions will be polynomials.  Given the file structure above, the file specifying this class should look like

{% highlight scala %}

package org.ztbarry.finitedifferences.nicefunctions

class Polynomial {

}

{% endhighlight %}

The package declaration tells the compiler that everything following should be put in the `org.ztbarry.finitedifferences.nicefunctions` project.  Everything inside this project will also be visible within the file I've started above.  So if I want to reference some other class from the `nicefunctions` project, say ContinuousFunctions, I can do that here.

### Next Steps

Now that I have the structure in place, I can start the fun part - implementing the function classes and finite difference classes. Stay tuned! This project will be hosted on [Github](https://github.com/NamaChikara/FiniteDifferences).


