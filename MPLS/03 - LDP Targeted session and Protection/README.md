# LDP Targeted session and Session Protection

## Topology

![image](https://user-images.githubusercontent.com/17289045/147263833-94fd628b-63d3-4fb1-99d6-a4fcdc3224b2.png)


## Targeted session

There are some situations where we need to establish an LDP session between non adjacent router:
- AtoM (Any transport over MPLS)
- MPLS TE Tunnel

In such situations multicast hello packets can't be exchanged between the router and LDP session can't be established

LDP session can be stablished with a command that specifies the IP address of the neighbor.
As a best practice the configuration should use the IP address configured as LDP transport address (or router-id, which is usually the same)

The command is
> mpls ldp neighbor <IP> targeted

After that, the router will send out unicast Hello packets that will reach the non adjacent router
This can be configured in the same way (most secure)
Or it can be cnfigured to accept targeted hellos with 
> mpls ldp discovery targeted-hello accept [from ACL]
This is considered to be less secure but can be filter with an ACL.

Once done, we can observe in wireshark, that the hello from 1.1.1.1 andfrom 3.3.3.3 are sent as unicast and the LDP session is established:
![image](https://user-images.githubusercontent.com/17289045/147269149-5462ae74-8437-4353-89a5-a24931fea8b0.png)

In the wireshark we can also see the multicast from R3 looking for other neighbors.
When R1 and R3 exchange the first two hellos they become neighbors.
And R3 now has this LDP bindings
  ![image](https://user-images.githubusercontent.com/17289045/147269335-28423f7c-d7be-47f5-8274-0e609b21d4b0.png)

 
## Session Protection 

Whenever a link between two LDP neighbors fails the two routers drop the LDP session and invalidates LDP bindings.
This can be avoided if there's a backup link (or an alternate route to reach the neighbor).
In this case the LDP bindings can be kept valid until the LDP session is recovered using the alternate path to reach the neighbor.
By default the session protection is kept for an infinite time (i.e. LDP bindings are not discarded, even if the neighbor is not reachable for a looong time.
The most important requirements in this case, is that it works only with neighbors that are configured with a targeted session

In this case let's consider R3 and R4 LDP session
Let's configure the targeted session between them and then shutdown the interface between them
  
Then let's use LDP protection
