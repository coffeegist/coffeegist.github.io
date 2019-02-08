---
title: Domain Fronting, Beacons, and TLS!
category: Security
tags: [red-team, pentest, hacking, security, cobaltstrike]
---


These posts have been too few and far in-between lately. But today I came across something that may save some poor red teamer a little bit of troubleshooting time. With my wife recently taking a trip out of town, I decided I had a few minutes this evening to write it up. What exciting times we live in.


## Setting Up

During some recent research, I found myself going through the process of setting up teamservers for domain fronting. If you don't know what domain fronting is, please go check out Raffi's blog post here: <https://blog.cobaltstrike.com/2017/02/06/high-reputation-redirectors-and-domain-fronting/>.

Once I got everything set up with a CDN, the first step is obviously to test and make sure traffic flow is happening as expected. So, I whip out `curl` and start making requests to the reputable domain, with our CDN address in the host header. CobaltStrike is seeing the traffic hit the web logs, and I'm thinking I'm almost finished.


## Road Blocks

I move over to a Windows box to fire up a test beacon, the stager launches, and nothing. I wait a few minutes, and still get nothing. I regenerate a beacon, and try again. It's still not working. Time to fire up Wireshark.

I see the download cradle successfully launches and pulls down the stager. The stager fires, does a DNS query for the fronted domain, and initiates a connection with a CDN caching server. Then the packets start to look interesting.


## TLS to the Failure!

{% include figure image_path="/assets/images/2019-02-07-domain-fronting-beacons-and-tls/wireshark-failed-handshakes.png" %}

Three failed TLS/SSL handshakes, and then it's over. But why are the handshakes failing? Let's look a little closer.

{% include figure image_path="/assets/images/2019-02-07-domain-fronting-beacons-and-tls/wireshark-failure-forty.png" %}

So, from this we can see that we're getting a handshake failure (represented by 0x28 or a decimal value of 40). I surfed around trying to find a definitive answer to what this meant. Someone suggested it was a Cipher Mismatch. Let's investigate that.


### Cipher Suites

We can see the list of cipher suites offered in the `Client Hello` message from beacon in the image below. Things to note are the TLS version being used (TLSv1.0), and the list of 12 cipher suites.

{% include figure image_path="/assets/images/2019-02-07-domain-fronting-beacons-and-tls/supported-ciphers-beacon.png" %}


#### TLSv1.2

For domain 1, we'll check to see if the allowed cipher suites line up with the above cipher suites. To do this, let's use [SSL Labs](https://www.ssllabs.com/ssltest/) to check and see what versions of TLS and Cipher Suites are accepted.

{% include figure image_path="/assets/images/2019-02-07-domain-fronting-beacons-and-tls/supported-ciphers-domain1.png" %}

We see here that only TLSv1.2 is accepted, and none of the cipher suites match. So, it makes sense that the handshake would fail. Beacon goes on to offer the same TLS protocol and cipher suites again, and then offers SSLv3, which we know won't do the trick. Let's find a domain accepting TLSv1.1.


#### TLSv1.1

Again using SSL Labs, we check domain 2 to see if we can find some cipher suite overlap.

{% include figure image_path="/assets/images/2019-02-07-domain-fronting-beacons-and-tls/supported-ciphers-domain2.png" %}

Here we see some overlap on ciphers! Beacon offers `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`, (or `0xc013`), and we see that `0xc013` is listed in the accepted cipher suites for our target domain. So, I assumed beacon might work with this setup. However, upon testing we get the same results: three failed handshakes.


### TLS Protocol Version

The last thing I knew to try was to find a frontable domain that accepts TLSv1.0. Below is the SSL Labs report, for completeness :)

{% include figure image_path="/assets/images/2019-02-07-domain-fronting-beacons-and-tls/supported-ciphers-domain3.png" %}

Here we see that TLSv1.0 is supported, and there is plenty of overlap on cipher suites. I fire up a beacon, and see that the TLS handshake works like a charm.


## Coffee Break!

By this point, I'm convinced that beacon will not be able to connect to sites that do not accept <=TLSv1.0. If this is the case, this will impact fronting due to organizations improving their domain security practices and cutting off older versions of the TLS protocol. This is an easy workaround for beacon, but it could potentially be tough to implement without breaking compatibility with older systems. It would be nice to see this as a configurable parameter one day (Malleable C2 perhaps?!), but until then, happy hacking!


#### Bonus!

For quick methods of checking TLSv1.0, TLSv1.1, and TLSv1.2 support for a site, you can use the following commands:

```
➜  ~ curl --tlsv1.0 --verbose --header 'Host: [your-cdn-caching-server]' 'https://[reputable-domain]/'
➜  ~ curl --tlsv1.1 --verbose --header 'Host: [your-cdn-caching-server]' 'https://[reputable-domain]/'
➜  ~ curl --tlsv1.2 --verbose --header 'Host: [your-cdn-caching-server]' 'https://[reputable-domain]/'
```
