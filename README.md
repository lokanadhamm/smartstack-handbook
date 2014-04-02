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
- Ruby 1.9.3, for the Smartstack daemons, that run with Ruby 1.9.3 - or JRuby
- A UNIX-like operating system supported by HAProxy: Linux, Solaris, FreeBSD, OpenBSD
- Chef for configuration - not mandatory but very very very recommended. Chef-solo, without a central server, is sufficient.

The most common choice is Linux Ubuntu 12.04 LTS

But with a little adaptation of the Chef cookbooks (software package names mostly), you'll also be totally fine on RedHat, FreeBSD, etc, it's not like installing Ruby 1.9.3 and HAProxy is rocket science.

Architecture
------------

TODO

- Nerve architecture details
- Synapse architecture details
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
- Memcached

BONUS! Going a bit further with Serf/Chef
- Munin setup
- ElasticSearch cluster
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
