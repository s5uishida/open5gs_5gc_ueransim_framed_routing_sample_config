# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Framed Routing with Open5GS UPF
This describes a very simple configuration that uses Open5GS and UERANSIM for Framed Routing.

This feature has been merged into Open5GS via the following pull requests by **@mitmitmitm**.

- [Framed routing](https://github.com/open5gs/open5gs/pull/2009)
- [Framed routes udr](https://github.com/open5gs/open5gs/pull/2022)
- [[SMF/PFCP] Send framed routes in both UL and DL pdrs](https://github.com/open5gs/open5gs/pull/2356)

The related documents can be found below.
- https://github.com/gonalobastos/5G-Framed-Routing

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Overview of Open5GS 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of Open5GS 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of Open5GS 5GC U-Plane](#changes_up)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN](#changes_ran)
    - [Changes in configuration files of UE0 (IMSI-001010000000000)](#changes_ue0)
    - [Changes in configuration files of UE1 (IMSI-001010000000001)](#changes_ue1)
- [Network settings of Open5GS 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of Open5GS 5GC U-Plane](#network_settings_up)
  - [Network settings of External Node](#network_settings_ext)
  - [Network settings of VM3](#network_settings_vm3)
    - [Add netns](#add_netns)
    - [Setup veth pair for UE0 and PC1/PC4](#setup_ue0)
    - [Setup veth pair for UE1 and PC2/PC3](#setup_ue1)
- [Add Framed Routes to Subscriber information](#add_framed_routes)
  - [Add Framed Routes to UE0](#add_framed_routes_ue0)
  - [Add Framed Routes to UE1](#add_framed_routes_ue1)
- [Build Open5GS and UERANSIM](#build)
- [Run Open5GS 5GC and UERANSIM UE / RAN](#run)
  - [Run Open5GS 5GC C-Plane](#run_cp)
  - [Run Open5GS 5GC U-Plane](#run_up)
  - [Run UERANSIM](#run_ueran)
    - [Start gNodeB](#start_gnb)
    - [Start UE0](#start_ue0)
    - [Start UE1](#start_ue1)
  - [Run tcpdump on PC1](#run_pc1)
  - [Run tcpdump on PC2](#run_pc2)
  - [Run tcpdump on PC3](#run_pc3)
  - [Run tcpdump on PC4](#run_pc4)
- [Ping Framed Routes](#ping)
  - [Ping IP address (192.168.20.100/24) of Framed Routes of UE0 on PC1](#ping_pc1)
  - [Ping IP address (192.168.21.100/24) of Framed Routes of UE1 on PC2](#ping_pc2)
  - [Ping IP address (192.168.22.100/24) of Framed Routes of UE1 on PC3](#ping_pc3)
  - [Ping IP address (192.168.23.100/24) not configured for Framed Routes](#ping_pc4)
- [Changelog (summary)](#changelog)
---
<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

I created a 5GC simulation mobile network for  the purpose of using  the IP routes (Framed Routes) behind the UE.

The following minimum configuration was set as a condition.
- Two UEs have the same DNN and connect to the same DN.
- Two UEs have different Framed Routes. On the UPF VM, make sure to be able to ping the Framed Routes via the IP address (Tunnel GW/uesimtun0) assigned to each UE.
- Confirm not to be able to ping to a network that is not configured in the Framed Routes.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The following figure shows the netns and veth pairs within VM3.

<img src="./images/netns-overview.png" title="./images/netns-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.8.0 (2026.09.16) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.3.0 (2026.09.06) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | CPU<br>(Min) | Mem<br>(Min) | HDD<br>(Min) |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24<br>192.168.14.111/24 | Ubuntu 24.04 | 1 | 2GB | 20GB |
| VM2 | Open5GS 5GC U-Plane  | 192.168.0.151/24<br>192.168.13.151/24<br>192.168.14.151/24<br>**192.168.16.151/24** | Ubuntu 24.04 | 1 | 1GB | 10GB |
| EXT | External Node | 192.168.0.152/24<br>**192.168.16.152/24** | Ubuntu 24.04 | 1 | 1GB | 10GB |
| VM3 | UERANSIM RAN (gNodeB) | 192.168.0.131/24<br>192.168.13.131/24 | Ubuntu 24.04 | 1 | 1GB | 10GB |
|| UERANSIM UE0 | **192.168.20.1/24<br>192.168.23.1/24** | -- | -- | -- | -- |
|| UERANSIM UE1 | **192.168.21.1/24<br>192.168.22.1/24** | -- | -- | -- | -- |
|| PC1 Internal Node | **192.168.20.100/24** | -- | -- | -- | -- |
|| PC2 Internal Node | **192.168.21.100/24** | -- | -- | -- | -- |
|| PC3 Internal Node | **192.168.22.100/24** | -- | -- | -- | -- |
|| PC4 Internal Node | **192.168.23.100/24** | -- | -- | -- | -- |

Pairs of network namespaces and virtual network interfaces are follows.
| Role | netns | veth | veth | netns | Role |
| --- | --- | --- | --- | --- | --- |
| UE0 | ueransim-001010000000000-internet-psi1 | veth-ue0-pc1<br>**192.168.20.1/24** | veth-pc1<br>**192.168.20.100/24** | pc1 | PC1 |
||| veth-ue0-pc4<br>**192.168.23.1/24** | veth-pc4<br>**192.168.23.100/24** | pc4 | PC4 |
| UE1 | ueransim-001010000000001-internet-psi1 | veth-ue1-pc2<br>**192.168.21.1/24** | veth-pc2<br>**192.168.21.100/24** | pc2 | PC2 |
||| veth-ue1-pc3<br>**192.168.22.1/24** | veth-pc3<br>**192.168.22.100/24** | pc3 | PC3 |

Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files. As of 2023.01.29, Framed Routes cannot be set with the WebUI. Also, if you change the `open5gs-dbctl` script, it seems that you can register these with this script, but I could not register.**
| UE # | IMSI | DNN | OP/OPc | Framed Routes | Internal IP address |
| --- | --- | --- | --- | --- | --- |
| UE0 | 001010000000000 | internet | OPc | **192.168.20.0/24** | **192.168.20.1** |
| UE1 | 001010000000001 | internet | OPc | **192.168.21.0/24<br>192.168.22.0/24** | **192.168.21.1<br>192.168.22.1** |

**Note. <ins>192.168.23.0/24</ins> is not configured for Framed Routes.**

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

The DN is as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 |

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.8.0 (2026.09.16) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.3.0 (2026.09.06) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2026-09-16 20:29:38.000000000 +0900
+++ amf.yaml    2026-09-16 23:23:49.573633085 +0900
@@ -20,27 +20,27 @@
         - uri: http://127.0.0.200:7777
   ngap:
     server:
-      - address: 127.0.0.5
+      - address: 192.168.0.111
   metrics:
     server:
       - address: 127.0.0.5
         port: 9090
   guami:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       amf_id:
         region: 2
         set: 1
   tai:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       tac: 1
   plmn_support:
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
       s_nssai:
         - sst: 1
   security:
```
- `open5gs/install/etc/open5gs/nrf.yaml`
```diff
--- nrf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ nrf.yaml    2025-05-04 08:13:05.973154453 +0900
@@ -11,8 +11,8 @@
 nrf:
   serving:  # 5G roaming requires PLMN in NRF
     - plmn_id:
-        mcc: 999
-        mnc: 70
+        mcc: 001
+        mnc: 01
   sbi:
     server:
       - address: 127.0.0.10
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2025-01-15 04:12:06.000000000 +0900
+++ smf.yaml    2025-01-15 04:26:36.000000000 +0900
@@ -20,16 +20,14 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.14.111
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.14.151
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.14.111
   metrics:
     server:
       - address: 127.0.0.4
@@ -37,20 +35,17 @@
   session:
     - subnet: 10.45.0.0/16
       gateway: 10.45.0.1
-    - subnet: 2001:db8:cafe::/48
-      gateway: 2001:db8:cafe::1
+      dnn: internet
   dns:
     - 8.8.8.8
     - 8.8.4.4
-    - 2001:4860:4860::8888
-    - 2001:4860:4860::8844
   mtu: 1400
 #  p-cscf:
 #    - 127.0.0.1
 #    - ::1
 #  ctf:
 #    enabled: auto   # auto(default)|yes|no
-  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
+#  freeDiameter: /root/open5gs/install/etc/freeDiameter/smf.conf
 
 ################################################################################
 # SMF Info
```

<a id="changes_up"></a>

### Changes in configuration files of Open5GS 5GC U-Plane

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-05-02 19:52:00.000000000 +0900
+++ upf.yaml    2024-05-19 12:38:00.000000000 +0900
@@ -11,18 +11,18 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.14.151
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.13.151
   session:
     - subnet: 10.45.0.0/16
       gateway: 10.45.0.1
-    - subnet: 2001:db8:cafe::/48
-      gateway: 2001:db8:cafe::1
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2026-06-20 16:34:40.000000000 +0900
+++ open5gs-gnb.yaml    2026-06-20 17:11:26.359194021 +0900
@@ -1,17 +1,17 @@
-mcc: '999'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
 linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.13.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.5
+  - address: 192.168.0.111
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<a id="changes_ue0"></a>

#### Changes in configuration files of UE0 (IMSI-001010000000000)

First, copy `open5gs-ue0.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue0.yaml
```
Next, edit `open5gs-ue0.yaml`.
- `UERANSIM/config/open5gs-ue0.yaml`
```diff
--- open5gs-ue.yaml.orig        2026-09-07 04:20:36.000000000 +0900
+++ open5gs-ue0.yaml    2026-09-16 23:29:48.866981423 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI
@@ -32,7 +32,7 @@
 tunNetmask: '255.255.255.0'
 
 # Create the UE TUN interface inside a dedicated Linux network namespace.
-useNamespace: false
+useNamespace: true
 
 # Optional prefix used when deriving the namespace name.
 nsNamePrefix: 'ueransim'
```

<a id="changes_ue1"></a>

#### Changes in configuration files of UE1 (IMSI-001010000000001)

First, copy `open5gs-ue1.yaml` from `open5gs-ue.yaml`.
```
# cd UERANSIM/config
# cp open5gs-ue.yaml open5gs-ue1.yaml
```
Next, edit `open5gs-ue1.yaml`.
- `UERANSIM/config/open5gs-ue1.yaml`
```diff
--- open5gs-ue.yaml.orig        2026-09-07 04:20:36.000000000 +0900
+++ open5gs-ue1.yaml    2026-09-16 23:30:06.019079454 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-999700000000001'
+supi: 'imsi-001010000000001'
 # Mobile Country Code value of HPLMN
-mcc: '999'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '70'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI
@@ -32,7 +32,7 @@
 tunNetmask: '255.255.255.0'
 
 # Create the UE TUN interface inside a dedicated Linux network namespace.
-useNamespace: false
+useNamespace: true
 
 # Optional prefix used when deriving the namespace name.
 nsNamePrefix: 'ueransim'
```

<a id="network_settings"></a>

## Network settings of Open5GS 5GC and UERANSIM UE / RAN

<a id="network_settings_up"></a>

### Network settings of Open5GS 5GC U-Plane

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure the TUNnel interface and set the routings towards Framed Routes.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

ip route add 192.168.20.0/24 dev ogstun
ip route add 192.168.21.0/24 dev ogstun
ip route add 192.168.22.0/24 dev ogstun
ip route add 192.168.23.0/24 dev ogstun
```

<a id="network_settings_ext"></a>

### Network settings of External Node

Set the routings towards UEs and Framed Routes.
```
ip route add 10.45.0.0/16 via 192.168.16.151
ip route add 192.168.20.0/24 via 192.168.16.151
ip route add 192.168.21.0/24 via 192.168.16.151
ip route add 192.168.22.0/24 via 192.168.16.151
ip route add 192.168.23.0/24 via 192.168.16.151
```

<a id="network_settings_vm3"></a>

### Network settings of VM3

Delete default GW.
```
# ip route del default
```

<a id="add_netns"></a>

#### Add netns

First, create 3 netns for the terminals.
```
ip netns add pc1
ip netns add pc2
ip netns add pc3
ip netns add pc4
```
From here on, I will explain how to setup netns and veth, but please note that these settings will be applied to the netns created by running UE0 and UE1.
In other words, run UE0 and UE1 before performing these operations.

<a id="setup_ue0"></a>

#### Setup veth pair for UE0 and PC1/PC4

This explanation assumes that running UE0 will create `ueransim-001010000000000-internet-psi1` as netns.

First, move to netns:`ueransim-001010000000000-internet-psi1`.
```
ip netns exec ueransim-001010000000000-internet-psi1 bash
```
Enable IP forwarding.
```
sysctl -w net.ipv4.ip_forward=1
```
Create `veth-ue0-pc1` and `veth-pc1`, then move `veth-pc1` to netns:`pc1`. Assign `192.168.20.1/24` to `veth-ue0-pc1` and enable `veth-ue0-pc1`.
```
ip link add veth-ue0-pc1 type veth peer name veth-pc1
ip link set veth-pc1 netns pc1
ip addr add 192.168.20.1/24 dev veth-ue0-pc1
ip link set veth-ue0-pc1 up
```
Similarly, create `veth-ue0-pc4` and `veth-pc4`, then move `veth-pc4` to netns:`pc4`. Assign `192.168.23.1/24` to `veth-ue0-pc4` and enable `veth-ue0-pc4`.
```
ip link add veth-ue0-pc4 type veth peer name veth-pc4
ip link set veth-pc4 netns pc4
ip addr add 192.168.23.1/24 dev veth-ue0-pc4
ip link set veth-ue0-pc4 up
```
Next, move to netns:`pc1`.
```
ip netns exec pc1 bash
```
Assign `192.168.20.100/24` ​​to `veth-pc1` and enable `veth-pc1`. Then, set `192.168.20.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.20.100/24 dev veth-pc1
ip link set veth-pc1 up
ip route add default via 192.168.20.1 dev veth-pc1
ip link set lo up
```
Similarly, move to netns:`pc4`.
```
ip netns exec pc4 bash
```
Assign `192.168.23.100/24` ​​to `veth-pc4` and enable `veth-pc4`. Then, set `192.168.23.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.23.100/24 dev veth-pc4
ip link set veth-pc4 up
ip route add default via 192.168.23.1 dev veth-pc4
ip link set lo up
```

<a id="setup_ue1"></a>

#### Setup veth pair for UE1 and PC2/PC3

This explanation assumes that running UE1 will create `ueransim-001010000000001-internet-psi1` as netns.

First, move to netns:`ueransim-001010000000001-internet-psi1`.
```
ip netns exec ueransim-001010000000001-internet-psi1 bash
```
Enable IP forwarding.
```
sysctl -w net.ipv4.ip_forward=1
```
Create `veth-ue1-pc2` and `veth-pc2`, then move `veth-pc2` to netns:`pc2`. Assign `192.168.21.1/24` to `veth-ue1-pc2` and enable `veth-ue1-pc2`.
```
ip link add veth-ue1-pc2 type veth peer name veth-pc2
ip link set veth-pc2 netns pc2
ip addr add 192.168.21.1/24 dev veth-ue1-pc2
ip link set veth-ue1-pc2 up
```
Similarly, create `veth-ue1-pc3` and `veth-pc3`, then move `veth-pc3` to netns:`pc3`. Assign `192.168.22.1/24` to `veth-ue1-pc3` and enable `veth-ue1-pc3`.
```
ip link add veth-ue1-pc3 type veth peer name veth-pc3
ip link set veth-pc3 netns pc3
ip addr add 192.168.22.1/24 dev veth-ue1-pc3
ip link set veth-ue1-pc3 up
```
Next, move to netns:`pc2`.
```
ip netns exec pc2 bash
```
Assign `192.168.21.100/24` ​​to `veth-pc2` and enable `veth-pc2`. Then, set `192.168.21.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.21.100/24 dev veth-pc2
ip link set veth-pc2 up
ip route add default via 192.168.21.1 dev veth-pc2
ip link set lo up
```
Similarly, move to netns:`pc3`.
```
ip netns exec pc3 bash
```
Assign `192.168.22.100/24` ​​to `veth-pc3` and enable `veth-pc3`. Then, set `192.168.22.1` as the default route. Finally, enable interface `lo`.
```
ip addr add 192.168.22.100/24 dev veth-pc3
ip link set veth-pc3 up
ip route add default via 192.168.22.1 dev veth-pc3
ip link set lo up
```

<a id="add_framed_routes"></a>

## Add Framed Routes to Subscriber information

[MongoDB Compass](https://www.mongodb.com/products/compass) is a useful GUI tool for working with MongoDB data.
I used this tool to add Framed Routes in the following operations.

<a id="add_framed_routes_ue0"></a>

### Add Framed Routes to UE0

The UE0's sample subscriber information registered in MongoDB is as follows in JSON format.
Among these, the items indicated by the arrows are Framed Routes to be added.
```json
{
  "_id": {
    "$oid": "672e210b2a5baf13e3c51a26"
  },
  "ambr": {
    "downlink": {
      "value": 1,
      "unit": 3
    },
    "uplink": {
      "value": 1,
      "unit": 3
    }
  },
  "schema_version": 1,
  "msisdn": [],
  "imeisv": "4370816125816151",
  "mme_host": [],
  "mme_realm": [],
  "purge_flag": [],
  "access_restriction_data": 32,
  "subscriber_status": 0,
  "operator_determined_barring": 0,
  "network_access_mode": 0,
  "subscribed_rau_tau_timer": 12,
  "imsi": "001010000000000",
  "security": {
    "k": "465B5CE8 B199B49F AA5F0A2E E238A6BC",
    "amf": "8000",
    "op": null,
    "opc": "E8ED289D EBA952E4 283B54E8 8E6183CA",
    "sqn": {
      "$numberLong": "1344"
    }
  },
  "slice": [
    {
      "_id": {
        "$oid": "672e210b2a5baf13e3c51a27"
      },
      "sst": 1,
      "default_indicator": true,
      "session": [
        {
          "qos": {
            "arp": {
              "priority_level": 8,
              "pre_emption_capability": 1,
              "pre_emption_vulnerability": 1
            },
            "index": 9
          },
          "ambr": {
            "downlink": {
              "value": 1,
              "unit": 3
            },
            "uplink": {
              "value": 1,
              "unit": 3
            }
          },
          "_id": {
            "$oid": "672e210b2a5baf13e3c51a28"
          },
-->       "ipv4_framed_routes": [
-->         "192.168.20.0/24"
-->       ],
          "name": "internet",
          "type": 1,
          "pcc_rule": []
        }
      ]
    }
  ],
  "__v": 0
}
```

<a id="add_framed_routes_ue1"></a>

### Add Framed Routes to UE1

The UE1's sample subscriber information registered in MongoDB is as follows in JSON format.
Among these, the items indicated by the arrows are Framed Routes to be added.
```json
{
  "_id": {
    "$oid": "691e43a812ac4d03469f1cff"
  },
  "ambr": {
    "downlink": {
      "value": 1,
      "unit": 3
    },
    "uplink": {
      "value": 1,
      "unit": 3
    }
  },
  "schema_version": 1,
  "msisdn": [],
  "imeisv": "4370816125816151",
  "mme_host": [],
  "mme_realm": [],
  "purge_flag": [],
  "access_restriction_data": 32,
  "subscriber_status": 0,
  "operator_determined_barring": 0,
  "network_access_mode": 0,
  "subscribed_rau_tau_timer": 12,
  "imsi": "001010000000001",
  "security": {
    "k": "465B5CE8 B199B49F AA5F0A2E E238A6BC",
    "amf": "8000",
    "op": null,
    "opc": "E8ED289D EBA952E4 283B54E8 8E6183CA",
    "sqn": {
      "$numberLong": "385"
    }
  },
  "slice": [
    {
      "_id": {
        "$oid": "691e43a812ac4d03469f1d00"
      },
      "sst": 1,
      "default_indicator": true,
      "session": [
        {
          "qos": {
            "arp": {
              "priority_level": 8,
              "pre_emption_capability": 1,
              "pre_emption_vulnerability": 1
            },
            "index": 9
          },
          "ambr": {
            "downlink": {
              "value": 1,
              "unit": 3
            },
            "uplink": {
              "value": 1,
              "unit": 3
            }
          },
          "_id": {
            "$oid": "691e43a812ac4d03469f1d01"
          },
-->       "ipv4_framed_routes": [
-->         "192.168.21.0/24",
-->         "192.168.22.0/24"
-->       ],
          "name": "internet",
          "type": 1,
          "pcc_rule": []
        }
      ]
    }
  ],
  "__v": 0
}
```

<a id="build"></a>

## Build Open5GS and UERANSIM

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.8.0 (2026.09.16) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.3.0 (2026.09.06) - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on Open5GS 5GC C-Plane machine.
It is not necessary to install MongoDB on Open5GS 5GC U-Plane machine.

<a id="run"></a>

## Run Open5GS 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_cp"></a>

### Run Open5GS 5GC C-Plane

First, run Open5GS 5GC C-Plane.

- Open5GS 5GC C-Plane
```
./install/bin/open5gs-nrfd &
sleep 2
./install/bin/open5gs-scpd &
sleep 2
./install/bin/open5gs-amfd &
sleep 2
./install/bin/open5gs-smfd &
./install/bin/open5gs-ausfd &
./install/bin/open5gs-udmd &
./install/bin/open5gs-udrd &
./install/bin/open5gs-pcfd &
./install/bin/open5gs-nssfd &
./install/bin/open5gs-bsfd &
./install/bin/open5gs-eird &
```

<a id="run_up"></a>

### Run Open5GS 5GC U-Plane

Next, run Open5GS 5GC U-Plane.
```
./install/bin/open5gs-upfd &
```

<a id="run_ueran"></a>

### Run UERANSIM

First, do an NG Setup between gNodeB and 5GC, then register the UE with 5GC and establish a PDU session.

Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="start_gnb"></a>

#### Start gNodeB

Start gNodeB as follows.
```
# ./nr-gnb -c ../config/open5gs-gnb.yaml
UERANSIM v3.3.0
[2026-09-17 00:14:39.129] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2026-09-17 00:14:39.133] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2026-09-17 00:14:39.133] [sctp] [debug] SCTP association setup ascId[8]
[2026-09-17 00:14:39.133] [ngap] [debug] Sending NG Setup Request
[2026-09-17 00:14:39.140] [ngap] [debug] NG Setup Response received
[2026-09-17 00:14:39.140] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
09/17 00:14:39.148: [amf] INFO: gNB-N2 accepted[192.168.0.131]:38756 in ng-path module (../src/amf/ngap-sctp.c:113)
09/17 00:14:39.148: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:823)
09/17 00:14:39.153: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1349)
09/17 00:14:39.154: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:870)
```

<a id="start_ue0"></a>

#### Start UE0

Start UE0 as follows. This will register the UE with 5GC and establish a PDU session.
Also, UE0 moves to netns:`ueransim-001010000000000-internet-psi1` and runs there.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml
UERANSIM v3.3.0
[2026-09-17 00:15:28.022] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2026-09-17 00:15:28.022] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2026-09-17 00:15:28.023] [nas] [info] Selected plmn[001/01]
[2026-09-17 00:15:28.023] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2026-09-17 00:15:28.023] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2026-09-17 00:15:28.023] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2026-09-17 00:15:28.023] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2026-09-17 00:15:28.023] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-09-17 00:15:28.023] [nas] [debug] Sending Initial Registration
[2026-09-17 00:15:28.024] [rrc] [debug] Sending RRC Setup Request
[2026-09-17 00:15:28.024] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2026-09-17 00:15:28.024] [rrc] [info] RRC connection established
[2026-09-17 00:15:28.024] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2026-09-17 00:15:28.024] [nas] [info] UE switches to state [CM-CONNECTED]
[2026-09-17 00:15:28.032] [nas] [debug] Authentication Request received
[2026-09-17 00:15:28.032] [nas] [debug] Received SQN [000000001061]
[2026-09-17 00:15:28.032] [nas] [debug] SQN-MS [000000000000]
[2026-09-17 00:15:28.037] [nas] [debug] Security Mode Command received
[2026-09-17 00:15:28.037] [nas] [debug] Selected integrity[2] ciphering[0]
[2026-09-17 00:15:28.047] [nas] [debug] Registration accept received
[2026-09-17 00:15:28.047] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2026-09-17 00:15:28.047] [nas] [debug] Sending Registration Complete
[2026-09-17 00:15:28.047] [nas] [info] Initial Registration is successful
[2026-09-17 00:15:28.047] [nas] [debug] Sending PDU Session Establishment Request
[2026-09-17 00:15:28.047] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-09-17 00:15:28.252] [nas] [debug] Configuration Update Command received
[2026-09-17 00:15:28.266] [nas] [debug] PDU Session Establishment Accept received
[2026-09-17 00:15:28.267] [nas] [info] PDU Session establishment is successful PSI[1]
Cannot open network namespace "ueransim-001010000000000-internet-psi1": No such file or directory
[2026-09-17 00:15:28.317] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up in namespace[ueransim-001010000000000-internet-psi1].
```
The Open5GS C-Plane log when executed is as follows.
```
09/17 00:15:28.039: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:668)
09/17 00:15:28.039: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:3049)
09/17 00:15:28.039: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:884)
09/17 00:15:28.039: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:2064)
09/17 00:15:28.039: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1818)
09/17 00:15:28.039: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1709)
09/17 00:15:28.040: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:186)
09/17 00:15:28.040: [sbi] INFO: [4c684d0e-b1e1-41f1-932f-e51eb2123a0a] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:349)
09/17 00:15:28.040: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.041: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:15:28.041: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.042: [nrf] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520:0] NF-Discovered [NF-Type:UDR,NF-Status:REGISTERED,IPv4:1,IPv6:0,FQDN:NULL] (../src/nrf/nnrf-handler.c:1226)
09/17 00:15:28.042: [sbi] WARNING: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NRF-discover) NF has already been added [type:UDR] (../lib/sbi/nnrf-handler.c:1561)
09/17 00:15:28.042: [sbi] INFO: [UDR] NFInstance associated [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2498)
09/17 00:15:28.042: [sbi] WARNING: UnRef NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:15:28.042: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:15:28.042: [sbi] INFO: [nudr-dr] NFService associated [4c6f7836-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2138)
09/17 00:15:28.043: [sbi] WARNING: UnRef NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:15:28.043: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:15:28.043: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NF-discover) NF Profile updated [type:UDR validity:30s] (../lib/sbi/nnrf-handler.c:1609)
09/17 00:15:28.045: [sbi] INFO: [UDR] (SCP-discover) NF registered [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/path.c:212)
09/17 00:15:28.045: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:227)
09/17 00:15:28.046: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:152)
09/17 00:15:28.047: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.048: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.050: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:339)
09/17 00:15:28.051: [gmm] INFO: [imsi-001010000000000] Security mode complete (../src/amf/gmm-sm.c:2784)
09/17 00:15:28.052: [gmm] INFO: [imsi-001010000000000] Skip 5G-EIR check [message:65,enabled:0] (../src/amf/gmm-sm.c:2683)
09/17 00:15:28.052: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:15:28.052: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.054: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:15:28.054: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.056: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.057: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.057: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:431)
09/17 00:15:28.057: [sbi] INFO: [4c6ac4b2-b1e1-41f1-aa22-e5d78e651783] Setup NF Instance [type:PCF] (../lib/sbi/path.c:349)
09/17 00:15:28.058: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.058: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:150)
09/17 00:15:28.059: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:349)
09/17 00:15:28.059: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.061: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
09/17 00:15:28.267: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:3458)
09/17 00:15:28.267: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:609)
09/17 00:15:28.267: [gmm] INFO:     UTC [2026-09-16T15:15:28] Timezone[0]/DST[0] (../src/amf/gmm-build.c:556)
09/17 00:15:28.267: [gmm] INFO:     LOCAL [2026-09-17T00:15:28] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:561)
09/17 00:15:28.267: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:3070)
09/17 00:15:28.267: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] LBO[0] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1452)
09/17 00:15:28.267: [gmm] INFO: V-SMF Instance [4c7ecd5e-b1e1-41f1-b051-1b6e94afe145](LIST) (../src/amf/gmm-handler.c:1529)
09/17 00:15:28.267: [gmm] INFO: [4c7ecd5e-b1e1-41f1-b051-1b6e94afe145] Setup NF Instance [type:SMF] (../src/amf/gmm-handler.c:1531)
09/17 00:15:28.267: [gmm] INFO: V-SMF Instance [4c7ecd5e-b1e1-41f1-b051-1b6e94afe145] (../src/amf/gmm-handler.c:1541)
09/17 00:15:28.267: [gmm] INFO: V-SMF discovered in Non-Roaming or LBO-Roaming[0] (../src/amf/gmm-handler.c:1610)
09/17 00:15:28.267: [gmm] INFO: nsmf_pdusession [1:0x56d826459ef8:(nil)] (../src/amf/gmm-handler.c:1650)
09/17 00:15:28.267: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.268: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1069)
09/17 00:15:28.268: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3625)
09/17 00:15:28.268: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:326)
09/17 00:15:28.268: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:15:28.269: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.271: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.272: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:473)
09/17 00:15:28.272: [sbi] INFO: [4c6ac4b2-b1e1-41f1-aa22-e5d78e651783] Setup NF Instance [type:PCF] (../lib/sbi/path.c:349)
09/17 00:15:28.272: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.272: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:140)
09/17 00:15:28.273: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:482)
09/17 00:15:28.273: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:349)
09/17 00:15:28.273: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.274: [sbi] INFO: [4c67ed0a-b1e1-41f1-a58d-7f57fe11bac2] Setup NF Instance [type:BSF] (../lib/sbi/path.c:349)
09/17 00:15:28.274: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.275: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:125)
09/17 00:15:28.276: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:414)
09/17 00:15:28.276: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:657)
09/17 00:15:28.276: [pfcp] INFO: PFCP encode Framed-Route in PDR[1]: 192.168.20.0/24 0.0.0.0 1 (../lib/pfcp/build.c:365)
09/17 00:15:28.277: [pfcp] INFO: PFCP encode Framed-Route in PDR[2]: 192.168.20.0/24 0.0.0.0 1 (../lib/pfcp/build.c:365)
09/17 00:15:28.278: [gtp] INFO: gtp_connect() [192.168.13.151]:2152 (../lib/gtp/path.c:60)
09/17 00:15:28.278: [sbi] INFO: [4b2f1c60-b1e1-41f1-9d57-b7389ac6d53e] Setup NF Instance [type:AMF] (../lib/sbi/path.c:349)
09/17 00:15:28.278: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.281: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.282: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:15:28.282: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:15:28.283: [nrf] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520:0] NF-Discovered [NF-Type:UDR,NF-Status:REGISTERED,IPv4:1,IPv6:0,FQDN:NULL] (../src/nrf/nnrf-handler.c:1226)
09/17 00:15:28.284: [sbi] WARNING: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NRF-discover) NF has already been added [type:UDR] (../lib/sbi/nnrf-handler.c:1561)
09/17 00:15:28.284: [sbi] INFO: [UDR] NFInstance associated [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2498)
09/17 00:15:28.284: [sbi] WARNING: UnRef NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:15:28.284: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:15:28.284: [sbi] INFO: [nudr-dr] NFService associated [4c6f7836-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2138)
09/17 00:15:28.284: [sbi] WARNING: UnRef NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:15:28.284: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:15:28.284: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NF-discover) NF Profile updated [type:UDR validity:30s] (../lib/sbi/nnrf-handler.c:1609)
09/17 00:15:28.284: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/path.c:217)
09/17 00:15:28.284: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:227)
09/17 00:15:28.285: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:1036)
```
The Open5GS U-Plane log when executed is as follows.
```
09/17 00:15:28.162: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:231)
09/17 00:15:28.162: [pfcp] INFO: Apply Create PDR: PDR-ID[1] (../lib/pfcp/handler.c:886)
09/17 00:15:28.162: [pfcp] INFO: Apply Create PDR: PDR-ID[2] (../lib/pfcp/handler.c:886)
09/17 00:15:28.162: [pfcp] INFO: Apply Create PDR: PDR-ID[3] (../lib/pfcp/handler.c:886)
09/17 00:15:28.163: [pfcp] INFO: Apply Create PDR: PDR-ID[4] (../lib/pfcp/handler.c:886)
09/17 00:15:28.163: [pfcp] INFO: Apply Create FAR: FAR-ID[1] (../lib/pfcp/handler.c:1399)
09/17 00:15:28.163: [pfcp] INFO: Apply Create FAR: FAR-ID[2] (../lib/pfcp/handler.c:1399)
09/17 00:15:28.163: [pfcp] INFO: Apply Create FAR: FAR-ID[3] (../lib/pfcp/handler.c:1399)
09/17 00:15:28.163: [pfcp] INFO: Apply Create URR: URR-ID[1] (../lib/pfcp/handler.c:1966)
09/17 00:15:28.163: [pfcp] INFO: Apply Create QER: QER-ID[1] (../lib/pfcp/handler.c:1669)
09/17 00:15:28.163: [pfcp] INFO: Apply Create BAR: BAR-ID[1] (../lib/pfcp/handler.c:1799)
09/17 00:15:28.163: [gtp] INFO: gtp_connect() [192.168.14.111]:2152 (../lib/gtp/path.c:60)
09/17 00:15:28.163: [pfcp] WARNING: Set FAR-ID[3] GTP-U peer [TEID:0x1] (../lib/pfcp/context.c:1262)
09/17 00:15:28.163: [pfcp] INFO: Register Error Indication F-TEID[0x1] [FAR-ID:3] (../lib/pfcp/context.c:2047)
09/17 00:15:28.163: [upf] INFO: UE F-SEID[UP:0x455 CP:0xd01] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:575)
09/17 00:15:28.163: [upf] INFO: UPF registered IPv4 framed route[192.168.20.0/24 0.0.0.0 1] for SEID[0x455] (../src/upf/context.c:724)
09/17 00:15:28.163: [upf] INFO: UPF registered IPv4 framed route[192.168.20.0/24 0.0.0.0 1] for SEID[0x455] (../src/upf/context.c:724)
09/17 00:15:28.163: [pfcp] INFO: Register local F-TEID[0xeb4c] [PDR-ID:2 type:1] (../lib/pfcp/context.c:1623)
09/17 00:15:28.163: [pfcp] INFO: Register local F-TEID[0x4424] [PDR-ID:3 type:1] (../lib/pfcp/context.c:1623)
09/17 00:15:28.167: [upf] INFO: Session Modification Request [xid:9] [UP-SEID:0x455 CP-SEID:0xd01] (../src/upf/n4-handler.c:312)
09/17 00:15:28.167: [pfcp] INFO: Mark rules [PDR:4 FAR:3 URR:1 QER:1 BAR:1] (../lib/pfcp/context.c:1377)
09/17 00:15:28.167: [gtp] INFO: gtp_connect() [192.168.13.131]:2152 (../lib/gtp/path.c:60)
09/17 00:15:28.167: [pfcp] WARNING: Set FAR-ID[1] GTP-U peer [TEID:0x1] (../lib/pfcp/context.c:1262)
09/17 00:15:28.167: [pfcp] INFO: Register Error Indication F-TEID[0x1] [FAR-ID:1] (../lib/pfcp/context.c:2047)
09/17 00:15:28.167: [pfcp] INFO: Updated FAR GTP-U tunnel: FAR-ID[1] TEID[0x0->0x1] (../lib/pfcp/handler.c:1544)
09/17 00:15:28.167: [pfcp] INFO: Apply Update FAR: FAR-ID[1] (../lib/pfcp/handler.c:1551)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2026-09-17 00:15:28.317] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up in namespace[ueransim-001010000000000-internet-psi1].
```
Just in case, after logging in VM3 from another terminal, move to netns:`ueransim-001010000000000-internet-psi1` and make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip netns exec ueransim-001010000000000-internet-psi1 ip addr show
...
10: uesimtun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::c673:5c21:84d3:bfdf/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
**Don't forget [Setup veth pair for UE0 and PC1/PC4](#setup_ue0).**

<a id="start_ue1"></a>

#### Start UE1

Start UE1 as follows. This will register the UE with 5GC and establish a PDU session.
Also, UE1 moves to netns:`ueransim-001010000000001-internet-psi1` and runs there.
```
# ./nr-ue -c ../config/open5gs-ue1.yaml 
UERANSIM v3.3.0
[2026-09-17 00:19:47.628] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2026-09-17 00:19:47.628] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2026-09-17 00:19:47.629] [nas] [info] Selected plmn[001/01]
[2026-09-17 00:19:47.629] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2026-09-17 00:19:47.629] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2026-09-17 00:19:47.629] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2026-09-17 00:19:47.629] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2026-09-17 00:19:47.629] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-09-17 00:19:47.630] [nas] [debug] Sending Initial Registration
[2026-09-17 00:19:47.630] [rrc] [debug] Sending RRC Setup Request
[2026-09-17 00:19:47.630] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2026-09-17 00:19:47.630] [rrc] [info] RRC connection established
[2026-09-17 00:19:47.630] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2026-09-17 00:19:47.630] [nas] [info] UE switches to state [CM-CONNECTED]
[2026-09-17 00:19:47.639] [nas] [debug] Authentication Request received
[2026-09-17 00:19:47.639] [nas] [debug] Received SQN [000000000601]
[2026-09-17 00:19:47.639] [nas] [debug] SQN-MS [000000000000]
[2026-09-17 00:19:47.643] [nas] [debug] Security Mode Command received
[2026-09-17 00:19:47.643] [nas] [debug] Selected integrity[2] ciphering[0]
[2026-09-17 00:19:47.653] [nas] [debug] Registration accept received
[2026-09-17 00:19:47.653] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2026-09-17 00:19:47.653] [nas] [debug] Sending Registration Complete
[2026-09-17 00:19:47.653] [nas] [info] Initial Registration is successful
[2026-09-17 00:19:47.653] [nas] [debug] Sending PDU Session Establishment Request
[2026-09-17 00:19:47.653] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2026-09-17 00:19:47.861] [nas] [debug] Configuration Update Command received
[2026-09-17 00:19:47.874] [nas] [debug] PDU Session Establishment Accept received
[2026-09-17 00:19:47.875] [nas] [info] PDU Session establishment is successful PSI[1]
Cannot open network namespace "ueransim-001010000000001-internet-psi1": No such file or directory
[2026-09-17 00:19:47.923] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up in namespace[ueransim-001010000000001-internet-psi1].
```
The Open5GS C-Plane log when executed is as follows.
```
09/17 00:19:47.645: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:668)
09/17 00:19:47.646: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:3049)
09/17 00:19:47.646: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:884)
09/17 00:19:47.646: [amf] INFO: [suci-0-001-01-0000-0-0-0000000001] Unknown UE by SUCI (../src/amf/context.c:2064)
09/17 00:19:47.646: [amf] INFO: [Added] Number of AMF-UEs is now 2 (../src/amf/context.c:1818)
09/17 00:19:47.646: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1709)
09/17 00:19:47.646: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:186)
09/17 00:19:47.646: [sbi] INFO: [4c684d0e-b1e1-41f1-932f-e51eb2123a0a] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:349)
09/17 00:19:47.646: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.647: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:19:47.647: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.648: [nrf] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520:0] NF-Discovered [NF-Type:UDR,NF-Status:REGISTERED,IPv4:1,IPv6:0,FQDN:NULL] (../src/nrf/nnrf-handler.c:1226)
09/17 00:19:47.649: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NRF-discover) NF registered [type:UDR] (../lib/sbi/nnrf-handler.c:1557)
09/17 00:19:47.649: [sbi] INFO: [UDR] NFInstance associated [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2498)
09/17 00:19:47.649: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:19:47.649: [sbi] INFO: [nudr-dr] NFService associated [4c6f7836-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2138)
09/17 00:19:47.649: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:19:47.649: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NF-discover) NF Profile updated [type:UDR validity:30s] (../lib/sbi/nnrf-handler.c:1609)
09/17 00:19:47.651: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/path.c:217)
09/17 00:19:47.651: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:227)
09/17 00:19:47.652: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:152)
09/17 00:19:47.654: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.654: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.657: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:339)
09/17 00:19:47.658: [gmm] INFO: [imsi-001010000000001] Security mode complete (../src/amf/gmm-sm.c:2784)
09/17 00:19:47.658: [gmm] INFO: [imsi-001010000000001] Skip 5G-EIR check [message:65,enabled:0] (../src/amf/gmm-sm.c:2683)
09/17 00:19:47.658: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:19:47.658: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.660: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:19:47.660: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.662: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.663: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.663: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:431)
09/17 00:19:47.663: [sbi] INFO: [4c6ac4b2-b1e1-41f1-aa22-e5d78e651783] Setup NF Instance [type:PCF] (../lib/sbi/path.c:349)
09/17 00:19:47.664: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.664: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:150)
09/17 00:19:47.664: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:349)
09/17 00:19:47.665: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.666: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
09/17 00:19:47.874: [gmm] INFO: [imsi-001010000000001] Registration complete (../src/amf/gmm-sm.c:3458)
09/17 00:19:47.875: [amf] INFO: [imsi-001010000000001] Configuration update command (../src/amf/nas-path.c:609)
09/17 00:19:47.875: [gmm] INFO:     UTC [2026-09-16T15:19:47] Timezone[0]/DST[0] (../src/amf/gmm-build.c:556)
09/17 00:19:47.875: [gmm] INFO:     LOCAL [2026-09-17T00:19:47] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:561)
09/17 00:19:47.875: [amf] INFO: [Added] Number of AMF-Sessions is now 2 (../src/amf/context.c:3070)
09/17 00:19:47.875: [gmm] INFO: UE SUPI[imsi-001010000000001] DNN[internet] LBO[0] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1452)
09/17 00:19:47.875: [gmm] INFO: V-SMF Instance [4c7ecd5e-b1e1-41f1-b051-1b6e94afe145](LIST) (../src/amf/gmm-handler.c:1529)
09/17 00:19:47.875: [gmm] INFO: [4c7ecd5e-b1e1-41f1-b051-1b6e94afe145] Setup NF Instance [type:SMF] (../src/amf/gmm-handler.c:1531)
09/17 00:19:47.875: [gmm] INFO: V-SMF Instance [4c7ecd5e-b1e1-41f1-b051-1b6e94afe145] (../src/amf/gmm-handler.c:1541)
09/17 00:19:47.875: [gmm] INFO: V-SMF discovered in Non-Roaming or LBO-Roaming[0] (../src/amf/gmm-handler.c:1610)
09/17 00:19:47.875: [gmm] INFO: nsmf_pdusession [1:0x56d826459ef8:(nil)] (../src/amf/gmm-handler.c:1650)
09/17 00:19:47.875: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.876: [smf] INFO: [Added] Number of SMF-UEs is now 2 (../src/smf/context.c:1069)
09/17 00:19:47.876: [smf] INFO: [Added] Number of SMF-Sessions is now 2 (../src/smf/context.c:3625)
09/17 00:19:47.876: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:326)
09/17 00:19:47.876: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:19:47.876: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.879: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.879: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:473)
09/17 00:19:47.879: [sbi] INFO: [4c6ac4b2-b1e1-41f1-aa22-e5d78e651783] Setup NF Instance [type:PCF] (../lib/sbi/path.c:349)
09/17 00:19:47.880: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.880: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:140)
09/17 00:19:47.880: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:482)
09/17 00:19:47.881: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:349)
09/17 00:19:47.881: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.882: [sbi] INFO: [4c67ed0a-b1e1-41f1-a58d-7f57fe11bac2] Setup NF Instance [type:BSF] (../lib/sbi/path.c:349)
09/17 00:19:47.882: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.883: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:125)
09/17 00:19:47.884: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:414)
09/17 00:19:47.884: [smf] INFO: UE SUPI[imsi-001010000000001] DNN[internet] IPv4[10.45.0.3] IPv6[] (../src/smf/npcf-handler.c:657)
09/17 00:19:47.884: [pfcp] INFO: PFCP encode Framed-Route in PDR[1]: 192.168.21.0/24 0.0.0.0 1 (../lib/pfcp/build.c:365)
09/17 00:19:47.884: [pfcp] INFO: PFCP encode Framed-Route in PDR[1]: 192.168.22.0/24 0.0.0.0 1 (../lib/pfcp/build.c:365)
09/17 00:19:47.884: [pfcp] INFO: PFCP encode Framed-Route in PDR[2]: 192.168.21.0/24 0.0.0.0 1 (../lib/pfcp/build.c:365)
09/17 00:19:47.884: [pfcp] INFO: PFCP encode Framed-Route in PDR[2]: 192.168.22.0/24 0.0.0.0 1 (../lib/pfcp/build.c:365)
09/17 00:19:47.885: [sbi] INFO: [4b2f1c60-b1e1-41f1-9d57-b7389ac6d53e] Setup NF Instance [type:AMF] (../lib/sbi/path.c:349)
09/17 00:19:47.886: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.889: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.890: [sbi] INFO: [4c68cefa-b1e1-41f1-981b-e723b74875cd] Setup NF Instance [type:UDM] (../lib/sbi/path.c:349)
09/17 00:19:47.890: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:583)
09/17 00:19:47.891: [nrf] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520:0] NF-Discovered [NF-Type:UDR,NF-Status:REGISTERED,IPv4:1,IPv6:0,FQDN:NULL] (../src/nrf/nnrf-handler.c:1226)
09/17 00:19:47.891: [sbi] WARNING: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NRF-discover) NF has already been added [type:UDR] (../lib/sbi/nnrf-handler.c:1561)
09/17 00:19:47.891: [sbi] INFO: [UDR] NFInstance associated [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2498)
09/17 00:19:47.891: [sbi] WARNING: UnRef NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:19:47.891: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:80] (../lib/sbi/context.c:2503)
09/17 00:19:47.891: [sbi] INFO: [nudr-dr] NFService associated [4c6f7836-b1e1-41f1-8265-197911ad5520] (../lib/sbi/context.c:2138)
09/17 00:19:47.891: [sbi] WARNING: UnRef NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:19:47.891: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../lib/sbi/context.c:2141)
09/17 00:19:47.891: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] (NF-discover) NF Profile updated [type:UDR validity:30s] (../lib/sbi/nnrf-handler.c:1609)
09/17 00:19:47.892: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [4c69ec4a-b1e1-41f1-8265-197911ad5520] (../lib/sbi/path.c:217)
09/17 00:19:47.892: [sbi] INFO: [4c69ec4a-b1e1-41f1-8265-197911ad5520] Setup NF Instance [type:UDR] (../lib/sbi/path.c:227)
09/17 00:19:47.893: [amf] INFO: [imsi-001010000000001:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:1036)
```
The Open5GS U-Plane log when executed is as follows.
```
09/17 00:19:47.767: [upf] INFO: [Added] Number of UPF-Sessions is now 2 (../src/upf/context.c:231)
09/17 00:19:47.767: [pfcp] INFO: Apply Create PDR: PDR-ID[1] (../lib/pfcp/handler.c:886)
09/17 00:19:47.767: [pfcp] INFO: Apply Create PDR: PDR-ID[2] (../lib/pfcp/handler.c:886)
09/17 00:19:47.767: [pfcp] INFO: Apply Create PDR: PDR-ID[3] (../lib/pfcp/handler.c:886)
09/17 00:19:47.767: [pfcp] INFO: Apply Create PDR: PDR-ID[4] (../lib/pfcp/handler.c:886)
09/17 00:19:47.767: [pfcp] INFO: Apply Create FAR: FAR-ID[1] (../lib/pfcp/handler.c:1399)
09/17 00:19:47.767: [pfcp] INFO: Apply Create FAR: FAR-ID[2] (../lib/pfcp/handler.c:1399)
09/17 00:19:47.767: [pfcp] INFO: Apply Create FAR: FAR-ID[3] (../lib/pfcp/handler.c:1399)
09/17 00:19:47.767: [pfcp] INFO: Apply Create URR: URR-ID[1] (../lib/pfcp/handler.c:1966)
09/17 00:19:47.767: [pfcp] INFO: Apply Create QER: QER-ID[1] (../lib/pfcp/handler.c:1669)
09/17 00:19:47.767: [pfcp] INFO: Apply Create BAR: BAR-ID[1] (../lib/pfcp/handler.c:1799)
09/17 00:19:47.767: [pfcp] WARNING: Set FAR-ID[3] GTP-U peer [TEID:0x2] (../lib/pfcp/context.c:1262)
09/17 00:19:47.767: [pfcp] INFO: Register Error Indication F-TEID[0x2] [FAR-ID:3] (../lib/pfcp/context.c:2047)
09/17 00:19:47.767: [upf] INFO: UE F-SEID[UP:0xeda CP:0xfe8] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:575)
09/17 00:19:47.768: [upf] INFO: UPF registered IPv4 framed route[192.168.21.0/24 0.0.0.0 1] for SEID[0xeda] (../src/upf/context.c:724)
09/17 00:19:47.768: [upf] INFO: UPF registered IPv4 framed route[192.168.22.0/24 0.0.0.0 1] for SEID[0xeda] (../src/upf/context.c:724)
09/17 00:19:47.768: [upf] INFO: UPF registered IPv4 framed route[192.168.21.0/24 0.0.0.0 1] for SEID[0xeda] (../src/upf/context.c:724)
09/17 00:19:47.768: [upf] INFO: UPF registered IPv4 framed route[192.168.22.0/24 0.0.0.0 1] for SEID[0xeda] (../src/upf/context.c:724)
09/17 00:19:47.768: [pfcp] INFO: Register local F-TEID[0x933e] [PDR-ID:2 type:1] (../lib/pfcp/context.c:1623)
09/17 00:19:47.768: [pfcp] INFO: Register local F-TEID[0x5202] [PDR-ID:3 type:1] (../lib/pfcp/context.c:1623)
09/17 00:19:47.772: [upf] INFO: Session Modification Request [xid:35] [UP-SEID:0xeda CP-SEID:0xfe8] (../src/upf/n4-handler.c:312)
09/17 00:19:47.772: [pfcp] INFO: Mark rules [PDR:4 FAR:3 URR:1 QER:1 BAR:1] (../lib/pfcp/context.c:1377)
09/17 00:19:47.772: [pfcp] WARNING: Set FAR-ID[1] GTP-U peer [TEID:0x2] (../lib/pfcp/context.c:1262)
09/17 00:19:47.772: [pfcp] INFO: Register Error Indication F-TEID[0x2] [FAR-ID:1] (../lib/pfcp/context.c:2047)
09/17 00:19:47.772: [pfcp] INFO: Updated FAR GTP-U tunnel: FAR-ID[1] TEID[0x0->0x2] (../lib/pfcp/handler.c:1544)
09/17 00:19:47.772: [pfcp] INFO: Apply Update FAR: FAR-ID[1] (../lib/pfcp/handler.c:1551)
```
Looking at the console log of the `nr-ue` command, UE1 has been assigned the IP address `10.45.0.3` from Open5GS 5GC.
```
[2026-09-17 00:19:47.923] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up in namespace[ueransim-001010000000001-internet-psi1].
```
Just in case, after logging in VM3 from another terminal, move to netns:`ueransim-001010000000001-internet-psi1` and make sure it matches the IP address of the UE1's TUNnel interface.
```
# ip netns exec ueransim-001010000000001-internet-psi1 ip addr show
...
11: uesimtun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.3/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::b8bb:ac26:2af2:f9e3/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
**Don't forget [Setup veth pair for UE1 and PC2/PC3](#setup_ue1).**

