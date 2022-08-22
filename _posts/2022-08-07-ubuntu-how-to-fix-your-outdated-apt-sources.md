---
layout: post
title: "Ubuntu: How to fix your outdated apt sources"
author: "Phitrapy"
date:   2022-08-07 16:27:00 +0200
tags: ubuntu linux config apt upgrade tech 
lastModifiedDate: 2022-08-22 16:34:16 +0200
---

Today, as I was trying to install the dependencies to run jekyll on my computer, I ran into a common yet annoying problem: `apt update` would not work anymore...

## The problem 🥲

It turned that my ubuntu version 21.10 reached its EOL (end of life) in July, and I couldn't reach the apt resources to update it anymore!

It resulted in multiple 404 errors as `apt` tried to reach the repositories... ouch.

Fortunately, all the sources are still stored, archived, somewhere else.  We just have to tell `apt` to look there instead of the original repositories.

## The fix 👌

All the work is done in the file `/etc/apt/sources.list`.
In order to target the old releases, we can use the `sed` tool. ([click here to learn more](https://linuxize.com/post/how-to-use-sed-to-find-and-replace-string-in-files/))

So we will run in the console:

{% highlight bash %}
sudo sed -i -r 's/([a-z]{2}.)?archive.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo sed -i -r 's/security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
{% endhighlight %}

There, we replaced the main repos for `old-releases.ubuntu.com`.

And that's it! Now we can do all the remaining updates in order to upgrade the distro!

## Reminder
In order to avoid this annoying situation, here is a friendly reminder to:
* Update your distro **regularly**
* Consider using a *Long Term Support* version (LTS) rather than the others
* Use **apt full-upgrade** and the **do-release-upgrade** tool to easily upgrade your distro
* Take a look at the [Ubuntu List of Releases](https://wiki.ubuntu.com/Releases) to check which versions are LTS, and when they reach their EOL.

That's all for today... Keep your env up to date, and stay hydrated.