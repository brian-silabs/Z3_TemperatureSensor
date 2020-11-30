---
sort: 3
---

# Running the example

-   Use the Z3_GatewayHost sample application
* Create a Network
* Open the network
* Start the find and Bind target procedure

To do so in the Gateway's CLI:
```console
plugin network-creator start 1
plugin network-creator-security open-net
plugin find-and-bind target 1
```

* Initiate Steering on the thuderboard by pressing one of the 2 push buttons
If all goes well, you should see the below in the output :

```console
Z3_TemperatureSensor>EMBER_NETWORK_DOWN
NWK Steering stack status 0x91
NWK Steering: issuing scan on primary channels (mask 0x0318C800)
NWK Steering: Start: 0x00
Join network start: 0x00
NWK Steering scan complete. Beacons heard: 1
NWK Steering State: Scan Primary Channels and use Install Code
Error: NWK Steering could not setup security: 0xB5
Examining beacon on channel 25 with panId 0x7ED1
NWK Steering joining 0x7ED1 on channel 25
EMBER_NETWORK_UP 0x78E7
NWK Steering stack status 0x90
NWK Steering network joined.

Processing message: len=12 profile=0000 cluster=0013
RX: ZDO, command 0x0013, status: 0x00
Device Announce: 0x78E7

pJoin for 180 sec: 0x70
NWK Steering: Broadcasting permit join: 0x00
NWK Steering Stop.  Cleaning up.

Find and bind initiator start: 0x00
Find and bind initiator complete: 0x00
```
After that, the device will start emitting Temperature reports to the Gateway at the period set in the ZCL Configuration
