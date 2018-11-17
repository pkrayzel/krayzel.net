---
title: "Running hugo on AWS"
date: 2018-11-17T19:32:30+01:00
draft: true
---

Hello visitor! 

I've just created this blog site using <a href="https://gohugo.io/" target="_blank">hugo</a> and I'm excited about it! :) I'll be using this place to share my ideas, thoughts and experience. 

The first "real" blog post will most likely be about <a href="https://www.intervention.ninja/" target="_blank">Intervention Ninja</a>. That's my spare time project, which I've built to demonstrate how simple is to get stuff done on AWS.

This site itself is also hosted on AWS. It actually sits in AWS S3 and it's delivered to you through AWS CloudFront (content delivery network). 
Domain is registered with AWS Route 53 (DNS) and do you see that *https* and valid certificate near address bar? 
That's AWS ACM (amazon certificate manager). :) 

If you'd like to host your own static website on AWS, then check this project on GitHub <a href="https://github.com/pkrayzel/aws-static-site" target="_blank">aws-static-site</a> - I'll share a blog post about it later.
