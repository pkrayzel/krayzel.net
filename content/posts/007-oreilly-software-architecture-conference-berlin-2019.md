+++
draft = false
date = 2019-11-07T18:12:37+01:00
title = "O'Reilly SA Conference - Berlin, November 2019"
slug = ""
tags = ["oreilly", "software-architecture", "oreillysacon", "education"]
categories = []
thumbnail = "images/thumbnail.png"
description = "O'Reilly Software Architecture Conference - Berlin, November 2019"
+++

I've attended O'Reilly Software Architecture conference in Berlin.
I have to say it was the best conference I've been so far in my life, 
and I don't think it's only because I'm writing this still on a train from there. :) 

I've bought the **platinum** pass, which is veeery expensive, however it's worth every single penny (actually every cent).

With platinum pass I've had access to the 2-Days Training Courses, 1 year access to O'Reilly online platform and access to all the talks on Wednesday and Thursday.  

## 2-Days Training Course

The first big challenge was what training to actually choose - there were more of them that I was interested in.

My candidates were: 

- 1. [Building evolutionary and incremental architectures](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/detail/79331) by [Allen Holub](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/speaker/246541)
- 2. [Fundamentals of software architecture](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/detail/79344) by [Mark Richards](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/speaker/339235)
- 3. [Moving to microservices and beyond](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/detail/79513) with [Sam Newman](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/speaker/75964)

At the end I've decided for Sam Newman's training **Moving to microservices and beyond**, as it seemed to be focused on the topics most relevant to what I'm dealing with at the moment (moving parts of the domain of the core legacy system - into highly available & customer facing microservice).

Sam wrote a book [Building Microservices](http://buildingmicroservices.com/) back in 2015 and he's working on a second edition which should be published soon.
As a side product of the second edition he wrote [Monolith to Microservices](https://samnewman.io/books/monolith-to-microservices/) and he gave us all signed early-release version.

We went through a lots of topics (observability, CAP theory, SAGAS and approaches to distributed transactions, ...) and few group exercises (architect real-life scenario, event storming, ...) during those 2 days. 
What I've really enjoyed was his pragmatism and focus on business value when deciding whether the microservices architecture is the right fit for the situation or not. 
Also his sense of humor and sarcasm was brilliant :)
  
<img alt="workshop topics" style="width:25%" src="images/007/snewman_topics.png" />

**Few things that resonate with me**: 

- If you're thinking about adoption, first try to solve **log aggregation** problem across all of your current services. 
That can show you whether you're ready to face challenges connected to microservices.
- The first rule of **distributed transactions club** is don't use distributed transactions :-D 

## 3 best talks

It's not easy to choose just three talks, however I think these would be the ones I've enjoyed the most or I've learned the most at. 

[The rise and fall of microservices](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/detail/78649) by [Mark Richards](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/speaker/339235)

Mark likens the fall of Roman empire with the current situation of microservices architecture style.
He mainly focused on when it's not a good idea to use that at all, 
what are the business benefits it can bring you and what are the trade-offs then.


[Migration to Microservices](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/detail/78648) by [Mark Richards](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/speaker/339235)

I've enjoyed his second talk even more than the first one, it was pragmatic manual how to do the refactoring 
of the current system, what to do before we even step into microservices world (and what that even means), 
how can we use tools to estimate the scope of changes and where the main pain points are etc.


[How I learned to love the rebuild: How to know when to reinvest in your systems](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/detail/78767) by [Rob Zuber](https://conferences.oreilly.com/software-architecture/sa-eu/public/schedule/speaker/350284) 

Rob is the CTO of CircleCI and as he works mostly in a start-up world, he gave very interesting "cash back" argument, why it's almost never a good idea 
to rewrite the current monolith / solution from scratch.
 
Some of his tips from CircleCI were things, we've actually used in the past at MADE.COM as well, which is always reassuring :) 
Eg. when your service has multiple parts running on the same container, and you don't have time to split the service properly but need to scale, use the same docker image however control
what part of the service runs at it using different env. variables.


## Architectural Katas

I've attended the Architectural Katas event hosted at the conference (and missed the first half of Barcelona vs Slavia Prague match because of that).
 
First challenge was to be selected from all those ~200 people to one of the 6 teams with only 5 people on each of them. 
I've had huge amount of luck and I've drawn piece of paper with the **number 6** on it from the Neil Ford's drawing bowl.   

Our kata was **Sysop Squad** from [Neil Ford's list of katas](http://nealford.com/katas/list.html).

We've only had 45 minutes to solve the problem and the time pressure was the most challenging part - especially when working with 4 people you have never met before.

To make the long story short, our team actually won the 1st place as you can notice on the tweet bellow: 

{{< tweet 1192049376329437184 >}} 

However that's not the main reason I'm mentioning it here... ok, I did want to mention that. Anyway, it was such a great experience that I've decided 
to try organize such thing myself - see [Architectural Katas](/katas) and stay tuned.
 
## Meet the experts 

There was a great chance to spend ~45 minutes with Neil Ford and Mark Richards sitting at the same round table and asking them questions.

**Things that resonate with me**:

- **Neil Ford**: "Your job as an architect is not to come up with the best solution. It's to know the trade offs of all potential solutions, come back to business with them and choose the least worst for given trade offs."
- **Mark Richards**: "If you want to become a good software architect you need to breadth your knowledge. This might be very difficult first when you want to be an expert in particular technology and you need to shift your thinking."  