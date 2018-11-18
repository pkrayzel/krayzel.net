+++ 
draft = false
date = 2018-11-18T14:45:39+01:00
title = "Intervention Ninja - flask app running on AWS ECS (#01)"
slug = "" 
tags = ["intervention-ninja", "aws"]
categories = []
thumbnail = "images/tn.png"
description = ""
+++

When I was working at Skyscanner, we had the whole tribe responsible for continuous integration and continuous deployment tools. 
Getting your service out to production (AWS ECS), was just the matter of merging PR to master and the rest was like "magic". You didn't need to worry about that.

It is must have for fast-paced company like Skyscanner, that their engineers rather spend time delivering business features then worrying about build and deployment. 
  
In the following blog series, I want to convince you, that deploying services to AWS is simple, even if you don't have the whole tribe behind your back.

**What are we going to build?**

Imagine you have a friend, who drinks too much. Or that you have a colleague, who smells ~~like sh*t~~ bad. 
When you don't have the social skills to deal with that - you can use our *imaginary* website <a href="https://www.intervention.ninja/index.html" target="_blank">Intervention Ninja</a> for it.
It will let them know about their problems (anonymously) via email. All you need is just their email address.

Ok that site is not that imaginary now :)  It's exactly what I've built, in order to prove myself, that I can get my stuff into AWS on my own.
And I want to share that experience with you, step by step.

**Technologies**

To keep things simple we're going to use the following set of technologies:

- html5 with <a href="https://getbootstrap.com/" target="_blank">bootstrap</a> - nice & simple responsive landing page theme is what we need
- <a href="http://flask.pocoo.org/" target="_blank">flask</a> - python micro-framework with Jinja 2 templates
- Python for business logic
- Docker to wrap our application

**What will you need?**

- Text editor / IDE of your choice (i'm using <a href="https://www.jetbrains.com/pycharm/download/" target="_blank">PyCharm community edition</a>)
- <a href="https://www.docker.com/get-started" target="_blank">Docker</a>
- <a href="https://realpython.com/installing-python/" target="_blank">Python 3</a> possibly with <a href="https://virtualenv.pypa.io/en/latest/" target="_blank">virtualenv</a> (if you want to run things locally outside Docker)
- <a href="https://aws.amazon.com/free/" target="_blank">AWS account</a> - if you don't have one, register today, it's free :) 
- AWS command line interface (<a href="https://docs.aws.amazon.com/cli/latest/userguide/installing.html" target="_blank">aws cli</a>) 

**Whenever you're ready...**

Once you'll have all this prepared, let's get this [party started](/posts/003-intervention-ninja-deploy-flask-hello-world-to-aws-ecs)!

