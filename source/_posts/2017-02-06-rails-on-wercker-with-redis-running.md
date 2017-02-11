---
layout: post
title: "Rails on Wercker with redis running"
permalink: rails-on-wercker-with-redis-running
date: 2017-02-06 00:21:00
comments: true
description: "Rails on Wercker with redis running"
keywords: "wercker ci rails ruby test integration redis"
categories: rails ci test redis

tags: rails redis test ci

---

Hi guys, I know I have been silent for kinda a looong long time, I had loads on my mind, but now new year new resolutions I am not going to keep up with, but one of them I should, because I got loads here that I was dealing with and was not the easiest, but got it solved.

First on the agenda is my overuse of free or really cheap but really nice solutions. Today is CI. I hope all of us are familiar with CI, I am talking about continuous integration here. To sum it up basically runner that goes through your test and rubocop and such stuff each time you push to git. Proven very useful in many cases and must have when you develop something and you want to keep an eye on quality of your work.

Today is not the time to share all the tools I use for development although couple of things will be visible in config over here, but I want to talk about CI I use for one of my free time love which is this coffee subscription - [inboxcoffee.cz](https://inboxcoffee.cz) which I work on in my free time.

What I use is [wercker.com/](http://www.wercker.com/). Wercker does not seem to me very popular or such, but for myself is still free even for private github repositories and serves the purpose well.

One of the pitfalls of it is the setup. I am one of those that want to do all I do without any description or how to, because I mean why, they mentioned is user friendly so I should be able to set it up… well we will see about that in a bit.

I am running pretty much standard stack on the page, you can check some of it out here: [stackshare.io/redrick/inbox-coffee-company](https://stackshare.io/redrick/inbox-coffee-company)

So lets have a look here, running ruby 2.3… check not a problem, wercker work with docker images, so we can move on. Using postgres as service is also no problem as you may be used from any other CI. But then redis comes in, I am testing also functionality of things going through sidekiq, mocking them and so, but as it is recurring subscription service, some of them are run 17x quicker than real time and other fun stuff to assure quality of service. Nevermind that weird stuff I just said, even for smoke testing I need redis running right…. well not as straight forward as I thought there, but works at present time just fine.

Let me walk you through it, but first I will just paste it here:

{% highlight ruby %}
services:
  - redis
build:
  steps:
    - install-packages:
      packages: redis-server redis-tools

    - script:
      name: start redis
      code: |
        redis-server --daemonize yes
        redis-cli ping
{% endhighlight %}

basically what I had to do there is to install redis as package in the ruby container I run tests as first step in the build process and next daemonize it (as bad as it may seem) which would probably help me if I was about to use it restart and so on but also automatically starts the instance which is exactly what we need right here. And yeah well so you can feel good, you know the drill, sen ping to redis-cli and if it’s running answers with funny PONG. and that is all. I myself am a mac guy so that is maybe why I had to google even simple solution like this, but we learn as we go.

So finally full config to run your tests including redis may look like this:

{% highlight ruby %}
ruby: 2.3.1
box: ruby
services:
  - postgres
  - redis
build:
  steps:
    - install-packages:
      packages: redis-server redis-tools

    - script:
      name: start redis
      code: |
        redis-server --daemonize yes
        redis-cli ping

    - bundle-install
    - rails-database-yml

    - script:
      name: Set up db
      code: bundle exec rake db:schema:load RAILS_ENV=test

    - script:
      name: rspec
      code: bundle exec rspec
{% endhighlight %}

OK, my real config includes more than 60 lines, so probably I should walk you though it whole, but this one serves ok for purpose of demonstrating what does it take to run rspec tests including redis on wercker.

Happy coding and see you all next time.
