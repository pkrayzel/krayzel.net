+++
draft = false
date = 2018-11-25T20:05:37+01:00
title = "Continuous Deployment with Hugo to AWS"
slug = ""
tags = ["continuous-deployment", "continuous-delivery", "continuous-integration", "hugo", "aws"]
categories = []
thumbnail = "images/thumbnail.png"
description = "Build your own continuous deployment pipeline with hugo today!"
+++

As you're reading this article about continuous deployment - you should know, that it was also automatically built and deployed to production.
Today, I wanna show you how - because it's incredibly simple!

![It's simple](https://media.giphy.com/media/3o6Zt16nOfEI0C9sPu/giphy.gif)

[The previous article](/posts/005-hosting-static-website-aws-s3.md) was about serving a static website from AWS S3. 
If you want to use this for running your own blog - you should take a look at <a href="https://gohugo.io/" target="_blank">Hugo</a> 
- a framework for building static websites. And what's more important - it's pretty simple to setup **continuous deployment** pipeline with it. 

If you're not familiar with continuous deployment, let's check the terminology here first. If you already have some experience with it, 
feel free to skip to the next paragraph.

> Continuous Integration (CI) and Continuous Deployment (CD) describe practices for automating the release of software. The two terms are semantically different, but go hand in hand. You will often see them referenced together or used interchangeably.

> Continuous Integration minimizes integration problems by constantly integrating and testing changes to the code. This is achieved by running automated tests whenever code is contributed to a project.

> Continuous Deployment is the practice of automatically releasing updates to production. CD is an extension of CI — you will want to run your integration tests before deploying your code, relying on automated testing to ensure the integrity of the production software.

Yeah that's great, but how it actually works? Well, as soon as I finish writing this article on my local machine, I'll simply **push** the changes to the git repository and it will get automatically deployed and published.

![Just like that!](https://media.giphy.com/media/3o85xkg5PK5JLBg796/giphy.gif) 

I'm using <a href="https://circleci.com/" target="_blank">circleci</a> with GitHub to generate final output from hugo, 
upload it to AWS S3 and then invalidate CloudFront distribution to get it served to all potential users. 

**What do you need to set this up?** 

First, you have to add <a href="https://circleci.com/integrations/github/" target="_blank">circleci to your GitHub account</a>.
Then, you need to create **.circleci/config.yml** file in your git repository (more about this later).
However I've skipped one important part - how to actually generate the static content?
With hugo it's quite simple. Once you're happy with your markdown file content, you can just run: 

``` 
hugo -v -d dist/
```

This command will create static files (as index.html etc) in the **dist/** directory and we can upload it's content to the AWS S3 bucket.

Because we'll be interacting with AWS from circleci, it's a good practice to create a dedicated IAM user in AWS with locked-down 
permissions just for the circleci. I've actually created a group in <a href="https://console.aws.amazon.com/iam/home#/groups" target="_blank">AWS IAM</a>
 with permissions below and then a user which I've assigned to this group. 
 
Group has two policies - one for listing the S3 bucket and uploading files to it. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::www.krayzel.net"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::www.krayzel.net/*"
      ]
    }
  ]
}
```

Second for invalidating CloudFront Distribution. I've actually enabled this on all CloudFront resources in my account 
as I'll be using this role for multiple websites.

``` 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAllCloudFrontPermissions",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
```

Then we need to setup **circleci** for AWS integration. We need access key and secret access key for our new user from AWS.
You should add them as environment variables **AWS_ACCESS_KEY_ID** and **AWS_SECRET_ACCESS_KEY** to your project in circleci. 
You can find more details on <a href="https://circleci.com/docs/2.0/deployment-integrations/#aws" target="_blank">this official website</a>.

![circleci project environment variables](images/006/circleci_environment.png)



As you've probably noticed on the image above, I also have two other environment variables - **S3_BUCKET_NAME** and **CLOUDFRONT_DISTRIBUTION_ID**.
These are used in the following **.circleci/config.yml**, which is the last piece of the puzzle.


```
version: 2
jobs:
  build:
    docker:
      - image: library/python:3.7-alpine3.8
    steps:
      - checkout
      - run: 
            name: install hugo
            command: apk --update upgrade && apk add hugo
      - run: 
            name: install aws command line tool
            command: pip install awscli
      # TODO run some smoke tests ?
      - run: 
            name: create final distribution of the website
            command: hugo -v -d dist/
      - run: 
            name: upload it to the s3
            command: aws s3 sync dist/ $S3_BUCKET_NAME
      - run: 
            name: invalidate the whole cloudfront distribution 
            command: aws cloudfront create-invalidation --distribution-id $CLOUDFRONT_DISTRIBUTION_ID --paths "/*"
```

Whenever I push to master on my <a href="https://github.com/pkrayzel/krayzel.net" target="_blank">blog git repo</a>, 
circleci will trigger a build and all my changes will automatically appear on the internet. Magic! Of course, 
for a proper continuous deployment pipeline, you should also run some tests to actually check that your recent commit didn't break your website. 
I don't have this yet, but I'm checking <a href="https://github.com/gjtorikian/html-proofer">html-proofer</a> as I'm writing this.

![circleci job pipeline](images/006/circleci_job_detail.png)

That's it! And as always, If you have any question, drop me an email at **pkrayzel (at) gmail.com**