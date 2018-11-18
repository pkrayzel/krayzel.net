+++
draft = true
date = 2018-11-18T18:21:39+01:00
title = "Intervention Ninja - build flask hello-world docker image (#02)"
slug = ""
tags = ["intervention-ninja", "python", "docker", "aws"]
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

Ok, coding part is done. Next task will be to ship this incredibly useful application to AWS. Stay tuned!
