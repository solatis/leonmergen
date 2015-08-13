---
layout:          post
title:           "On the state of cryptography in Haskell"
date:            2015-03-21 12:22:02
categories:      haskell crypto
seo_descr:       "When you are serious about security, you must put on a tinfoil hat and assess the quality of your application's cryptographic dependencies."
seo_keywords:    "haskell, cryptography"
author:          Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github:   solatis
---
<img src='/images/posts/blog3.jpg' class='blogimage' title='Somebody is watching you' />

In the past months, I was attempting to write an application that uses cryptographic primitives in Haskell. In the process I found out some disturbing things about the state of cryptography in Haskell of which I think more people should be aware. I am trying to present you with as many facts as possible, but I will also draw my own conclusions from these facts. My conclusions are fairly paranoid, since I think developing crypto software requires a certain level of professional paranoia. You might not necessarily agree with this and are free to draw your own conclusions.

##### Cryptography is hard

First off, I am by no mean a cryptography expert. On the contrary, a lot of my friends know a *lot* more about cryptography and security than I do. As such, I have a kind of Socratic view on cryptography: I know only one thing about cryptography, and that is that I know nothing about cryptography.

This fact, however, does not mean I am unable to assess some basic facts about a library. When I decide to use a certain library, I would like to know more about how it implemented certain primitives and compare that to what other people on the internet say. In other words: I am able to recognize when a library is implemented improperly, but not the other way around.

In this article I will take you through three of the major primitives I had to use in my application: entropy, random numbers and algorithms.

##### Entropy

Entropy is where all cryptography starts: you often need to *seed* some algorithm with random data. This is one of those areas that is easier said than done: computer processors are by definition deterministic, and that is the last thing you want your random data to be. As such, operating systems usually provide primitives for applications to get cryptographically secure random data: this is generated from background noise, user input (mouse movements) and recently Intel's [RdRand](http://en.wikipedia.org/wiki/RdRand) instruction.

