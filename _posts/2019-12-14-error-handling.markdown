---
layout: post
title: Error Handling Methods
date: 2019-12-14 18:25:00
description: FiniteDifferences Project - Part III
---
This is the third post in my series documenting the process of writing a Scala package.  In the first two posts, I covered (1) the SBT file structure of a Scala package; (2) making a package available in the Scala REPL.  Now I want to discuss a crucial part of any function design -- error handling.  

Three popular patterns for error handling are: Option/Some/None, Either/Left/Right, and Try/Success/Failure.
To give this comparison a point of reference, let's assume that we're implementing our own division function for integers.  I'll start by defining this function with no error handling and then use each of the patterns I mentioned.


### No Error Handling

As a baseline for comparison, let's consider the case where nothing is done to handle errors. The function will either return an output successfully or throw an error.  

#### Function definition

{% highlight scala %}
def frac(a: Int, b: Int): Double = {
  a / b
}
{% endhighlight %}

#### Handling the return

The only type something is returned is when the function successfully returns a value type Double.  So there's nothing to "handle" per-se.  To avoid an unforeseen error that crashes our program, we could wrap calls to `frac` in a Try-Catch block:

{% highlight scala %}
try {
  frac(2, 5)
} catch {
  case e: Exception => // do something with the exception
}
{% endhighlight %}

#### Pros and cons

The upside to this option is the brevity of the code; the con is that the caller is responsible for checking conditions and handling errors.  

### Option/Some/None

This is the method I read about most frequently when I started my Scala journey.  Functions that employ this method return `None` if they cannot return a value, and `Some(value)` if they can.  

#### Function definition

The return signature is set as `Option[successType]`. If the function is successful, `Some(value: successType)` is returned; otherwise `None` is returned.

{% highlight scala %}
def frac(a: Int, b: Int): Option[Double] = {
  try {
    Some(a / b)
  } catch {
    case e: Exception => None
  }
}
{% endhighlight %}

#### Handling the return

A caller can handle an Option in several different ways:

1) use `getOrElse` to retrieve the actual value if the function succeeded, or a default value if it failed:

{% highlight scala %}
frac(0, 1).getOrElse(0)
{% endhighlight %}

2) treat the Option as a collection (containing 0 values in the case of `None` and 1 value in the case of `Some`):

{% highlight scala %}
frac(0, 1).foreach{ result => println(f"Inverse found: $result") }
{% endhighlight %}

3) use `map` to apply a function $$f$$ to the Option, returning `Some(f(a))` if the Option was `Some(a)` and returning `None` otherwise:

{% highlight scala %}
frac(0, 1).map(x => x + 1)
{% endhighlight %}

#### Pros and cons

This method fails to return any information to the caller that would be useful in understanding a why it failed to produce a value.  For this reason, the methods below are more attractive.  However, Option can be useful if it makes sense for a function to not have an output for every input.  In this case, returning None to the caller indicates that nothing failed but that there is no sensical output for the given input.


### Either/Left/Right

This method is similar to Option/Some/None in that a successful function call returns the value wrapped in a container.  A successful return looks like `Right(value: successType)` while a failure looks like `Left(failValue: failType)`.  `failType` can be any type, allowing the function to send an error message to the caller.

#### Function definition

The return signature is set to `Either[failureType, successType]`.  If the function is successful, `Right(value: successType)` is returned; otherwise `Left(value: failureType)` is returned. *Note:* using `Right` for successful returns is conventional, it is not required (but you really should).

{% highlight scala %}
def frac(a: Int, b: Int): Either[String, Double] = {
  if (b != 0) {
    Right(a / b)
  } else {
    Left("The second argument (b) cannot be equal to 0.")
  }
}
{% endhighlight %}

#### Handling the return

1) Pattern matching:

{% highlight scala %}
frac(0, 4) match {
  case Left(s)  => // do something with error message
  case Right(d) => // do something with returned Double
}
{% endhighlight %}

2) use `map` to apply a function $$f$$ to `Right(x: successType)`, yielding `Right(f(x))`; if the same map is applied to `Left(y: failType)`, `Left(y)` is returned unchanged:
{% highlight scala %}
frac(0, 4).map(d => f(d))
{% endhighlight %}

3) a) for-yield comprehension is similar to option (2) in case there is one result:
{% highlight scala %}
for {
  d <- frac(2, 5)
} yield {
  // do something with d
} 
{% endhighlight %}
In the case that `frac(2, 5)` is Left, the yield section is skipped and the Left value is returned. 

3) b) for-yield is especially useful for multiple results:
{% highlight scala %}
for {
  a <- frac(1, 4)
  b <- frac(4, 7)
  c <- frac(2, 3)
} yield {
  // do something with `a`, `b`, and `c`
}
{% endhighlight %}

In the case that any of the function calls return a Left, the yield section is skipped and the first Left value is returned.

**Note:** Return handling methods (2) and (3) make use of the right-biased nature of Either.  As put by the [docs](https://www.scala-lang.org/api/2.12.0/scala/util/Either.html), "right-biased" means that "Right is assumed to be the default case to operate on.  If it is Left, operations like map, flatMap, ... return the Left value unchanged."

#### Pros and cons

Unlike Option/Some/None, Either allows the function to return a descriptive error message.  However, other functions that call functions with return signature `Either[typeA, typeB]` must always implement methods to deal with `typeA` and `typeB`.  In the case of many nested functions, this can lead to a lot of boilerplate.


### Try/Success/Failure

Try is essentially the same as Either where the Left type is restricted to Throwable. 

#### Function definition

{% highlight scala %}
import scala.util.{Try,Success,Failure}

def frac(a: Int, b: Int): Try[Int] = Try {
  a / b
}
{% endhighlight %}

#### Handling the return

All of the return handling methods for Either/Left/Right apply here as well.  Instead of `Right(x: successType)`, there is `Success(x: successType)`; instead of `Left(y: failType)`, there is `Failure(y: Throwable)`.


#### Pros and cons

Since the "Left" return type is always Throwable, this reduces the complexity of handling returns from a function with signature Try[A].  However, this also makes `Try` inherently less flexible than `Either`.


### Parting Thoughts

After careful consideration of the above methods, I ultimately chose to use Either/Left/Right for my package.  I started out with Option/Some/None for the ease of use but soon realized I wanted to provide users with flexible error messages. Try/Success/Failure was a close second, but I didn't want to deal with the complexity of developing my own exceptions.  

Try implementing each of the above methods for a set of nested functions and see which one you like best! 

Here are a few write-ups that I found useful:

- [https://docs.scala-lang.org/overviews/scala-book/functional-error-handling.html#trysuccessfailure](https://docs.scala-lang.org/overviews/scala-book/functional-error-handling.html#trysuccessfailure)
- [https://alvinalexander.com/scala/scala-either-left-right-example-option-some-none-null](https://alvinalexander.com/scala/scala-either-left-right-example-option-some-none-null)
- [https://stackoverflow.com/questions/29682208/what-are-the-differences-between-either-and-option/29683007#29683007](https://stackoverflow.com/questions/29682208/what-are-the-differences-between-either-and-option/29683007#29683007)
- [https://www.scala-lang.org/api/2.12.7/scala/util/Either.html](https://www.scala-lang.org/api/2.12.7/scala/util/Either.html)


