---
layout:          post
title:           "On scalability, correctness and stateless software design."
date:            2015-08-13 14:06:00
categories:      sysadmin
seo_descr:       "Stateless software design is gaining in popularity. This article explains why you should care."
seo_keywords:    "stateless, scalability, correctness"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           true
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

Statelessness is a design principle applied in software design, in order to develop scalable and robust software. This article explores the fundamentals of statelessness and why it is important to make it a fundamental part of the way you design your applications. 

##### Just what is state ?

<img src='/images/posts/blog6a.png' title='Stateful versus stateless' style='display: block; margin-left: auto; margin-right: auto;' />

Statelessness defines whether or not a computation is designed to depend on one or more preceding events in a sequence of interactions. A stateful application carefully manages an internal state when from within a computation. A stateless application, however, decouples the computation from the state and manages the state at the *interactions* between computations. As such, a computation doesn't need to worry about managing any state, only its input and output.

As you can see ib the diagram above, we do not get rid of the state altogether; we separate the code from the state in such a way that state becomes an immutable object: instead of *mutating* the state, you *transform* one state into another. This what computer scientists call [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science)): the ability to replace a computation with the value it returns without changing the behavior of the program.

Referential transparency is closely related to [object immutability](https://en.wikipedia.org/wiki/Immutable_object), the trait of an object that it cannot be changed after it has been created. As such, object immutability forces you use referential transparency, since the state cannot be altered; you can only create a new state, typically as a return value of a function.

###### State stacking

Now, at this point, you might think: stateless is a lie! Even if I design my application to use an immutable state, the underlying infrastructure might not! As such, it is impossible to build a true stateless application.

This is a correct assessment: statelessness applies only to your own problem domain. The [HTTP Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is often praised as an example of the simplicitly you can achieve using a stateless protocol. However, it builds upon a stateful protocol ([TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)), which in turn builds upon a stateless protocol ([IP](https://en.wikipedia.org/wiki/Internet_Protocol)). This is a great example of separation of concerns: HTTP is able to be stateless *because* TCP does the hard work of managing state.

At the end of the article, I will show you under which circumstances a stateful design might be appropriate.

###### Example: Erlang hot code reloading

To explore the concept of state a little further, we will take a look at a language that is famous for its statelessness and scalability: [Erlang](http://www.erlang.org).

Erlang is well known for popularizing the [actor model](https://en.wikipedia.org/wiki/Actor_model) which conforms to the stateless ideology described above: instead of having an actor manage internal state, it communicates state from one actor to another.

Erlang takes the concept of immutability a little bit further than most programming languages: it also applies the concept of immutability to its own interpreter. Let's take a look at the typical operation of the Erlang interpreter.

<img src='/images/posts/blog6b.png' title='Erlang interpreter' style='display: block; margin-left: auto; margin-right: auto;' />

Where most interpreters mutate the state directly, Erlang uses an immutable state and recursively calls itself with the new state. Now why would anyone do that, doesn't that make the whole thing an incredibly expensive operations ?

*Except* that you can invert this where you keep the state the same but upgrade the code:

<img src='/images/posts/blog6c.png' title='Erlang hot code reload' style='display: block; margin-left: auto; margin-right: auto;' />

When you tell Erlang to upgrade your code, it tells the old code to remember the program's state and call the new code recursively. While not impossible, without referential transparency this problem would be much harder to solve.


##### State and correctness

In mathematics, many models of physical systems are deterministic: a core trait of mathematics is that you need to be able to prove your assertions, and determinism is extremely important for such a model. Computer theory and mathematical logic were closely related for a very long time; this was because computer systems were mostly doing relatively simple computations, and reasoning about these computations was deterministic.

Something catastrophical happened to computer theory in the 60s, however: the introduction of non-determinism because of exponential time algorithms (e.g. the halting problem) and randomness. By pure coincidence, this occured roughly at the same time of the introduction of parallelism in computing systems, which introduced all kinds of non-determinism in computing systems. We had a problem: how do we deal with all this non-deterministic behaviour?

Mathematicians dealt with this problem with the introduction of probability: you prove non-deterministic functions using the laws of probability just like you prove other functions. A [stochastic process](https://en.wikipedia.org/wiki/Stochastic_process) is a great example to see how mathematics deals with probability.

In computer science, a more elaborate solution was required, since it is even more important there: if your computation may have several results, they compose to have even more possible results. Since computing systems are usually composed of many smaller systems, this could easily lead to an infinite amount of possible outcomes. Clearly a new method for reasoning about non-deterministic computing systems was required; it was not just about making individual subsystems correct, it was about reasoning about the correctness of the computing system as a whole.

One of the scientists that devoted much of his life to solving this problem was [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare), who some of you might know as the inventor of [quicksort](https://en.wikipedia.org/wiki/Quicksort). One of his other contributions that I like to focus on is more relevant in the light of this article: [Communicating sequential processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes) (CSP). CSP is a formal language for describing interactions in parallel and concurrent systems, and is one of the most succesful models for providing high-level linguistic support for concurrency:

$$
VendingMachine = \mu X : \{ coin, choc \} \bullet (coin \rightarrow (choc \rightarrow X))\\_\text{CSP Example for a chocolate vending machine}
$$

The industrial application of CSP has been focused on safety critical systems, but we do not have to fully apply CSP to derive any meaning from it: one of the key insights is being event-driven and have processes communicate based on message passing via channels. Sounds familiar?

Yes, in fact, Erlang is similar in spirit to. It also inspired many other programming languages, such as [go](http://golang.org/doc/faq#csp) and has been the direct foundation for [occam](https://en.wikipedia.org/wiki/Occam_(programming_language)). Modeling your complex computing system using CSP will result in splitting it up in many smaller, stateless subsystems that communicate using channels. This allows you to better compose and reason about complex systems to the point where you can [validate your model](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=345823).

CSP is limited to a *language* about concurrent systems only, but perhaps you can already see why I talk about it in this article: it has no shared state. Systems communicate exclusively using message passing and computations do not depend upon earlier events. If you would introduce state to these components, it would make life a lot harder for you, as a mere mortal, to ensure that your system behaves correctly:

* it is hard(er) to change around the order of components;
* there are less ways to refactor and reason about the components;
* you do not know what other interactions a component might have.

Making state explicit in the communications between these components makes you see the possible interactions better. This is partially a consequence of making it more difficult to pass around the state, so you are more inclined to only use state where you need it. In other words, the state is more *visible* so it is easier to see when it's unnecessary.

So, my case for correctness in programming is to use that idea and apply it to your daily life as a software engineer: if we can model big and complex systems as communicating sequential processes, can't we model the smaller part of our computing systems, the individual components, in the same way?

As a matter of fact, this is actually a fundamental part of [functional programming](https://en.wikipedia.org/wiki/Functional_programming) and its [function composability](https://en.wikipedia.org/wiki/Function_composition_(computer_science)), in which functions are self-contained and stateless. You do not need to use a functional language in order to use function composability, however; imperative languages allow for function compsability just fine:

{% highlight c linenos %}
choc_t vending_machine() {
    return choc(coin());
}
{% endhighlight %}

The example above is a perfectly fine way in C to compose functions `f` and `g` together into function `z`.

<!--

I don't know about you, but the code I feel good about is code that I know for sure works correctly, with optional bonus points for solving a complex problem at the same time. But how do we know code works correctly? Rougly, this can be divided up into two different categories:

* writing (lots of) tests;
* static analysis.

Writing tests can be a lot of work, and often can be seen as a work of art: it takes a lot of knowledge about a system to actually know what to test, and uses up a lot of creativity at the same time. Static analysis, however, is something that can be performed by computers, and as such makes for a much more ideal candidate to make your code correct: static analysis can provide you with actual *guarantees* about a piece of code.

Having said that, static analysis is often very difficult to perform: ideally, we provide the computer with lots of hints about what we actually want to do, so it can make assertions whether these traits still hold. 

-->

##### State and scalability

##### State and performance

<!---

- state and correctness

"
<ski> solatis : stateful is harmful because it's harder to change around the
      order of the components, there's less ways to refactor and reason about
      it
<ski> solatis : another big part is having to assume the worst (because you
      can't recall what things a call may change (and thus interact implicitly
      with other stuff), or you don't know because you don't have source)
<ski> solatis : making state explicit means that you see the possible
      interactions better, partly as a consequence of it being a PITA so
      you're more inclined to only use state where you need it"
      "
"
<|f`-`|f> In addition to that, with what ski mentions, since the state is
          always "visible" or there, it's easier to see where it's unnecesarry
                                                                        [16:09]
<|f`-`|f> there are functions to dealing with splitting and recombining pure
          values with the state values                                  [16:10]
"


* feel good about making software: correct code
* correct code: compiler guarantees and writing tests
* writing tests is hard and boring, compilers can guarantee things!
* allows you to reason about code
* you don't need an FP language to get your compiler to reason about code (c++)


- state and scalability

* having state is ok, sharing state is not!
* shared state is ok, if it is close together
** OS kernel has state: necessary;

* last 10 years, CPUs started bringing more cores, increasing the gap;
* the next 10 years, the gap will only widen;
* every time you lock, you make something single-threaded;
 -> how is a mutex implemented ?

* in the future, the difference between scalability and performance will only decrease! -> stateless design for performance!

- state and performance;

* having state is sometimes required when performance is an issue;
* extreme example: relay network of NASA, protocol is stateful due to bandwidth limitations;

-->
