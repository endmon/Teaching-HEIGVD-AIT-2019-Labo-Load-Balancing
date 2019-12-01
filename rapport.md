#Task 1

### 1.
We can see that the proxy handles requests like this :
It will address the requests on each server sequentially. It means that one time it will be sent to one server and the next request coming will go threw the other server.
The NODESESSID is changing at each request which is not so surpirsing because each time we visit the site or actualize we have an ID which is invalid for the server requested (because the precendent response came from the other server). 
### 2. 
We expect the load balancer to address requests coming in with a session ID to the server who knows this session (so the server who created the session). If not, the session management can't be done.
### 3.
Here is our sequence diagram for task 1 (roundrobin). This sequence diagram shows 2 requests by the same user, we see that the behaviour is not correct because both requests don't go on the same server as expected.
![Sequence Diagram 1](./pictures/seqDiagramRound1.png)
### 4.
Here is the test in JMeter, we can see that assertions are false because the counters doesn't increment, in fact th server don't know the users coming because of balancing strategy used here.
![JMeter results](./pictures/jmeterFirst.png)
### 5.
Here we see the sequence diagram for this task, we can see that because there's only one server running it responds and knows each user (because he serves all the requests), and so the counter increments correctly.
![Sequence Diagram for one server](./pictures/seqDiagramRound2.png)
We can see the JMeter report too :
![JMeter results](./pictures/jmeterSecond.png)
## Task 2
### 1.
Sequence Diagram to show differences on stickiness
We think that the SERVERID alternative is more effective because the balancer can just evaluate the SEVERID and redirect the request accordingly.
### 2.
We can see the modified configuration at the root of the git repo : `haproxy_sticky.cfg`. The modifications are juste on the `backend nodes` section. We've just added `cookie SERVERID insert indirect nocache` and the value we want to add on the cookie for each server so :
`server s1 ${WEBAPP_1_IP}:3000 cookie s1 check
 server s2 ${WEBAPP_2_IP}:3000 cookie s2 check`
### 3.
To see if we have achieved the right behaviour we can check like that :
First we visit the site and we see that the load balancer crete the SERVERID cookie and that if we refresh the page we keep the same server :
![sticky session 1](./pictures/sticky1.png)
![sticky session 2](./pictures/sticky2.png)
But if we open an incognito (to remove all cookies) we can see that the roundrobin strategy apply and we have the other server attribued :
![sticky session 3](./pictures/sticky3.png)
And if we refresh this page we keep the same server again because the SERVERID cookie is well set.
![sticky session 4](./pictures/sticky4.png)
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
## Task 3
## Task 4
### 1.
We have been asked to do a run for base data with the base config, so we set the delays to 0 for both servers (we took the JMeter conf. used for Task 2, 2 thread groups):
![Base data JMeter](./pictures/jmeterBase.png)
### 2.
We set a 250ms delay to s1 :
![JMeter 250ms](./pictures/jmeter250.png)

### 3.
### 4.
### 5.
### 6.