<a id="run_pc1"></a>

### Run tcpdump on PC1

On PC1, run `tcpdump` on `veth-pc1` to check Frame Routing of UE0 (`192.168.20.0/24`).
```
# ip netns exec pc1 tcpdump -l -i veth-pc1 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc1, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc2"></a>

### Run tcpdump on PC2

On PC2, run `tcpdump` on `veth-pc2` to check Frame Routing of UE1 (`192.168.21.0/24`).
```
# ip netns exec pc2 tcpdump -l -i veth-pc2 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc2, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc3"></a>

### Run tcpdump on PC3

On PC3, run `tcpdump` on `veth-pc3` to check Frame Routing of UE1 (`192.168.22.0/24`).
```
# ip netns exec pc3 tcpdump -l -i veth-pc3 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc4"></a>

### Run tcpdump on PC4

On PC4, run `tcpdump` on `veth-pc4` and confirm that no frame routing is configured for UE0 (`192.168.23.0/24`).
```
# ip netns exec pc4 tcpdump -l -i veth-pc4 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on veth-pc4, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="ping"></a>

## Ping Framed Routes

<a id="ping_pc1"></a>

### Ping IP address (192.168.20.100/24) of Framed Routes of UE0 on PC1

On EXT (External Node), ping IP address (`192.168.20.100/24`) of Framed Routes of UE0 and confirm with `tcpdump` running on PC1.
```
# ping 192.168.20.100
PING 192.168.20.100 (192.168.20.100) 56(84) bytes of data.
64 bytes from 192.168.20.100: icmp_seq=1 ttl=62 time=0.688 ms
64 bytes from 192.168.20.100: icmp_seq=2 ttl=62 time=0.685 ms
64 bytes from 192.168.20.100: icmp_seq=3 ttl=62 time=0.673 ms
```
The `tcpdump` log on PC1 is as follows.
```
00:39:10.846856 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 1452, seq 1, length 64
00:39:10.846868 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 1452, seq 1, length 64
00:39:11.878737 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 1452, seq 2, length 64
00:39:11.878747 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 1452, seq 2, length 64
00:39:12.902728 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 1452, seq 3, length 64
00:39:12.902738 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 1452, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC2 and PC3.**

<a id="ping_pc2"></a>

### Ping IP address (192.168.21.100/24) of Framed Routes of UE1 on PC2

On EXT (External Node), ping IP address (`192.168.21.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on PC2.
```
# ping 192.168.21.100
PING 192.168.21.100 (192.168.21.100) 56(84) bytes of data.
64 bytes from 192.168.21.100: icmp_seq=1 ttl=62 time=0.818 ms
64 bytes from 192.168.21.100: icmp_seq=2 ttl=62 time=0.760 ms
64 bytes from 192.168.21.100: icmp_seq=3 ttl=62 time=0.706 ms
```
The `tcpdump` log on PC2 is as follows.
```
00:41:15.672966 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 1461, seq 1, length 64
00:41:15.672977 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 1461, seq 1, length 64
00:41:16.678277 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 1461, seq 2, length 64
00:41:16.678287 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 1461, seq 2, length 64
00:41:17.702238 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 1461, seq 3, length 64
00:41:17.702249 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 1461, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC1 and PC3.**

<a id="ping_pc3"></a>

### Ping IP address (192.168.22.100/24) of Framed Routes of UE1 on PC3

On EXT (External Node), ping IP address (`192.168.22.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on PC3.
```
# ping 192.168.22.100
PING 192.168.22.100 (192.168.22.100) 56(84) bytes of data.
64 bytes from 192.168.22.100: icmp_seq=1 ttl=62 time=0.927 ms
64 bytes from 192.168.22.100: icmp_seq=2 ttl=62 time=0.854 ms
64 bytes from 192.168.22.100: icmp_seq=3 ttl=62 time=0.754 ms
```
The `tcpdump` log on PC3 is as follows.
```
00:42:04.865645 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 1463, seq 1, length 64
00:42:04.865655 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 1463, seq 1, length 64
00:42:05.894201 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 1463, seq 2, length 64
00:42:05.894211 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 1463, seq 2, length 64
00:42:06.917996 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 1463, seq 3, length 64
00:42:06.918005 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 1463, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC1 and PC2.**

<a id="ping_pc4"></a>

### Ping IP address (192.168.23.100/24) not configured for Framed Routes

On EXT (External Node), ping IP address (`192.168.23.100/24`) that is not configured in Framed Routes, and confirm no packets with `tcpdump` running on PC4.
```
# ping 192.168.23.100
PING 192.168.23.100 (192.168.23.100) 56(84) bytes of data.
```
**Also make sure there are no tcpdump logs on PC1, PC2 and PC3.**

---
I was able to confirm the very simple configuration for Framed Routing.
In practice, I think that PSA-UPF and UE will require more complex network routing configuration.
In this article, I kept the minimum settings necessary to check Framed Routing.
Also in this scenario, UE0 and UE1 only serve routing and not NAT. You may run `ping` and `iperf3` commands bidirectionally between PC1, PC2, PC3 and EXT.

I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2026.09.16] Updated to Open5GS v2.8.0 (2024.09.16).
- [2026.04.25] Changed to the method that uses network namespaces for UERANSIM gNodeB and UE.
- [2026.02.11] Changed to the scenario where N3/N4/N6 interfaces of UPF are separated into different networks.
- [2025.11.22] Added information related to Open5GS Framed Routing feature to the top of this article.
- [2025.11.21] Modified the scenario to verify Framed Routing to explain it in a bit more detail.
- [2024.03.31] [This commit](https://github.com/open5gs/open5gs/commit/e8a3b76af395a9986234b7d339a7a96dc5bb537f) fixed the issue where SMF crashes without `gtpc` section in `smf.yaml`. So deleted the `gtpc` section in `smf.yaml` for 5G use.
- [2024.03.29] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.03.18] Updated to Open5GS v2.6.1 (2023.03.18) and UERANSIM v3.2.6 (2023.03.17).
- [2023.03.11] Added the description about ping between UE0 and UE1, and ping between Framed routes belonging to different UEs.
- [2023.01.29] Initial release.
