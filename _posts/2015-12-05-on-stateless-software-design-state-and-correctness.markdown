---
layout:          post
title:           "On stateless software design: state and correctness"
date:            2015-12-05 09:00:00
categories:      code
seo_descr:       "Stateless software design is gaining in popularity. This article explains why you should care."
seo_keywords:    "stateless, state, correctness"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           false
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

| [1. What is state ?]({% post_url 2015-12-04-on-stateless-software-design-what-is-state %}) | 2. State and correctness | [3. State and scalability]({% post_url 2015-12-06-on-stateless-software-design-state-and-scalability %}) | [4. State and performance]({% post_url 2015-12-07-on-stateless-software-design-state-and-performance %}) | [5. Conclusion]({% post_url 2015-12-08-on-stateless-software-design-conclusion %})

##### State and correctness

In mathematics, many models of physical systems are deterministic: a core trait of mathematics is that you need to be able to prove your assertions, and determinism is extremely important for such a model. Computer theory and mathematical logic were closely related for a very long time; this was because computer systems were mostly doing relatively simple computations, and reasoning about these computations was deterministic.

Something catastrophical happened to computer theory in the 60s, however: the introduction of non-determinism because of exponential time algorithms (e.g. the halting problem) and randomness. By pure coincidence, this occured roughly at the same time of the introduction of parallelism in computing systems, which introduced all kinds of non-determinism in computing systems. We had a problem: how do we deal with all this non-deterministic behaviour?

