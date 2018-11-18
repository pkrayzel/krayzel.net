+++
draft = true
date = 2018-11-18T20:36:39+01:00
title = "Intervention Ninja - deploy flask hello-world to AWS ECS (#03)"
slug = ""
tags = ["intervention-ninja", "aws"]
categories = []
thumbnail = "images/tn.png"
description = ""
+++

**Preparing AWS ECS**

AWS Elastic Container Service (ECS) is Amazon's original solution for running docker containers on cluster of machines (EC2 boxes). 
Nowadays, you would most likely use EKS - Kubernetes on AWS, but let's keep things simple for now.

Let's sign into <a href="https://console.aws.amazon.com" target="_blank">AWS console</a> and 
open service <a href="https://console.aws.amazon.com/ecs/" target="_blank">AWS ECS</a>.

First, we need to create a cluster - on the left side click on *Clusters* and then on *Create cluster* button.
It will be enough for us to use *Networking only* template, so select this option and press *Next*. 
Then type name of your new cluster and press *Create*. Now you should see active cluster in list of clusters!

**TBD image ?**

Next, we'll need docker container repository - on the left side click on *Repositories* and then on *Create repository*.
Then type **intervention-ninja-flask** as a name of your repository and press *Create*. Now you should have one repository in your list. 
Let's open it's detail (click on it's name) and then press *View Push Commands* button.

**TBD image ?**

You should see step by step manual how to push docker image into this repository.

**Pushing docker image to your AWS repository**

**Warning** - following steps require aws cli being properly setup on your machine with aws credentials as well. 
<a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html" target="_blank">Check this manual</a> for help. 

First let's login to our new repository.

```
$(aws ecr get-login --no-include-email --region us-east-1)
```

Hopefully you'll see ```Login Succeeded```.

Then we need to tag our local docker image and push it into our AWS repository. Run following two commands 
(change parameters in your case as AWS told you in *View Push Commands* manual):

``` 
docker tag intervention-ninja-flask:latest 166058053690.dkr.ecr.us-east-1.amazonaws.com/intervention-ninja-flask:latest
docker push 166058053690.dkr.ecr.us-east-1.amazonaws.com/intervention-ninja-flask:latest
```

It might take a while, you should see output similar to this:

``` 
The push refers to repository [166058053690.dkr.ecr.us-east-1.amazonaws.com/intervention-ninja-flask]
3a1a7a42d977: Pushed 
40fe83e8a33e: Pushed 
9d47dcf94482: Pushing [================================>                  ]  3.823MB/5.929MB
9d47dcf94482: Pushed 
66bd78ef2de1: Pushing [=>                                                 ]  1.655MB/70.17MB
66bd78ef2de1: Pushed 
ebf12965380b: Pushed 

latest: digest: sha256:70059d082a08176eae045982f4f7de115f2d0c5cf76e92e42fc0e66bf20c0922 size: 1786
```

 Once the command successfully finishes, you should see a docker image in your repository in AWS console.
 
**TBD image**

**Creating task definition in AWS**

In AWS console let's get back to ECS and on the left side, let's choose *Task definitions*. 
Click on *Create new Task Definition* button. Let's choose *Fargate* type as it's easier to manage and press *Next step*.
Let's put *intervention-ninja-flask* as task name, then choose *0.5GB* as task memory and *0.25vCPU* as task CPU.
Also click on *Add container* button and put *intervention-ninja-flask-container* as a name 
and URL to the docker image in your repository. In my case it's *166058053690.dkr.ecr.us-east-1.amazonaws.com/intervention-ninja-flask:latest*.

We don't need to fill anything else for this simple case, so let's click on *Create* (container) and *Create* (Fargate task definition).

We should now have one active task *intervention-ninja-flask* in our list of task definitions.

**TBD Image?**