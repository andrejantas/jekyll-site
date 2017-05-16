---
layout: post
title: "SSH on MacOS Sierra"
permalink: ssh-on-macos-sierra
date: 2017-02-12 00:21:00 CET
comments: true
description: "SSH on MacOS Sierra"
keywords: "SSH MacOS Sierra agent"
categories: ssh macos

tags: ssh macos

---

As a shout in the dark I have to write this.... You cannot fathom what I just survived with setting up some stuff after "tiny loss of data" and recovery from timemachine backup. As the story goes I got my stuff back, but somehow iTerm got a bit off so I started over with it. Fresh install and so, I thought it is going to be easier... As far as brew that I use on MacOS goes and libraries I was missing from the backup all went well...

Trouble started once I punched in `ssh SERVERNAME` to log into our company development machine...

First I started off with new annoying problem that happens above macOS 10.12. Mac simply forgets your identities.
Before it was enough to get him to use Keychain Access by running `ssh -K /path/to/key` but that does not work anymore and according to what I read through SO it seems it desired behaviour by apple... well not by us right ?

So starting here, we need to create file `~/.ssh/config` if you got none, that is the place for the configuration. Here the rules go: server name and under it indented by tab all the options/properties or how you call them. So here I found out that I can mimic the old behaviour by only adding couple things in config. Just for someone that does not know, the old behaviour means that you mac keeps your identitties between restarts. And the piece of code is here:

{% highlight ruby %}
Host *
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_rsa
{% endhighlight %}

As simple as the config can be right? Well not quiet yet. We are going to have other trouble. How I am used to work is that from the remote server I open apps that require window server (X11) to be running to display output from the dev machine that I am using. so lets add X11 forwarding there...

{% highlight ruby %}
Host *
  ForwardX11 yes
  UseKeychain yes
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_rsa
{% endhighlight %}

And of course this will not work out for you if you do not have X11 installed on your mac, now is XQuartz app that you can just google away, because you need to use most up to date one anyway. (just a hint: get it from here https://www.xquartz.org/)

So by now X11 forwarding should be working... well it's not, you will probably meet one of these:

`Warning: untrusted X11 forwarding setup failed: xauth key data not generated`

This first one can be easily solved by adding a line in config:

{% highlight ruby %}
ForwardX11Trusted yes
{% endhighlight %}

Which now leaves us with warning:

`Warning: No xauth data; using fake authentication data for X11 forwarding.`

That we can get rid off by pointing to right xauth location since it has changed in new version of macOS and no one told us specifically... well maybe they did, but sure as hell nobody reads changelogs that are couple pages long... So this line will save the day here:

{% highlight ruby %}
XAuthLocation /usr/X11/bin/xauth
{% endhighlight %}

And that should be it for the setup. So I go to dev machine, seems ok no warning on login, X11 opens up when I run something that requires window server, but once I get to `git pull` it goes off with:

`Permission denied (publickey)`

Well of course it will since on remote I got no ssh keys and instead I forward mine, so I need to also add that in the config there.

{% highlight ruby %}
ForwardAgent yes
{% endhighlight %}

ok we are good to go now. So at the end our `~/.ssh/config` looks like this:

{% highlight ruby %}
Host *
    ForwardX11Trusted yes
    ForwardX11 yes
    ForwardAgent yes
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_rsa
    XAuthLocation /usr/X11/bin/xauth
{% endhighlight %}

Which is fair enough right, but to add a few extra lines, mine looks more or less like this:

{% highlight ruby %}
Host *
    ForwardX11Trusted yes
    ForwardX11 yes
    Compression yes
    CompressionLevel 9
    ForwardAgent yes
    UseKeychain yes
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_rsa
    XAuthLocation /usr/X11/bin/xauth
    ServerAliveInterval 60
{% endhighlight %}

What is extra there is just Compression turned on while not everywhere I got high speed internet and I use vim a lot these days, so it really sucks once vim is lagging while you are writing. And also the last line is pinging server I am connecting to every 60 second to ensure I am not logged off for no good reason (for example talking to people about stuff I am doing...).

And that concludes the ssh on macOS Sierra where you have to manually paste in configs to make it work like you were used to... But I always calm myself by loads of advantages the system has except making my life more painful by googling things that were working before last update. I hope I helped somebody with this. Happy coding :)
