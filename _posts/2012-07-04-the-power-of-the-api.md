---
layout: post
title: "The power of the API, and why so many developers are doing it wrong"
permalink: /2012/07/the-power-of-the-api
---

When times were simple for web developers, we used to create "a web site" that is viewed in "a browser" on "a computer". Not so, any more. Users now not only would like, but *expect* that a given brand will have a desktop site, a tablet site, a mobile site, apps, widgets and everything in between. From a technical perspective, the simplest way to achieve that goal is to base your entire existence on a central API. This is exactly what I've spent the last year of my life working on.

Where I work, we have three brands that collectively operate what used to be eight web sites. Supported by our new platform, that eight web sites has become eight "tablet-ready" desktop web sites, eight mobile sites, three mobile apps, regular email newsletters, and so much more. At the heart of that new platform is our JSON-based ReST API.

![The Power of the API](/assets/images/power-of-api.jpg){: .img-fluid}

<img class="alignnone  wp-image-13" title="" alt="" src="" width="576" height="432" />

What makes that approach a success is that every part of the system - our web sites, our CMS, and even our internal weekly web stats email - are all created, managed and delivered using the API. Nothing goes into or comes out of our database(s) by any other means. Given the way that all of this works, it can take a little leg work to get a completely new feature off the ground, but the payoff is absolutely worth it. With that approach, we were able to take the API end-points we made available for our internal web stats interface and rapidly create a weekly stats summary email that gets automatically sent to the entire company, and better yet, we are now taking the enhancements we made to support that weekly stats email and building a "most popular content" widget for our sites. Having your API at the core of everything you do practically forces you to completely cut-out reinventing the wheel within your own system, in just the same way as the open source movement and sites like Github do for avoiding doing work that someone else has already done.

The problem is, none of this works unless you can totally embrace it. I've seen an awful lot of more traditional-style web applications such as eCommerce systems, forums, and so on, try and bolt an API on the side of their existing system. Day to day, their developers write code that interfaces directly with internal classes, which in turn interface with the database, and then separately build an API that "external apps" can use, that does exactly the same thing - but with a completely separate implementation. This can only lead to misery for everyone. You end up with bugs in your own system that your API users don't encounter (and vice versa), features they can't access because a developer didn't port it to the API, and an unimaginable amount of pain for any major refactoring project.

This is an exciting time to be a developer with the luxury of working on a new piece of software, a "software as a service" solution and an altogether more saddening time for anyone who is expected to make all of this work with a legacy web app. Taking the risk of creating a new platform from scratch has given us the benefit of being able to embrace the many channels available, allowing us to create a piece of content and share it online, on mobile, socially and more, instantly.
