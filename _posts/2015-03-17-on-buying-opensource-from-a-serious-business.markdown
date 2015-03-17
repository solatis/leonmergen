---
layout: post
title:  "On buying opensource from a serious business"
draft:  true
categories: business
author: Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github: solatis
---

![Building bridges](/images/posts/blog2.jpg)

There has been a trend in the last years where software companies completely embrace the opensource model and release their product as opensource for the community. This has been largely fueled by the free software movement: they have shown that groups of individuals, making software for the sake of sharing it with others, are just as capable as releasing quality software as traditional software businesses are. In a struggly to keep themselves relevant, some businesses have been founded on the idea of embracing free software and build their business around this product. In this post I will make an argument to show that this is a flawed way of conducting business, and why you should be wary of buying into these companies' products.

##### The case

I am going to take the database market as a case study for various reasons:

* the database market has traditionally been a *for-profit* market, with only a few key players, all being big software firms, most of which are still around;
* there have been free software alternatives to these products which have gained major traction;
* there is a lot of innovation happening in this market as we speak;
* various new *for-profit* businesses have been founded in this sector that released that core product as free software.

The rationale from a business perspective for releasing your product as free software is relatively clear: for databases it is desirable to have a healthy developer ecosystem around it and it is easier to build this ecosystem when your product is free. The assumption here is that there is little room for small players.

##### The problem

This was the short term vision. For the long-term profitability of these businesses, I can see two strategies:

1. become a service company and aim for big support contracts from big customers;
2. exit by being acquired by a bigger firm that has other means of monetization (for example, as part of a larger platform of services).

There is one variable lacking in these strategies, though: investors. They are the ones paying the bill for gaining the market share in the starting phase, and the only reason they invested is for your business to become a cash cow. Selling support contract hardly is a way to become a cash cow: it doesn't scale well and there is no way to be "printing money" when all your income comes from selling hours in support. As such, aiming for a big exit is the only viable business model for a business that releases its core product as free software.

##### The consumer wins?

That depends on your definition of consumer and winning. For the right kind of audience that wants to quickly develop a MVP and wants that to be as painless as possible, then I agree that this is a very good development. However, if you are a serious business looking to do serious business with another business, you want to be sure you're not buying into anything with only a short-term vision and sustainability. You want to know the long-term vision of profitability. And if that vision is "we will be selling support", and they have investors behind them, you know you are being lied to: the investors are funding the development of the technology, and are not committed towards the software; they are committed to being acquired by a big player. 

##### The solution

The business I co-founded was a SaaS in a competitive market. As one of the few exceptions to the rule, we seemed to be one of the only businesses that were bootstrapped from scratch, instead of being investor funded. This put some serious constraints on the way we priced our software: we had to be *profitable*. Sales conversations often went like this:

- Customer: "We are not happy with our current solution."
- Me: "We are better!"
- Customer: "We worry about your business' sustainability, the competition is bigger and has more money."
- Us: "That's because they have investors."
- Customer: "And they have the same features and are cheaper!"
- Us: "That's because the investors are paying half your bill; they are actually losing money."
- Customer: "We'll call you."

In the database market, it's exactly the same at the moment: customers are being spoiled and getting used to the fact that a database server should be free software. Now, don't get me wrong: as long as there is a healthy ecosystem of a true free software community surrounding a project (like PostgreSQL), and there are true support-oriented businesses surrounding that ecosystem, then I would fully support the decision to go ahead with such a solution. However, if there is only one business funding the development of the software, then it is in fact no better than being a freemium product: the premium price being paid as soon as *you* are committed to the software with your entire codebase and the acquisition taking place.

The solution is simple: it is not bad to pay for software, really. If you want to make sure your software vendor remains in business, pay a fair price for the software. Realize that the only problem a business is solving when giving away its technology as free software, is the problem of market acquisition; in return it makes the problem of monetization almost unsolvable. But if a business is truly confident in that it has good technology, and it solves real problems, people will pay for it anyway; it is only the businesses that are unsure about their competitive advantage of their technology that have to worry about customer acquisition, so they have nothing to lose with providing their software for free.
