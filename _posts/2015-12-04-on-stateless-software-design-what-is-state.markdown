---
layout:          post
title:           "On stateless software design: what is state ?"
date:            2015-12-04 09:00:00
categories:      code
seo_descr:       "Stateless software design is gaining in popularity. This article explains why you should care."
seo_keywords:    "stateless, scalability, correctness"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           false
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

| 1. What is state ? | [2. State and correctness]({% post_url 2015-12-05-on-stateless-software-design-state-and-correctness %}) | [3. State and scalability]({% post_url 2015-12-06-on-stateless-software-design-state-and-scalability %}) | [4. State and performance]({% post_url 2015-12-07-on-stateless-software-design-state-and-performance %}) | [5. Conclusion]({% post_url 2015-12-08-on-stateless-software-design-conclusion %})

Statelessness is a design principle applied in software design, in order to develop scalable and robust software. This article explores the fundamentals of statelessness and why it is important to make it a fundamental part of the way you design your applications.

##### Just what is state ?

<img src='/images/posts/blog6a.png' title='Stateful versus stateless' style='display: block; margin-left: auto; margin-right: auto;' />

Statelessness defines whether or not a computation is designed to depend on one or more preceding events in a sequence of interactions. A stateful application carefully manages an internal state when from within a computation. A stateless application, however, decouples the computation from the state and manages the state at the **interactions** between computations. As such, a computation doesn't need to worry about managing any state, only its input and output.

As you can see ib the diagram above, we do not get rid of the state altogether; we separate the code from the state in such a way that state becomes an immutable object: instead of **mutating** the state, you **transform** one state into another. This what computer scientists call [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science)): the ability to replace a computation with the value it returns without changing the behavior of the program.

Referential transparency is closely related to [object immutability](https://en.wikipedia.org/wiki/Immutable_object), the trait of an object that it cannot be changed after it has been created. As such, object immutability forces you use referential transparency, since the state cannot be altered; you can only create a new state, typically as a return value of a function.

###### State stacking

Now, at this point, you might think: stateless is a lie! Even if I design my application to use an immutable state, the underlying infrastructure might not! As such, it is impossible to build a true stateless application.

This is a correct assessment: statelessness applies only to your own problem domain. The [HTTP Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is often praised as an example of the simplicitly you can achieve using a stateless protocol. However, it builds upon a stateful protocol ([TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)), which in turn builds upon a stateless protocol ([IP](https://en.wikipedia.org/wiki/Internet_Protocol)).

This is a great example of separation of concerns: TCP has enabled many great applications to be built on top of it *because* it completely hides all the complexity and state management for the applications that are built on top of it. At the end of this article I will discuss the circumstances where a stateful design makes sense.

Another example of state stacking are garbage collectors. In C-like languages, you have the freedom to manually manage your memory allocations. This comes with the cost of the added complexity of managing the state of a pointer in your application, whether the memory address the pointer points to is currently allocated or not. Garbage collectors move the state of this pointer (allocated/free) from application code to the core of the language. Garbage collectors are notorious for being complex beasts, but so is manual memory management. By hiding this complexity from the application programmer it reduces state management in the code and as such improves correctness.

###### Example: Erlang hot code reloading

To explore the concept of state a little further, we will take a look at a language that is famous for its statelessness and scalability: [Erlang](http://www.erlang.org).

Erlang wouldn't be the same without its [Open Telecom Platform](http://learnyousomeerlang.com/what-is-otp) (OTP); I frequently call Erlang a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language) for OTP. OTP grew out of a need of reducing error and increasing availability in telecom operations: when you have one switch connecting multiple telecom networks, even the slightest downtime would be highly inconvenient and undeseriable. OTP takes care of this and has been carefully engineered and battle-hardened over years.

One of the more commonly used patterns in OTP is the `gen_server`: this is the module for implementing the server in a client-server relation. One of the more interesting parts of this pattern is the way it handles state:

<img src='/images/posts/blog6b.png' title='Erlang gen_server' style='display: block; margin-left: auto; margin-right: auto;' />

Where most servers mutate the state directly, Erlang's `gen_server` uses an immutable state and recursively calls itself with the new state. Now why would anyone do that, doesn't that make the whole thing an incredibly expensive operations ?

*Except* that you can invert this where you keep the state the same but upgrade the code:

<img src='/images/posts/blog6c.png' title='Erlang hot code reload' style='display: block; margin-left: auto; margin-right: auto;' />

When you tell Erlang to upgrade your code, it tells the old code to remember the program's state and call the new code recursively. And, when an upgrade fails, you can revert to a known good state. While not impossible, without referential transparency this problem would be much harder to solve. 

---

| 1. What is state | &#187; | [2. State and correctness]({% post_url 2015-12-05-on-stateless-software-design-state-and-correctness %}) |

