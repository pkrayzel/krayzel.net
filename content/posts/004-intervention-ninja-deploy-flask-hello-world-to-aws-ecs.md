+++
draft = true
date = 2018-11-28T20:36:39+01:00
title = "Intervention Ninja - deploy flask docker app to AWS ECS (#03)"
slug = ""
tags = ["intervention-ninja", "aws ecs", "docker", "flask"]
categories = []
thumbnail = "images/004/intervention_ninja_ecs.png"
description = ""
+++

>**Warning** If you haven't read the previous articles from the Intervention Ninja series, it might not make sense. 
So rather check following parts before reading (at least #02):

>- [INTERVENTION NINJA - FLASK APP RUNNING ON AWS ECS (#01)](/posts/002-intervention-ninja-flask-app-running-on-aws-ecs.md) 
>- [INTERVENTION NINJA - BUILD FLASK APP DOCKER IMAGE (#02)](/posts/003-intervention-ninja-build-flask-app-docker-image.md)


**AWS Elastic Container Service**

AWS ECS is Amazon's original solution for running docker containers on cluster of machines (EC2 instances). 
Nowadays, you would most likely use EKS - Kubernetes on AWS, however let's keep things simple for now.

**1. Create cluster**

I think the first step is obvious - we need to create ECS cluster, where our job will run.
Let's sign in to <a href="https://console.aws.amazon.com" target="_blank">AWS console</a> and 
open service <a href="https://console.aws.amazon.com/ecs/" target="_blank">AWS ECS</a>. 
On the left side click on **Clusters** and then on **Create cluster** button. 
As we want to keep things simple - we'll use **EC2 Linux + Networking** template. 
With this template, we'll also create first EC2 instance that will be added into cluster and we'll be able to 
start running things straight away.

So let's select **EC2 Linux + Networking** template and press **Next step**. The only things we need to specify on the next page are:

- **Cluster name**: intervention-ninja-cluster
- **EC2 instance type**: t2.micro 
    - **!!! careful** - if you select larger machine, you won't be eligible for 
<a href="https://aws.amazon.com/free/" target="_blank">free tier</a> and you might spend some money 

![AWS ECS List of clusters](images/004/ecs_create_cluster.png)

As we've exposed our flask application on port 80, we don't need to change anything in security group's section, 
as this should be default inbound rule there already.

![AWS ECS List of clusters](images/004/ecs_create_cluster_sg.png)

Click on the **Create** button and wait until the cluster is created.

**2. Create container repository** 

Next, we'll need docker container repository - the place where we'll store our docker images.
You could also use <a href="https://hub.docker.com/" target="_blank">https://hub.docker.com/</a> or something else. 
If you want to use ECS, click on **Repositories** (on the left side menu) and then on **Create repository**.
Then type **intervention-ninja-flask** as a name of your repository and press **Create**. Now you should have one repository in your list. 

![AWS ECS Repository](images/004/ecs_repository.png)

Let's open it's detail (click on it's name) and then press **View Push Commands** button.
You should see step-by-step manual how to push docker image into this repository.

**3. Pushing docker image to your AWS repository**

> **Warning** - following steps require aws cli being properly setup on your machine with aws credentials as well. 
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
 
![AWS ECS Repository with image](images/004/ecs_repository_image.png)

**4. Creating a task definition in AWS**

Next, we need to create our task definition, which we'll run on our cluster. 
We can do this from command line as well. Let's create a file called **task_definition.json** and put following content to it:
{{< highlight go "linenos=table,linenostart=1" >}}
{
    "family": "intervention-ninja-flask",
    "containerDefinitions": [
        {
            "name": "intervention-ninja-flask",
            "image": "166058053690.dkr.ecr.us-east-1.amazonaws.com/intervention-ninja-flask",
            "cpu": 100,
            "memory": 256,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80
                }
            ],
            "essential": true
        }
    ]
}
{{< / highlight >}}

In your case, change the **image** on line 6 to URL of your docker repository.

Then let's run the following command:

``` 
aws ecs register-task-definition --cli-input-json file://task.json
```

If everything works as expected, you should see output similar to this:

{{< highlight go "linenos=table,linenostart=1" >}}
{
    "taskDefinition": {
        "taskDefinitionArn": "arn:aws:ecs:us-east-1:166058053690:task-definition/intervention-ninja-flask:8",
        "containerDefinitions": [
            {
                "name": "intervention-ninja-flask",
                "image": "166058053690.dkr.ecr.us-east-1.amazonaws.com/intervention-ninja-flask",
                "cpu": 100,
                "memory": 256,
                "portMappings": [
                    {
                        "containerPort": 80,
                        "hostPort": 80,
                        "protocol": "tcp"
                    }
                ],
                "essential": true,
                "environment": [],
                "mountPoints": [],
                "volumesFrom": []
            }
        ],
        "family": "intervention-ninja-flask",
        "revision": 8,
        "volumes": [],
        "status": "ACTIVE",
        "requiresAttributes": [
            {
                "name": "com.amazonaws.ecs.capability.ecr-auth"
            }
        ],
        "placementConstraints": []
    }
}
{{< / highlight >}}

Note down your task definition arn as you'll need it in a bit. It is on **line 3** - in my case it's: 
```
arn:aws:ecs:us-east-1:166058053690:task-definition/intervention-ninja-flask:8
```

**5. Running a job on a cluster**

OK, final round!!! Let's submit a job to our cluster! 
 
![Final round](https://media.giphy.com/media/tKsjKnxt7LX44/giphy.gif)

There are two types of jobs - services and tasks.

> A Task is created when you run a Task directly, which launches container(s) (defined in the task definition) until they are stopped or exit on their own, at which point they are not replaced automatically. Running Tasks directly is ideal for short running jobs, perhaps as an example things that were accomplished via CRON.

> A Service is used to guarantee that you always have some number of Tasks running at all times. If a Task's container exits due to error, or the underlying EC2 instance fails and is replaced, the ECS Service will replace the failed Task. This is why we create Clusters so that the Service has plenty of resources in terms of CPU, Memory and Network ports to use. To us it doesn't really matter which instance Tasks run on so long as they run. A Service configuration references a Task definition. A Service is responsible for creating Tasks. 

For the purpose of this tutorial, we'll stick with *tasks* because it's simpler. 
There are only two arguments necessary for them - the name of the cluster and the name / arn of the task definition.

```
aws ecs run-task --cluster intervention-ninja-cluster --task-definition arn:aws:ecs:us-east-1:166058053690:task-definition/intervention-ninja-flask:8
```

And again, if everything works as expected, you should see output similar to this:

``` 
{
    "tasks": [
        {
            "taskArn": "arn:aws:ecs:us-east-1:166058053690:task/51c71262-55eb-4076-9485-b0b1216d569b",
            "clusterArn": "arn:aws:ecs:us-east-1:166058053690:cluster/intervention-ninja-cluster",
            "taskDefinitionArn": "arn:aws:ecs:us-east-1:166058053690:task-definition/intervention-ninja-flask:8",
            "containerInstanceArn": "arn:aws:ecs:us-east-1:166058053690:container-instance/304cf913-2163-411b-8b02-dc77f5f5e9a9",
            "overrides": {
                "containerOverrides": [
                    {
                        "name": "intervention-ninja-flask"
                    }
                ]
            },
            "lastStatus": "PENDING",
            "desiredStatus": "RUNNING",
            "containers": [
                {
                    "containerArn": "arn:aws:ecs:us-east-1:166058053690:container/cfb0d6b7-0521-4bb6-bd61-2d2ae9092fec",
                    "taskArn": "arn:aws:ecs:us-east-1:166058053690:task/51c71262-55eb-4076-9485-b0b1216d569b",
                    "name": "intervention-ninja-flask",
                    "lastStatus": "PENDING"
                }
            ],
            "version": 1,
            "createdAt": 1543441148.711,
            "group": "family:intervention-ninja-flask"
        }
    ],
    "failures": []
}
```

Now let's go back to AWS console and let's check our cluster - we should have 1 instance running with 1 running task.

![AWS ECS cluster running task](images/004/ecs_cluster_task.png)

Let's get into our cluster detail and open the tab **ECS instances**.
Then click on the name of the only running container instance. 
You should see a page with the detail of container instance.

Copy the **Public DNS** from here and open new browser window with it - in my case it's *http://ec2-34-238-118-24.compute-1.amazonaws.com* (the link won't work - i've shut it down, sorry:).

![AWS ECS running flask](images/004/ecs_running_flask.png)

That's it for now! And as always, If you have any question, drop me an email at **pkrayzel (at) gmail.com**