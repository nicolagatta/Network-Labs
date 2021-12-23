# LDP Targeted session and Session Protection

# Targeted session

There are some situations where we need to establish an LDP session between non adjacent router:
- AtoM (Any transport over MPLS)
- MPLE TE Tunnel

In such situations multicast hello packets can't be exchanged between the router and LDP session can't be established

LDP session can be stablished with a command that specifies the IP address of the neighbor.
As a best practice the configuration should use the IP address configured as LDP transport address (or router-id, which is usually the same)

The command is
> mpls ldp neighbor <IP> targeted
  
After that, the router will send out unicast Hello packets that will reach the non adjacent router
This can be configured in the same way (most secure)
Or it can be cnfigured to accept targeted hellos (less secure) with  
> mpls ldp discovery targeted-hello accept [from ACL]
  
  
 # Session Protection 