Mathematicians dealt with this problem with the introduction of probability: you prove non-deterministic functions using the laws of probability just like you prove other functions. A [stochastic process](https://en.wikipedia.org/wiki/Stochastic_process) is a great example to see how mathematicians deal with probability.

In computer science, a more elaborate solution was required, since it is even more important there: if your computation may have several results, they compose to have even more possible results. Since computing systems are usually composed of many smaller systems, this could easily lead to an infinite amount of possible outcomes. Clearly a new method for reasoning about non-deterministic computing systems was required; it was not just about making individual subsystems correct, it was about reasoning about the correctness of the computing system as a whole.

###### Communicating Sequential Processes

<img src='/images/posts/blog6d.png' title='Communicating Sequential Processes' align='right' style='padding-left: 20px' />

One of the scientists that devoted much of his life to solving this problem was [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare), who some of you might know as the inventor of [quicksort](https://en.wikipedia.org/wiki/Quicksort). One of his other contributions that I like to focus on is more relevant to this article: [Communicating sequential processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes) (CSP). CSP is a formal language for describing interactions in parallel and concurrent systems, and is one of the most succesful models for providing high-level linguistic support for concurrency.

The industrial application of CSP has been focused on safety critical systems, but we do not have to fully apply CSP to derive any meaning from it: one of the key insights is being event-driven and have processes communicate based on message passing via channels. Sounds familiar?

CSP inspired many programming languages, and yes, Erlang and [Go](http://golang.org/doc/faq#csp) are two of them. Modeling your complex computing system using CSP will result in splitting it up in many smaller, stateless subsystems that communicate using channels. This allows you to better compose and reason about complex systems to the point where you can [validate your model](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=345823).

###### No shared state

CSP is limited to a *language* about concurrent systems only, but perhaps you can already see why I mention about it: it has no shared state. Systems communicate exclusively using message passing and computations do not depend upon earlier events. If you would introduce state to these components, it would make life a lot harder for you, as a mere mortal, to ensure that your system behaves correctly.

Making state explicit in the communications between these components makes you see the possible interactions better. This is partially a consequence of making it more difficult to pass around the state, so you are more inclined to only use state where you need it. In other words, the state is more **visible** so it is easier to see when it's unnecessary.

So, my case for correctness in programming is to use that idea and apply it to your daily life as a software engineer: if we can model big and complex systems as communicating sequential processes, can't we model the smaller part of our computing systems, the individual components, in the same way?

As a matter of fact, this is actually a fundamental part of [functional programming](https://en.wikipedia.org/wiki/Functional_programming) and its [function composability](https://en.wikipedia.org/wiki/Function_composition_(computer_science)), and advocates of those languages are already well aware of the advantages of stateless design. However, you do not need to use a functional language in order to use function composability; imperative languages allow for function composability just fine. For those of you that come from these type of languages, I can already hear you thinking: I **need** object mutability, how else am I going to apply changes to these objects?!

The answer is that, in fact, you don't. Let's take an example of a Counter written in a popular object-oriented language:

{% highlight java linenos %}
public class Counter {
  private Integer count;

  public void add (Integer amount) {
    this.count += amount;
  }
}

/* .. */

Counter c = new Counter ();
c.add(2);

/* c.count == 2 */

c.add(2);

/* c.count == 4 */

{% endhighlight %}

This, however, is *wrong* since now you are leaking state: you mutate `this.count`, and as such, lose referential transparency; when you apply the function twice, you end up with a different result.

Ok, so how do we get from here to a more stateless example? By separating the code from the data! It's easier than you think:

{% highlight java linenos %}
public class Counter {
  private Integer count;

  public static Counter add (Counter c, Integer amount) {
    Counter tmp = c;
    tmp.count += amount;
    return tmp;
  }
}
{% endhighlight %}

So what is going on here? Let's look at the individual steps we take in more detail:

{% highlight java linenos %}
/* Create a local copy of the object's state. */
Counter tmp = c;
/* Mutate our local state. */
tmp.count += amount;
/* Return the new Counter state. */
return tmp;
{% endhighlight %}

The key insight here is that instead of mutating an object, we return its new state; we do not mutate `Counter` anymore, we return the new object we stored in `tmp`. This means that the function remains functionally transparent: for any given state of our Counter, it will always return the same result.

This might seem a little cumbersome in this language, because it doesn't have nice constructs to create and update an object in place. The same example in a more functional friendly language would look like this:

{% highlight c++ linenos %}
class Counter {
  const int count_;

public:
  Counter(int count)
    : count_(count) { }

  Counter add(int amount) const {
    return count_ + amount; // Implicit constructor
  }
};

/* .. */

Counter c1(0);
/* c1.count_ == 0 */
Counter c2 = c1.add(2);
/* c1.count_ == 0, c2.count_ == 2 */
Counter c3 = c2.add(2);
/* c1.count_ == 0, c2.count_ == 2, c3.count_ == 4 */
}
{% endhighlight %}

Or, if you want to go all the way and completely decouple state from computations, I would advice taking a "real" functional approach by getting rid of classes altogether:

{% highlight c++ linenos %}
struct Counter {
  int count_;
}

Counter add (Counter const cur, int const amount) {
  return Counter { cur.count_ + amount };
}
{% endhighlight %}

This introduces type safety by wrapping our data in a `Counter` struct, and ensures that the objects remain unchanged as part of the function signature's use of `const`. This allows you to better reason about the function without looking at the implementation, and let the compiler do a lot of the hard work for you: ensuring that there is no shared state.

As some of you might understand by now, I am clearly advocating *against* object oriented programming, due to the fact that it strongly favours tightly coupling data with computations: the computations are actually modeled as a *part* of the data. This is the wrong approach for modeling our computing systems for the reasons stated earlier.

I hope you are now convinced that the decoupling of data and computation is the correct approach for the following reasons:

* there are more ways to refactor and reason about the components;
* it is easier to change around the order of components;
* you know what interactions the component has.

Applying this technique will help you write higher quality code and as such makes you more productive by making the compiler do the hard work of guaranteeing the claims above hold.

---

| [1. What is state]({% post_url 2015-12-04-on-stateless-software-design-what-is-state %}) | &#171; | 2. State and correctness | &#187; | [3. State and scalability]({% post_url 2015-12-06-on-stateless-software-design-state-and-scalability %}) |


