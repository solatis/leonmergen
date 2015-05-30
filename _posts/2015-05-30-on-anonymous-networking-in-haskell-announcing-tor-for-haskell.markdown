---
layout:     post
title:      "On anonymous networking in Haskell: announcing Tor and I2P for Haskell"
date:       2015-05-30 12:17:00
categories: haskell privacy
draft:      true
author:     Leon Mergen
author_gravatar: a00ff1a0178e0e6bda0108bf6d40eefc
author_facebook: lmergen
author_github: solatis
---
<img src='/images/posts/blog4.jpg' class='blogimage' title='Privacy in a world where everybody watches you' />



During the development of a Bitcoin-related project, I found the need to perform anonymous peer-to-peer communication in Haskell. When people want a solution for anonymous networking, they usually point to either <a href='https://www.torproject.org/'>Tor</a> or the lesser-known <a href='http://geti2p.net/'>I2P</a>.

I have developed a Haskell API for both projects. In this post I will outline the differences between these projects and illustrate how to use them within Haskell.

##### Tor

<img src='/images/posts/blog4a.png' style='width: 33%; margin-left: 10px; margin-bottom: 10px;' align='right' title='Tor network design' />

Tor is easily the most popular of the two projects, and many people are familiar with it. For an end-user to start using Tor is incredibly easy: just download and install the <a href='https://www.torproject.org/projects/torbrowser.html.en'>Tor Browser Bundle</a> and you're set. The browser bunde also shows Tor's major target audience: people that want to browse the web anonymously. However, recent changes in <a href='https://gitweb.torproject.org/torspec.git/tree/control-spec.txt#n1299'>Tor's Control Protocol</a> allows application developers to set up ad-hoc <a href='https://www.torproject.org/docs/hidden-services.html.en'>Hidden Services</a> using Tor, opening the way for easy anonymous peer-to-peer communication between applications.

Tor's design relies on a separation between relays and exit nodes, where exit nodes bridge the gap between the Tor network and the 'public' internet. Due to the set of exit nodes being relatively small, the major complaint against Tor is that is more vulnerable to traffic analysis attacks than I2P.

Being the more popular option of the two, Tor also gained significant interest from the parties it tries to protect you from: searching on Google <a href='http://www.spiegel.de/media/media-35543.pdf'>gives</a> <a href='http://www.spiegel.de/media/media-35538.pdf'>many</a> <a href='http://www.spiegel.de/media/media-35540.pdf'>reports</a> where <a href='http://en.wikipedia.org/wiki/Five_Eyes'>adversaries</a> research various techniques to deanonymize the Tor network. These adversaries consider Tor a major obstacle in their mission and as such spend considerable resources to deanonymize it. Without diving too much into the details, the bottom line is that it is incredibly hard and all techniques essentially boil down to various traffic analysis methods. The conclusion: <a href='http://www.spiegel.de/media/media-35543.pdf'>"The required data is not collected at present."</a>.

I will let you draw your own conclusions on what this means for the integrity of your privacy you will get with Tor.

###### Haskell and Tor

As mentioned above, since version 0.2.7.1-alpha and above, Tor's control protocol supports the ADD_ONION command: this creates a new hidden service without any manual configuration, and cleans up the hidden service address as soon as the application quits. This is ideal for creating short-lived, ad-hoc services.

I recently developed the <a href='http://hackage.haskell.org/package/network-anonymous-tor'>Network.Anonymous.Tor</a> library which makes use of this command. This library has the small(ish) goal of facilitating anonymous communication between applications and as such is far far from a complete implementation of the Tor control protocol.

Usage of the library is pretty straightforward. To set up a server:

{% highlight haskell linenos %}
main = Tor.withSession withinSession

where
  withinSession :: Socket -> IO ()
  withinSession sock = do
    onion <- Tor.accept sock 80 worker
    -- At this point, 'onion' contains the base32 representation of
    -- our hidden service, without the trailing '.onion' part, and any
    -- incoming connections will be redirected to our 'worker' function.
    --
    -- Once again, when we leave this function, all registered mappings
    -- will be lost.

  worker sock = do
    -- Now you may use sock to communicate with the remote.
    return ()
{% endhighlight %}

Connecting to a client is as simple as using Tor's SOCKS proxy, since connections are bidirectional. I provide a convenience function for you that handles the discovery of the SOCKS port and connection through SOCKS for you:

