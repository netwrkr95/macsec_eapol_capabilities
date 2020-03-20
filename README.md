## Enhancements for running MACsec over a public Carrier Ethernet Transport

The purpose of this document is to provide several key enhancements Cisco IOS-XE, IOS-XR, and NX-OS has added to their MACsec feature capabilities for overcoming challenges typically seen when running MACsec encryption over a public carrier Ethernet transport (E-LINE, E-LAN, etc.).

### What is WAN MACsec?

To gather a more in-depth understanding of WAN MACsec, how it benefits network designs, and key areas operators should be understanding, see key reference material below [2] that will aid in understanding the vital concept for securing high speed data networks.

### Common Challenges

Prior to these enhancements, establishing MACsec connectivity over any transport than dark fiber, was non-existent, and for some of the carrier ethernet providers, highly discouraged due to the known challenges that existed, specifically for the key authentication.  The reason?  The MACsec Key Agreement (MKA), which is transported over the EAP over LAN (EAPoL) encapsulation leverages a well-known MAC address (01:80:C2:00:00:03) and Ether-type (0x888e) within the key exchange.  The problem is that this well-known MAC/ethertype are used for other communications, such as 802.1X.  Given this, when two MACsec routers attempt to negotiate a MACsec session over MKA/EAPoL,the provider backbone bridge supporting the ethernet transport sees an Ethernet frame with this MAC/ethertype, and the logic says it is a "for me" frame, and consumes it for further processing.  Upon further processing, it deciphers it is not for the transit bridge, and drops the frame.  Now, the two MACsec endpoints have an unsuccessful key establishment as the MKA frame sent for key nogotiation never made it to the destination end-point.

### Solution - EAPoL Enhancements

To overcome this critical challenge for running MACsec over a public Ethernet transport, Cisco introduced the ability for an operator to modify the EAPoL MAC address and ethertype from the default settings of 802.1X.  The change options remain standard attributes and are only relevant to the MACsec MKA end-points, thus remaining transparent to the underlying carrier Ethernet provider transporting the frames, but now, unlike in the default settings, these EAPoL MKA frame remain untouched by the transit backbone bridges.

### WAN MACsec EAPoL Command Example

Below is an IOS-XE example of the two primary enhancement commands to the EAPoL key exchange transport:

```
interface TenGigabitEthernet0/0/0
...
 eapol destination-address broadcast-address
 eapol eth-type 876F
 ...
 macsec
!
```
The first of these commands, the "**eapol destination-address broadcast-address**" command, allows the EAPoL destination MAC address to be modified, using a standard (all "F's") broadcast address.  This allows the Layer-2 transport network to treat the EAPoL traffic as a standard broadcast type packet, and more importantly, transparently flooding it to all receivers un-processed by the transit backbone bridges. 

The second command, "**eapol eth-type 876F**", modifies the ether-type of the EAPoL frame to an ether-type that is "unknown" (Cisco owns this ether-type value) encoding, changing the behavior of how the public Ethernet transport transit bridge processes the frame.  In essence, it allows the transit bridge(s) to ignore this ether-type and for the frame to be sent to the MACsec destination MACsec end-point, un-processed.  The output from a Wifreshark capture can be found a the following link in the "docs" section of the repo ([EAPoL Wireshark capture output](https://github.com/netwrkr95/macsec_eapol_capabilities/blob/master/docs/EAPoL_Capture.txt)).

The final command in the example, "**macsec**", simply enables MACsec on the shown interface.
```
It should be noted that not all Ethernet transports and/or provider backbone bridges react on EAPoL related identifiers, specifically MAC address or ether-types, so in these cases, the operator can ignore enabling these commands on the MACsec interface.  As a best practice, these EAPoL commands are recommended, as it provides a deterministic MAC address/ether-type for the EAPoL packets that are ingnored by all transports and bridges/switches.
```

### Ansible Playbooks for Automating the Deployment of these EAPoL Commands in IOS-XE

I created basic Ansible playbooks from a customer demonstration for applying these commands to an IOS-XE router running MACsec, in addition to other detailed examples for dynamic MKA key rotation within IOS-XE for pre-shared keys.

These playbooks and configuration examples can be found at:

[Reference Link to configuration examples - MACsec EAPoL Demo's](https://github.com/netwrkr95/macsec_eapol_demo)

### References

[1] Cisco IOS-XE EAPoL Documentation: https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/macsec/configuration/xe-3s/macsec-xe-3s-book.html#task_1573E9C7150F4451981F493D0B7859CB

[2] WAN MACsec Overview - White Paper:  https://www.cisco.com/c/dam/en/us/td/docs/solutions/Enterprise/Security/MACsec/WP-High-Speed-WAN-Encrypt-MACsec.pdf 


