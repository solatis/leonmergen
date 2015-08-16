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

#### Glossary

While I try to make this article as easy as possible to read for as broad an audience as possible, I cannot ignore the fact that there is terminology for a lot of concepts I will be touching. Since computer scientists and mathematicians use a terminology that often uses closely related (or even competing) terms, below is a short glossary of relevant terms that I use in this article:

##### Just what is state ?

<img src='/images/posts/blog6a.png' title='Stateful versus stateless' style='display: block; margin-left: auto; margin-right: auto;' />

Statelessness defines whether or not a computation is designed to depend on one or more preceding events in a sequence of interactions. A stateful application carefully manages an internal state when from within a computation. A stateless application, however, decouples the computation from the state and manages the state at the *interactions* between computations. As such, a computation doesn't need to worry about managing any state, only its input and output.

As you can see ib the diagram above, we do not get rid of the state altogether; we separate the code from the state in such a way that state becomes an immutable object: instead of *mutating* the state, you *transform* one state into another. This what computer scientists call [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science)): the ability to replace a computation with the value it returns without changing the behavior of the program.

Referential transparency is closely related to [object immutability](https://en.wikipedia.org/wiki/Immutable_object): the trait of an object that it cannot be changed after it has been created. As such, state immutability forces you use referential transparency, since the state cannot be altered: you can only create a new state, typically as a return value of a function.

###### State stacking

Now, at this point, you might think: stateless is a lie! Even if I design my application to use an immutable state, the underlying infrastructure might not! As such, it is impossible to build a true stateless application.

This is a correct assessment: statelessness applies only to your own problem domain. The [HTTP Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is often praised as an example of the simplicitly you can achieve using a stateless protocol. However, it builds upon a stateful protocol ([TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)), which in turn builds upon a stateless protocol ([IP](https://en.wikipedia.org/wiki/Internet_Protocol)). This is a great example of separation of concerns: HTTP is able to be stateless *because* TCP does the hard work of managing state. 

###### Example: Erlang hot code reloading

To explore the concept of state a little further, we will take a look at a language that is famous for its statelessness and scalability: [Erlang](http://www.erlang.org).

Erlang is well known for popularizing the [actor model](https://en.wikipedia.org/wiki/Actor_model) which conforms to the stateless ideology described above: instead of having an actor manage internal state, it communicates state from one actor to another.

Erlang takes the concept of immutability a little bit further than most programming languages: it also applies the concept of immutability to its own interpreter. Let's take a look at the typical operation of the Erlang interpreter.

<img src='/images/posts/blog6b.png' title='Erlang interpreter' style='display: block; margin-left: auto; margin-right: auto;' />

Where most interpreters mutate the state directly, Erlang uses an immutable state and recursively calls itself with the new state. Now why would anyone do that, doesn't that make the whole thing an incredibly expensive operations ?

*Except* that you can invert this where you keep the state the same but upgrade the code:

<img src='/images/posts/blog6c.png' title='Erlang hot code reload' style='display: block; margin-left: auto; margin-right: auto;' />

When you tell Erlang to upgrade your code, it tells the old code to remember the program's state and call the new code recursively. While not impossible, without referential transparency this problem would be much harder to solve.

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


-->
