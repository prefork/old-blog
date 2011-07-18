---
layout: post
title: The Github Firehose
---

{{ page.title }}
================

<p class="meta">17 July 2011 - Lincoln</p>

I spent last week abroad and I'm still tired so this will be a short post...

## The Basics
The firehose is implemented as a PuSH server.  To subscribe, send a HTTP POST to http://githubub.superfeedr.com/ with the following details:

    hub.mode : subscribe
    hub.verify : sync or async
    hub.callback : <YOUR-CALLBACK-HERE>
    hub.topic : http//tesla.grapepudding.com:8080/

[Superfeedr](http://superfeedr.com) has good documentation on how to setup a PuSH client.

Once you get everything up and running, you should start receiving POST's about Github activity in realtime.

Let [me](http://twitter.com/rdnck76) know if you have questions/comments!
