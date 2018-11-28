+++
draft = false
date = 2018-11-23T21:38:37+01:00
title = "Hosting a static website in AWS S3 with AWS CloudFront and AWS Route 53"
slug = ""
tags = ["hugo", "aws s3", "aws cloudfront", "aws route 53", "cloudformation", "content delivery network"]
categories = []
thumbnail = "images/005/aws_static_site_hosting.png"
description = "Article describes how you can host your static website in AWS S3 with AWS CloudFront and AWS Route 53. It also provides step by step manual and CloudFormation template for you."
+++

This article describes how you can host your static website in AWS S3 with AWS CloudFront and AWS Route 53.
And because it's best to use real world examples - I'll show you how this is done for the website you're reading right now.

The following diagram shows the components of this solution.  

![AWS static content hosting](images/005/aws_static_site_hosting.png)

**User** can be from anywhere right? Anywhere around the world, from Asia, Australia, Europe to America. 
The goal is to serve our website to the user fast, no matter where he / she is connecting from. 

**AWS ACM** is a certificate manager from Amazon. You can quickly register trusted certificate for your domain 
in order to deliver trusted experience through **https**. Price per one certificate is $0.75 / month / region.

**AWS Route 53** is scalable DNS solution from Amazon. If you want to learn more about it's concept, 
I recommend checking <a href="https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/route-53-concepts.html" target="_blank">the official documentation</a>.
Price per hosted zone / month is **$0.50** for the first 25 hosted zones.

I've registered domain **krayzel.net** through AWS Route 53 (yes Route 53 is registrar as well). 
However if you already have domain registered with different registrar, you have two options. 
You can either transfer your domain to Amazon or you can just set 
the name servers in your DNS provider to the Amazon name servers. 

After successful domain registration, I have one **Hosted Zone** for domain **krayzel.net** in AWS Route 53. 
For hosted zone you can specify records for your website as you need. 
For this static website, I have two **A level** records for **www.krayzel.net** and **krayzel.net**. 
Both of them are routed to the same AWS CloudFront Distribution DNS record.

**AWS CloudFront** is content delivery network - it's a network of geographically dispersed servers,
which delivers cached content from websites to end users based on the geographic locations of the user, 
the origin of the static content and a content delivery edge server. Thanks to this, your users can have 
similar user experience whenever they are.  

<a href="https://aws.amazon.com/cloudfront/pricing" target="_blank">Pricing model for CloudFront</a> is more complex - it depends on the traffic - data in and out, number of requests 
and also the amount of the file invalidation requests (first 1000 invalidation requests in month are free).

![Content Delivery Network](images/005/cdn.png)

**AWS S3** is the actual storage for your static content. We need one bucket with public access to it, 
where we store the actual content (eg. index.html and other files for your website). 

**AWS CloudFormation** - this piece of the puzzle is not visible on the diagram. It's infrastructure as a code solution.
It's purpose is to model and provision all your AWS cloud infrastructure resources in a repeatable manner. 

**How to set this up in AWS**

Because I'm using this setup on multiple sites already, I've created CloudFormation template for it.
You can use it as well for your own sites - check my <a href="https://github.com/pkrayzel/aws-static-site" target="_blank">aws-static-site repository</a> on github. 

Check this <a href="https://github.com/pkrayzel/aws-static-site/blob/master/README.md" target="_blank">readme</a> for detailed step-by-step manual how to deploy the content and invalidate your CloudFront distribution.

If you have any questions, drop me an email at **pkrayzel (at) gmail.com**. 
