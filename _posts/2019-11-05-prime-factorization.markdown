---
layout: post
title: Fast Factorization
date: 2019-11-04 18:25:00
description: Exercism - Perfect Numbers
---
Given a positive integer $$N$$, how would you find all of its divisors? You might try dividing $$N$$ by every number between $$2$$ and $$N/2$$.  This would be fine for small numbers, but clearly becomes inefficient as $$N$$ grows -- there must be a smarter way.  The "Perfect Numbers" [exercise]() asks a related question: given $$N$$, is the sum of its divisors (excluding $$N$$ itself) less than, greater than, or equal to $$N$$? Using the exhaustive search as a benchmark solution, I set out to see how much faster I could solve the problem.

---
### Background Research

Something about even my solution being slow when large primes are factors

#### Guiding Example

A quick Google search [reveals](https://www.quora.com/What-is-an-efficient-algorithm-to-find-divisors-of-any-number) that prime factorization can be exploited to quickly find all of $$N$$'s divisors in cases where $$N$$ has no large prime factors.  Specifically, we find the prime factorization of $$N$$ and then vary the frequency of each factor. This technique exploits the [fundamental theorem of arithmetic](https://en.wikipedia.org/wiki/Fundamental_theorem_of_arithmetic), which states that every integer greater than 1 can be uniquely represented as the product of prime numbers. 

As an example, consider $$N=18$$:

$$ 18 \ \ = 2^1 \times 3^2  \ \ \ (\text{prime factorization})$$

$$ 6 \ \ \ \ = 2^1 \times 3^1$$

$$ 2 \ \ \ \ = 2^1 \times 3 ^ 0$$


$$ 9 \ \ \ \ = 2^0\times 3^2$$

$$ 3 \ \ \ \ = 2^0\times 3^1 $$

$$ 1 \ \ \ \ = 2^0\times 3^0$$

Notice that we held 2's exponent constant as 3's exponent decreased from 2 to 1, and then we reduced 2's exponent and repeated the variation for 3.  

---
### Proof that varying powers finds all factors.

#### Statement.

Suppose that $$N$$ is a positive integer, $$N\geq 2$$, with prime factorization

$$N = x_1^{a_1}x_2^{a_2}\cdots x_n^{a_n}, \ \ 1\leq a_i, \ \ x_i \ \ \text{prime}$$.

Suppose also that $$p$$ is an integer divisor of $$N$$.  Then $$p$$'s prime factorization can be written as

$$ p = x_1^{b_1}x_2^{b_2}\cdots x_n^{b_n}, \ \ 0\leq b_i\leq a_i $$.

#### Proof.

If $$p$$ is a factor of $$N$$, then there exists an integer $$q$$ such that $$pq = N$$. By the fundamental theorem of arithmetic, $$q$$ has its own prime factorization:

$$ q = y_1^{c_1}y_2^{c_2}\cdots y_m^{c_m}$$.

By substitution we get

$$ p \times (y_1^{c_1}y_2^{c_2}\cdots y_m^{c_m}) = x_1^{a_1}x_2^{a_2}\cdots x_n^{a_n} $$.

By the fundamental theorem, prime factorizations are unique and thus for each $$y_i$$ there exists $$x_j$$ such that $$y_i = x_j$$.  Moreover, it must be that $$c_i \leq a_i$$.  Construct $$\{\gamma_1, \ \ldots, \gamma_m, \gamma_{m+1}, \ldots, \gamma_n\}$$ so that $$y_i = x_{\gamma_i}$$ for $$1\leq i\leq m$$. Then by reordering the right hand side we have

$$ p \times (y_1^{c_1}y_2^{c_2}\cdots y_m^{c_m}) = x_{\gamma_1}^{a_{\gamma_1}}x_{\gamma_2}^{a_{\gamma_2}}\cdots x_{\gamma_n}^{a_{\gamma_n}} $$.

Divide both sides by $$y_i^{c_i}$$ to get

$$ p = x_{\gamma_1}^{a_{\gamma_1} - c_1}x_{\gamma_2}^{a_{\gamma_2} - c_2}\cdots x_{\gamma_m}^{a_{\gamma_m} - c_m}x_{\gamma_{m+1}}^{a_{\gamma_{m+1}}}\cdots x_{\gamma_n}^{a_{\gamma_n}} $$.

Without loss of generality, let $$b_i = a_{\gamma_i} - c_i$$ for $$1\leq i\leq l$$ and $$b_i = \gamma_i$$ otherwise:

$$ p = x_{\gamma_1}^{b_1}x_{\gamma_2}^{b_2}\cdots x_{\gamma_n}^{b_n} $$.

We have expressed $$p$$ as a product of the prime factors of $$N$$ with powers $$b_i$$, $$0\leq b_i\leq a_i$$.  

QED.

---
### Algorithm Sketch

There are two pieces to this solution: 1) finding the prime factorization, 2) calculating each product made from varying the powers in the prime factorization.



Graph of distribution of times depending on the size of N.


The problem "Sum Of Multiples" is to find the sum of all the unique multiples of particular numbers up to but not including a given number.  For example, suppose the limit number is 8 and the sequence numbers are 2 and 3.  Then the result would be 2 + 3 + 4 + 6 = 15. (Notice that 6 was not counted twice, even though it is a multiple of both 2 and 3.)

