# LDP IGP Synchronization

## Topology

![image](https://user-images.githubusercontent.com/17289045/147365472-6240794b-6d00-49ca-8aa0-86f9af3f24ce.png)


LDP - IGP synchronization can help in situations when IGP is fully sinchronized (i.e. everything is stable), but LDP is still no synchronized.
This situation can generate black holes (i.e. routing with MPLS that is sending packets in the wrong path.

This feature is valid for IGP (nbut not supported by EIGRP).
Basically, whenever there is an issue with an LDP neighbor, LDP asks to IGP to increase the metric of the impacted destination to have them routed to an alternate path.

The behavior can be configured to last for a specific time, or indefintely until LDP issues is fixed.

It can be configured in ruter config mode with
> mpls ldp sync

It can also be disabled on a per-interface base with:
> no mpls ldp igp sync

Let's configure the topoly without LDP-IGP synch and try to see the routing table for 4.4.4.4 for R1:

![image](https://user-images.githubusercontent.com/17289045/147365628-9e8dc43f-91ad-4afb-9294-633dd454e266.png)

Then let's create a filter for LDP packages in GNS3:

![image](https://user-images.githubusercontent.com/17289045/147365741-203e16f4-9c5e-49b4-b523-287054e855aa.png)

We see that the IGP routes are still there and equal.
![image](https://user-images.githubusercontent.com/17289045/147365788-952e8796-7390-4021-87f1-df26786df0c6.png)

With a traceroute we can see that R1 is still loa balancing te traffic, to R2 without labels and to R3 with labels.
![image](https://user-images.githubusercontent.com/17289045/147365819-be2114e7-f78f-4cbb-8323-6debfe8e53cf.png)

If we don't want this, we can configure R1 for LDP IGP sync.
Let's remove the filter and only after that configure sync.

Then enable the filter again and we can observe that IGP removes the route using R2, even if it still a OSPF neighbor 
![image](https://user-images.githubusercontent.com/17289045/147366063-35021d69-50fc-4cd1-8b31-09c12e2d052c.png)
And the OSPF LSDB contains 4.4.4.4
![image](https://user-images.githubusercontent.com/17289045/147366092-dcdf98d2-9334-45fd-902a-7885e02ef658.png)

OSPF would like to operate correctly but LDP convinced him to drop the path using R2.

If we put a timer to LDP sync in global config mode 
> mpls ldp igp sync holddown XXX

We can ask to the router to keep the IGP route valid for a specific time (XXX milliseconds) after the LDP neighbor 




