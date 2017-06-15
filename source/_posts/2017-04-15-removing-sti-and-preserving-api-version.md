---
layout: post
title: "Removing STI and preserving API version?"
permalink: removing-sti-and-preserving-api-version
date: 2017-04-15 00:21:00 CET
comments: true
description: "Removing STI and preserving API version?"
keywords: "rails STI API"
categories: rails api sti

tags: rails api sti

---


Hi guys, so today it is kind of funny thing what I was and will be solving for a bit now at work...
So the main problem is that we have functional a helpful intranet where many things are represented and mainly there is a wee bit of overuse of STI. Which many would argue is horrible mistake, but sometimes really really makes sense...

I am not here to make a case for it because in this particular case it also made completely no sense... But here it was gradual changes to both models represented by the common table. So I reached the point where almostly only thing they had in common was the actual table there... so STI off you go right ? Well it is not as easy as that, there are many many tutorials and blog posts to help you get rid of STI even if you have it there for a while (in my case 6 years probably more...), for example these:

[FutureLearn article](https://about.futurelearn.com/blog/refactoring-rails-sti/)

[Arkency article](http://blog.arkency.com/2013/07/sti/)

They solve and show you how to solve some problems right?

But here is my case... You have roundup of older and nicely functional tools that use this apps API to get your base STI model with ID.... since it is STI there is no way IDs of one or the other model inheriting from base are going to be same right ? So this way you don't really have to have API for each model from STI, base is really enough since you are able to find it by ID.

OK but now I tore it apart, is two tables now and this way my IDs will meet and there is a problem right there, even if I create myself nice facade to search above these two models I am not able to find the right one by ID because it will result in two records each time, but my user only wants one...

So first of all what I did is preserve existing IDs, so by the time I migrate no two ids meet, that is a given from situation before and then I worked out that best would be to let these two tables share common sequence in this version of API and with the new one I will have to rewrite couple tools. But interesting thing is how really easy that was:

{% highlight sql %}
CREATE SEQUENCE common_base_id_seq START WITH 50000;
{% endhighlight %}

This way I just set up some common sequence I could use, then I just altered existing ones in my tables:

{% highlight sql %}
ALTER TABLE breads ALTER COLUMN id SET DEFAULT nextval('common_base_id_seq');
ALTER TABLE cucumbers ALTER COLUMN id SET DEFAULT nextval('common_base_id_seq');
{% endhighlight %}

What this will do for me now is that it will ensure between oth these tables ID will be unique.

I can sleep nice now and continue little bits here and there preserving my API version as it was preparing for the changes to come in future.

OK so we are all set, happy coding then and see you around.
