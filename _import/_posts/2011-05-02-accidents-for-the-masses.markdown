--- 
layout: post
title: Accidents for the masses!
wordpress_id: 3
wordpress_url: http://blog.stdio.me/?p=3
date: 2011-05-02 18:38:17 -05:00
---
<em>The talk I should have given at Hacks/Hackers  last week.</em>

I gave a high-level talk on web scraping for <a title="Hacks/Hackers Heartland" href="http://twitter.com/HacksHackersHLD" target="_blank">Hacks/Hackers Heartland</a> last week.   There was nothing explicitly technical about the subject matter, its web scraping.  You pretend to be a web browser, retrieve some data, and you filter out the junk you don't want.  It ought to be simple to explain, but I botched it.  So, writing this post is a way for me to think through, step by step, how one goes about borrowing some data -- start to finish.
<h3>Find the source:</h3>
We're going to target the Lincoln Police Department's site listing accidents on a day to day basis.  It looks something like this:

<em><a href="http://www.flickr.com/photos/34149492@N07/5684924527/"><img class="aligncenter" title="LPD Home Page" src="http://farm6.static.flickr.com/5067/5684924527_6cb75bac8d_z.jpg" alt="Accident Reporting Page" width="640" height="495" /></a>
</em>

There's only one form on the page, so that makes navigation easier.

Two options are available for searching the database:
<ol>
	<li>By Case Number</li>
	<li>By Date</li>
</ol>
Since we're doing this in bulk, going by date would probably be more reliable.  The thing to figure out is the earliest date that we can get access to data.  There isn't any good way to figure this out except experimenting.  I started at Jan 1, 2005 and worked my way back 1 year at a time.  After messing with it, we know data became available on Jan 1, 2000 (pretty early all things considered.).  So all we have to do get the complete dataset is to start at Jan 1, 2000 and request every day until we get to the present.  That's not so bad, except that it would take a while by hand, and it would be boring.  That's why its an excellent job for a computer; computers are boring <em>and</em> fast.

Let's figure out what sort of information it sends to the server.  Google Chrome's developer tools work well for this.  (From Chrome) Right click on the page and pick "Inspect Element".  By default it shows the HTML markup for the page.  While we could wade through it and find the markup for the form, we really don't have to.  The good stuff is under the network tab, so click over there.  Don't worry, its empty by default.

With the network tab open, type 1-1-2000 into the form and click "submit query". Your browser probably looks something like this now:

<a href="http://www.flickr.com/photos/34149492@N07/5685003217"><img class="aligncenter" title="LPD HTTP Requests" src="http://farm6.static.flickr.com/5066/5685003217_1bca16088a_z.jpg" alt="LPD HTTP Requests" width="640" height="495" /></a>

The network tab now shows all the requests your browser made to load the page (darn nice of it eh?).  This is every image, style sheet, and script.  All we care about is what we sent to the server, so scroll up to the top and click on the row called "ACC.COM".  More HTML!  Now click on the little tab called "Headers".  Now you should see something like this:

<a href="http://www.flickr.com/photos/34149492@N07/5685589776"><img class="aligncenter" title="LPD HTTP POST" src="http://farm6.static.flickr.com/5225/5685589776_b4aef1477c_z.jpg" alt="LPD HTTP POST" width="640" height="495" /></a>

Most of that information we still don't care about.  Scraping is kind of wasteful that way.  I've done some beautiful work with paint to show you what really matters.

<a href="http://www.flickr.com/photos/34149492@N07/5685590006"><img class="aligncenter" title="LPD HTTP POST mod_paint" src="http://farm6.static.flickr.com/5302/5685590006_6312d64e65_z.jpg" alt="LPD HTTP POST mod_paint" width="640" height="495" /></a>

That's right, all that work for three lines.  The first tells us how we requested information from the server (called the method), using an HTTP POST.  The other two lines tell us what kind of form information we sent to the server.  The first field, called "rky" is empty.  I have no idea what it's used for, but its required.  The second field is "date", and its the same date we typed in (magic!).  Now we have everything we need to write some code.
<h3>Write some code</h3>
Some sort of disclaimer is probably in order here.  Maybe a subset of these:
<ul>
	<li><em>Your mileage may vary</em></li>
	<li><em>Continue at your own risk</em></li>
	<li><em>This is probably not the best way, but its the way I picked</em></li>
