---
layout: post
title: Jekyll Migration, DNS, and URL Redirection
date: 2015-3-21 17:02:54.000000000 -05:00
tags: []
status: publish
comments: true
type: post
published: true
author:
  login: scummins
  email: sean@scummins.com
  display_name: Sean Cummins
  first_name: Sean
  last_name: Cummins
---

So I finally bit the bullet, and I've migrated this blog from Wordpress on AWS to Jekyll and Disqus on Github Pages. Because that's what all the cool kids are doing.

Actually, my reasoning was twofold:

1. I'm a cheapskate.
2. I wanted more git in my life.

My final solution ended up being free (gratis & libre), and deployment is managed entirely via git.

I haven't exactly been prolific in my blog posting, so the migration was, for the most part, pretty straightforward. There are a million posts out there that discuss making the switch, so I won't reiterate all that here. The one topic I couldn't find much information on, though, was how to handle URL redirection in the event that your Wordpress install retained the [default](http://www.elegantthemes.com/blog/tips-tricks/wordpress-permalinks) Wordpress URL scheme (where all posts appear as `<baseurl>/?p=<postid>`).

Github pages is able to perform some basic redirection, but it works by explicitly mapping old page URLs to new page URLs. With the default Wordpress URL scheme, there is essentially only one page (the implicit index.php page), and the `?p=<postid>` portions of the URL are query strings -- key/value pairs that are being passed into index.php. Github Pages [doesn't support](https://github.com/jekyll/jekyll-redirect-from/issues/28) redirection based on query strings, so I began searching for options that would allow me to migrate to Github Pages without breaking my old links.

The typical solution to this is to create HTTP 301 redirects, but this requires access to the web server's config files (e.g. Apache's .htaccess file). This [isn't an option](https://help.github.com/articles/redirects-on-github-pages/) with Github Pages. So my initial solution was to spin up a small Digital Ocean VM, point 'blog.scummins.com' (the base URL for all of my old links) to that VM, and create some nginx 301 redirects so that my old URL patterns were redirected to my Jekyll blog on Github Pages. This worked, but it meant that I had to maintain (and pay for) a Digital Ocean VM just to have HTTP 301 redirects. It also just felt kind of janky, which is why all the redirects pointed to janky.scummins.com. I've needed a reason to use the work janky since hearing Alex Ohanian's keynote at Varrow Madness 2015 ;)

So I continued researching, and found a free DNS provider (CloudFlare) that provides HTTP 301 redirection services. I was kind of reluctant to change DNS providers, as I have a lifetime dyndns membership that I feel kind of compelled to use -- but CloudFlare could do what I needed to do, and the switchover cost was zero. CloudFlare also provides the ability to specify a CNAME as the apex/bare domain without breaking the DNS spec. Breaking the DNS spec can result in problems. Using a CNAME is Github's recommendation for custom domains with Github Pages, but per the DNS spec, bare domains can only be A records. CloudFlare does some [CNAME flattening](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/) magic to effectively create CNAMEs on bare domains while conforming to the DNS spec.

CloudFlare also provides three free HTTP 301 redirects. I used two for my most-visited direct links, and one catch-all, so any remaining traffic sent to blog.scummins.com/* will be redirected to scummins.com. My page redirect rules look like this:

![]({{ site.url }}/assets/Page_Rules_for_scummins_com_CloudFlare.png)

As a bonus, CloudFlare is also provides a bunch of other services, like caching/acceleration/compression via its content delivery network; analytics; and threat management. It will even cache & serve your static content in the event that your services are down. These services are offered free of charge, and there are paid plans with additional features and scalability limits.

There are definitely other DNS providers out there with similar 301 rediction services and CNAME-flattening-equivalent services -- for example, [DNS Made Easy](http://help.dnsmadeeasy.com/managed-dns/records/http-redirection-record/), [easyDNS](http://helpwiki.easydns.com/index.php/URL_Forwarding), and even [Dyn](https://help.dyn.com/setting-up-http-redirect/) -- but I don't know of any other free ones. ;)
