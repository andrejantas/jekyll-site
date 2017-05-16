---
layout: post
title: "Recursion with nokogiri are friends"
permalink: recursion-with-nokogiri-are-friends
date: 2017-03-12 00:21:00 CET
comments: true
description: "Recursion with nokogiri are friends"
keywords: "recursion nokogiri ruby"
categories: ruby

tags: ruby

---

So as a quick one I found out some time ago that I need to migrate 'wiki' like
information from one system to postres DB... That basically meant for me that I
need to crawl those pages cut out paragraphs I want and move only those.

And there you can see what I found out... Now I don't say is the right solution,
but for sure quick and kinda neat for what it does :)

So first of all just a small notice, this was just one time rake task that I
removed afterward since it served no purpose anymore, but was quick to write
way to move all that data from wiki without actually hiring two people for
weekend to copy paste it... We are developers, there is nothign script cannot
do for us right?

So here I exactly as I described needed to cut out only paragraphs that are
essential for me, so half of the information there was redundant anyway.

So long story short I needed only to find my two entities between which is my
desired text. As I want to provide you more examples, so lets say we are
looking for paragraph that starts with link with text 'Hello' and ends with `hr` tag.
(note I am getting response from Net::HTTP)

{% highlight ruby %}
response = Nokogiri::HTML(response.body)
start = response.search("a:contains('Hello')").first
stop = response.search("hr").first
{% endhighlight %}

Notice, that these are presented as array to you, but in my case I am sure
there is only one of each on the page, so I can affort to just go ahead and
call `.first` on two entities beginning and ending my paragraph.

And now just a nifty method getting all between and we are good to go :)

{% highlight ruby %}
def collect_between(start, stop)
  start == stop ? [start] : [start, *collect_between(start.next_element, stop)]
end
{% endhighlight %}

And there goes recursion, where I am filling array with next and next element
untill I hit the last one. You could do with `while`, but for me seems nice to
use recursion. In my daily job I hardly ever meet it since it could be tricky and not
so easy to debug if you make a mistake. But in this case is just nicer.

Ok so here we have it. We call this bad boy with our two elements and are good
to go and since I am am from here forward in app using WYSIWYG editor I want to
store it as HTML that it was on the page and will apply similar styling to it
so I save it like string:

{% highlight ruby %}
result = collect_between(start, stop)
result.map(&:to_html).join
{% endhighlight %}

Thank you for reading and hopefully it was usefull for somebody :)
