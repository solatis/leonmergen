---
layout:          post
title:           "On stateless software design: state and scalability"
date:            2015-12-06 09:00:00
categories:      code
seo_descr:       "Stateless software design is gaining in popularity. This article explains why you should care."
seo_keywords:    "stateless, state, scalability"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           false
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

| [1. What is state ?]({% post_url 2015-12-08-on-stateless-software-design-what-is-state %}) | [2. State and correctness]({% post_url 2015-12-08-on-stateless-software-design-state-and-correctness %}) | 3. State and scalability | [4. State and performance]({% post_url 2015-12-08-on-stateless-software-design-state-and-performance %}) | [5. Conclusion]({% post_url 2015-12-08-on-stateless-software-design-conclusion %})

##### State and scalability

So far we have only discussed applying stateless programming techniques to increase the correctness of your application. But as I mentioned earlier, CSP was the result of two independent problems arising in computing theory: **determinism** and **parallelism**. If CSP is good for increasing the determinism of your code, what about scalability?

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

Now, what will happen if multiple threads will try to update the counter at the same time? Because of the nature how computers work, a so-called [race condition](https://en.wikipedia.org/wiki/Race_condition#Software) will occur. As you can see in the diagram on the below, even though two threads update the counter to increment by one, only a single mutation is stored because by the time `Thread B` will store the new count, its view of the counter is outdated, and it wil overwrite and ignore the increment of `Thread A`:

| Thread A      | Thread B      |       | Count |
| ------------- | ------------- |-------|-------|
| Read ()       |               |&#8592;| 0     |
| Increment ()  | Read ()       |&#8592;| 0     |
| Write ()      | Increment ()  |&#8594;| 1     |
|               | Write ()      |&#8594;| 1     |

Ignoring some [low-level](https://en.wikipedia.org/wiki/Compare-and-swap) and [complex](https://en.wikipedia.org/wiki/Software_transactional_memory) alternatives, the go-to solution is to synchronize access to the counter by introducing [locks](https://en.wikipedia.org/wiki/Lock_(computer_science)). Locking effectively synchronizes access to a resource, making sure that only one thread can access a certain area at the same time. As such, with the introduction of locks, our `Counter` is now thread-safe:

| Thread A      | Thread B      |       | Count |
| ------------- | ------------- |-------|-------|
| Read ()       |               |&#8592;| 0     |
| Increment ()  |               |       | 0     |
| Write ()      |               |&#8594;| 1     |
|               | Read ()       |&#8592;| 1     |
|               | Increment ()  |       | 1     |
|               | Write ()      |&#8594;| 2     |

And to apply it to the code means adding the `synchronized` keyword to the functions that you wish to synchronize:

{% highlight java linenos %}
public class Counter {
  private Integer count;

  public synchronized void add (Integer amount) {
    this.count += amount;
  }
}
{% endhighlight %}

The `synchronized` keyword in this case ensures only one thread at the same time can access this object. However, this comes with a hefty cost: we have added **sequential code** to our system which, as we determined earlier, is something we like to avoid. 

###### Unlocking scalability

So what is the core problem here? As we discussed earlier, in order to achieve scalability, we must make our computing system able to divide up a job into smaller tasks and run these concurrently. In the example above we still have shared state, which means we have to introduce a lock. **Every time you introduce a lock to your application, you make it behave more like a single-threaded application.**

For a simple counter this might not mean much, but when you apply this idea to more complex operations (e.g. [I/O](https://en.wikipedia.org/wiki/Scalability#Examples), a cache, etc) or higher scalability, this problem becomes more and more visisble: it forces you to carefully examine all the interactions your computation has with any global state.

When you look back at the previous chapter where I advocated stateless programming for correctness, one of the arguments I raised was that it made interactions with (global) state more visisble. If you want to scale your application, you need to be extremely aware of all interactions your code has with your data. This was one of the key traits of stateless software design: it makes the interactions your code has with any state more explicit, and as such forces you to either reduce these interactions or deal with them in a consolidated, centralized way. Exactly what we want when we want to scale our applications.

So what would a scalable, stateless alternative of our request counter look like? Let's compare the stateful versus the stateless example:

<img src='/images/posts/blog6g.png' title='Counter as global, synchronized state' style='display: block; margin-left: auto; margin-right: auto;' />

We now assign a counter to each thread, a technique known as [thread-local storage](https://en.wikipedia.org/wiki/Thread-local_storage). Instead of having to synchronize access to a global counter, each thread now has its own counter and as such removed a scalability bottleneck in our code. But wait, we now have a lot of different counters and don't really have any way to access them! How is this going to help us?

This is a consequence of thinking differently about state, and thinking about computations as multiple discrete components which have only a single way to interact with them. As such, we interact with these threads in a single, uniform way, using a single request and a response pipeline:

<img src='/images/posts/blog6h.png' title='Local counter' style='display: block; margin-left: auto; margin-right: auto;' />

We now have a far better picture of the interactions a thread has with the outside world, and as such the state becomes easier to manage: we have a single path a thread uses to communicate with the outside world, and we only have to test this path. Hence the case that stateless design enhances both correctness *and* scalability.

###### Example: Shared pointers

At this point you might think that keeping a global counter in memory is not a realistic thing anyone would do and a non-problem. But these kind of abstractions are everywhere, and they are [leaky](http://www.joelonsoftware.com/articles/LeakyAbstractions.html): a famous example is a shared pointer. Using shared pointers might seem like a great idea: instead of manually managing memory allocation, the implementation of a shared pointer automatically cleans up as soon as all references to this memory area are gone. This hides complexity and state and as such allows you to write cleaner code.

But hiding the state here can be a very bad idea: hiding **state** within a **shared** pointer means we are **sharing state**. When an application allocates or copies a shared pointer, its internal reference counter will be increased, and will be decreased once the shared pointer goes out of scope. As you might have guessed, increasing and decreasing this reference count needs to be thread-safe and as such access to this reference count needs to be synchronized, otherwise memory leaks (or even worse, multiple cleanups) might occur. All this in addition to sequencing mutations of the memory the pointer points at.

As such, these shared pointers **leak state**: they try to hide the internal state of the reference count and introduce a major scalability bottleneck in the process, which is **invisible**. This is a consequence of shared pointers having side-effects that are invisible to the user. A much more sane alternative is something like C++11's [unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr), in which ownership of the allocated memory is made explicit using [move semantics](http://www.cprogramming.com/c++11/rvalue-references-and-move-semantics-in-c++11.html). The application programmer is made aware of the underlying state of the object, and this abstraction doesn't leak state.

###### Applying CSP

While thread local storage might be a perfectly acceptable solution if you wish to scale this system within a single computer, it has drawbacks when you want to scale it over multiple machines: when we want to get an overview of state of the system as a whole, we have to aggregate the statistics. The reason is that we still have state; the state is now hidden as a counter inside each thread. This, as you might guess, doesn't scale: when you have vast clusters of hundreds of computing systems the time it takes to aggregate these statistics will be non-trivial. 

When we fully apply the techniques of CSP this problem goes away:

<img src='/images/posts/blog6i.png' title='Stateless counter' style='display: block; margin-left: auto; margin-right: auto;' />

###### Scalability is a tradeoff

A tradeoff is made here: a real-world implementation might queue the logs and process the statistics in bulk, orchestrating multiple aggregators together, etcetera. This is an approach that many real-world scalable computing systems take, and they all have tradeoffs. Let's look at a few:

* Hadoop's [HDFS](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Staging) caches writes to its distributed file on the local server until 64MB of data has been accumulated and will then store the block. As such, the system is not realtime and is prone to data loss.
* Apache's [Storm](https://storm.apache.org/) provides a stream-processing platform very much inspired by CSP, and guarantees consistency and is very much suitable for real-time queries, but is not suitable for bulk/offline data processing.
* Key/value stores that are based on Amazon's [Dynamo](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) architecture achieve very high availability in exchange for very weak consistency: it is prone to race conditions, and applies a method called [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency) to resolve these.
* Google has a whole range of scalable software solutions, each of which has its own tradeoffs:
  * [BigTable](http://en.wikipedia.org/wiki/BigTable) does not provide atomicity or strong consistency;
  * [MegaStore](http://research.google.com/pubs/pub36971.html), [Spanner](http://en.wikipedia.org/wiki/Spanner_(database)) and [F1](http://research.google.com/pubs/pub38125.html) do not support high-throughput on geo-replicated datacenters, and as such cannot be used for strong consistency accross multiple datacenters;
  * [Mesa](http://research.google.com/pubs/pub42851.html) provides a stream-oriented data store that works with geo-replicated datacenters and high-throughput; updates are processed in batches, and as such have a delay of several minutes before they become visisble;

The takeaway is that when designing a scalable system, you need to clearly define the goal of the system and make the right tradeoffs. They all have on thing in common, though: they are all almost stateless.

---

| [2. State and correctness]({% post_url 2015-12-08-on-stateless-software-design-state-and-correctness %}) | &#171; | 3. State and scalability | &#187; | [4. State and performance]({% post_url 2015-12-08-on-stateless-software-design-state-and-performance %}) |
