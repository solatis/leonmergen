---
layout:          post
title:           "On stateless software design: state and performance"
date:            2015-12-07 09:00:00
categories:      code
seo_descr:       "Stateless software design is gaining in popularity. This article explains why you should care."
seo_keywords:    "stateless, state, performance"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           false
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

| [1. What is state ?]({% post_url 2015-12-08-on-stateless-software-design-what-is-state %}) | [2. State and correctness]({% post_url 2015-12-08-on-stateless-software-design-state-and-correctness %}) | [3. State and scalability]({% post_url 2015-12-08-on-stateless-software-design-state-and-scalability %}) | 4. State and performance | [5. Conclusion]({% post_url 2015-12-08-on-stateless-software-design-conclusion %})

##### State and performance

Scalability and performance are, albeit related, entirely different beasts. Where scalability is all about keeping your computing systems busy during increasing workload, performance is about making your computing systems **do less**. This might seem trivial, but it is something that many people overlook: as we have shown in the previous example, increasing scalability often has a negative impact on performance!

Introducing scalability to a system means you have to decrease its shared state; instead of using a single pool of **state**, we have to copy our entire state and pass it around all the time. This is clearly less efficient, but sometimes a necessity in order to make your system scale, and very often an acceptable tradeoff. Better yet, when you first make your system **correct**, then make it **scalable**, it is often **easier** to apply optimizations to make it more **performant**. Yet, many people seem to have this order mixed up: for example, they first focus on making their code performant and correct, but forgetting all about scalability; this is *wrong*. Even worse, since there is a clear trend with processor manufacturs to add more cores to their CPUs, the line between having a computing system and a system that scales becomes more and more blurred.

Having said that, there are legitimate constraints when you should prioritize performance over scalability:

* the link between individual components is unreliable;
* the link between individual components has limited bandwidth;
* the link between individual components has a great latency.

To introduce a situation where all these reasons apply, let's look at an extreme example: [the recommended standard for a Space Data System File Delivery Protocol](http://public.ccsds.org/publications/archive/727x0b4.pdf). Space missions have to cope with limited systems with limited computational power and memory and unreliable connections due to environmental restrictions. As such, this protocol focuses mostly of working in this constrained environment with protocol optimizations and doesn't worry about scalability that much.

<img src='/images/posts/blog6j.png' title='Space Data System File Delivery Protocol' style='display: block; margin-left: auto; margin-right: auto;' />

As you can see in the diagram above, the different steps in the protocol depend upon earlier events: files are split in segments, a transaction is created, segments are transferred and the source is notified when the entire file has been transferred. It has the ability to retransmit segments of a file in case the transmission was corrupted: where a **stateless** protocol would retransmit the entire file, a sensible optimization is for the destination to hold the **state** of the succesfully received segments in memory while it awaits retransmission of the corrupted segments.

---

| [3. State and scalability]({% post_url 2015-12-08-on-stateless-software-design-state-and-scalability %}) | &#171; | 4. State and performance | &#187; | [5. Conclusion]({% post_url 2015-12-08-on-stateless-software-design-conclusion %}) |
