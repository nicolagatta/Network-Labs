# LDP IGP Synchronization

LDP - IGP synchronization can help in situations when IGP is fully sinchronized (i.e. everything is stable), but LDP is still no synchronized.
This situation can generate black holes (i.e. routing with MPLS that is sending packets in the wrong path.

This feature is valid for IGP (nbut not supported by EIGRP).
Basically, whenever there is an issue with an LDP neighbor, LDP asks to IGP to increase the metric of the impacted destination to have them routed to an alternate path.

The behavior can be configured to last for a specific time, or indefintely until LDP issues is fixed.
