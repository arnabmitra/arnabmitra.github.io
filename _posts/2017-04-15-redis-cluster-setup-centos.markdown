---
layout: post
title:  "Setting up a Redis instance."
date:   2017-04-15 20:52:29 -0700
categories: jekyll update
---

>Redis is an open source, BSD-licensed, key-value data store that also comes with a messaging system. The server is freely available at http://redis.io/download.

If you use a Mac with homebrew

>brew install redis
Once you unpack Redis, you can launch it with default settings.
redis-server
You should see a message like this:
[35142] 01 May 14:36:28.939 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
[35142] 01 May 14:36:28.940 * Max number of open files set to 10032
                _._
              _.-``__ ''-._
        _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 64 bit
    .-`` .-```.  ```\/    _.,_ ''-._
  (    '      ,       .-`  | `,    )     Running in stand alone mode
  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
  |    `-._   `._    /     _.-'    |     PID: 35142
    `-._    `-._  `-./  _.-'    _.-'
  |`-._`-._    `-.__.-'    _.-'_.-'|
  |    `-._`-._        _.-'_.-'    |           http://redis.io
    `-._    `-._`-.__.-'_.-'    _.-'
  |`-._`-._    `-.__.-'    _.-'_.-'|
  |    `-._`-._        _.-'_.-'    |
    `-._    `-._`-.__.-'_.-'    _.-'
        `-._    `-.__.-'    _.-'
            `-._        _.-'
                `-.__.-'

[35142] 01 May 14:36:28.941 # Server started, Redis version 2.6.12
[35142] 01 May 14:36:28.941 * The server is now ready to accept connections on port 6379

If you want to install it on CentOs or RHEL systems.
wget http://download.redis.io/releases/redis-3.2.6.tar.gz
yum install make gcc gcc-c++ kernel-devel
sudo wget http://prdownloads.sourceforge.net/tcl/tcl8.6.0-src.tar.gz
sudo tar xzvf tcl8.6.0-src.tar.gz
cd tcl8.6.0/unix
sudo ./configure
sudo make
sudo make install
cd ~
sudo tar xzvf redis-3.2.6.tar.gz
sudo cp -p redis-3.2.6/utils/redis_init_script /etc/init.d/redis
sudo mkdir /etc/redis
sudo cp -p redis.conf /etc/redis/6379.conf
sudo vi /etc/redis/6379.conf

@@ -14,11 +14,11 @@

 # By default Redis does not run as a daemon. Use 'yes' if you need it.

 # Note that Redis will write a pid file in /var/run/redis.pid when daemonized.

-daemonize no

+daemonize yes



 # When running daemonized, Redis writes a pid file in /var/run/redis.pid by

 # default. You can specify a custom pid file location here.

-pidfile /var/run/redis.pid

+pidfile /var/run/redis_6379.pid



 # Accept connections on the specified port, default is 6379.

 # If port 0 is specified Redis will not listen on a TCP socket.

@@ -61,12 +61,12 @@

 # verbose (many rarely useful info, but not a mess like the debug level)

 # notice (moderately verbose, what you want in production probably)

 # warning (only very important / critical messages are logged)

-loglevel notice

+loglevel debug



 # Specify the log file name. Also 'stdout' can be used to force

 # Redis to log on the standard output. Note that if you use standard

 # output for logging but daemonize, logs will be sent to /dev/null

-logfile stdout

+logfile /var/log/redis.log



 # To enable logging to the system logger, just set 'syslog-enabled' to yes,

 # and optionally update the other syslog parameters to suit your needs.

@@ -150,7 +150,7 @@

 # The Append Only File will also be created inside this directory.

 #

 # Note that you must specify a directory here, not a file name.

-dir ./

+dir /usr/local/redis/


If you want your redis server to accept connections from instances other than local host
Do the following (**********Don't do this in PROD**************)

################################## NETWORK #####################################



# By default, if no "bind" configuration directive is specified, Redis listens

# for connections from all the network interfaces available on the server.

# It is possible to listen to just one or multiple selected interfaces using

# the "bind" configuration directive, followed by one or more IP addresses.

#

# Examples:

#

# bind 192.168.1.100 10.0.0.1

# bind 127.0.0.1 ::1

#

# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the

# internet, binding to all the interfaces is dangerous and will expose the

# instance to everybody on the internet. So by default we uncomment the

# following bind directive, that will force Redis to listen only into

# the IPv4 lookback interface address (this means Redis will be able to

# accept connections only from clients running into the same computer it

# is running).

#

# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES

# JUST COMMENT THE FOLLOWING LINE.

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#bind 127.0.0.1
# Protected mode is a layer of security protection, in order to avoid that

# Redis instances left open on the internet are accessed and exploited.

#

# When protected mode is on and if:

#

# 1) The server is not binding explicitly to a set of addresses using the

#    "bind" directive.

# 2) No password is configured.

#

# The server only accepts connections from clients connecting from the

# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain

# sockets.

#

# By default protected mode is enabled. You should disable it only if

# you are sure you want clients from other hosts to connect to Redis

# even if no authentication is configured, nor a specific set of interfaces

# are explicitly listed using the "bind" directive.

protected-mode no

---------------------------------------------------------------------------
sudo mkdir /usr/local/redis
sudo /etc/init.d/redis start

sudo vi /etc/init.d/redis

@@ -10,6 +10,10 @@

+# chkconfig:   - 85 15

+# description:  redis-server

+# processname: redis

--------------------------------------------------------------------------
sudo /sbin/chkconfig --add redis
sudo /sbin/chkconfig redis on
sudo /sbin/chkconfig --list | grep redis


Now you can start and stop your redis service via

service redis stop

service redis start