{% highlight haskell linenos %}
main = Tor.withSession withinSession

where
  constructDestination =
    SocksAddress (SocksAddrDomainName (BS8.pack "2a3b4c.onion")) 80

  withinSession :: Socket -> IO ()
  withinSession sock = do
    Tor.connect' sock constructDestination worker

  worker sock =
    -- Now you may use sock to communicate with the remote.
    return ()

{% endhighlight %}

This is a safe function that will connect to 2a3b4c.onion port 80.

###### A note about authentication

Tor's control socket requires authentication: however, using clever protocol negotiation mechanics, there is a so-called COOKIE authentication method: a file is stored in a secure location on the user's computer, and you can authenticate with the Tor server if you prove you know the contents of that file. This is currently the only authentication method supported by the Tor library, but is sufficient in most cases: a user can run the Tor Browser Bundle, which launches Tor, and the Haskell library is automatically able to detect, authenticate and use this Tor service. Yes, it is *that* trivial, which is awesome!

##### I2P

<img src='/images/posts/blog4b.png' style='width: 33%; margin-left: 10px; margin-bottom: 10px;' align='right' title='Tor network design' />

I2P clearly is the underdog of the two. Where Tor is focussing primarily on anonymous web browsing (or at least, that's what it has come to be), I2P's focus is on anonymous communication between applications. This is an important distinction: I2P has facilities for programmatically creating anonymous endpoints from the ground up, and provides many several ways for an application developer to set up communications, depending on the quality of service the application requires.

From what I can tell on a high level, the major difference in design between I2P and Tor is that I2P is more resilient against traffic analysis attacks, for two reasons:

* there is no distinction between relays and exit-nodes: in I2P, every client is running an exit-node by default, further hiding the traffic of a client and increasing the amount of exit points;
* connections in Tor as bidirectional, where connections in I2P are unidirectional: for bidirectional communication in I2P, two completely different routes will be taken for received / sent packets between two nodes (as illustrated by the green and blue lines in image on the right), making it more difficult to perform traffic analysis.

In terms of operation, I2P is not as user-friendly as Tor. Installation and configuration requires more expertise, and to set up a web browser for anonymous web browsing is more difficult than just downloading a browser bundle. However, if this is not a concern to you, I2P might definitely be the better choice for you.

###### Haskell and I2P

I2P provides multiple APIs for an application to communicate with the I2P service: the choice basically boils down to <a href='https://geti2p.net/en/docs/api/bob'>BOB</a> and <a href='https://geti2p.net/en/docs/api/samv3'>SAM</a>. For the Haskell API, I chose the SAM API, mainly because it provides functionality for automatic cleanup of any mappings and endpoints in case of an unclean shutdown of an application. To use this library, please use the <a href='http://hackage.haskell.org/package/network-anonymous-i2p'>Network.Anonymous.I2P package</a>.

Once you have this library installed, using I2P to set up anonymous connections is relatively trivial. To set up a server, this is all that's required:

{% highlight haskell linenos %}
main = withSocketsDo $ withSession defaultEndPoint VirtualStream withinSession

where
  withinSession :: Context -> IO ()
  withinSession ctx =
    serveStream ctx worker

  worker (sock, addr) = do
    -- Now you may use sock to communicate with the remote; addr contains
    -- the address of the remote we are connected to, which we might want
    -- to store to send back messages asynchronously.
    return ()

{% endhighlight %}

This will launch a server and start accepting connections. Launching a client to connect to this server is equally trivial:

{% highlight haskell linenos %}
main = withSocketsDo $ withSession defaultEndPoint VirtualStream withinSession

where
  dest :: PublicDestination
  dest = undefined -- This is for you to fill in

  withinSession :: Context -> IO ()
  withinSession ctx =
    connectStream ctx dest worker

  worker (sock, addr) = do
    -- Now you may use sock to communicate with the remote; addr contains
    -- the address of the remote we are connected to, which on our case
    -- should match dest.
    return ()

{% endhighlight %}

For more examples on using datagram connections, please refer to the package documentation or test suite.

##### Conclusion

I personally consider Haskell a great language to write privacy-concious applications in, because of its rich type system and focus on correctness. In this post I have introduced the existence of two libraries which make it almost trivial to use either Tor or I2P for anonymous communication in Haskell.

These libraries aim to be simple rather than feature complete: if you have any suggestions for improvements, do not hesitate to let me know and open a ticket on Github.
