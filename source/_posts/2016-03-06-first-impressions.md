---
layout: post
title: "First impressions"
permalink: first-impressions
date: 2016-03-06 21:51:00 CET
comments: true
description: "First impressions"
keywords: "jekyll impressions github"
categories: jekyll github

tags: jekyll github

---


Hey guys, so I am finally moving over from really nice, but at the end quiet painful experience with [publify](http://www.publify.co).

I mean don't get me wrong, but I found out all my friends using [jekyll](http://jekyllrb.com) and I simply had to check it out for myself.

I am always using pretty widely [DigitalOcean](https://www.digitalocean.com) as host for my projects, but this time I also wanted to checkout [Github Pages](https://pages.github.com) and what they provide for free.

So far I have to say for free I never had better experience.

My setup was to find out from basic tutorial on jekyll pages, what it is and so on and then I just went ahead over to [Jekyll Themes](http://jekyllthemes.org) and there I picked this really cute one I quiet like and will probably use for longer time now. Can be found over [here](http://jekyllthemes.org/themes/end2end/).

Made me think for a bit thou. How the theme is set up and prepared for you, it is not a setup you want to put into your USERNAME.github.io repository and I will explain why.

I always really like to store my changes in git and this time if you did all from the readme, you found yourself in a place when every `rake publish` is doing `git push origin master --force` which is not really something you are going to appreciate.

So instead I placed repository with sources elsewhere and filled out info so it force pushes real repository USERNAME.github.io instead. Win Win.

Well not completely at this stage, because I happen to own domain with my surname in it, so what else should be there than this page right?

There you go, easily follow tutorial Github nice provides [here](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider) but as you see I cannot create alias in my DNS admin, so I point two IPs to root which is not a real problem (well lasted hour to take effect...).

Then you go and create that `CNAME` file they tell you to and you find out that with next `rake publish` the `CNAME` file is not anymore there... Well of course is not, you force pushed it there right?

So naturally you go have a look in `Rakefile` provided by the theme to see where the problem may be...

and you see:

{% highlight ruby %}
desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    cp_r "_site/.", tmp

    pwd = Dir.pwd
    Dir.chdir tmp

    system "git init"
    system "git checkout --orphan #{GITHUB_REPO_BRANCH}"
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -am #{message.inspect}"
    system "git remote add origin git@github.com:#{GITHUB_REPONAME}.git"
    system "git push origin #{GITHUB_REPO_BRANCH} --force"

    Dir.chdir pwd
  end
end
{% endhighlight %}

So yes it will disappear every time you do it because it overrides you previous commit, so your repository will stay one commit all the time.

But there is nothing easier than fixing it, after you create your `tmp` and jump into it, lets create this `CNAME` file with domain in it and then commit and force push.

So it will look like this:

{% highlight ruby %}
desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    cp_r "_site/.", tmp

    pwd = Dir.pwd
    Dir.chdir tmp

    File.open("CNAME", 'w') {|f| f.write("YOURDOMAIN") }

    system "git init"
    system "git checkout --orphan #{GITHUB_REPO_BRANCH}"
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -am #{message.inspect}"
    system "git remote add origin git@github.com:#{GITHUB_REPONAME}.git"
    system "git push origin #{GITHUB_REPO_BRANCH} --force"

    Dir.chdir pwd
  end
end
{% endhighlight %}

And you are all setup and once DNS changes take effect, will automatically point to your custom domain.

And then you just change info needed to yourself and there you go :)

Well to be precise there are two more things I noticed afterwards, because I had no DISQUS setup, so I had to go [here](https://disqus.com/admin/create/) and create one so I have name to paste in here.

The completely last thing was once creating new post (like this one here), I noticed, that it's not displaying it... well what the hell I thought, but it was simple mistake of not including timezone with time given so it would probably appear after build in couple hours or so. But to be sure you are publishing what you want, set timezone, like i did: `date: 2016-03-06 21:51:00 CET`.

And now I am loving it and never going back + hopefully will be active, previously did not completely work out, because I fell in love with startup culture so much I simply had no free time.

Happy coding everybody :)