</ul>
Ruby has a couple nice tools to scrape websites, but most of them are overkill for this project.  Lets take a look at the tools we need.
<ul>
	<li>Rest-client - a ruby gem that can fake the HTTP POST request from earlier. (<a href="https://github.com/archiloque/rest-client">Github</a>)</li>
	<li>Nokogiri - a ruby gem for making the HTML returned by the server more useful (<a href="http://nokogiri.org/">Homepage</a>)</li>
</ul>
I'm going to assume you know how to get Ruby and Rubygems up and running on your platform.  We need to install a couple dependencies.  Open up terminal, shell, cygwin or wherever you type magic words and do the following.
<pre class="prettyprint">gem install nokogiri
gem install rest-client</pre>
Now, we want to play a bit.  So lets open up irb:
<pre class="prettyprint">irb</pre>
Now you should see the ruby shell.  We need to tell ruby to load the tools we just installed, so enter these three lines:
<pre class="prettyprint">gem install rest-client
require 'rubygems'
require 'nokogiri'
require 'rest-client'</pre>
Now we're ready to do fun stuff!  Lets tell ruby to submit the same form from earlier and save the response in a variable called "page":
<pre class="prettyprint">page = RestClient.post 'http://cjis.lincoln.ne.gov/HTBIN/ACC.COM', :date =&gt; "1-1-2000", :rky =&gt; ""</pre>
You'll probably get a whole bunch of HTML back in your console, Ruby is telling you the results of the command you just gave it.  So now, if we were able to see inside the variable "page", it would be full of HTML.  Too bad people don't understand HTML very well.  So now we need to use another tool called Nokogiri to interpret the page we got back.
<pre class="prettyprint">doc = Nokogiri::HTML(page.to_str)</pre>
Now, all the information stored in the HTML has been parsed (interpreted) by Nokogiri and saved in a new variable called "doc".  Now let's go through and find every row in the table on the results page:
<pre class="prettyprint">rows = doc.xpath('//table[1]//tr')</pre>
But we've still got HTML!  These next few lines will tell Ruby to clean up the table and convert it to CSV (a format readable by Microsoft Excel and friends.
<pre class="prettyprint">rows.each do |row|
      tempStr=""
      row.xpath('td').each do |cell|
         tempStr += '"' + cell.text.gsub("\n", '').gsub('"', '\"').gsub(/(\s){2,}/m, '').gsub('?','').gsub('??','').strip + "\","
      end
      tempStr += "\n"
      if tempStr.include? '-'
        unless tempStr.include? 'View the accident report by \"clicking\" on the BLUE case number at the left of the line.'
           puts tempStr
        end
      end
   end</pre>
Cool! We just wrote a bunch of data to the screen, if we wanted to save all of that to a file instead, all we'd have to do is add a line to the top that said:
<pre class="prettyprint">aFile = File.new("lpd.csv", "w+")</pre>
And change that single line from:
<pre class="prettyprint">puts tempStr</pre>
to:
<pre class="prettyprint">aFile.syswrite(tempStr)</pre>
<h3>Wrapping up</h3>
To wrap things up, lets automate the process.  I've made a script available here which captures all the data from the accident reports and saves it as CSV.  All that I did to change our script above to run for a whole bunch of days instead of just one.  You can check it out below:
<pre class="prettyprint">require 'rubygems'
require 'date'
require 'nokogiri'
require 'rest-client'

startDate = Date::civil(2000, 1, 1) # When LPD started making data available
endDate   = Date.today              # The current system date

aFile = File.new("lpd.csv", "w+")

