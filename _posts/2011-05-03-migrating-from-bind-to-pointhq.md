---
layout: post
title: Migrating from BIND to PointHQ
---

{{ page.title }}
================

<p class="meta">03 May 2011 - Lincoln</p>

The last few weeks, I migrated a bunch of my DNS to PointHQ.  Their support is quick, the website is pretty, and they have an API (which makes me happy.).  But they don't support* importing BIND zone files, which is what pretty much every other site exports.  So I wrote my own little tool in ruby.<!--more-->

<ol>

	<li>Install the dependencies like so:

<pre class="prettyprint">gem install point zonefile</pre>

</li>

	<li>Download or clone the script from <a title="PointHQ Importer" href="https://gist.github.com/954279" target="_blank">Github</a>.</li>

	<li>Update the configuration stuff at the top. (the zoneID is the number you see in the URL when you view that domain in Point.)</li>

	<li>Run it! <br /><pre class="prettyprint">ruby dns.rb</pre></li>

</ol>

This tool DOES NOT support TXT records that don't have their value wrapped in quotes. &nbsp;It's a known bug in the gem. &nbsp;I could write my own parser pretty easy (or patch zonefile) but I don't usually have that many TXT records so I don't care that much.



<em>*According to their staff, support is on the way.</em>