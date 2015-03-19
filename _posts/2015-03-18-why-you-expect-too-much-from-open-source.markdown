---
layout:     post
title:      "Why you expect too much from open source"
date:       2015-03-18 15:20:02
categories: business
author:     Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github: solatis
---

![Building bridges](/images/posts/blog2.jpg)

There has been a trend in the last years where software companies completely embrace the open source model and release their product as open source for the community. This has been largely fueled by the free software movement: they have shown that groups of individuals, making software for the sake of sharing it with others, are just as capable as releasing quality software as traditional software businesses are. In a struggle to keep themselves relevant, some businesses have been founded on the idea of embracing free (as in speech) software and build their business around this product. In this post I will make an argument to show that this is a flawed way of conducting business, and why you should be wary of buying into these companies' products. 

##### The case

I am going to take the database market as a case study for various reasons:

* the database market has traditionally been a *for-profit* market, with only a few key players, all being big software firms, most of which are still around;
* there have been free software alternatives to these products which have gained major traction;
* there is a lot of innovation happening in this market as we speak;
* various new *for-profit* businesses have been founded in this sector that released their core product as free software.

The rationale from a business perspective for releasing your product as free software is relatively clear: for databases it is desirable to have a healthy developer ecosystem around it and it is easier to build this ecosystem when your product is free. The assumption here is that there is little room for small players.

##### The problem

This was the short term vision. For the long-term profitability of these businesses, I can see three strategies:

1. become a service company and aim for big support contracts from big customers;
2. become a platform to sell plug-ins;
3. exit by being acquired by a bigger firm that has other means of monetization (for example, as part of a larger platform of services).

The first two options are very viable ways to build a business. [MySQL](http://www.mysql.com/) used to be a successful service company, and [SQLite](https://sqlite.org/) seems to be doing pretty well selling compression and encryption plugins. However, there is one crucial variable lacking in these strategies: investors. They are the ones paying the bill for gaining the market share in the starting phase and the only reason they invested is for your business to become a cash cow. The first two options are unlikely to meet that demand: selling support does not scale, and selling plug-ins becomes very hard if your product is a big success and there is an actual ecosystem of volunteers surrounding it. This means that both these options are not attractive when you take on investors, and aiming for a big exit is the only viable business model for a business that releases its core product as free software.

##### The consumer wins?

That depends on your definition of consumer and winning. For the right kind of audience that wants to quickly develop a MVP and wants that to be as painless as possible, then I agree that this is a very good development. However, if none of your customers would pay for your services or product, you agree that you wouldn't be able to run a business. If you use the product of another business without making sure they can make a living out of it, you are killing your business partners and therefore shooting yourself in the foot. Thus, you want to be sure you're not buying into anything with only a short-term vision of sustainability; you want to know the long-term vision of profitability. And if that vision is "we will be selling support", and they have investors behind them, you know you are being lied to. The investors are funding the development of the technology and are not committed towards the software: they are committed to being acquired by a big player. Acquiring you as a customer is a means towards that goal, rather than the goal itself.

One could argue that any business could be acquired, even when you pay for the software. However, when the product is profitable, there is far less need to change it. When the product is given away for free, other means of monetization need to be implemented after the acquisition.

##### There is no pure market

Underneath the previous argument lies the assumption that there is a pure market in which there are only market-based production of goods. The market, however, is not pure: if the product gains traction, non-market goods may also be produced (i.e. an ecosystem of volunteers). The theory is that free software puts the power to the people: if the original producer misbehaves, is acquired or goes bankrupt a rich ecosystem of volunteers will fork the project and continue the effort.

The first thing you should do when deciding to commit yourself to a specific software vendor is try to figure out whether they will still be in business a several years down the road. For young businesses, this can be difficult: if a software vendor is releasing its product as open source and is losing money, and investors are paying your bill, this becomes a big risk to their sustainability. This shows that the argument that releasing a software product as free software makes it more resilient in the long term is flawed since it ignores the fact that a business that doesn't earn any money with its product is far more likely to go out of business in the first place. 

##### Stories from the field

The business I co-founded was a SaaS in a competitive market. As one of the few exceptions to the rule, we seemed to be one of the only businesses that were bootstrapped from scratch, instead of being investor funded. This put some serious constraints on the way we priced our software: we had to be *profitable*. Sales conversations often went like this:

- Customer: "We are not happy with our current solution."
- Me: "We are better!"
- Customer: "We worry about your business' sustainability, the competition is bigger and has more money."
- Us: "That's because they have investors."
- Customer: "And they have the same features and are cheaper!"
- Us: "That's because the investors are paying half your bill; they are actually losing money."
- Customer: "We'll call you."

In the database market, it's exactly the same at the moment: customers are being spoiled and getting used to the fact that a database server should be free software. Now, don't get me wrong: as long as there is a healthy ecosystem of a true free software community surrounding a project (like [PostgreSQL](http://www.postgresql.org/)), and there are true support-oriented businesses surrounding that ecosystem, then I would fully support the decision to go ahead with such a solution. However, if there is only one business funding the development of the software, then it is in fact no better than being a freemium product: the premium price being paid as soon as *you* are committed to the software with your entire codebase and the acquisition taking place. 

##### Paying for software is ok

The solution is simple: it is not bad to pay for software, really. If you want to make sure your software vendor remains in business, pay a fair price for the software. Realize that the only problem a business is solving when giving away its technology as free software, is the problem of market acquisition; in return it makes the problem of monetization almost unsolvable. However, if a business is truly confident in that it has good technology, and it solves real problems, people will pay for it anyway; it is only the businesses that are unsure about their competitive advantage of their technology that have to worry about customer acquisition, so they have nothing to lose with providing their software for free.
