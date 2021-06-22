# Details on the YANG Model Support in IOS-XE for EAPoL Commands

The content below describes the YANG model description of the relevent EAPoL commands used in certain WAN MACsec deployments.


## WAN MACsec EAPoL YANG Model Leaf (view via 'pyang -f tree')

Execute the PYANG command with a tree depth of 4 levels for the 'Cisco-IOS-XE-ethernet.yang' native model in IOS-XE

```
pyang -f tree --tree-depth 4 Cisco-IOS-XE-ethernet.yang
```


Output:

```
+--rw eapol
    |  +--rw announcement?          empty
    |  +--rw destination-address
    |  |  +--rw (address-option)?
    |  |  |  +--:(mac-address)
    |  |  |  |     ...
    |  |  |  +--:(bridge-group-address)
    |  |  |  |     ...
    |  |  |  +--:(broadcast-addr)
    |  |  |  |     ...
    |  |  |  +--:(lldp-multicast-address)
    |  |  |        ...
    |  |  x--rw broadcast-address?              empty
    |  +--rw eth-type?              enumeration
```



Below is the output in the CLI version from an ASR 1001-X (IOS-XE example) for both EAPoL key exchange transport command options:

```
interface TenGigabitEthernet0/0/0
...
 eapol destination-address broadcast-address
 eapol eth-type 876F
 ...
 macsec
!
```

