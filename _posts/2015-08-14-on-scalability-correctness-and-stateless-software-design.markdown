---
layout:          post
title:           "On scalability, correctness and stateless software design."
date:            2015-08-20 14:06:00
categories:      engineering
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

Mathematicians dealt with this problem with the introduction of probability: you prove non-deterministic functions using the laws of probability just like you prove other functions. A [stochastic process](https://en.wikipedia.org/wiki/Stochastic_process) is a great example to see how mathematicians deal with probability.

In computer science, a more elaborate solution was required, since it is even more important there: if your computation may have several results, they compose to have even more possible results. Since computing systems are usually composed of many smaller systems, this could easily lead to an infinite amount of possible outcomes. Clearly a new method for reasoning about non-deterministic computing systems was required; it was not just about making individual subsystems correct, it was about reasoning about the correctness of the computing system as a whole.

###### Communicating Sequential Processes

<img src='/images/posts/blog6d.png' title='Communicating Sequential Processes' align='right' style='padding-left: 20px' />

One of the scientists that devoted much of his life to solving this problem was [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare), who some of you might know as the inventor of [quicksort](https://en.wikipedia.org/wiki/Quicksort). One of his other contributions that I like to focus on is more relevant to this article: [Communicating sequential processes](https://en.wikipedia.org/wiki/Communicating_sequential_processes) (CSP). CSP is a formal language for describing interactions in parallel and concurrent systems, and is one of the most succesful models for providing high-level linguistic support for concurrency.

The industrial application of CSP has been focused on safety critical systems, but we do not have to fully apply CSP to derive any meaning from it: one of the key insights is being event-driven and have processes communicate based on message passing via channels. Sounds familiar?

CSP inspired many programming languages, and yes, Erlang and [Go](http://golang.org/doc/faq#csp) are two of them. Modeling your complex computing system using CSP will result in splitting it up in many smaller, stateless subsystems that communicate using channels. This allows you to better compose and reason about complex systems to the point where you can [validate your model](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=345823).

###### No shared state

CSP is limited to a *language* about concurrent systems only, but perhaps you can already see why I mention about it: it has no shared state. Systems communicate exclusively using message passing and computations do not depend upon earlier events. If you would introduce state to these components, it would make life a lot harder for you, as a mere mortal, to ensure that your system behaves correctly.

Making state explicit in the communications between these components makes you see the possible interactions better. This is partially a consequence of making it more difficult to pass around the state, so you are more inclined to only use state where you need it. In other words, the state is more *visible* so it is easier to see when it's unnecessary.

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

##### State and scalability

So far we have only discussed applying stateless programming techniques to increase the correctness of your application. But as I mentioned earlier, CSP was the result of two independent problems arising in computing theory: determinism and *parallelism*. If CSP is good for increasing the determinism of your code, what about scalability?

Scalability is the capability of a system to handle a growing amount of work. We make computing systems scale by dividing up the jobs that need to be processed into smaller tasks, and run these tasks concurrently. The end result is that a system is able to process more work in the same amount of time by adding additional means for computation: in a small environment, this might mean making use of more computing cores, while in larger environments this most likely means dividing the work over multiple discrete computers.

<img src='/images/posts/blog6e.png' title="Amdahl's law" align='right' style='padding-left: 20px' />

###### Amdahl's law

Many very wise people have said wise things about achieving scalability, but what I like to focus on is [Amdahl's law](https://en.wikipedia.org/wiki/Amdahl%27s_law), which is often used in parallel computing to predict the maximum speedup when adding more processors or computers to a cluster. In layman's terms, it involves determine the parts of your program which need to be executed sequentially. For example, if 5% of your code needs to be executed sequentually, and your total computation takes 20 hours, it will be impossible to complete in less than 1 hour.

So, to phrase this knowledge differently, in order to succesfully scale a computing system, we need to reduce the amount of code that needs to be executed sequentially. How do we do that? Well, let me first take the time under which conditions your code truely needs to be sequential.

###### Race conditions

*Those of you who are already familiar with race conditions can safely skip this chapter.*

Let's say I am trying to scale a webserver that needs to keep track of the amount of requests it has processed in order to provide meaningful statistics. I have learned that I should use multiple threads to scale an application, and at the end of each request, our application will update the same global `Counter` we used earlier:

{% highlight java linenos %}
public class Counter {
  private Integer count;

  public void add (Integer amount) {
    this.count += amount;
  }
}
{% endhighlight %}

Now, what will happen if multiple threads will try to update the counter at the same time? Because of the nature how computers work, a so-called [race condition](https://en.wikipedia.org/wiki/Race_condition#Software) will occur. As you can see in the diagram on the below, even though two threads update the counter to increment by one, only a single mutation is stored because by the time `Thread B` will store the new count, its view of the counter is outdated, and it wil overwrite and ignore the increment of `Thread A`.

<img src='/images/posts/blog6f.png' title='Race condition versus synchronization' style='display: block; margin-left: auto; margin-right: auto;' />

Ignoring some [low-level](https://en.wikipedia.org/wiki/Compare-and-swap) and [complex](https://en.wikipedia.org/wiki/Software_transactional_memory) alternatives, the go-to solution is to synchronize access to the counter by introducing [locks](https://en.wikipedia.org/wiki/Lock_(computer_science)). Locking effectively synchronizes access to a resource, making sure that only one thread can access a certain area at the same time. As such, with the introduction of locks, our `Counter` is now thread-safe:

{% highlight java linenos %}
public class Counter {
  private Integer count;

  public synchronized void add (Integer amount) {
    this.count += amount;
  }
}
{% endhighlight %}

The `synchronized` keyword in this case ensures only one thread at the same time can access this object. However, this comes with a hefty cost: we have added *sequential code* to our system which, as we determined earlier, is something we like to avoid.

###### Unlocking scalability

So what is the core problem here? The problem is shared state. Shared state needs to be synchronized, which means we have to introduce a lock. **Every time you introduce a lock to your application, you make it behave more like a single-threaded application.** 

For a simple counter this might not mean much, but when you apply this idea to more complex operations (e.g. [I/O](https://en.wikipedia.org/wiki/Scalability#Examples), a cache, etc) or higher scalability, this problem becomes more and more visisble: it forces you to carefully examine all the interactions your computation has with any global state.

When you look back at the previous chapter where I advocated stateless programming for correctness, one of the arguments I raised was that it made interactions with (global) state more visisble. If you want to scale your application, you need to be extremely aware of all interactions your code has with your data. This was one of the key traits of stateless software design: it makes the interactions your code has with any state more explicit, and as such forces you to either reduce these interactions or deal with them in a consolidated, centralized way. Exactly what we want when we want to scale our applications.

So what would a scalable, stateless alternative of our request counter look like? Let's compare the stateful versus the stateless example:

<img src='/images/posts/blog6g.png' title='Counter as global, synchronized state' style='display: block; margin-left: auto; margin-right: auto;' />

We now assign a counter to each thread, a technique known as [thread-local storage](https://en.wikipedia.org/wiki/Thread-local_storage). Instead of having to synchronize access to a global counter, each thread now has its own counter and as such removed a scalability bottleneck in our code. But wait, we now have a lot of different counters and don't really have any way to access them! How is this going to help us?

This is a consequence of thinking differently about state, and thinking about computations as multiple discrete components which have only a single way to interact with them. As such, we interact with these threads in a single, uniform way, using a single request and a response pipeline:

<img src='/images/posts/blog6h.png' title='Local counter' style='display: block; margin-left: auto; margin-right: auto;' />

We now have a far better picture of the interactions a thread has with the outside world, and as such the state becomes easier to manage: we have a single path a thread uses to communicate with the outside world, and we only have to test this path. Hence the case that stateless design enhances both correctness *and* scalability.

While this might be a perfectly acceptable solution if you wish to scale this system within a single computer, it has drawbacks when you want to scale it over multiple machines: when we want to get an overview of state of the system as a whole, we have to aggregate the statistics. The reason is that we still have state; the state is now hidden as a counter inside each thread. This, as you might guess, doesn't scale: when you have vast clusters of hundreds of computing systems the time it takes to aggregate these statistics will be non-trivial. 

When we fully apply the techniques of CSP this problem goes away:

<img src='/images/posts/blog6i.png' title='Stateless counter' style='display: block; margin-left: auto; margin-right: auto;' />

A tradeoff is made here: a real-world implementation might queue the logs and process the statistics in bulk, orchestrating multiple aggregators together, etcetera. This is an approach that many real-world scalable computing systems take, and they all have tradeoffs. Let's look at a few:

* Hadoop's [HDFS](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Staging) caches writes to its distributed file on the local server until 64MB of data has been accumulated and will then store the block. As such, the system is not realtime and is prone to data loss.
* Apache's [Storm](https://storm.apache.org/) provides a stream-processing platform very much inspired by CSP, and guarantees consistency and is very much suitable for real-time queries, but is not suitable for bulk/offline data processing.
* Key/value stores that are based on Amazon's [Dynamo](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) architecture achieve very high availability in exchange for very weak consistency: it is prone to race conditions, and applies a method called [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency) to resolve these.
* Google has a whole range of scalable software solutions, each of which has its own tradeoffs:
  * [BigTable](http://en.wikipedia.org/wiki/BigTable) does not provide atomicity or strong consistency;
  * [MegaStore](http://research.google.com/pubs/pub36971.html), [Spanner](http://en.wikipedia.org/wiki/Spanner_(database)) and [F1](http://research.google.com/pubs/pub38125.html) do not support high-throughput on geo-replicated datacenters, and as such cannot be used for strong consistency accross multiple datacenters;
  * [Mesa](http://research.google.com/pubs/pub42851.html) provides a stream-oriented data store that works with geo-replicated datacenters and high-throughput; updates are processed in batches, and as such have a delay of several minutes before they become visisble;

The takeaway is that when designing a scalable system, you need to clearly define the goal of the system and make the right tradeoffs. They all have on thing in common, though: they are all almost stateless.


##### State and performance

##### Further reading

##### Credits

I would like to thank the following people for helping me and discussing the various topics I touched in this article:

* the people of [#haskell on freenode](https://wiki.haskell.org/IRC_channel) for the discussion on state and correctness;
* [Edouard Alligand](https://fr.linkedin.com/in/edouardalligand) for his wisdom on state and scalability;
* [Jeroen Habraken](https://www.linkedin.com/in/jeroenhabraken).



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
