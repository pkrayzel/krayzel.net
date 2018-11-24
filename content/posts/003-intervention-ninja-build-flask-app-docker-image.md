+++
draft = false
date = 2018-11-18T18:21:39+01:00
title = "Intervention Ninja - build flask app docker image (#02)"
slug = ""
tags = ["intervention-ninja", "python", "docker", "aws"]
categories = []
thumbnail = "images/thumbnail.png"
description = ""
+++

In the [previous post](posts/002-intervention-ninja-flask-app-running-on-aws-ecs), I've shared what we're going to build and why.
Today, we're gonna finally do some real work - we'll build flask application and we'll run it in docker. 

**Let's get started**

I'll start with building a docker image for our flask application. Let's write a Dockerfile for it. I expect you to have some experience with docker, if you don't - I recommend you to check <a href="https://docker-curriculum.com/" target="_blank">this website</a>.

{{< highlight go "linenos=table,linenostart=1" >}}
FROM library/python:alpine3.7

RUN pip install Flask==0.12.2

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

The main file for our flask application is *src/app.py* - I'll put only the most important pieces here and the rest will be available at the end of this post.

{{< highlight go "linenos=table,linenostart=1" >}}
@app.route("/")
def home():
    return render_template("index.html")

@app.route("/send", methods=["POST"])
def send_email():
    logging.info("Pretending to sent email...")
    return render_template("sent.html")
{{< / highlight >}}

So far our application have two endpoints - **the root** endpoint renders Jinja2 template file **index.html**.
The **/send** endpoint accepts POST request, sends an email (not really) and then renders Jinja2 template file **sent.html**. 

{{< highlight go "linenos=table,linenostart=1" >}}
@app.route("/_health")
def health_check():
    return make_response(
        jsonify({
            "version": os.environ.get("VERSION", "N/A"),
            "status": "AVAILABLE"
        }),
        200
    )
{{< / highlight >}}

The last section defines **_health** endpoint, which has one purpose - it should tell us 
(or to the load balancer) whether this container can be shown to our customers or not.
In real life, this endpoint should return status based on the current state of the app, checking also it's 
dependencies etc.

It's also a good practice to return version of the application from here, as it's very useful for debugging. 

I won't show the rest of the files here as they're not really interesting - it's just a static content and bootstrap template. 
Feel free to check all of them in the branch <a href="https://github.com/pkrayzel/intervention.ninja/tree/flask-ec2" target="_blank">flask-ec2</a>. 
If you're not familiar with Jinja2 templates or macros, check <a href="http://jinja.pocoo.org/docs/2.10/" target="_blank">the docs</a> first. 
  
The full structure of our project now looks like this:

![Intervention Ninja flask app project structure](images/in_flask_structure.png)

Great! Now let's build a docker image - run following command from your terminal.

```
docker build -t intervention-ninja .
```

Docker daemon should execute all the steps from Dockerfile and last two lines in your terminal 
should look similar to this (the *image id* will be different):

```
Successfully built 4549d6651033
Successfully tagged intervention-ninja:latest
```

**Running the service**

We'll run our new docker image, expose it on our host (localhost) on port 5000 and we'll run it as daemon (so we can still use our terminal).

```
docker run -p 5000:5000 -d intervention-ninja
```

Now let's check whether our docker container is running:

```
docker ps
```

should give you a following input

``` 
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                    NAMES
ba8b6ddb34c6        intervention-ninja   "python app.py"     3 seconds ago       Up 1 second         0.0.0.0:5000->5000/tcp   priceless_darwin
```

That looks promising. Now open your browser on url <a href="http://localhost:5000" target="_blank">http://localhost:5000</a> and you should see:

![Intervention Ninja Flask - homepage](images/in_flask_index.png)

Let's put some email address into the form and let's click on the **Send** button. You should now be on the second page:

![Intervention Ninja Flask - homepage](images/in_flask_send.png)

Ok, that was fun! Of course we're not sending real emails - to be honest - i've removed this part from the code for the purpose of this blog post.
Next task will be to ship this application to AWS ECS. Stay tuned!