Since Scala supports both functional and imperative programming, I thought it would be interesting to write two different solutions. Simply put, writing code in a functional style means avoiding the use of mutable variables while an imperative style has no such restriction.  This [article](https://codeburst.io/a-beginner-friendly-intro-to-functional-programming-4f69aa109569) provides a more thorough introduction to the differences between the two.

Near the end of this post, I compare the performance of my implementations to that of one of the top community solutions to the problem.  (Spoiler alert) the community solution topped my own by an order of magnitude. My thoughts on that difference at the end.

---
### Solution structure.

The test suite that Exercise provides with their problems informs the object/class design.  For this problem, each of the tests is similar to 

{% highlight scala %}
test("multiples of 3 or 5 up to 10") {
  SumOfMultiples.sum(Set(3, 5), 10) should be(23)
}
{% endhighlight %}

The solution will be an object or case class called SumOfMultiples with a public method "sum."  I chose to go with an object instead of a case class because the features of a case class (constructor implementation, equality checks, etc.) will not be necessary for this problem.

{% highlight scala %}
object SumOfMultiples {

  def sum(factors: Set[Int], limit: Int): Int = {
  
  // for each factor in factors, find all multiples less than limit

  // add all the distinct multiples and return

  }

}
{% endhighlight %}

*What data structure should the multiples be stored in?* Note that Scala's Set keeps only distinct values which is perfect for our use case.  We could use a list or an array, but then we would have to remove any duplicates before summing the multiples to get our solution. 

### Imperative solution.

For the imperative solution, we create a List variable to keep track of all the multiples as we loop over the set of factors.  This list is later turned into a Set to remove the duplicate values, and the Set is then summed.  I chose not to use a Set for keeping track of values because added elements to a List is less cumbersome (Set(x0).concat(Set(x1)) vs. x1 :: List(x0)).

{% highlight scala %}
object SumOfMultiples {
	
  def sum(factors: Set[Int], limit: Int): Int = {

    var multiples = List[Int]()               // create an empty list to store all multiples
                                              // (adding values to lists is easier than adding to sets)
    for (f <- factors) {
      var i = 1
      while ((i * f) < limit) {               // check that the current multiple is less than limit	
        multiples = (i * f) :: multiples      // append the multiple to the list
        i += 1
      }
    }
    
    multiples.toSet.sum  // duplicate values are removed when the list is converted to a set
  }

}
{% endhighlight %}

### Functional solution.

For the functional solution, I used for-yield comprehension to create a Set of Sets - one Set of multiples for each factor.  This Set of Sets is then flattened (removing the duplicate multiples) and summed.  

{% highlight scala %}
object SumOfMultiples {

  def sum(factors: Set[Int], limit: Int): Int = {

    val multiples = for (f <- factors) yield {  
      Iterator                                
        .iterate(f){ _ + f }	// see note (1) below
        .takeWhile(_ < limit)	// see note (2) below
        .toSet
    }

    multiples.flatten.sum  // multiples is a set of sets, we must 
                           // flatten it to one set before summing

  }
}
{% endhighlight %}

Some notes about the implementation:

1) iterate is a method that "generates the sequence resulting from the repeated application of a function to a start element".  For our problem, we need to repeatedly advance a number by one multiple until the limit is passed.  That is, given a starting number $$n$$, we want to create $$\{n, n + n, ...\}$$ until the sum is greater than the provided limit. In terms of the iterate method, the function we are repeatedly applying is $$f(x) = x + n$$ and the starting element is $$n$$. 

2) iterate needs to be told when to stop applying the function. We apply the takeWhile() method which is an iterator returning elements from the object it is called on as long as a condition is satisfied.  In our case, the condition is that the sum is less than the limit.

---
### Benchmarking solutions.

Okay, so I managed to write both an imperative and functional solution to the same problem... so what? While the functional code is cleaner, was there any performance benefit? It certainly took a lot longer to create the solution.  

To test performance, I timed how long it took each function to calculate the sum 100 times with factors = Set(2, 3, ..., 9, 23, 42), limit = 10,000.  I did this 10 times each to get min/mean/max values in case there was a wide distribution in performance.  Exercism allows users to sort community solutions by upvote count, so I decided to include one of the [highest ranked](https://exercism.io/tracks/scala/exercises/sum-of-multiples/solutions/26a2eef4e25b4a84835d5acc6468d607) solutions in my benchmark test. 

The min, mean, and max times (in seconds) were:

* Imperative: 3.26, 3.34, 3.45

* Functional: 3.15, 3.37, 3.53

* Community: 0.24, 0.25, 0.26


The imperative and functional implementations had approximately the same performance; both did far worse than the top community solution:

{% highlight scala %}
  def sum(factors: Set[Int], limit: Int): Int = {
    factors.flatMap(f => f until limit by f).sum
  }
{% endhighlight %}

---
### Parting thoughts.

In hindsight, I see that the route I took was too complicated.  Instead of looking into creating a range list in Scala (as in the community solution), I thought the best way would be to use a for-yield statement with a while condition (leading me to [this](https://stackoverflow.com/questions/26558120/may-a-while-loop-be-used-with-yield-in-scala) Stackoverflow post and the use of an Iterator).  **Ultimately, there is a balance to be found between researching implementations to exercises and finding your own way.**  Since the range list creation was (to me at least) the crux of the problem, I am not disappointed that my solution was slow.  On the flip side, I am glad that Exercism ranks community solutions so that I can learn from others' expertise!  


