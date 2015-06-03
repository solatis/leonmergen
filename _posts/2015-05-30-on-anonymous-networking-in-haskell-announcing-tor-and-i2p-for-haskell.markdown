---
layout:     post
title:      "On anonymous networking in Haskell: announcing Tor and I2P for Haskell"
date:       2015-05-30 12:17:00
categories: haskell privacy
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

Tor is easily the most popular of the two projects, and many people are familiar with it. For an end-user to start using Tor is incredibly easy: just download and install the <a href='https://www.torproject.org/projects/torbrowser.html.en'>Tor Browser Bundle</a> and you're set. The browser bundle also shows Tor's major target audience: people that want to browse the web anonymously. However, recent changes in <a href='https://gitweb.torproject.org/torspec.git/tree/control-spec.txt#n1299'>Tor's Control Protocol</a> allows application developers to set up ad-hoc <a href='https://www.torproject.org/docs/hidden-services.html.en'>Hidden Services</a> using Tor, opening the way for easy anonymous peer-to-peer communication between applications.

Tor's design relies on a separation between relays and exit nodes, where exit nodes bridge the gap between the Tor network and the 'public' internet. Due to the set of exit nodes being relatively small, the major complaint against Tor is that it's more vulnerable to traffic analysis attacks than I2P.

Being the more popular option of the two, Tor also gained significant interest from the parties it tries to protect you from: searching on Google <a href='http://www.spiegel.de/media/media-35543.pdf'>gives</a> <a href='http://www.spiegel.de/media/media-35538.pdf'>many</a> <a href='http://www.spiegel.de/media/media-35540.pdf'>reports</a> where <a href='http://en.wikipedia.org/wiki/Five_Eyes'>adversaries</a> research various techniques to deanonymize the Tor network. These adversaries consider Tor a major obstacle in their mission and as such spend considerable resources to deanonymize it. Without diving too much into the details, the bottom line is that it is incredibly hard and all techniques essentially boil down to various traffic analysis methods. The conclusion: <a href='http://www.spiegel.de/media/media-35543.pdf'>"The required data is not collected at present."</a>.

I will let you draw your own conclusions on what this means for the integrity of your privacy you will get with Tor.

###### A Tor relay in Haskell

As mentioned above, since version _0.2.7.1-alpha and above_, Tor's control protocol supports the <a href='https://gitweb.torproject.org/torspec.git/tree/control-spec.txt#n1299'>ADD_ONION</a> command: this creates a new hidden service without any manual configuration, and cleans up the hidden service address as soon as the application quits. This is ideal for creating short-lived, ad-hoc services.

I recently developed the <a href='http://hackage.haskell.org/package/network-anonymous-tor'>Network.Anonymous.Tor</a> library which makes use of this command. This library has the small(ish) goal of facilitating anonymous communication between applications and as such is far far from a complete implementation of the Tor control protocol.

As a proof-of-concept, I figured that writing a service that relays hidden service connections to a local server might be a good use case. We could actually do all this with a single command, since Tor supports relaying connections natively:

{% highlight haskell %}
onion <- Tor.mapOnion controlSock 8000 80
{% endhighlight %}

.. but that would not be a really demonstration. So lo' and behold, we are going to write a trivial relaying service for Tor.

Our main function is fairly simple: get our configuration data and start
a new Tor Session. As soon as this session completes, exit the program.

{% highlight haskell %}
main = withSocketsDo $ do
  torPort <- whichControlPort
  portmap <- getPortmap
{% endhighlight %}

Once we got the configuration data, launch a Tor session and will continue
execution in the 'withinSession' function.

{% highlight haskell %}
  Tor.withSession torPort (withinSession portmap)

  where
{% endhighlight %}

We need a simple function to detect which control port Tor listens at. By
default the Tor service uses port 9051, but the Tor Browser Bundle uses 9151,
to allow Tor and the TBB to run next to each other. This function is relatively
unsafe, since it assumes at least one Tor service is running (otherwise 'head'
will return an error).

{% highlight haskell %}
whichControlPort = do
  ports <- Tor.detectPort [9051, 9151]
  return (head ports)
{% endhighlight %}

Some boilerplate code: we need to set up a port mapping, and rather than hard-
coding it, we allow the user to provide is as command line arguments. The first
argument is the public port we will listen at, the second argument the private
port we relay connections to.

{% highlight haskell %}
getPortmap :: IO (Integer, Integer)
getPortmap = do
  [pub, priv] <- getArgs
  return (read pub, read priv)
{% endhighlight %}

Once a Tor session has been created and we are authenticated with the Tor
control service, let's set up a new onion service which redirects incoming
connections to the 'newConnection' function.

