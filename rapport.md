# Report for Lab 3 of AIT
### Authors : Mickael Bonjour & Miguel Gouveia

## Introduction
This lab will help us understand better some of the strategies of load balancing proposed by HAProxy. We will pass threw some options used to specify some behaviour in same balancing strategies.

## Task 1

### 1.
We can see that the proxy handles requests like this :
It will address the requests on each server sequentially. It means that one time it will be sent to one server and the next request coming will go threw the other server.
The NODESESSID is changing at each request which is not so surpirsing because each time we visit the site or actualize we have an ID which is invalid for the server requested (because the precendent response came from the other server). 
### 2. 
We expect the load balancer to address requests coming in with a session ID to the server who knows this session (so the server who created the session). If not, the session management can't be done.
### 3.
Here is our sequence diagram for task 1 (roundrobin). This sequence diagram shows 2 requests by the same user, we see that the behaviour is not correct because both requests don't go on the same server as expected.

![Sequence Diagram 1](./pictures/seqDiagramRound1.png)

That's because of the roundrobin strategy seen on the config file. This strategy consists to give a request to one server and the next to the next server. So in this case where it's a stateful ap (session dependant) it's anightmare because each time we mak a request we can go to another server than the previous request so our cookie will be reset (that's because the current server doesn't know the sessions of the previous server).

### 4.
Here is the test in JMeter, we can see that assertions are false because the counters doesn't increment, in fact th server don't know the users coming because of balancing strategy used here.

![JMeter results](./pictures/jmeterFirst.jpg)

### 5.
Here we see the sequence diagram for this task, we can see that because there's only one server running it responds and knows each user (because he serves all the requests), and so the counter increments correctly.

![Sequence Diagram for one server](./pictures/seqDiagramRound2.png)

Well, this one is a bit obvious, the load balancer detects a node down and doesn't forward any requests to it. So the session management don't have problems, but if there's too many users we will not be able to respond as well as the previous case.

We can see the JMeter report too :

![JMeter results](./pictures/jmeterSecond.jpg)

## Task 2
### 1.
With SERVERID Cookie : 

![SERVERID Diagram](./pictures/seqDiagramRound4.png)

With NODESESSID Cookie : 

![NodesessID diagram](./pictures/seqDiagramRound5.png)

We think that the SERVERID alternative is more effective because the balancer can just evaluate the SEVERID and redirect the request accordingly.
### 2.
We can see the modified configuration at annexes : `haproxy_sticky.cfg`. The modifications are just on the `backend nodes` section. We've just added `cookie SERVERID insert indirect nocache` and the value we want to add on the cookie for each server so :
```
cookie SERVERID insert indirect nocache
server s1 ${WEBAPP_1_IP}:3000 cookie s1 check
server s2 ${WEBAPP_2_IP}:3000 cookie s2 check
```
### 3.
To see if we have achieved the right behaviour we can check like that :
First we visit the site and we see that the load balancer crete the SERVERID cookie and that if we refresh the page we keep the same server :

![sticky session 1](./pictures/sticky1.png)

![sticky session 2](./pictures/sticky2.png)

But if we open an incognito (to remove all cookies) we can see that the roundrobin strategy apply and we have the other server attribued :

![sticky session 3](./pictures/sticky3.png)

And if we refresh this page we keep the same server again because the SERVERID cookie is well set.
### 4.
We can see here the diagram of the situation where 1 browser refresh the page and what happens if another browser opens a connection :

![Sequence Diagram for correct roundrobin](./pictures/seqDiagramRound3.png)

We can see that the SERVERID cookie is set at the HAProxy level, and it's removed at this level too. And because of the roundrobin if another browser connects to the infra it will be directed to s2 if s1 was the last to respond. This cookie is used by the HAProxy to redirect correctly to the right server.
### 5.
Here is the summary report of this task : 

![JMeter report](./pictures/jmeterThird.png)

We can explain the behaviour because how JMeter handles the cookies, because there's only one thread group and so one user. So that's normal that we just have one server who responds (because of the SERVERID Cookie).
### 7.
We can see the result of the manipulations we have done, in fact we added one "user". So the second user when doing his request he's redirected to the other server and all of these future requests will too (because of the cookie).

![Test JMeter](./pictures/jmeterTest.png)
## Task 3

### 1.

![](./pictures/statePageTask3-1.jpg)

![](./pictures/JsonTask3-1.jpg)

