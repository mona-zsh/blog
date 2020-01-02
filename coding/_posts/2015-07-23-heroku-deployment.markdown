---
layout: post
title:  "Deploying a Django App on Heroku: Interprocess Communication with APScheduler and Redis"
date:   2015-07-23
categories: coding
tags: python django heroku git
root: "../"
---

This is a story about git pushing to a Heroku app without killing all your processes.

(There is no happy ending.)

<!--more-->

<hr><br />

Let's say you have a Django app that you want to deploy to Heroku.

Heroku makes it easy to push code from a Git repository. You specify process types in a Procfile, set a remote to your Heroku app, and run `git push <app name> <local branch>:<remote branch>`. Heroku runs each process on a separate `dyno`, or "lightweight Linux container that runs a single user-specified command", which means Heroku will deploy your processes on separate virtual machines.

This is all well and good -- that is, until on deployment, you find this in your logs:

>heroku/clock.1: Stopping all processes with SIGTERM
>
>heroku/web.1: Stopping all processes with SIGTERM
>
>heroku/regworkers.1: Stopping all processes with SIGTERM

It shouldn't be all that surprising that Heroku spins down your dynos when deploying new code. After all, processes need to restart before changes can be made. Heroku [documentation]() is very clear about this:

> When the dyno manager restarts a dyno, the dyno manager will request that your processes shut down gracefully by sending them a SIGTERM signal...
> ...application processes have ten seconds to shut down cleanly.

At the time that we discovered these SIGTERMs in our own Heroku application, our clock process was a poorly constructed singleton instance of Advanced Python Scheduler, or [APScheduler](https://apscheduler.readthedocs.org/en/latest/), that took on the burden of executing as well as scheduling jobs. If we happened to push while our clock was scheduling and finishing up a job, it was quite possible that the job would never finish (since then, we have rallied our workers to do the job).

APScheduler is pretty nifty and "shuts down its job stores and executors and waits until all currently executing jobs are finished" by default.
This is super awesome. Shutting down cleanly is a matter of handling a SIGTERM in your clock process and call `scheduler.shutdown()`. 

But if you happened to have written APScheduler incorrectly, `scheduler.shutdown(wait=True)` can't withstand the 10 second countdown before a SIGKILL is sent by Heroku. Your longstanding job will be *finished*, as in K.O'd, and you will have an unhappy user who did not receive a scheduled service from your application.

So instead of having `git push` spin down all the dynos, it might be nice if we could ask the scheduler to shutdown _and then_ push.

**Attempt #1: a one-off management command and a global variable**

This attempt was rather embarrassing, and I only include this lesson in this story to help others never experience the pain of deeply misunderstanding how code works when run from different processes.

Our clock process was specified in our Procfile as `clock: python manage.py clock`. In a naive first implementation of shutting down the APScheduler instance in our clock process, I tried to make the scheduler in `clock.py` a globally defined variable. 

{% highlight python %}
app/clock.py

scheduler = APScheduler()
scheduler.start()
scheduler.schedule_job(# some job)
{% endhighlight %}

I then tried use another one-off command, `heroku run python manage.py shutdown_clock`, to shut it down. It looked a little like this:

{% highlight python %}
from app.clock import scheduler
scheduler.shutdown()
{% endhighlight %}

After some consternation, I printed out the memory addresses of these schedulers and realized that __these schedulers are completely separate instances on completely separate processes on completely separate dynos__. I learned that a global variable in one process on one dyno cannot be accessed by another process on another dyno. Another dyno that imports from the module will make a new instance of whatever you have, and two schedulers is the last thing you want in a live application.

**Attempt #2: Writing to a file and reading from a file**

This attempt never got off the ground. Again, with Heroku, you are on **two different dynos** and do not have access to the same filesystem.

**Attempt #3: [XML Remote Procedural Call](https://docs.python.org/3.4/library/xmlrpc.client.html?highlight=rpc)**

What we really want to do is wait until the scheduler finishes executing or scheduling its jobs, and then push when the coast is clear. That sounds like sending a request to the scheduler and receiving a response to proceed.

XMLRPC is a Python library that implements a lightweight IPC mechanism to pass XML via HTTP as transport. Our client (`shutdown_clock`) can call methods with parameters on our server (`clock.py`), which is named by a URL. 

Here's where this line of inqury ends. Heroku does not provide a URL for the clock process. Only web processes are assigned ports to receive requests from the outside world.

**4. Database as IPC**

So if we can't have direct interprocess communication, maybe we can have a middle man. 

Our Django application was using a PostgreSQL database, which in our desperation, we thought could serve as an intermediary to communicate between the clock process and our local machines. As you can imagine, a `clock.py` that makes a database query every 3 seconds puts an unnecessary load onto our database.

This is a known [antipattern](https://en.wikipedia.org/wiki/Database-as-IPC), meaning you should probably not do it even if everyone else is doing it.

> In computer programming, Database-as-IPC is an anti-pattern where a database is used as the message queue for routine interprocess communication in a situation where a lightweight IPC mechanism such as sockets would be more suitable. Using a database for this kind of message passing is extremely inefficient compared to other IPC methods and often introduces serious long-term maintenance issues, but this method enjoys a measure of popularity because the database operations are more widely understood than 'proper' IPC mechanisms.

Finally, we settled on a solution. 

(And yes, once again, we don't have a happy ending to this story.)

**5. Redis Queue as an Intermediary**

Using a Redis Queue as a message queue is really the same thing as using a database as a message queue.

But it works.

Sure, Redis is not lightweight like sockets or HTTP. We are still connecting to a Redis server and attempting to pop messages from the queue every 3 seconds. We like to think that an open Redis connection to a special queue is a lesser evil than reading and writing from our production database every 3 seconds. We also like to think that the [existence of Python libraries for Redis-as-RPC](https://pypi.python.org/pypi/redisrpc) is a sign of validation for this solution (though really, we know better... ).

Our server and client look something like this:

{% highlight python %}
class RedisServer(object):
   def listen(self):
       keep_listening = True
       while keep_listening:
           time.sleep(3)
           _, byte_values = self.conn.brpop("request")
           received = byte_values.decode("utf-8")
           self.handle_message(received) # Here, we might have a mapping of message to function call. If `received == "shutdown"`, we call `shutdown_clock`.
{% endhighlight %}


{% highlight python %}
class RedisClient(object):
   def send_request(self, message):
       self.push(message)
       self.listen() # Client implements a similar listen function that returns when server responds 

   def push(self, message):
       self.conn.lpush("request", message)
{% endhighlight %}

And our deployment script looks something like this:

{% highlight bash %}
heroku run python manage.py shutdown_clock
git push app master:master
{% endhighlight %}

When shutdown_clock finishes executing and returns with process code 0, we know that the clock has successfully executed its jobs and stored the unfinished ones, and that it is safe to push, destroy, and deploy.

<hr><br/>

This is the unhappy end of our story about Heroku interprocess communication, but maybe you have an alternative ending. Have you encountered anything like this, and what was your solution? [Let me know!](http://twitter.com/hazelynut)

Thanks for reading.

## The end (?)

