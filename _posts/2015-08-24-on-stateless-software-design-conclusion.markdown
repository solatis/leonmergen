---
layout:          post
title:           "On stateless software design: conclusion"
date:            2015-08-24 09:00:00
categories:      code
seo_descr:       "Stateless software design is gaining in popularity. This article explains why you should care."
seo_keywords:    "stateless, scalability, correctness"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
draft:           true
---
<img src='/images/posts/blog6.jpg' class='blogimage' title='Sharing a ride is a blocking operation' />

| [1. What is state ?]({% post_url 2015-08-24-on-stateless-software-design-what-is-state %}) | [2. State and correctness]({% post_url 2015-08-24-on-stateless-software-design-state-and-correctness %}) | [3. State and scalability]({% post_url 2015-08-24-on-stateless-software-design-state-and-scalability %}) | [4. State and performance]({% post_url 2015-08-24-on-stateless-software-design-state-and-performance %}) | 5. Conclusion |

##### Conclusion

The way software developers interact with their computing systems is changing. Where so far the focus has been on correctness first and then focus on performance, scalability was mostly a non-concern. Software developer relied on CPU manufacturers to increase performance and the application would magically run faster on next generation computers.

This free lunch, however, is over. It is by now an accepted fact that an easy way to make your application more responsive is by taking advantage of multiple CPU cores concurrently. This means that the way we, as software developers, change the way we think about application design. The focus should be on **correctness** which allows for **scalability**, and after having achieved this, focus on optimizing for performance.

Stateless software design is an important way to achieve this, and I hope that by now I have convinced you to apply this technique in your daily life as a software engineer.

##### Further reading

* [Referential transparency](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science));
* [Memoization](https://en.wikipedia.org/wiki/Memoization), a compiler optimization technique closely related to referential transparency;
* [Communicating Sequential Processes](http://www.usingcsp.com/), the book written by Tony Hoare;

---

| [4. State and performance]({% post_url 2015-08-24-on-stateless-software-design-state-and-performance %}) | &#171; | 5. Conclusion |
