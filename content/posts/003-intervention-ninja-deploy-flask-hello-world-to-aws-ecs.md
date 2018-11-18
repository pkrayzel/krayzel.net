+++
draft = true
date = 2018-11-18T18:21:39+01:00
title = "Intervention Ninja - deploy flask hello-world to AWS ECS (#02)"
slug = ""
tags = ["intervention-ninja", "aws"]
categories = []
thumbnail = "images/tn.png"
description = ""
+++

In the [previous post](posts/002-intervention-ninja-flask-app-running-on-aws-ecs), I've shared what we're going to build and why.
Today, we're gonna finally do some real work - we'll build simple flask hello-world application and we'll deploy it into AWS. 

**Let's get started**

I'll start with building a docker image for our flask application. Let's write a Dockerfile for it. I expect you to have some experience with docker, if you don't - I recommend you to check <a href="https://docker-curriculum.com/" target="_blank">this website</a>.

{{< highlight go "linenos=table,linenostart=1" >}}
FROM library/python:alpine3.7

RUN pip install flask

ADD ./src /opt/intervention-ninja
WORKDIR /opt/intervention-ninja

EXPOSE 5000
CMD ["python", "app.py"]
{{< / highlight >}}

Nothing really fancy here, we're using alpine version of Linux with python 3 (line 1). 
We're installing flask on it using <a href="https://pypi.org/project/pip/" target="_blank">pip</a> (line 3) 
and copying source files from *src* directory (line 5).
Line 6 is setting work directory to */opt/intervention-ninja* - that's where we've just copied source code of our application to.
Then we can run *python app.py* from that directory (line 9).

There is one more file required for this to work - *src/app.py* - and this is it's content:

{{< highlight go "linenos=table,linenostart=1" >}}
from flask import Flask
app = Flask(__name__)


@app.route('/')
def hello_world():
    return 'Hello, World!'


if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000)
{{< / highlight >}}

So the structure of this *"project"* should look like this:

```
- Dockerfile
- src
  - app.py
```

Great! Now let's build a docker image - run following from your terminal.

```
docker build -t intervention-ninja-flask .
```

Docker daemon should execute all the steps from Dockerfile and last two lines in your terminal 
should look similar to this (the *image id* will be different):

```
Successfully built efcd9aeea72a
Successfully tagged intervention-ninja-flask:latest
```

**Running the service**

We'll run our new docker image, expose it on our host (localhost) on port 5000 and we'll run it as daemon (so we can still use our terminal).

```
docker run -p 5000:5000 -d intervention-ninja-flask
```

Now let's check whether our docker container is running:

```
docker ps
```

should give you a following input

``` 
CONTAINER ID        IMAGE                      COMMAND             CREATED             STATUS              PORTS                    NAMES
319721e320d5        intervention-ninja-flask   "python app.py"     5 seconds ago       Up 2 seconds        0.0.0.0:5000->5000/tcp   sleepy_perlman
```

That looks promising. Now open your browser on url <a href="http://localhost:5000" target="_blank">http://localhost:5000</a> and you should see:

```
Hello, World!
```

Ok, coding part is done for today. Now it's time to ship this incredibly useful application to AWS. 

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

