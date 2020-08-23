---
layout: post
title: Importing A Class - Scala REPL
date: 2019-12-02 18:25:00
description: FiniteDifferences Project - Part II
---
I decided to kick off the first package in my FiniteDifferences project on an easy note --- implementing a class for polynomial functions. After writing this class, I wanted to import it into the Scala REPL so that I could test user friendliness and identify bugs. This process was more difficult than I anticipated, so I wanted to capture the method(s) I ended up using here.

For those who don’t know, REPL stands for Read-Eval-Print-Loop.  A REPL is a programming environment that takes single user inputs, evaluates them, and returns the result to the user.  The command-line Scala REPL can be accessed by opening a terminal and typing “scala” (assuming it is installed on your machine).  Why use it?  It's a very fast way to quickly test code segments and run small experiments (and is one of Alvin Alexander’s [best practices](https://alvinalexander.com/scala/scala-best-practices-idioms-cookbook)). 

### Two Methods

Note: The package I am writing is called `com.ztbarry.nicefunctions` and the class I wanted to import is called `Polynomial`.

#### Method 1: Compile and Import

[This guide](https://stackoverflow.com/a/47874911/11407644) shows how to compile a class via the command line and then import it into a Scala REPL. I've adopted it to my `Polynomial` class here:

1) *Navigate to the folder that contains the package's classes.*  In this case my package is called `nicefunctions` and I can navigate to it's folder by typing

<pre>$ cd ~/Documents/FiniteDifferences/src/main/scala/com/ztbarry/nicefunctions</pre>

2) *Compile the class using `scalac`.* This part is easy:

<pre>$ scalac Polynomial.scala</pre>

3) *Start the REPL with the folder that contains the class in classpath.* Classpath tells the compiler where to look for additional classes and can be specified by typing `-cp pathtofolder` after `scala`. Since I navigated to the `nicefunctions` folder in Step 1, I can reference that folder with a dot `.` on the command line.  

<pre>$ scala -cp .</pre>

4) *Import the class.*  It is important to note that the formal name of my package is `com.ztbarry.nicefunctions`, *not* simply `nicefunctions`. So to import the `Polynomial` class I need to type:

<pre>scala> import com.ztbarry.nicefunctions.Polynomial</pre>

5) *Use the class!.* I'll create the polynomial $$4.2 + 2.1x + 0.7x^2$$:

<pre>scala> val parabola = new Polynomial(Vector[Double](4.2, 2.1, 0.7))</pre>

#### Method 2: SBT

I was unable to get the above method to work initially because I was trying to import all of `nicefunctions` by typing `import com.ztbarry.nicefunctions` instead of typing `import com.ztbarry.nicefunctions._` (this silly mistake cost me hours!).  Luckily, I [posted to stackoverflow](https://stackoverflow.com/questions/59121622/sbt-compiled-package-not-working-properly-in-scala-repl?noredirect=1#comment104515151_59121622) and someone was kind enough to help. 

They pointed me towards using the command `sbt console`, which "load[s] up the same REPL environment as `$ scala` but with the addition of all of the project code" ([source](https://stackoverflow.com/a/27193122/11407644)).  Many of the same steps as Method 1 apply, but I'll list all of the steps for this method for clarity:

1) *Navigate to the folder that contains the package's classes.*  In this case my package is called `nicefunctions` and I can navigate to it's folder by typing

<pre>$ cd ~/Documents/FiniteDifferences/src/main/scala/com/ztbarry/nicefunctions</pre>

2) *Load SBT console.*

<pre>$ sbt console</pre>

3) *Import the class.*  It is important to note that the formal name of my package is `com.ztbarry.nicefunctions`, *not* simply `nicefunctions`. So to import the `Polynomial` class I need to type:

<pre>scala> import com.ztbarry.nicefunctions.Polynomial</pre>

4) *Use the class!.* I'll create the polynomial $$4.2 + 2.1x + 0.7x^2$$:

<pre>scala> val parabola = new Polynomial(Vector[Double](4.2, 2.1, 0.7))</pre>

*Note:* I could have run `sbt console` from within the `FiniteDifferences` directory to include the project code for *all* packages within the project, including `com.ztbarry.nicefunctions`.  This is the method I'll be using going forward.

### Parting Thoughts

When you're learning a new language or technology, some things that seem simple can take a lot longer than anticipated.  This is all part of the learning experience. Often times, the extra time leads to additional learnings.  If I had typed `import com.ztbarry.nicefunctions._` instead of `import com.ztbarry.nicefunctions` when trying Method 1 for the first time, I wouldn't have found Method 2 as quickly - even though it is much more useful.  Hopefully this post helps someone get moving faster!