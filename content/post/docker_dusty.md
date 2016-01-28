---
title: "Get your sh*t together: Docker development environments with Dusty"
date: "2016-01-28"
description: "This article will give you an overview how you can use Docker and Dusty to streamline your dev environments."
---

I've seen hell, when it comes to software development and actual production environments. Inconsitent platforms, packages and a plethora of hacks to 'get everything to work' on all developer machines. I've seen whole teams screwing with a shared 'dev database' on a root server somewhere, creating a horrible mess of inconsistent behavior and ultimately slow down development and testing. This article will give you an overview how you can use Docker and Dusty to streamline your dev environments.

In production, we get a similar picture in many companies. A bunch of servers, running somewhere and a whole lot of custom bash scripts doing stuff no one really wants to know. Luckily we live in a world now where everyone has heard of Docker. With docker you can containerize everything and mix and match whatever you need.

The purpose of this post is not to give you an intro to Docker and Dusty, but to basically present you an example dev environment with some useful addons that help you with logging, monitoring and config management. If you don't know about Dusty yet, read their excellent docs at https://dusty.readthedocs.org.

## Developing with Docker

Developing with Docker has always been a bit tricky. Sure, there's Docker Compose and you can mount your code from your disk into your container. But it always seemed a bit messy and since there wasn't any clean structure around developing in containerized environments it tended to get messy again.

Fortunately, the great folks at GameChanger came up with Dusty. A neat little tools that makes working with dockerized dev environments fun fun fun. Dusty allows you to configure your whole environment with yaml files (basically like docker compose) but adds useful components and structure to the whole thing. Especially mixing exactly the components you need and decide which code you want to mount into a containers is dead simple with Dusty.

## Get Started with Dusty

Follow the Dusty install docs and set everything up. Once you've installed Dusty and all its requirements, run `$ dusty setup` and follow the prompts. If you want to used my example repository, enter `github.com/marianzange/dusty-boilerplate` as specs repo.

If you now run `$ dusty repos list` you should see something like this:

```
+----------------------------------------------------+-------------------------+----------------+
|                     Full Name                      |        Short Name       | Local Override |
+----------------------------------------------------+-------------------------+----------------+
|      github.com/marianzange/dusty-boilerplate      |    dusty-boilerplate    |                |
| github.com/marianzange/dusty-boilerplate-flask.git | dusty-boilerplate-flask |                |
+----------------------------------------------------+-------------------------+----------------+
```

The first repo contains our specs and the second repo our actual app. Now run `$ dusty bundles list` to see the available bundles.
If you want to run everything, activate them with `$ dusty bundles activate boilerplate` and `$ dusty bundles activate devtools`.

Well, and now it's time to fire up the whole thing! Run `$ dusty up` and watch the magic happen. Please note, that it can take ages for the environment to start up when you run
it for the first time because it will pull all the docker images it needs. Subsequent starts will be super fast though.
If everything is working, you should see something along these lines:

```
[...]
Compiling together the assembled specs
Compiling the port specs
Compiling the nginx config
Creating setup and script bash files
Compiling docker-compose config
Saving port forwarding to hosts file
Configuring NFS
Saving updated nginx config to the VM
Saving Docker Compose config and starting all containers
Warning: --allow-insecure-ssl is deprecated and has no effect.
It will be removed in a future version of Compose.
Creating dusty_dustyInternalNginx_1...
Creating dusty_cadvisor_1...
Creating dusty_etcd_1...
Creating dusty_etcd-browser_1...
Creating dusty_logio_1...
Creating dusty_logio-harvester_1...
Creating dusty_memcached_1...
Creating dusty_postgres_1...
Creating dusty_flask_1...
Your local environment is now started!
```

Now it's time to access your awesome app. Open your browser and go to http://flask.dusty-boilerplate.io.
You should see an awesome website telling you how awesome Docker and Dusty are. If you're now wondering
why you can access your local container via dusty-boilerplate.io, it's simple: Dusty runs and nginx
proxy that automatically proxies all your requests to the correct container. This way you can use
vanity URLs for local development. I would recommend using a standard compliant domain that's not registered by anyone
to avoid browser autocorrect etc.

## Specs Structure