{% highlight haskell %}
withinSession (publicPort, privatePort) controlSock = do
  onion <- Tor.accept controlSock publicPort (newConnection privatePort)
  putStrLn ("hidden service descriptor: " ++ show onion)
{% endhighlight %}

If we would leave this function at this point, our connection with the Tor
control service would be lost, which would cause Tor to clean up any mappings
and hidden services we have registered.

Since this is just an example, we will now wait for 5 minutes and then exit.

{% highlight haskell %}
threadDelay 300000000
{% endhighlight %}

This function is called for all incoming connections. All it needs to do is
establish a connection with the local service and relay the connections to the
local, private server.

{% highlight haskell %}
newConnection privatePort sPublic =
  NST.connect "127.0.0.1" (show privatePort) $ \(sPrivate, addr) -> spliceSockets sPublic sPrivate
{% endhighlight %}

And to demonstrate that the sockets we deal with are just regular, normal
network sockets, we implement a function using the `splice` package that
creates a bidirectional pipe between the public and private sockets.

{% highlight haskell %}
spliceSockets sLhs sRhs = do
  hLhs <- socketToHandle sLhs ReadWriteMode
  hRhs <- socketToHandle sRhs ReadWriteMode
  _ <- forkIO $ Splice.splice 1024 (sLhs, Just hLhs) (sRhs, Just hRhs)
  Splice.splice 1024 (sRhs, Just hRhs) (sLhs, Just hLhs)
{% endhighlight %}

And that's all there's to it. To test out the service, let's fire up a simple python webserver:

{% highlight sh %}
python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
{% endhighlight %}

And let's launch our relay

{% highlight sh %}
./dist/build/tor-relay/tor-relay.exe 80 8000
hidden service descriptor: Base32String "6MQF77PTMIV3BL6S"
{% endhighlight %}

This means our webserver will be available at <a href='http://6mqf77ptmiv3bl6s.onion/'>http://6mqf77ptmiv3bl6s.onion/</a>. When we connect to this address using our Tor Browser, we see a webpage and our python server servicing requests:

{% highlight sh %}
127.0.0.1 - - [01/Jun/2015 10:15:07] "GET / HTTP/1.1" 200 -
{% endhighlight %}

Victory! We have just built ourselves a (very unreliable) Tor relaying service in Haskell!

You can find the complete example in <a href='https://github.com/solatis/haskell-network-anonymous-tor/tree/master/examples'>Github repository</a>.

###### A note about authentication

Tor's control socket requires authentication: however, using clever protocol negotiation mechanics, there is a so-called COOKIE authentication method: a file is stored in a secure location on the user's computer, and you can authenticate with the Tor server if you prove you know the contents of that file. This is currently the only authentication method supported by the Tor library, but is sufficient in most cases: a user can run the Tor Browser Bundle, which launches Tor, and the Haskell library is automatically able to detect, authenticate and use this Tor service. Yes, it is *that* trivial, which is awesome!

##### I2P

<img src='/images/posts/blog4b.png' style='width: 33%; margin-left: 10px; margin-bottom: 10px;' align='right' title='Tor network design' />