We can see in the screenshot above that it is the node "s2" that answers.

### 2.

![](./pictures/statePageTask3-drain.jpg)

We pass the node "s2" on DRAIN mode, it become blue "soft stopped" on the state page.

### 3.

![](./pictures/JsonTask3AfterDrain.jpg)

It's the same node than answer our request. In DRAIN mode, all the new traffic will be redirected to the other nodes, but the current sesssion continue to make request to the node in DRAIN mode.

### 4.

![](./pictures/ChromeJsonTask3-1.png)

In the new browser , we start a new session and we reach the node "s1".

### 5.

![](./pictures/ChromeJsonTask3-2.png)

We clear the cookies on the new browser and we still reach the node "s1". Because "s2" is DRAIN mode, the new session can't reach the "s2". The new browser can only reach "s1".

![](./pictures/JsonTask3AfterChrome.jpg)

We refresh the first browser and it is still on the node "s2". But if we clear the cookie in the first browser, it will not be able to reach "s2" too.

### 6.

![](./pictures/statePageTask3-ready.jpg)

We pass "s2" on READY mode. On the state page, it becomes like at beginning.

It balanced sequentially between the two nodes.

![](./pictures/ChromeJsonTask3-3.png)

We clear the cookies in the new browser and it reach the node "s2". If we clear again, it will change sequentially.

![](./pictures/JsonTask3AfterReady.jpg)

On the first browser, it still on the node "s2".

### 7.

![](./pictures/statePageTask3-maint.jpg)

We pass the node "s2" on MAINT mode, it become brown "down" on the state page.

On MAINT mode, all the traffic is redirected to the other nodes, including existing sessions. The node "s2" can't be reached by request.

![](./pictures/ChromeJsonTask3-4.png)

The new browser reach the node "s1". If we clear the cookies, we still only reach "s1".

![](./pictures/JsonTask3Aftermaint.jpg)

The first browser is not anymore on the node "s2". And has a new session on the node "s1".

## Task 4
### 1.
We have been asked to do a run for base data with the base config, so we set the delays to 0 for both servers (we took the JMeter conf. used for Task 2, 2 thread groups):

![Base data JMeter](./pictures/jmeterBase.png)

### 2.
We set a 250ms delay to s1 :

![JMeter 250ms](./pictures/jmeter250.png)

We can see with this run that it took a really long time (25min), but it works well in fact. What happens it's that the first user takes the connection to s1 and it will be long because the cookie specify s1 for each request. And the second user go to s2 without problems and takes each time s2 because of the cookie.
### 3.
Proof that we set correctly the delay :

![Proof delay 2500ms](./pictures/proof2500.png)

We can see on the proof that HA detects s1 as DOWN.
We set a 2500ms delay to s1 :

![JMeter 2500ms](./pictures/jmeter2500.png)

We see that apparently the load balancer managd to see that the s1 server was too slow and directed all requests to s2 (see proof above). But that's really weird because this timeout for connecting to a server is defined as 5000ms.
### 4.
No we didn't have any error on these tasks, we think that the load balancer is smart enough to balance quite well, and we didn't make enough requests to cause an error.
### 5.
After doing the weight config (haproxy_weight.cfg in annexes):
```
server s1 ${WEBAPP_1_IP}:3000 cookie s1 check weight 2
server s2 ${WEBAPP_2_IP}:3000 cookie s2 check weight 1
```
We have this behaviour in the JMeter tests : 

![JMeter weights 250ms](./pictures/jmeter250weight.png)

Nothing change from the 2. apart of the time taken. That's because the weights doesn't change anything for 2 users.
### 6.
Now the weights takes effects because it's like we have another user each time, because we clean cookies for each iteration. So the weights influence the behaviour of the load balancer in this way :

![JMeter weights 250ms clean cookies](./pictures/jmeter250weight2.png)

Where we have more weight the load balancer will try to send more requests, accordingly to the incoming requests.
## Task 5
### 1. 
We have choosen the first and leastconn strategies, because they seems interesting to us. In fact, the first strategy is interesting because it's used to only use one server for a certain amount of connection and shut down the oter servers in the farm (ecological for this times). The leastconn strategy is interesting because it balance between the number of open connection in servers, if the server "s1" has 2 open connections and "s1" has none, leastconn strategy will send the new connection to "s1". This avoids having one server with 10 open connections and the other without any.
### 2.
#### First
In the annexes you can see the haproxy_first.cfg which is the default config we used for the tests, but we will change maxconn values as indicated. 