To work with Dusty, you need to create a specs repo. Specs are a simple set of yml files specifying how your environment looks like. My example specs repo (https://github.com/marianzange/dusty-boilerplate) has the following structure:

```
.
├── LICENSE
├── apps
│   ├── cadvisor.yml
│   ├── etcd-browser.yml
│   ├── flask.yml
│   ├── logio-harvester.yml
│   └── logio.yml
├── bundles
│   ├── boilerplate.yml
│   └── devtools.yml
├── libs
└── services
    ├── etcd.yml
    ├── memcached.yml
    ├── mysql.yml
    ├── postgres.yml
    └── solr.yml
```

In the example, the app we want to work with is called 'flask'. It's a simple Flask hello world example which is hosted here: https://github.com/marianzange/dusty-boilerplate-flask



## Managing Configurations with etcd

There are many different ways to manage configs. One of my favorite ways is etcd. It's lightweight and you can use it locally and in production the same way.
Since we're trying to get a fully isolated mix-n-match development environment, our etcd is contained in our Dusty specs as a service.

Additionally we have an app called etcd-browser which allows you to populate your etcd key-value store with whatever settings you need.
Now we just need our processes to pickup and monitor relavant values from etcd. To do this, I'm using etcdenv (https://github.com/upfluence/etcdenv)
which wraps your process and restarts it automatically if it detects changes to a specific subset of keys on etcd.

If you take a look at my example `apps/flask.yml`, you'll find the following configuration in the 'commands' section:

```
commands:
  once:
    - curl -L https://github.com/upfluence/etcdenv/releases/download/v0.3.1/etcdenv-linux-amd64-0.3.1 > /usr/local/bin/etcdenv
    - chmod +x /usr/local/bin/etcdenv
    - pip install -r requirements.txt
  always:
    # etcdenv wraps the the executed python process and provides the environment
    # variables from etcd. In this example it looks for kv stored under
    # /apps/boilerplate/flask and will restart the py process on any detected change.
    - |
      etcdenv \
      -s http://etcd0:4001 \
      -n /apps/boilerplate/flask \
      -b restart \
      python app.py
```

As you can see, we get a built of etcdenv and write it to the container. Since this is a specific strategy for my development
enviroment, it wouldn't make sense to embed it in the docker app container. The main command for my container is etcdenv which wraps my app.py.
Now, whenever I change anything on etcd in the `/apps/boilerplate/flask` subset, etcdenv will automatically restart the wrapped process.

The nice parts is, you can use a fairly similar strategy to configure your production containers. So you can get pretty close to having
the same configuration flow both on production and in development.

To add and change values in your etcd, my example repo comes with etcd-browser. Open http://etcd.dusty-boilerplate.io in your browser and enter `http://192.168.99.100:4001` in
the connect	field. Now this is not very elegant to be honest. I couldn't find the time to improve it though, so coming up with a better way for editing your etcd data is left as an exercise to the reader.

All etcd data is persisted, so no need to worry when restarting your etcd container.

## Logging

If you develop an application, you obviously want to have a realtime stream of relavant logs always open. There are multiple ways to do so with Dusty. Either your run `$ dusty logs app_name` or you can aggregate your docker logs with log.io or another tool.

My boilerplate repo contains a 'logio' and a 'logio-server' app. The first one is a harvester, collecting everything it gets from Docker and the second one is the actual log.io server. If your environment with these apps is up and running, simply head to http://logs.dusty-boilerplate.io and you'll see a beautiful real-time aggregation of logs from your containers.

![log.io stream](/images/posts/docker-logio.jpg)

## Monitoring

There are many moving parts in a development environment and sometimes you're wondering why your CPU is suddenly feeling the heat.
To get some quick insights into what going on with your containers, I've added Google CAdvisor (https://github.com/google/cadvisor) to the toolbox. CAdvisor
shows you real-time stats about your docker containers and makes it easy to identify high level problems in case a containers goes haywire.

Just open up http://cadvisor.dusty-boilerplate.io and you'll see some pretty graphs:

![CAdvisor](/images/posts/cadvisor.jpg)

I hope you found this article helpful. As I've mentioned earlier, it was not my intention to give an intro to Docker and Dusty,
but present some opinionated aspects and additional tools that can help you when working with Docker and Dusty.