I2P clearly is the underdog of the two. Where Tor is focussing primarily on anonymous web browsing (or at least, that's what it has come to be), I2P's focus is on anonymous communication between applications. This is an important distinction: I2P has facilities for programmatically creating anonymous endpoints from the ground up, and provides several ways for an application developer to set up communications, depending on the quality of service the application requires.

From what I can tell on a high level, the major difference in design between I2P and Tor is that I2P is more resilient against traffic analysis attacks, for two reasons:

* there is no distinction between relays and exit-nodes: in I2P, every client is running an exit-node by default, further hiding the traffic of a client and increasing the amount of exit points;
* connections in Tor are bidirectional, where connections in I2P are unidirectional: for bidirectional communication in I2P, two completely different routes will be taken for received / sent packets between two nodes (as illustrated by the green and blue lines in image on the right), making it more difficult to perform traffic analysis.

In terms of operation, I2P is not as user-friendly as Tor. Installation and configuration requires more expertise, and to set up a web browser for anonymous web browsing is more difficult than just downloading a browser bundle. However, if this is not a concern to you, I2P might definitely be the better choice for you.

###### Haskell and I2P

I2P provides multiple APIs for an application to communicate with the I2P service: the choice basically boils down to <a href='https://geti2p.net/en/docs/api/bob'>BOB</a> and <a href='https://geti2p.net/en/docs/api/samv3'>SAM</a>. For the Haskell API, I chose the SAM API, mainly because it provides functionality for automatic cleanup of any mappings and endpoints in case of an unclean shutdown of an application. To use this library, please use the <a href='http://hackage.haskell.org/package/network-anonymous-i2p'>Network.Anonymous.I2P package</a>.

Once you have this library installed, using I2P to set up anonymous connections is relatively trivial. Once again, we will use a relay as example. You will notice that the code is very similar to the Tor code above, probably because, well, both libraries have been designed by me.

Our main function is fairly simple: get our configuration data and start a new I2P Session. As soon as this session completes, exit the program.

{% highlight haskell %}
main = withSocketsDo $ do
  port <- getPort
{% endhighlight %}

Once we got the configuration data, launch a Tor session and will continue execution in the 'withinSession' function.

{% highlight haskell %}
I2P.withSession I2P.defaultEndPoint S.VirtualStream (withinSession port)
where
{% endhighlight %}

Some boilerplate code: we need to set up a port mapping, and rather than hard-coding it, we allow the user to provide it as command line arguments. We will accept a single argument, which is the private service we want to redirect connections to (I2P doesn't support ports, only endpoints).

{% highlight haskell %}
getPort :: IO Integer
getPort =
  fmap (read . head) getArgs
{% endhighlight %}

Once an I2P session has been created and we are connected with the I2P SAM
bridge, let's set up a new endpoint which redirects incoming connections to the
'newConnection' function.

{% highlight haskell %}
withinSession port ctx = do
  I2P.serveStream ctx (newConnection port)
{% endhighlight %}

If we would leave this function at this point, our connection with the I2P SAM bridge would be lost, which would cause I2P to clean up any mappings and endpoints we have registered.

Since this is just an example, we will now wait for 5 minutes and then exit.

{% highlight haskell %}
threadDelay 300000000
{% endhighlight %}

This function is called for all incoming connections. All it needs to do is establish a connection with the local service and relay the connections to the local, private server.

{% highlight haskell %}
newConnection port (sPublic, _) =
NST.connect "127.0.0.1" (show port) $ \(sPrivate, addr) -> spliceSockets sPublic sPrivate
{% endhighlight %}

The `spliceSockets` function is exactly the same as defined in the Tor example, and as such I will omit it.

Once again, this is it. To test out the service, let's fire up the Python webserver again.

{% highlight sh %}
python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
{% endhighlight %}

And let's launch our relay

{% highlight sh %}
./dist/build/i2p-relay/i2p-relay.exe 8000
Now serving connections at: PublicDestination "GxZvvhZjulGE3E7lWsTHhJN44N32wyYXNqQBrr01Jm~Dw~zaVK7-xXw~k1co-HTcfTwpWRHHEd-bh9OagFYDIPQFGltAZ9FQ~b5bGwoULXtFtad6koFn57HZIl7tNuO7gDeJJdgGWzVjA0JHN1-hD5HVV-lkwPzEhMFrufHSsmIUD9CUSXuyi-uJK65DX7GwfDWo0bXpdtfsggvxhLUHrlpPib2M3tlGoMoMGzH3OQM2v6j9BqF8BFSvVfLsE2thiRLB1C-bw-hkH9fSrRVpZKBH~QdoypHAplEbLouPiHO6-V7jbhNjXMIDCCxtTkaGVSsB~uL26MiyPjpwzIIhri6wEUyAOFGw~PtQs1N8LX096AXyZF1cbualxGhFyAY3gW6rN6AOWg5IWuYlJwrQ32tYi40R9YdoQhkQ9bw4rqyDZUz6VRAQz7ZMoYKcwA68HOSWoViZ7yBT0sFGTQ0r3oycE-lxOVVzcWByRhCq2fDBVg36wWp-bvtuknsmgoA9AAAA"
{% endhighlight %}

You can see that, by design, I2P endpoints are a lot larger than Onion hidden services. This is because Tor uses a central directly lookup service, whereas in I2P you work with public and private keys only; in this case, the entire public key is packed into the destination.

If we connect to that address using a browser, we get:

{% highlight sh %}
127.0.0.1 - - [01/Jun/2015 08:31:05] "GET / HTTP/1.1" 200 -
{% endhighlight %}

Once again, we have proven how simple it can be to setup a relay, but you can also see how the approaches in both Tor and I2P differ.

You can find the complete example in <a href='https://github.com/solatis/haskell-network-anonymous-i2p/tree/master/examples'>Github repository</a>.


##### Conclusion

I personally consider Haskell a great language to write privacy-conscious applications in, because of its rich type system and focus on correctness. In this post I have introduced the existence of two libraries which make it almost trivial to use either Tor or I2P for anonymous communication in Haskell.

These libraries aim to be simple rather than feature complete: if you have any suggestions for improvements, do not hesitate to let me know and open a ticket on Github.
