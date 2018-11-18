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

AWS Elastic Container Service (ECS) is Amazon's original solution for running docker containers on cluster of machines (EC2 boxes). 
Nowadays, you would most likely use EKS - Kubernetes on AWS, but let's keep things simple for now.

Let's sign into <a href="https://console.aws.amazon.com" target="_blank">AWS console</a> and 
open service <a href="https://console.aws.amazon.com/ecs/" target="_blank">AWS ECS</a>.

First, we need to create a cluster - on the left side click on **Clusters** and then on **Create cluster** button.
It will be enough for us to use **Networking only** template, so select this option and press **Next**. 
Then type name of your new cluster and press **Create**. Now you should see active cluster in list of clusters!

![AWS ECS List of clusters](images/ecs_clusters.png)

Next, we'll need docker container repository - on the left side click on **Repositories** and then on **Create repository**.
Then type **intervention-ninja-flask** as a name of your repository and press **Create**. Now you should have one repository in your list. 

![AWS ECS Repository](images/ecs_repository.png)

Let's open it's detail (click on it's name) and then press **View Push Commands** button.
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

It might take a while and then you should see output similar to this:

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

 Once the command successfully finishes, you should also have a docker image in your repository in AWS console.
 
![AWS ECS Repository with image](images/ecs_repository_image.png)

**Creating task definition in AWS**

Next, we need to create our task definition, which we'll run on our cluster. On the left side, let's choose **Task definitions**. 
Click on **Create new Task Definition** button. Let's choose **Fargate** type as it's easier to start with press **Next step**.
You see all the details I've filled on the following image and you also need to click on the **Add container** button. 

![AWS ECS Task definition fargate](images/ecs_task_definition_fargate.png)

There's a lot of options for new container. However we only need to specify it's name, image and port mapping. 

![AWS ECS Task definition - add new container](images/ecs_task_definition_container.png)

Once we click on **Create** button for both container and task definition, we should have one active task *intervention-ninja-flask* in our list.

**Running a task on a cluster**

Now final round - let's click on the **Clusters** menu and open detail for our cluster. 
There's again a lot of options here - what we want for now, is to create a new task.

If you're asking yourself what's the difference between task and a service: 

> A Task is created when you run a Task directly, which launches container(s) (defined in the task definition) until they are stopped or exit on their own, at which point they are not replaced automatically. Running Tasks directly is ideal for short running jobs, perhaps as an example things that were accomplished via CRON.

> A Service is used to guarantee that you always have some number of Tasks running at all times. If a Task's container exits due to error, or the underlying EC2 instance fails and is replaced, the ECS Service will replace the failed Task. This is why we create Clusters so that the Service has plenty of resources in terms of CPU, Memory and Network ports to use. To us it doesn't really matter which instance Tasks run on so long as they run. A Service configuration references a Task definition. A Service is responsible for creating Tasks. 

So on the **Tasks** tab click on **Create task** and fill the form as seen on the next image.

![AWS ECS Cluster - run task](images/ecs_run_task.png)