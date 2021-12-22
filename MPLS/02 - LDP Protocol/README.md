# LDP protocol

## Topology
![image](https://user-images.githubusercontent.com/17289045/147063746-099854a6-0a24-4a4e-aa40-10572fe077b1.png)

## Initial Setup
OSPF is configured for all routers to have full reachability, including loopback interfaces.

## LDP Setup
On R1 we can setup MPLS using 

> mpls ldp router-id lo0
> 
> mpls label range 1000 1999
> 
> mpls label protocol ldp
> 
> int e0/0
> 
> mpls ip

As soon as we give the above comment, the router starts sending out multicast packets (Hello packets)
- DST MAC: 01:00:5e:00:00:02
- DST IP: 224.0.0.2
- UDP port 646 (src and dst)
- in the payload there are some information like
  - LDP Version
  - Hello packet type
  - Hello parameters (i.e. timers)
  - IPv4 Transport address (the IP used for further unicast comunication with neighbors)
 This is usually referred as "Discovery message", to discover neighbors
 
![image](https://user-images.githubusercontent.com/17289045/147068650-20a24bdd-97b3-4e5d-bfd3-3f7fbbb02f23.png)

# LDP Neighborship and Session 
If we configure also R2 with the commands (note the labels IDs in a different range):

> mpls ldp router-id lo0
> 
> mpls label range 2000 2999
> 
> mpls label protocol ldp
> 
> int e0/0
> 
> mpls ip
> 
> int e0/1
> 
> mpls ip

We see that R2 sends out an Hello packet and immediately R1 and R2 start communicating on port tcp 646
![image](https://user-images.githubusercontent.com/17289045/147071449-e472db73-20e0-4f42-a36d-24bb52079cf1.png)

Some remarks:
- after Hello exchange, both routers know their router-id (IP of Loopback 0)
- lower router-id will be passive (i.e. listen tcp/646
- They complete 3-way handshake
- they send keep alive messages (and they sends them periodically) (oSession management messages)
- finally they exchange their labels inside (advertisement messages)

An example of advertisement from R2:
![image](https://user-images.githubusercontent.com/17289045/147072421-cd59e7b0-790f-41b3-870a-2ec563c1229f.png)
It's unicast and it contains 
- header
- Address message with a list of its interfaces IPs
- a list of Label Mapping Messages

Let's see one of this label mapping messages
![image](https://user-images.githubusercontent.com/17289045/147073007-98ef1ec8-672c-4260-a29c-6f1f8a920a56.png)

It has three sections: header, FEC and Label:

![image](https://user-images.githubusercontent.com/17289045/147073129-0c02e26f-e35e-4e9e-9049-7ac3fddd6057.png)

The FEC section contains the prefix 3.3.3.3/32 and its associated label: 0x7d1 (2001 in decimal)
This means that R2 advertises it's prefixes (all!) with its label to R1

in LDP the label advertisment is upstream (i.e. follows the opposite directions as the traffic):
The traffic from R1 to R3 is downstream (R1 send to R2 for routing)
The label for R3 lo0 are communicated upstream (R2 assigns and communicate its label to R1)

This can be verifed on router R2 with the command:
> show mpls forwarding table
![image](https://user-images.githubusercontent.com/17289045/147073972-655468ed-0e12-4e26-8d1c-d524a8d6f17c.png)


![image](https://user-images.githubusercontent.com/17289045/147073933-a013b420-eaeb-43b0-8043-49080b3fadff.png)



There is one particular (reserved) LDP label that is related to directly attached addresses, like 2.2.2.2
The labels is #3 (implicit NULL label) which means that R2 is saying to R1: I'm the last hop to reach 2.2.2.2, so don't use a label when you send me packets for 2.2.2.2
This is called "penultimate Hop popping": the penultimate rpouter for a destination can safely remove the label, because the last router doesn't really need to do a routing decision
This is clear analyzing a ping with wireshark

Let's try a ping from router R1 (1.1.1.1) to R3 (3.3.3.3)

![image](https://user-images.githubusercontent.com/17289045/147074510-206003e9-d80c-4734-8e06-8428247fae33.png)

Let's look at the first ICMP packet from R1 to R3:
It has a header betwwen layer2 and layer containing the MPLS header with the Label 2001

If we do the same thing ping R2 from R1, R1 won't put any LDP label, because it doesn't know the label (since it is the penultimate router and R2 doesn't send it to R1)
![image](https://user-images.githubusercontent.com/17289045/147075693-6cd32636-6950-42c7-8093-9a520a9c428f.png)

# Complete the MPLS configuration

Now configure R3 for MPLS and try the same ping from R1 to R3, this time using wireshark between R2 and R3

> mpls ldp router-id lo0
> mpls label range 3000 3999
> mpls label protocol ldp
> int e0/0
> mpls ip
> int e0/1
> mpls ip

Let's open wireshark also between R2 and R3

Let's look at ping from R1 to R3
![image](https://user-images.githubusercontent.com/17289045/147076538-26f237c8-89ac-450b-94e2-597712aece36.png)

R1 sends out packet using the label 2001.
R2 pop the label and sends out the packet with no labels since it is the penultimate hop for 3.3.3.3

Let's look at ping from R1 to R4
![image](https://user-images.githubusercontent.com/17289045/147076706-d5a063f6-747b-42d5-b668-a872451ee483.png)

R1 sends out packet using the label 2002.
R2 pop the label 2002 and will push the label 3002 
R3 will have the role of penultimape hop

We can also configure R4 for MPLS and look also at the returning traffic for the ping packets