So for this balancing strategy we need this config :
```
balance first
server s1 ${WEBAPP_1_IP}:3000 cookie s1 check maxconn 1
server s2 ${WEBAPP_2_IP}:3000 cookie s2 check
```
The maxconn 1 define 1 connection before it switches to the other server.
Like we see on this JMeter test both servers have one connection. 

![Jmeter first strat](./pictures/jmeterStratFirst.png)

In this screenshot we see one connection for each (because of 2 threads).
If we want to see better this strategy we put maxconn to 3 and 4 threads. But we lower the iterations to 100 for each thread. And we can see this behaviour :

![First strat to maxconn 3](./pictures/jmeterStratFirst3.png)

With these parameters we can better see what's hapenning, it manages to have only 3 connections at a time. we can see that with the 300 requests to s1 and only 100 to s2, because s1 was already respondy to 3 different connections.

I've made tests with 10 threads but it's less obvious what happens because s1 take moe than 3 connections (because when the 100 iterations finish the connection is no more effective).

I think that's the best way to see really the impact. Maybe if you have a node that has better hardware than an other you can try this option to tweak the behaviour of your load balancing and go to mid-full capacity on the server you want not to overheat.

#### Leastconn

For this balancing strategy we need to modify the /ha/config/haproxy.cfg with this config (available in annexes as haproxy_leastconn.cfg) :

```
balance leastconn
```

We will test with JMeter the strategy. For testing the leastconn strategy, we need to create an imbalance between the two servers.

We will launch two JMeter at same time. One with 3 users who keep cookies and a second test with one user who clear cookies each iteration.

![Task5-leastconnJmeter1](./pictures/Task5-leastconnJmeter1.JPG)

![Task5-leastconnJmeter2](./pictures/Task5-leastconnJmeter2.JPG)

With these parameters we can see than the leastconn strategy balanced servers with new connections that came from the second JMeter test.

### 3.

For this lab, First is better because it allows to shut down no used server in no-intensive hours and leastconn is not useful here, because http connection don't last enough to create a unbalance that the leastconn strategy needs to correct.

## Conclusion
This lab was really interesting because HAProxy is really a complete tool for load balancing and it's interesting to see the optimizations we can do and that it's well used by profesionnals. And we learned many options of JMeter.


# Annexes