This RdRand instruction deserves some more attention: it is designed to be an efficient way for a CPU to generate random data. However, even though it is based on an open standard, we know very little about the actual implementation Intel uses. In a post-Snowden area, I would say it would be a very bad idea to rely solely on this instruction: Intel engineers have (unsuccesfully) [pressured a Linux kernel maintainer](https://plus.google.com/+TheodoreTso/posts/SDcoemc9V3J) into using the RdRand exclusively as the source of random data for /dev/random. The conclusion to this discussion [caused the kernel maintainer of /dev/random to quit](http://lists.randombit.net/pipermail/cryptography/2013-July/004728.html), because he did not want to have an unauditable RdRand to bypass the entropy pool. Why would Intel want to be the exclusive source? Thanks to Snowden, it is a [public fact](http://www.nytimes.com/2013/09/06/us/nsa-foils-much-internet-encryption.html?pagewanted=all) that the NSA has infiltrated enryption chips with backdoors:

> By this year, the Sigint Enabling Project had found ways inside some of the encryption chips that scramble information for businesses and governments, either by working with chipmakers to insert back doors [...]
<cite>New York Times, Sept. 5 2013</cite>

This does not by all means mean it is bad to use RdRand as *a* source for random data; I would only be very, very wary to use it as the *only* source of random data. Linux uses RdRand as one of the sources of random data, which is perfectly fine; FreeBSD has ditched the RdRand instruction altogether.

Now, back to the question: how do we get cryptographically secure random data in Haskell? There are really three ways, really: by using the [OpenSSL bindings](https://hackage.haskell.org/package/HsOpenSSL), the [entropy](https://hackage.haskell.org/package/entropy) package or the [crypto-random](https://hackage.haskell.org/package/crypto-random) package. For the OpenSSL bindings, it is pretty clear what we can expect: for better or worse, there are a huge amount of systems using OpenSSL in production, and the disucssion about how OpenSSL acquires its random data is beyond the scope of this article. So we will focus on the other two libraries.

###### entropy

So, how does the entropy package get its random data? Well, if RdRand is available, [it uses RdRand](https://github.com/TomMD/entropy/blob/master/System/EntropyNix.hs#L45), of course! To [quote the maintainer](https://github.com/TomMD/entropy/issues/14) when concerns about *exclusively* using RdRand were raised:

> So far as can be seen these concerns are unfounded tinfoil-hattedness. RDRAND in published design and observed use is faster, safer and more portable than urandom.
<cite>Thomas DuBuisson, Jun. 11 2014</cite>

So, there you have it. It is tinfoil-hattedness to raise these concerns. The *one* area in computer science where tinfoil-hattedness is of absolutely importance is cryptography; and the maintainer of the only Haskell entropy package dismisses these very valid concerns as tinfoil-hattedness. I almost cannot believe I'm reading this: there are innocent people who don't study this code who rely on it to be secure. Why would you decide to override an operating system's default behavior, *which already mixes the RdRand instruction with other sources*, and solely rely on the RdRand instruction? It just makes no sense at all, other than a case of NIH-syndrome. The Linux kernel maintainers [mix it with other entropy sources](http://lxr.linux.no/linux+v3.11/drivers/char/random.c#L956): but even then, djb asserted that [RdRand could be spying on what it's being mixed into](http://blog.cr.yp.to/20140205-entropy.html) (the idea being that it is theoretically possible to backdoor any MOV instructions after the RDRAND instruction has been executed, to see what pool of random data it is being mixed into). After some heated discussion the maintainer of the `entropy` package decided to XOR the urandom data with the RdRand data, but I still have no idea what to make of this. At least Debian has been smart about this: they refused to upgrade to the 0.3 version of this package after they found out about this.

I really don't like to do ad hominem attacks, but this person is [behind quite a lot](https://hackage.haskell.org/user/ThomasDuBuisson) of Haskell's cryptography libraries. If this way of thinking is common across the design of his libraries, I would default to not trusting anything this person has made as cryptographically secure.

###### crypto-random

So, how does the crypto-random package get its random data? This feels like a deja-vu, but well, if RdRand is available, [it uses RdRand exclusively](https://github.com/vincenthz/hs-crypto-random/blob/master/Crypto/Random/Entropy.hs#L47), of course! I have [opened a ticket](https://github.com/vincenthz/hs-crypto-random/issues/10) about this issue 10 weeks prior to writing this article, and while there have been updates to the code base, the author has yet to respond to the ticket. In other words, once again, I don't think this author takes this very seriously.

Once again, it seems like this person is [behind quite a lot](https://hackage.haskell.org/user/VincentHanquez) of Haskell's cryptography libraries. So where does this leave us? Haskell's entire cryptography ecosystem seems to be developed by two persons who don't even take entropy seriously. That's not a good sign.

###### HsOpenSSL

So, my advice: when you are looking for a source of entropy in Haskell, I would advice to use the [HsOpenSSL.Random module](http://hackage.haskell.org/package/HsOpenSSL-0.11.1.1/docs/OpenSSL-Random.html). I would never think I would actually be recommending OpenSSL as the best alternative, but when you design software that uses cryptographic primitives and you have are wearing your tinfoil hat, this really seems to be the best option.

##### Random numbers

Well, now that we have secure entropy in the form of random bytes, we often want to convert it to a random *number* that lies within a certain range. This has its pitfalls; in cryptography, the goal is to have an uniform distribution of numbers. An obviously flawed strategy would be to use a modulo operation like this:

{% highlight python linenos %}
int x;
int n = 3;
x = rand() % n;
return x;
{% endhighlight %}

<img src='/images/posts/blog3a.png' style='width: 25%' align='right' title='Modulo bias' />

This however suffers from [modulo bias](http://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle#Modulo_bias), as illustrated in the picture on the right. To illustrate this problem, let's assume that our `rand()` function above always returns numbers within the range `0 =< n < 5`; when we align these numbers according to a modulo 3 range, the problem becomes more clear:

| **0** | **1** | **2** |
| **3** | **4** |  |

As you can clearly see, when you apply a modulo 3 to this list of numbers, number 2 has only a 20% probability of occurring, while numbers 0 and 1 have a combined probability of 80% of occuring. That's a pretty heavy bias! So, how *do* we get numbers with a uniform distribution within a range, then? Well, easy:

{% highlight python linenos %}
int x;
int n = 3;
while (x = rand() >= n) {};
return x;
{% endhighlight %}

What is going on here? We simply loop until the random number returned falls within the range we are looking for. There will be no bias: all numbers have the same probability of occurring, and there will be a uniform distribution. When we do some clever bitshifting, we can make each loop iteration have a 50% chance of falling within the range and as such should performance should be adequate for most circumstances.

What options for generating cryptographically secure random numbers in Haskell do we have? Well, there is [crypto-random](https://hackage.haskell.org/package/crypto-random), but in the previous chapter we have determined that the entropy source of this library uses RdRand exclusively, so I would advice against that. So, then we have the [crypto-numbers](https://hackage.haskell.org/package/crypto-numbers), which uses an entropy source of choice. However, when I was looking closely, I found [this little gem](https://github.com/vincenthz/hs-crypto-numbers/blob/695772b3fd22a87b78281fdbc9aed47c544751cb/Crypto/Number/Generate.hs#L25):

{% highlight haskell linenos %}
-- | generate a positive integer x, s.t. 0 <= x < m
--
-- Note that depending on m value, the number distribution
-- generated by this function is not necessarily uniform.
generateMax :: CPRG g => g -> Integer -> (Integer, g)
generateMax rng m = withRandomBytes rng (lengthBytes m) $ \bs ->
    os2ip bs `mod` m
{% endhighlight %}

So, you are designing a library called `crypto-numbers`, which is supposed to be provide cryptographically secure numeric operations. You have one thing to get right: no bias. And what do you do? You use `mod`. *Sigh*. In defense of the author, he did add the comment that depending on the requested range, the function does not necessarily generate uniformly random numbers. I would like to rephrase that into "unless your requested range is exactly based on a power of 2, in which case you can safely just use random entropy bits, this function does not generate uniform random numbers". Secondly, I fixed the issues, [requested a pull request](https://github.com/vincenthz/hs-crypto-numbers/commit/bceb5493dbe96d216c40efdf0b8eff5cc2e909cd) and the author has patched the issue since. But it does show something about the reactive instead of proactive attitude the designers have about these kind of issues. On top of that, this library *also* depends upon the flawed [crypto-random](https://hackage.haskell.org/package/crypto-random) library's entropy function, so I would advice against using this library just for that reason.

So are there no ways of generating a cryptographically secure random number in Haskell then? Well, yes, and I feel even more ashamed to repeat myself here: [use HsOpenSSL.BigNum](http://hackage.haskell.org/package/HsOpenSSL-0.11.1.1/docs/OpenSSL-BN.html#g:5). There are random number functions there, and without blinking an eye I would trust these functions over the Haskell-specific libraries.

##### Algorithms

So the old saying goes: whenever you want to write cryptographic code yourself, don't. Cryptography is all about standing on the shoulders of giants, but in Haskell, however, we have [quite a collection of home-grown cryptography libraries](https://hackage.haskell.org/packages/#cat:Cryptography). Because I have addressed serious concerns about the generating random data and deriving random numbers from that data is Haskell, I would be very, very wary of using any of these libraries in production code. It is impossible for me to assess the quality of all these libraries, but I will share a story about AES.

###### AES

So, AES is particularly interesting since it suffers from the same flaws as entropy: there is a native CPU instruction that can do the job for you, called [AESNI](http://en.wikipedia.org/wiki/AES_instruction_set). Colin Percival already [called AES-NI out on his blog](http://www.daemonology.net/blog/2014-09-06-zeroing-buffers-is-insufficient.html):

> Nearly every AES implementation using AESNI will leave two values in registers: The final block of output, and the final round key. The final block of output isn't a problem for encryption operations — it is ciphertext, which we can assume has leaked anyway — but for encryption an AES-128 key can be computed from the final round key, and for decryption the final round key is the AES-128 key.

So, in other words, when you use AESNI it will leave the decryption key sitting around in memory. This might not seem to dangerous to you, but consider the fact that there have been exploits in the past of [privilege escalation in virtualization software](https://blog.xenproject.org/2012/06/13/the-intel-sysret-privilege-escalation/), and you suddenly get a feeling about how dangerous this could be.

So, what is the most popular Haskell AES library? It seems like [cipher-aes](https://hackage.haskell.org/package/cipher-aes) is the most popular candidate. Let's examine what it does.

First of all, I am not too surprised by the fact that the library, of course, features [a complete home-grown implementation of AES](https://github.com/vincenthz/hs-cipher-aes/blob/master/cbits/aes.c). When AESNI is available, [it uses these instructions exclusively](https://github.com/vincenthz/hs-cipher-aes/blob/master/cbits/aes.c#L142). So, what about expecting the user is dumb and preventing them from making any errors? AES is known to be tricky: various modes are vulnerable to a [padding oracle attack](http://en.wikipedia.org/wiki/Padding_oracle_attack). So, looking at the code, it only seems like the newer OCB mode has built-in padding (which is required by the standard). When you use CBC mode, no padding is provided: no warnings for this, either.

The fact that this is a home-grown implementation of AES, instead of a wrapper around a known secure, peer-reviewed implementation of AES, makes me very wary. If I would use this library, I would advice my users that they should not trust their lives and safety on this code.

###### RSA

And just to raise the level of tinfoil-hatness we have going on here, I am going to leave this here: [RSA]() is a popular (the only?) Haskell implementation of the RSA assymetric encryption algorithm. It is a widely used public/private keypair encryption algorithm.

Other than the fact that it relies on the RdRand instruction through the `entropy` package, the developer of the RSA package is [Galois, Inc](http://galois.com/), a [known military contractor](http://www.defense.gov/contracts/contract.aspx?contractid=5076); they help the government on cyber warfare projects. So the developer of the implementation of the most widely used assymetric encryption algorithm, which relies on a library that uses RdRand solely as the source of random data, helps the US defense in cyber warfare. Draw your own conclusions.

###### OpenSSL

To be fair, other than the fact that they are home-grown implementations, is that the Haskell packages do not do a lot to prevent the user from making mistakes. OpenSSL is also well known to only provide building blocks, but do nothing at all to prevent a user from making errors (instead, sometimes it seems like they go out of their way to make it *more* likely that the user makes error). It is your own responsibility to apply a padding algorithm to prevent a padding oracle attack, for example.

There is a recent attempt called [NaCl](http://nacl.cr.yp.to/)/[libsodium](http://doc.libsodium.org/) which implements cryptographic primitives in such a way that it makes it very hard for the end-user to make mistakes. There is a Haskell interface for libsodium called [saltine](https://hackage.haskell.org/package/saltine), but that library warns that it is not production ready; something that other libraries could learn something from, I think.

##### Conclusion

When using cryptographic primitives, you need to be absolutely paranoid about other people's code you rely on. Many of the arguments in this article are based on the fact that cryptography libraries rely on primitives that are either known to be flawed, or possibly flawed. In these cases, it is most wise to avoid using such a library.

The good news is that all this is fixable: the most obvious place to start fixing these problems are the libraries that generate random entropy: this is where all encryption starts. Once these are fixed, the random number libraries are a lot more safe to use, as is the RSA library.

The bad news is that the maintainers of these packages, for some reason, seem to be against these kind of fixes. I think Haskell suffers badly from a not-invented-here syndrome when it comes to cryptography and at the moment, everyone will be better off using HsOpenSSL or waiting for saltine.
