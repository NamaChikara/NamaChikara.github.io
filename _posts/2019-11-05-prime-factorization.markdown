---
layout: post
title: Fast(er) Factorization
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

Without loss of generality, let $$b_i = a_{\gamma_i} - c_i$$ for $$1\leq i\leq l$$ and $$b_i = a_{\gamma_i}$$ otherwise:

$$ p = x_{\gamma_1}^{b_1}x_{\gamma_2}^{b_2}\cdots x_{\gamma_n}^{b_n} $$. 

QED.

---
### Implementation

There are three steps to solving the Perfect Numbers problem using the factorization method from above. 

1) find the prime factorization of $$N$$
2) calculate each factor of $$N$$ by calculating each product made from varying the powers in the prime factorization
3) excluding $$N$$, sum each of its factors and compare the result to $$N$$

#### Step 1: Prime Factorization

I'm going to use direct search factorization for this solution.  It is easy to implement and fast, provided that $$N$$ doesn't have any large prime factors.  An outline of the method is:

{% highlight scala %}
factors = ()
d = 2

while (N > 1) 
  if (N % d == 0) 
    factors += d
    N = N / d
  else
    if (d == 2)
      d + 1
    else
      d + 2
{% endhighlight %}

Note that even though d is not always prime, non-prime odd numbers will never be factors of the current value of N; their own prime factors will have already been divided out. Instead of dividing by every odd number, one could create a list of prime numbers from 2 to N in advance using a method such as the [Sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes).

To keep track of the factors, I'm going to use a Map with factors as keys and their frequencies in the factorization as values.  This will set us up nicely for step (2). 

{% highlight scala %}
val next_odd_number: Int => Int = (x) => (x + 1) / 2 * 2 + 1

def get_prime_factors(value: Int, factor: Int = 2,
                        all_factors: Map[Int, Int] = Map[Int, Int]().withDefaultValue(0)):
Map[Int, Int] = {

  if (value == 1) {
    all_factors
  } else if (value % factor == 0) {
    // divide the factor out of value, increment the value it is pointing to, and repeat
    get_prime_f(value / factor, factor, all_factors + (factor -> (all_factors(factor) + 1)))
  } else {
    // check the next odd number 
    get_prime_f(value, next_odd_number(factor), all_factors)
  }

}
{% endhighlight %}

#### Step 2: Calculate Each Product of Primes

This part is a little more complicated.  Given a prime factorization $$x_1^{a_1}\cdots x_n^{a_n}$$, we need to calculate every possible product $$x_1^{b_1}\cdots x_n^{b_n}$$ where $$0\leq b_i\leq a_i$$.  We'll make sure we hit every product by moving each $$b_i$$ along a sequence: 

$$b_n$$) Starting at $$a_n$$, decrease $$b_n$$ by 1 after each calculation. When $$b_n$$ would decrease to -1, cycle back to $$a_n$$.

$$b_i$$, $$2\leq i\leq n-1$$) Starting at $$a_i$$, decrease $$b_i$$ by 1 after every $$(a_{i+1}+1)\cdots(a_{n}+1)$$ calculations. When $$b_i$$ would decrease to -1, cycle back to $$a_i$$. 

$$b_1$$) Starting at $$a_1$$, decrease $$b_1$$ by 1 after every $$(a_{2}+1)\cdots(a_{n}+1)$$ calculations. When $$b_1$$ would decrease to -1, all factors have been found.

If you look back to the example at the top of the page, you'll see that $$b_n = 2$$ decreases by 1 until it gets to 0 and then returns to its initial value of 2.  Meanwhile, $$b_1$$ decreases by 1 after every $$b_n$$ calculations until the next decrease would take it to -1: at this point, all the factors have been found.  Try the pattern for yourself with a larger number such as $$90 = 2\cdot3^2\cdot5$$,.

Before sketching the algorithm, let $$w_i$$ represent the "repeat count" (i.e. the number of calculations that $$b_i$$ remains constant for).  Then $$w_n=1$$ and $$w_i = (a_{i+1} +1)\cdots(a_{n} +1)$$, $$1\leq i\leq n-1$$. Also, note that the "cycle back from 0 to $$a_i$$" behavior is simply expressed by taking $$b_i \ \text{mod} \ a_i$$ at each step. With that in mind:

{% highlight scala %}
factors = ()
j = 0

while j < num_factors
  factor = 1
  for i in 1:n
    bi = ai - (1 / wi) % (ai +1)
    factor = factor * (xi ^ bi)
  factors += factor
{% endhighlight %}

A functional implementation of this algorithm in Scala is:

{% highlight scala %}
def get_all_factors(input: Int): Set[Int] = {

  val prime_factors = get_prime_factors(input)

  // create a map from each factor to its repeat count, using 
  // reduceOption to handle the case of the largest prime factor
  // where there are no keys larger than it
  val repeat_count = prime_factors.keys.map(
    x => x -> prime_factors
      .view
      .filterKeys(_ > x)
      .values
      .map(_ + 1)
      .reduceOption((a, b) => a * b)
      .getOrElse(1)  
    ).toMap
    
  // calculate the overall number of factors
  val num_factors = prime_factors.values.map(_ + 1).reduce(_ * _)

  val all_factors = for (i <- 0 until num_factors) yield {
    // raise each prime factor to the current sequence value its
    // exponent is at, then multiply all the results together
    prime_factors.keys.map(x =>
      scala.math.pow(x.toDouble, prime_factors(x) - (i / repeat_count(x)) % (prime_factors(x) + 1))
    ).reduce(_ * _).toInt
  }

  all_factors.toSet
}
{% endhighlight %}

#### Step 3: Check If Perfect

This part is easy now that we have the functions from steps 1 and 2. We'll limit the domain of this function to $$N > 0$$.  We'll use the [Either/Left/Right pattern](https://alvinalexander.com/scala/scala-either-left-right-example-option-some-none-null) so that an error message can be returned to the user.

{% highlight scala %}
def classify(input: Int): Either[String, NumberType.Value] = {
  if (input < 1) Left("Classification is only possible for natural numbers.")
  else {
    val aliquot = get_all_factors(input).filter(_ != input).sum
    aliquot match {
      case `input` => Right(NumberType.Perfect)
      case x if x > `input` => Right(NumberType.Abundant)
      case _ => Right(NumberType.Deficient)
    }
  }
}
{% endhighlight %}

---
### Benchmarking


Graph of distribution of times depending on the size of N.  


---
### Parting Thoughts

This post took a lot longer to write than I thought it would, but it was a great exercise in technical exposition.  

The hardest section to write was definitely describing the algorithm in Step 2.  I figured out the formulation of the cyclic exponent sequences on a piece of scratch paper, and I'm sure I could explain it in person.  However, "in person" implies back and forth conversation, not a one-way recitation.  I think that I could improved my explanation by including a graphic that shows what the sequences look like for a large number such as $$2^3\times 4^2\times 5^4$$. 

My favorite section is the proof that varying powers finds all factors. While fairly trivial, I really miss the proof-based coursework of grad school. Writing elegant functions only partially scratches the itch to prove a good theorem.
