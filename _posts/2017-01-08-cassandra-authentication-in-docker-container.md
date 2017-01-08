---
layout: post
date: 2017-01-08
title: Turn on Cassandra Authentication in Docker Container
comments: true
---
I've recently been working on a project that uses a Cassandra database
running in a Docker container, with [this image](https://hub.docker.com/_/cassandra/).
It works great, but it does not have authentication enabled by default. This is
because unless you specify otherwise, the `authenticator` is set to 
`AllowAllAuthenticator`.

To enable authentication, just add this line to your `Dockerfile`:

~~~
# Require user & pass for accessing Cassandra instance within container
RUN echo "authenticator: PasswordAuthenticator" >> /etc/cassandra/cassandra.yaml
~~~

All this does is add a line to the [cassandra.yaml](https://docs.datastax.com/en/cassandra/2.1/cassandra/configuration/configCassandra_yaml_r.html)
file (the main config file for Cassandra) indicating that you want Cassandra to
require a username and password for authentication.

Now, when you start up your Cassandra Docker container, you'll be required to
specify a username and password to access the database. The default login's
username is `cassandra`, and the default password is also `cassandra`. So, if
you want to start a `cqlsh` session, you'll have to execute (after starting a 
shell in the container, of course - see 
[this](http://dev-smart.com/docker-cheetsheet/), unless you've mapped the
container to a host port):

~~~
cqlsh -u cassandra -p cassandra
~~~

or to execute a single query:

~~~
cqlsh -u cassandra -p cassandra --execute=INSERT_CQL_QUERY_HERE
~~~

## Extra Steps 
If you have enabled password authentication, you may also want to consider 
following the procedure outlined [here](https://docs.datastax.com/en/cassandra/2.1/cassandra/security/security_config_native_authenticate_t.html):

* Increase the replication factor of your `system_auth` keyspace to avoid getting
locked out if your lone replica node goes down.
* Create a new superuser to replace the default `cassandra` one.
* Use the new superuser to demote the default one to `NOSUPERUSER` status and
change its password from the default `cassandra` to something more secure.

### References
[https://docs.datastax.com/en/cassandra/2.1/cassandra/security/security_config_native_authenticate_t.html](https://docs.datastax.com/en/cassandra/2.1/cassandra/security/security_config_native_authenticate_t.html)          
[http://sempike.blogspot.com/2016/11/docker-set-up-cassandra-container-with.html](http://sempike.blogspot.com/2016/11/docker-set-up-cassandra-container-with.html)