## haproxy_sticky.cfg
```
# Global configuration for HAProxy
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3
global
    # Bind UNIX socket to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket /var/run/haproxy.sock mode 600 level admin

    # Bind TCP port to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket ipv4@0.0.0.0:9999 level admin

    # Define the timeout on the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3.1-stats%20timeout
    stats timeout 30s

    # Configure the way the logging is done
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#log
    log 127.0.0.1 local1 notice

# Configure defaults for all the proxies configuration (applied for all the next sections in the configuration)
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
defaults
    # Enable logging
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-log
    log     global

    # The default mode for all the services
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    mode    http

    # Enable the logging of HTTP requests
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httplog
    option  httplog

    # Enable the logging of null connections
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20dontlognull
    option  dontlognull

    # Configure the timeout to connect to a server
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20connect
    timeout connect 5000

    # Configure the timeout before cutting the connection of a client
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20client
    timeout client  50000

    # Same kind of configuration for the servers side
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20server
    timeout server  50000

# Open the metrics HAProxy page on the port 1936 on any network interface on the host
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
listen stats *:1936
    # Enable HAProxy to serve stats about himself and the nodes
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20enable
    stats enable

    # Define the URI to access the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20uri
    stats uri /

    # Avoid leaking more info than necessary with hiding the version of HAProxy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20hide-version
    stats hide-version

# Define the frontend configuration. In fact, that's the part that configure how HAProxy will handle
# the requests from the outside world:
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
frontend localnodes
    # Bind the port 80 to listen incoming outside connections (from the outside world)
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    bind *:80

    # Define which protocol is enabled on the binded ports.
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Use the backend configuration references by the backend name section in this configuration
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-default_backend
    default_backend nodes

# Define the backend configuration. In fact, that's the part that configure what is not directly
# accessible from the outside of the network.
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
backend nodes
    # Define the protocol accepted
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Define the way the backend nodes are checked to know if they are alive or down
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httpchk
    option httpchk HEAD /

    # Define the balancing policy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#balance
    balance roundrobin

    # Automatically add the X-Forwarded-For header
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20forwardfor
    # https://en.wikipedia.org/wiki/X-Forwarded-For
    option forwardfor

    # With this config, we add the header X-Forwarded-Port
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-http-request
    http-request set-header X-Forwarded-Port %[dst_port]

    # Define the list of nodes to be in the balancing mechanism
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-server
    cookie SERVERID insert indirect nocache
    server s1 ${WEBAPP_1_IP}:3000 cookie s1 check
    server s2 ${WEBAPP_2_IP}:3000 cookie s2 check

# Other links you will need later for this lab
#
# About cookies: http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-cookie
#

```
## haproxy_weight.cfg
```
# Global configuration for HAProxy
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3
global
    # Bind UNIX socket to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket /var/run/haproxy.sock mode 600 level admin

    # Bind TCP port to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket ipv4@0.0.0.0:9999 level admin

    # Define the timeout on the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3.1-stats%20timeout
    stats timeout 30s

    # Configure the way the logging is done
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#log
    log 127.0.0.1 local1 notice

# Configure defaults for all the proxies configuration (applied for all the next sections in the configuration)
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
defaults
    # Enable logging
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-log
    log     global

    # The default mode for all the services
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    mode    http

    # Enable the logging of HTTP requests
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httplog
    option  httplog

    # Enable the logging of null connections
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20dontlognull
    option  dontlognull

    # Configure the timeout to connect to a server
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20connect
    timeout connect 5000

    # Configure the timeout before cutting the connection of a client
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20client
    timeout client  50000

    # Same kind of configuration for the servers side
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20server
    timeout server  50000

# Open the metrics HAProxy page on the port 1936 on any network interface on the host
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
listen stats *:1936
    # Enable HAProxy to serve stats about himself and the nodes
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20enable
    stats enable

    # Define the URI to access the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20uri
    stats uri /

    # Avoid leaking more info than necessary with hiding the version of HAProxy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20hide-version
    stats hide-version

# Define the frontend configuration. In fact, that's the part that configure how HAProxy will handle
# the requests from the outside world:
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
frontend localnodes
    # Bind the port 80 to listen incoming outside connections (from the outside world)
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    bind *:80

    # Define which protocol is enabled on the binded ports.
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Use the backend configuration references by the backend name section in this configuration
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-default_backend
    default_backend nodes

# Define the backend configuration. In fact, that's the part that configure what is not directly
# accessible from the outside of the network.
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
backend nodes
    # Define the protocol accepted
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Define the way the backend nodes are checked to know if they are alive or down
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httpchk
    option httpchk HEAD /

    # Define the balancing policy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#balance
    balance roundrobin

    # Automatically add the X-Forwarded-For header
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20forwardfor
    # https://en.wikipedia.org/wiki/X-Forwarded-For
    option forwardfor

    # With this config, we add the header X-Forwarded-Port
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-http-request
    http-request set-header X-Forwarded-Port %[dst_port]

    # Define the list of nodes to be in the balancing mechanism
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-server
    cookie SERVERID insert indirect nocache
    server s1 ${WEBAPP_1_IP}:3000 cookie s1 check weight 2
    server s2 ${WEBAPP_2_IP}:3000 cookie s2 check weight 1

# Other links you will need later for this lab
#
# About cookies: http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-cookie
#

```
## haproxy_first.cfg
```
# Global configuration for HAProxy
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3
global
    # Bind UNIX socket to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket /var/run/haproxy.sock mode 600 level admin

    # Bind TCP port to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket ipv4@0.0.0.0:9999 level admin

    # Define the timeout on the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3.1-stats%20timeout
    stats timeout 30s

    # Configure the way the logging is done
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#log
    log 127.0.0.1 local1 notice

# Configure defaults for all the proxies configuration (applied for all the next sections in the configuration)
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
defaults
    # Enable logging
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-log
    log     global

    # The default mode for all the services
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    mode    http

    # Enable the logging of HTTP requests
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httplog
    option  httplog

    # Enable the logging of null connections
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20dontlognull
    option  dontlognull

    # Configure the timeout to connect to a server
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20connect
    timeout connect 5000

    # Configure the timeout before cutting the connection of a client
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20client
    timeout client  50000

    # Same kind of configuration for the servers side
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20server
    timeout server  50000

# Open the metrics HAProxy page on the port 1936 on any network interface on the host
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
listen stats *:1936
    # Enable HAProxy to serve stats about himself and the nodes
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20enable
    stats enable

    # Define the URI to access the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20uri
    stats uri /

    # Avoid leaking more info than necessary with hiding the version of HAProxy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20hide-version
    stats hide-version

# Define the frontend configuration. In fact, that's the part that configure how HAProxy will handle
# the requests from the outside world:
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
frontend localnodes
    # Bind the port 80 to listen incoming outside connections (from the outside world)
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    bind *:80

    # Define which protocol is enabled on the binded ports.
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Use the backend configuration references by the backend name section in this configuration
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-default_backend
    default_backend nodes

# Define the backend configuration. In fact, that's the part that configure what is not directly
# accessible from the outside of the network.
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
backend nodes
    # Define the protocol accepted
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Define the way the backend nodes are checked to know if they are alive or down
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httpchk
    option httpchk HEAD /

    # Define the balancing policy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#balance
    balance first

    # Automatically add the X-Forwarded-For header
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20forwardfor
    # https://en.wikipedia.org/wiki/X-Forwarded-For
    option forwardfor

    # With this config, we add the header X-Forwarded-Port
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-http-request
    http-request set-header X-Forwarded-Port %[dst_port]

    # Define the list of nodes to be in the balancing mechanism
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-server
    cookie SERVERID insert indirect nocache
    server s1 ${WEBAPP_1_IP}:3000 cookie s1 check maxconn 1
    server s2 ${WEBAPP_2_IP}:3000 cookie s2 check

# Other links you will need later for this lab
#
# About cookies: http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-cookie
#
```
## haproxy_leastconn.cfg
```
# Global configuration for HAProxy
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3
global
    # Bind UNIX socket to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket /var/run/haproxy.sock mode 600 level admin

    # Bind TCP port to get various stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#stats
    stats socket ipv4@0.0.0.0:9999 level admin

    # Define the timeout on the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#3.1-stats%20timeout
    stats timeout 30s

    # Configure the way the logging is done
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#log
    log 127.0.0.1 local1 notice

# Configure defaults for all the proxies configuration (applied for all the next sections in the configuration)
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
defaults
    # Enable logging
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-log
    log     global

    # The default mode for all the services
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    mode    http

    # Enable the logging of HTTP requests
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httplog
    option  httplog

    # Enable the logging of null connections
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20dontlognull
    option  dontlognull

    # Configure the timeout to connect to a server
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20connect
    timeout connect 5000

    # Configure the timeout before cutting the connection of a client
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20client
    timeout client  50000

    # Same kind of configuration for the servers side
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-timeout%20server
    timeout server  50000

# Open the metrics HAProxy page on the port 1936 on any network interface on the host
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
listen stats *:1936
    # Enable HAProxy to serve stats about himself and the nodes
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20enable
    stats enable

    # Define the URI to access the stats
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20uri
    stats uri /

    # Avoid leaking more info than necessary with hiding the version of HAProxy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-stats%20hide-version
    stats hide-version

# Define the frontend configuration. In fact, that's the part that configure how HAProxy will handle
# the requests from the outside world:
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
frontend localnodes
    # Bind the port 80 to listen incoming outside connections (from the outside world)
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-bind
    bind *:80

    # Define which protocol is enabled on the binded ports.
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Use the backend configuration references by the backend name section in this configuration
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-default_backend
    default_backend nodes

# Define the backend configuration. In fact, that's the part that configure what is not directly
# accessible from the outside of the network.
# http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4
backend nodes
    # Define the protocol accepted
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-mode
    mode http

    # Define the way the backend nodes are checked to know if they are alive or down
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20httpchk
    option httpchk HEAD /

    # Define the balancing policy
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#balance
    balance leastconn

    # Automatically add the X-Forwarded-For header
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-option%20forwardfor
    # https://en.wikipedia.org/wiki/X-Forwarded-For
    option forwardfor

    # With this config, we add the header X-Forwarded-Port
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-http-request
    http-request set-header X-Forwarded-Port %[dst_port]

    # Define the list of nodes to be in the balancing mechanism
    # http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-server
    cookie SERVERID insert indirect nocache
    server s1 ${WEBAPP_1_IP}:3000 cookie s1 check
    server s2 ${WEBAPP_2_IP}:3000 cookie s2 check

# Other links you will need later for this lab
#
# About cookies: http://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-cookie
#

```