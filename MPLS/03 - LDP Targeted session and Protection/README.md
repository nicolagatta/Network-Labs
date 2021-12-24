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
qshow ipThe most important requirements in this case, is that it works only with neighbors that are configured with a targeted session

In this case let's consider R3 and R4 LDP session and look the LDP binding of 4.4.4.4 on R3:
  ![image](https://user-images.githubusercontent.com/17289045/147362704-3bd88b03-7129-47f1-9708-5fd92fd5300d.png)
We see that it has binding from R1, R4, R5 (this because of the "liberal mode")

Taking down R3 e0/0 interface (link with R4) it will take down also the LDP session with R4 all bidingns from R4 are lost.
![image](https://user-images.githubusercontent.com/17289045/147362794-2d5aab7b-4774-48fc-9b5c-30403030de8a.png)

However, due to the redundant path using R5, the destination is still available using labels from R5

Session protection prevents such issues by enabling targeted hellos
We can see it below: just after typing the command
> mpls ldp session protection [duration [infinite | seconds ]


![image](https://user-images.githubusercontent.com/17289045/147363132-f1fa6e17-7c5d-4ee3-bb05-f8175bb65569.png)

If we take down the R3 interface e0/0 (connecting R3 and R4) we on the other wireshark between R3 and R5, that targeted hellos between 3.3.3.3 and 4.4.4.4 are going through it
  
![image](https://user-images.githubusercontent.com/17289045/147363629-f26c8a51-718a-4c0f-92fc-ac9b56c6b978.png)

And the bbinding on R3 are still there
  ![image](https://user-images.githubusercontent.com/17289045/147363670-026819aa-c43f-4e4f-be37-e88f847df6ce.png)

And we also see that session protection is doing its work:
  ![image](https://user-images.githubusercontent.com/17289045/147363704-3e0bc511-f4e8-4069-a256-bf8abe1a1309.png)

