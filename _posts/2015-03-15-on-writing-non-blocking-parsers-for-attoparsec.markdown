---
layout: post
title:  "On writing non-blocking network parsers for attoparsec"
date:   2015-03-15 14:56:02
categories: haskell attoparsec
author: Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github: solatis
---
The goal of Haskell's [Attoparsec](https://hackage.haskell.org/package/attoparsec) has always been clear: where Parsec provides you with a friendly parser combinator that produces nice error messages, Attoparsec does everything that is possible to give you the best speed possible. Error messages can be cryptic at best: `not enough bytes` is a common error when you would actually expect a real parse error.

However, when you are able to live with that, Attoparsec does bring real advantages: personally I do not care about raw parser performance *that* much, but this goal for best performance has had another more interesting effect: it supports incremental parsing.

##### Incremental parsing
Incremental parsing is ideal for network related programming, where data comes in and you do not know how much data you need to read from the socket beforehand, do not wish to do your own buffering, etcetera: the parser's state will be the buffer itself! So, this is why I developed a little helper library to help you with this, [Network.Attoparsec](http://hackage.haskell.org/package/network-attoparsec). It consists of two main interfaces: an incremental one and a singular one.

When you use this library for a little while, and write certain kind of parsers, you might suddenly find yourself in a nasty situation:

##### Why is my parser blocking?!

After developing the library, a situation I suddenly felt myself in was that my parser seemed to be expecting more input, while the parser should clearly have been in a completed state. Let's illustrate the problem by writing a small parser ourselves:

{% highlight haskell linenos %}
main :: IO ()
main = do
  let parser = Atto8.string "wombat" <|> Atto8.string "wombat"
      res    = Atto.parse parser "foo"

  putStrLn (show res)
{% endhighlight %}

Obviously, when you send the parser the input `foo` it should complete, or when you send it `bar` it should fail, right? Then why is attoparsec blocking? Let's look into the implementation of `string`:

{% highlight haskell linenos %}
string :: ByteString -> Parser ByteString
string s = takeWith (B.length s) (==s)
{% endhighlight %}

Wait a minute! Doesn't that mean... *yes*. If it is not obvious to you yet, let's rewrite the first parser to more accurately describe what is happening:

{% highlight haskell linenos %}
takeWith 6 (== "wombat") <|> takeWith 3 (== "foo")
{% endhighlight %}

By now the problem should be obvious to you: the parser is blocking because the first option, `wombat` requests 6 bytes of data from `takeWith`, even though it *should* know for sure that this branch can never match! This is probably done for performance reasons, and is not a problem in a non-incremental world, but since I live in an iterative world, this *is* a problem for me. So how can we fix this?

The obvious but error-prone solution is to reorder the parsers:

{% highlight haskell linenos %}
string "foo" <|> string "wombat"
{% endhighlight %}

This will ensure that the branch with the least required characters will be evaluated first, and as such, for valid input, will always work properly. However, it still does not work for invalid input: what if some evil enemy that is trying to perform a Denial-of-Service attack on your elegant Haskell parser only sends a malicious string, otherwise known as `z`? Once again the parser is in a blocking state, even though no branches could match. Could we fix this by writing a more secure/strict version of the parser that fails as soon as one character doesn't match anymore ?

The answer is, why yes, of course we can:

{% highlight haskell linenos %}
safeString :: Parser ByteString
safeString = mapM (Atto.satisfy . (==)) . unpack
{% endhighlight %}

As you can see here, the solution is threefold:

1. Unpack the input string as an array of characters: this is done with the `unpack` statement;
2. Create a parser for each of these characters: this is done in the `Atto.satisfy` statement, creating for example three different parsers: `[Atto.satisfy (==) 'f', Atto.satisfy (==) 'o', Atto.satisfy (==) 'o']`
3. Transform all those parsers `[Parser Char8]` into a single `Parser [Char8]` using `mapM` which can be applied on the input.

Since StackOverflow suggests I am not the only person running into this problem, I highly encourage the attoparsec team to adopt this implementation of the string parser as an alternative, since it will safe many headaches and provides more predictable results.