while startDate &lt;= endDate    # scrape some    target = startDate.month.to_s + "-" + startDate.day.to_s + "-" + startDate.year.to_s # is there a strfdate?    puts 'Fetching: ' + target    response = RestClient.post 'http://cjis.lincoln.ne.gov/HTBIN/ACC.COM', :date =&gt; target, :rky =&gt; ''
   doc = Nokogiri::HTML(response.to_str)

   doc.xpath('//table[1]//tr').each do |row|
      tempStr=""
      row.xpath('td').each do |cell|
         tempStr += '"' + cell.text.gsub("\n", '').gsub('"', '\"').gsub(/(\s){2,}/m, '').gsub('?','').gsub('??','').strip + "\","
      end
      tempStr += "\n"
      if tempStr.include? '-'
        unless tempStr.include? 'View the accident report by \"clicking\" on the BLUE case number at the left of the line.'
           aFile.syswrite(tempStr)
        end
      end
   end
   startDate = startDate + 1
end</pre>
For the not so ambitious, the resulting CSV file is <a title="LPD Accident Reports" href="http://dl.stdio.me/hhh/lpd/lpd.csv" target="_blank">available for download</a>.

Something interesting worth noting is that by just scraping the results table, we don't get that much useful information.  If we wanted detailed information on charges, tickets issued, etc. we need to go one step further.  Once we know the Case Number, we can request more details.  A sample implementation of that is below:
<pre class="prettyprint">def sleepyPD(ticketID)
   require 'rubygems'
   require 'nokogiri'
   require 'rest-client'
   nonascii = /[\x80-\xff]/ 

   # scrape some
   response = RestClient.post 'http://cjis.lincoln.ne.gov/HTBIN/ACC.COM', :tick =&gt; ticketID
   doc = Nokogiri::HTML(response.to_str)
   resp = doc.xpath('//table//tr[td[@style]]')
   i=0
   respclean = Array.new
   resp.each do |row|
     respclean[i] = row.to_s.gsub(/&lt;.*?&gt;/, "").gsub(nonascii, "").gsub("\n",",") #.each do |row|
     i = i + 1
   end
   jsonObj = '{'

   jsonObj += '"ticketNum":"'+respclean[0].split(',').at(1).gsub(" ",'')+'",'
   jsonObj += '"offenseDate":"'+respclean[0].split(',').at(2).gsub(" ",'')+'",'
   jsonObj += '"docketNum":"'+respclean[0].split(',').at(3).gsub(" ",'')+'",'
  # puts jsonObj
   jsonObj += '"charges":['
   # each
   respSize = respclean.size-5
   k=0
   while k &lt; respSize       if k&gt;2
         jsonObj += ','
      end
      jsonObj += '{"citedFor":"'+respclean[k+1].split(',').at(4).gsub("  ",'').strip+'",'
      jsonObj += '"chargedWith":"'+respclean[k+2].split(',').at(3).gsub("  ",'').strip+'",'
      jsonObj += '"amendedTo":"'+respclean[k+3].split(',').at(3).gsub("  ",'').strip+'",'
      jsonObj += '"disposition":"'+respclean[k+4].gsub("DISPOSITION: ",'').gsub("  ",'').strip+'"}'
      k = k+5
   end
   jsonObj += ']}'
   return jsonObj
end</pre>
It's a ruby function so if you include this into your project, you can get details on accidents in web-friendly JSON just by calling something like this:

sampleJson = sleepyPD("A0-000087")

It would take a really, really long time to scrape all the tickets.  But if you were going to do something like that, here's a script that reads Case Numbers out of lpd.csv and creates a massive JSON array.
<pre class="prettyprint">require 'rubygems'
require 'fastercsv'
require 'ticket'
doc = FasterCSV.read("lpd.csv")
aFile = File.new("master.json", "w+")
lFile = File.new("master.log", "w+")
aFile.syswrite('[')
doc.each{|row|
  unless(row.at(4).strip == "NONE")
     lFile.syswrite("Fetching: "+row.at(0).strip+"\n")
     begin
       temp = sleepyPD(row.at(0).strip)
       unless row == doc.last
         temp += ","
       end
       aFile.syswrite(temp)
     rescue
       lFile.syswrite(puts "Failed on: "+row.at(0).strip+"\n")
     end
  end
}
aFile.syswrite(']')</pre>
All of this source code is available <a title="LPD Scraper" href="https://github.com/rdnck76/lpd-scraper" target="_blank">on Github</a>.  Happy scraping!

&nbsp;

Questions/Comments?

&nbsp;
