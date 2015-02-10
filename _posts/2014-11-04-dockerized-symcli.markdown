---
layout: post
title: Dockerized SYMCLI
date: 2014-11-04 05:40:53.000000000 -05:00
categories:
- Automation
- SYMCLI
- Symmetrix
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
author:
  login: scummins
  email: sean@scummins.com
  display_name: Sean Cummins
  first_name: Sean
  last_name: Cummins
---
If you work with offline SYMAPI databases frequently, you've probably encountered cases where you can't open a DMX SYMAPI database on a recent version of Solutions Enabler. Getting around this typically means you must maintain several different versions of Solutions Enabler, to ensure that you can read SYMAPI databases produced by DMX, VMAX, and VMAX^3 arrays at diverse code levels. And because only one instance/version of Solutions Enabler can be installed on any given OS instance, you end up building and maintaining several virtual machines, each with a different version of Solutions Enabler installed.

No that big of a deal, really. But it does get kind of annoying when the states of these virtual machines diverge over time. For example, when you manually install ad-hoc packages, change security settings, apply patches, modify your .dotfiles, etc.

Also, every time you need to use a particular version of Solutions Enabler, you have to boot a virtual machine -- which takes a while, and chews up memory/CPU resources just to do the same thing your host OS is already doing -- and possibly just to do the same thing that other VMs you're hosting are also doing. In other words, you're dealing with the "VM Tax" -- there's a ton of 'effort duplication' going on here just to run a particular version of Solutions Enabler on your laptop.

*Docker can make this much easier.*

First though, please understand that I would only recommend doing this for offline/test/dev use. Solutions Enabler is not designed nor tested to be used in a container, so even though it works, it could produce unexpected results -- generally not a desirable attribute in a production VMAX :)

So, to reiterate: **THIS IS NOT FOR PRODUCTION ENVIRONMENTS.**

# So what can I use it for?

Ok, I get it; it's not for production. So what's the point then?

I can think of a few good use cases:

* Provides a safe "read-only" development sandbox for creating custom reports in environments with multiple generations of Symmetrix systems.
* You're in an EMC or Partner pre-sales, post-sales, or support role, and you work with Symmetrix systems on a regular basis. [Offline SYMCLI](http://blog.scummins.com/?p=56) is a very useful for people who help other organizations install, manage, and support their VMAX arrays.
* You work with Symmetrix systems on a regular basis, and you want to learn more about shiny new stuff like [Docker](https://docker.com/) and [Vagrant](https://www.vagrantup.com).

# Where can I get it?

The latest version is now easily deployed via Vagrant. The bits and installation instructions can all be found on GitHub here: [https://github.com/seancummins/dockerized_symcli](https://github.com/seancummins/dockerized_symcli)

If you have any questions/suggestions/problems, drop me a line here, or feel free to make changes and send me a pull request.
