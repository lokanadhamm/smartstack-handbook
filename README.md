The Smartstack Handbook
=======================

This work is licenced under Creative Commons 4.0 Attribution Licence
and Apache 2.0 for the software parts.

Copyright 2014 Patrick Viet

Please read the [LICENCE](https://github.com/patrickviet/smartstack-handbook/blob/master/LICENCE) file first.

- [LICENCE_DOCS.txt](https://github.com/patrickviet/smartstack-handbook/blob/master/LICENCE_DOCS.txt) for the text
- [LICENCE_SOFTWARE.txt](https://github.com/patrickviet/smartstack-handbook/blob/master/LICENCE_SOFTWARE.txt) for the code

What's in here
--------------

This will explain:

- How to test Smartstack with Vagrant (on a local virtual machine)
- How to setup Smarstack with a lot of real life - sometimes complex - examples
- Smarstack day to day operations, in a real life production environment

Some things will be totally obvious, but nonetheless useful.

When it's 2am, you're on call and get paged, it's easier to just follow the steps without having to think too much. I do realize that you most likely do know how to start/stop a runit service, where the logs are, or how to establish a SSH tunnel, but hey you might blank out under stress. Anyway, I do.

Smartstack requirements
-----------------------

Smartstack is a combination of a few very common tools, with two daemons to control them (Nerve, Synapse)

The full stack uses:

- A registration service: [Serf](http://www.serfdom.io/) or [ZooKeeper](http://zookeeper.apache.org/)
- Willy Tarreau's [HAProxy](http://haproxy.1wt.eu)
- Ruby 1.9.3, for the Smartstack daemons, that can run with Ruby 1.9.3 - or the 1.9 compatible JRuby
- A UNIX-like operating system supported by HAProxy: Linux, Solaris, FreeBSD, OpenBSD
- Chef for configuration - not mandatory but very very very recommended. Chef-solo, without a central server, is sufficient.

The most common OS choice is Linux Ubuntu 12.04 LTS

But with a little adaptation of the Chef cookbooks (software package names mostly), you'll also be totally fine on RedHat, FreeBSD, etc, it's not like installing Ruby 1.9.3 and HAProxy is rocket science.

Architecture
------------

### Base architecture
TODO: add pretty drawing

To better understand Smartstack, I'll describe a before/after. In this example I'll take a Java app that consumes ElasticSearch.

Before:
- User connects to Java app
- Java app does some processing. It's a search. It needs to query ElasticSearch
- Java app connects directly to the ElasticSearch server, via an IP address known via a DNS or Hosts entry
- ElasticSearch processes the request, and we go all back up the stack

The problem here is that if the IP address of the hosts file change, we need to restart the Java app, and while Java is very fast to run, it's not exactly fast to startup.

Sometimes, we will use a load balancer between the app and the backend. That's nice but means dedicated hardware, more management, more risks of failutre, (or in the case of am Amazon architecture, ELB crashes at the same time as EBS - not cool! - not to mention the extra costs, latency, etc)

After
- User connects to Java app
- Java app must connect to ElasticSearch
- __Java app connects to a localhost:someport HAProxy software!__ - which is dynamically configured by smartstack
- HAProxy connects to ElasticSearch
- Request goes back up the stack

Result
- HAProxy is very fast so no visible latency or performance loss is introduced
- Any reconfiguration is transparent for the app
- The app doesn't need to know about this discovery system: just connect to some local port as if it was the final service
- No extra hardware

Here are the elements of Smartstack:
- Nerve probes for services, and announces them to some registry (ZooKeeper or Serf)
- Synapse queries the registry to generate a HAProxy configuration
- HAProxy runs, on each node, making locally accessible some remote services
- ZooKeeper or Serf runs as a registry

### Nerve architecture details

TODO: add pretty drawing

Nerve is the daemon that watches the service and announces it (or stops announcing it) to the registry. It must run on the nodes that run a service that others nodes will want to consume. It's written in Ruby. The default installation, with Chef, puts it in /opt/smartstack/src, with its configuration in /opt/smartstack/config.json.

It's basically a loop that runs the watchers you configure. If the watchers (ie. a TCP probe to localhost, or an HTTP probe, etc) return OK, then it will in turn register that this service is OK for the node it's running on. With ZooKeeper, it's a persistent TCP connection to one of the machines of the ZooKeeper cluster, with Serf it's a tag set in the file /etc/serf/zzz_nerve_nameofservice_80.json.

And... That's all there is to it.

### Synapse architecture details

Synapse is Nerve's brother, it queries the registry (or other sources!) to know the state of a service and configure its local HAProxy accordingly. You must, for each service, specify:
- How it discovers the service (ZooKeeper, Serf, DNS, etc)
- The HAProxy configuration variables it will take from this service (see examples)
- Various timing/timeout variables

Just like Nerve, it's a loop, that runs through the _service_watchers_, and if something changes, will reconfigure the HAProxy. It has a few nice transparent features, such as, if a service comes up/down it will not restart HAProxy but send via UNIX socket an activate/deactivate command.

TODO
- Synapse plugins (DNS etc)
- The Serf port/plugin
- Operations with ZooKeeper
- Full configuration options/description
- The local DNS/hosts setup (useful for NewRelic)
- Health checks

Installation
------------

TODO

- Vagrant test environment
- With Serf
- With ZooKeeper
- Your first own service
- DRY (Don't Repeat Yourself): generate a config file that connects to the right port

Cookbooks - some real life typical setups
-----------------------------------------

TODO

- SQL Master
- SQL Slave - simple
- SQL Slave - advanced, with a replication monitor daemon
- Amazon RDS
- RabbitMQ (unique server)
- RabbitMQ (multiple servers, no cluster)
- Web load balancer - vanilla
- Web load balancer - with sticky sessions
- Web load balancer - with SSL and sticky sessions
- ElasticSearch
- Without HAProxy: memcached and circular hashing

### Configure your service: generate the file in Chef

#### Abstract

_DRY_ = that means Don't Repeat Yourself.

You have already reserved a port for a name in ```cookbooks/smartstack/attributes/ports.rb```.
You don't want to be hardcoding it anywhere else!

Provided you include the smartstack cookbook as a dependency in your cookbook meatadata, it is accessible via ```node.smartstack.service_ports['yourservicename']```

Most services can read a configuration file to know where to connect. You want to generate the configuration file with Chef, and trigger a restart.

#### Real life example : a daemon called _pusher_ accesses Rabbitmq

Original cookbook: direct access

```ruby
#cookbooks/pusher/attributes/default.rb
node.pusher.rmq_host = 'myrabbitserver'
node.pusher.rmq_host = 5672
...
```

cookbooks/pusher/recipes/default.rb
```ruby
[... stuff that installs pusher ...]

service 'pusher' do
  supports :reload => true
end

template '/opt/pusher/etc/pusher.conf' do
  user 'pusher'
  group 'pusher'
  mode '0600'
  notifies :reload, 'service[pusher]'
end

```

cookbooks/pusher/



I had this service that accessed RabbitMQ, let's call it _pusher_.

It is accessible via the 

The general pattern is:
- Configure your service in Chef, reserving a port

BONUS! Going a bit further with Serf/Chef
- Munin setup
- ElasticSearch cluster: don't reconfigure right away if it's down
- Firewall


Operations
----------

TODO

- Put a server in temporary maintenance
- Put a server in long term maintenance
- Take a server out of maintenance
- Force-restart everything (!)
- Full stack diagnostic walkthrough
- Create a new service
- Add a service to a server
- Add a server to a service
- Instant config switch (with Serf)
- If you don't have a VPN: SSH tunnel to diagnose

Author
------

I'm Patrick. I used to work at Airbnb as a systems engineer, and I used Smartstack a lot.
Since it's opensource, I have ported Smartstack to use Serf instead of ZooKeeper.
I have setup Smartstack in production with the Serf port at [GetYourGuide AG](http://www.getyourguide.com/)
