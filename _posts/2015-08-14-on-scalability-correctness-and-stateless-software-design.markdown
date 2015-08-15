---
layout:          post
title:           "On scalability, correctness and stateless software design."
date:            2015-08-13 14:06:00
categories:      sysadmin
seo_descr:       "Stateless software design is gaining in popularity. This post explains why you should care."
seo_keywords:    "stateless, scalability, correctness"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           true
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

Statelessness is a design principle applied in software design, in order to develop scalable and robust software. This post explores the fundamentals of statelessness and why it is important to make it a fundamental part of the way you design your applications.

##### Just what is state(lessness) ?

Statelessness defines whether or not a computation is designed to depend on one or more preceding events in a sequence of interactions.

<img src='/images/posts/blog6a.png' class='blogimage' title='Stateful operation' />

As you can see in the diagram above, a stateful application carefully manages an internal state when from within a computation. A stateless application, however, decouples the computation from the state and manages the state at the *interactions* between computations. As such, a computation doesn't need to worry about managing any state, only its input and output.

As you can see, stateless in this context doesn't imply getting rid of state altogether: it involves moving the management of the state from one place to another. The key difference is that state is never shared: *having* state is ok, *sharing* state is not. Stateless software design, as such, involves separating the *code* from the *state* in such a way that state becomes an immutable object: instead of *mutating* the state, you *transform* one state into another.

To explore this a little further, let's look at a language that is famous for its statelessness and scalability: [erlang](http://www.erlang.org).

###### Erlang hot code reloading

Erlang is well known for popularizing the [actor model](https://en.wikipedia.org/wiki/Actor_model) which conforms to the stateless ideology described above: instead of having an actor manage internal state, it communicates state from one actor to another.

This is a high level abstraction and can be considered a stateless *protocol* rather than a stateless application. However, Erlang takes it a step further than most programming languages and also applies it to its virtual machine by strictly separating code from state.


###### State stacking

Now, at this point, you might think: stateless is a lie! Even if I design my application to use an immutable state, the underlying infrastructure might not! As such, it is impossible to build a true stateless application.

This is a correct assessment: statelessness applies only to your own problem domain. The [HTTP Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is often praised as an example of the simplicitly you can achieve using a stateless protocol. However, it builds upon a stateful protocol ([TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)), which in turn builds upon a stateless protocol ([IP](https://en.wikipedia.org/wiki/Internet_Protocol)). This is a great example of separation of concerns: HTTP is able to be stateless *because* TCP does the hard work of managing state. 



<!---

- what is state ?

* everything has state !
* having state is ok, sharing state is not!
* stateless software design: separating code from state
* erlang hot code upgrade example;

- stateless design for scalability

* shared state is ok, if it is close together
** extreme example: relay network of NASA, protocol is stateless
** OS kernel has state: necessary;

* last 10 years, CPUs started bringing more cores, increasing the gap;
* the next 10 years, the gap will only widen;
* every time you lock, you make something single-threaded;
 -> how is a mutex implemented ?

* in the future, the difference between scalability and performance will only decrease! -> stateless design for performance!

- stateless design for correctness

* feel good about making software: correct code
* correct code: compiler guarantees and writing tests
* writing tests is hard and boring, compilers can guarantee things!
* allows you to reason about code
* you don't need an FP language to get your compiler to reason about code (c++)


-->
