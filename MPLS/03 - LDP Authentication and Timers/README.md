# LDP Authentication and Timers

# Topology
![image](https://user-images.githubusercontent.com/17289045/147111445-19a407aa-bb60-4960-9943-0850e4a6729a.png)


# Authentication

LDP can use MD5 to authenticate neighbors, using a shared secret (password).
This is done when establishing the session with TCP, therefore after 

It can done with these requirements:
- we know the Transport IP address of the two neighbors
- we can configure a shared password

Configurazione can be done by explicitely supplying a password for a neighbor.
On R1 we can use such global configuration:

> mpls ldp neighbor 2.2.2.2 password mys3cretpa$$

And on the other side, on R2 we can use:

> mpls ldp neighbor 1.1.1.1 password mys3cretpa$$

# Protecting authentication process

However we should notice that the dicovery process is not protected in any way.
Therefore there is a (little) space for attacks.
The best practice would be to protect the TCP session by using ACLs to let only authorized IPs
To do taht we have know which router will be passive (and therefore will be listening on port 646).
In our case R1 (1.1.1.1) will be the passive therefore we'll have to filter the traffic 


## R1
> ip access-list extended PERMIT_LDP
>
>  permit tcp host 2.2.2.2 host 1.1.1.1 eq 646
>
>  deny tcp any any eq 646
>
>  permit ip any any (3 matches)

> int e0/0
> ip access-group PERMIT_LDP in

## R2
> ip access-list extended PERMIT_LDP
>  permit tcp host 1.1.1.1 eq 646 host 1.1.1.1

>  deny tcp any eq 646 any


>  permit ip any any (3 matches)

> int e0/0
> ip access-group PERMIT_LDP in


# Timers
Since there are two communications channel between routers (Discovery and session), there are also different timers

- Hello Interval: interval of sending the Hello packets  (default 5 seconds)
- Hello holdtime: interval after a neighbor is considered dead (default 15 seconds)
- Session Holdtime: it established the timeout to drop a LDP session if the router doesn't get a Keepalive message (default 180 seconds)
  - Keepalive interval is automatically calculated as 1/3 of the Session Holtime
- Backoff time: If the two router can't agree on session parameter they retry to establish the session after N seconds and retrying with higher intervals (up to a maximum)

Configuration of timers:
Hello timers:
> mpls ldp discovery hello interval 10
> mpls ldp discovery hello holdtime 10

Session Holdtime:
> mpls ldp holdtime 10

Backoff Initial and Maximum timer:
> mpls ldp backoff 15 120
