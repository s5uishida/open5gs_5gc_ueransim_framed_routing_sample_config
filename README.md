# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Framed Routing
This describes a very simple configuration that uses Open5GS and UERANSIM for Framed Routing.

This feature has been merged into Open5GS via the following pull requests by <b>@mitmitmitm</b>.

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
  - [Network settings of UERANSIM UE0](#network_settings_ue0)
  - [Network settings of UERANSIM UE1](#network_settings_ue1)
  - [Network settings of PC1 (Internal Node)](#network_settings_pc1)
  - [Network settings of PC2 (Internal Node)](#network_settings_pc2)
  - [Network settings of PC3 (Internal Node)](#network_settings_pc3)
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
- [Ping Framed Routes](#ping)
  - [Ping IP address (192.168.20.100/24) of Framed Routes of UE0 on PC1](#ping_ue0)
  - [Ping IP address (192.168.21.100/24) of Framed Routes of UE1 on PC2](#ping_ue11)
  - [Ping IP address (192.168.22.100/24) of Framed Routes of UE1 on PC3](#ping_ue12)
  - [Ping IP address (192.168.23.100/24) of Framed Routes (not exist)](#ping_ue2)
- [Changelog (summary)](#changelog)
---
<a id="overview"></a>

## Overview of Open5GS 5GC Simulation Mobile Network

I created a 5GC simulation mobile network for  the purpose of using  the IP routes (Framed Routes) behind the UE.

The following minimum configuration was set as a condition.
- Two UEs have the same DNN and connect to the same DN.
- Two UEs have different Framed Routes. On the UPF VM, make sure to be able to ping the Framed Routes via the IP address (Tunnel GW/uesimtun0) assigned to each UE.

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - Open5GS v2.7.6 (2025.11.19) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.7 (2025.10.25) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Mem (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 24.04 | 2GB | 20GB |
| VM2 | Open5GS 5GC U-Plane  | 192.168.0.112/24<br><b>192.168.16.112/24</b> | Ubuntu 24.04 | 1GB | 20GB |
| EXT | External Node | 192.168.0.152/24<br><b>192.168.16.152/24</b> | Ubuntu 24.04 | 1GB | 10GB |
| VM3 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 24.04 | 1GB | 10GB |
| VM4 | UERANSIM UE0 | 192.168.0.132/24<br><b>192.168.20.1/24</b> | Ubuntu 24.04 | 1GB | 10GB |
| VM5 | UERANSIM UE1 | 192.168.0.133/24<br><b>192.168.21.1/24<br>192.168.22.1/24</b> | Ubuntu 24.04 | 1GB | 10GB |
| PC1 | Internal Node | 192.168.0.161/24<br><b>192.168.20.100/24</b> | Ubuntu 24.04 | 1GB | 10GB |
| PC2 | Internal Node | 192.168.0.162/24<br><b>192.168.21.100/24</b> | Ubuntu 24.04 | 1GB | 10GB |
| PC3 | Internal Node | 192.168.0.163/24<br><b>192.168.22.100/24</b> | Ubuntu 24.04 | 1GB | 10GB |

Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files. As of 2023.01.29, Framed Routes cannot be set with the WebUI. Also, if you change the `open5gs-dbctl` script, it seems that you can register these with this script, but I could not register.**
| UE # | IMSI | DNN | OP/OPc | Framed Routes | Internal IP address |
| --- | --- | --- | --- | --- | --- |
| UE0 | 001010000000000 | internet | OPc | <b>192.168.20.0/24</b> | <b>192.168.20.1</b> |
| UE1 | 001010000000001 | internet | OPc | <b>192.168.21.0/24<br>192.168.22.0/24</b> | <b>192.168.21.1<br>192.168.22.1</b> |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

The DN is as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 |

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.6 (2025.11.19) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.7 (2025.10.25) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ amf.yaml    2025-11-20 07:31:25.166453278 +0900
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
--- smf.yaml.orig       2025-04-27 11:38:05.000000000 +0900
+++ smf.yaml    2025-11-20 23:29:21.945846594 +0900
@@ -20,16 +20,14 @@
         - uri: http://127.0.0.200:7777
   pfcp:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.111
     client:
       upf:
-        - address: 127.0.0.7
-  gtpc:
-    server:
-      - address: 127.0.0.4
+        - address: 192.168.0.112
+          dnn: internet
   gtpu:
     server:
-      - address: 127.0.0.4
+      - address: 192.168.0.111
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
+++ upf.yaml    2025-11-20 23:29:48.863908116 +0900
@@ -11,18 +11,18 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.112
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.0.112
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
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:44.000000000 +0900
+++ open5gs-gnb.yaml    2024-03-29 16:25:09.415183762 +0900
@@ -1,17 +1,17 @@
-mcc: '999'          # Mobile Country Code value
-mnc: '70'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
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
--- open5gs-ue.yaml.orig        2025-03-16 15:49:12.000000000 +0900
+++ open5gs-ue0.yaml    2025-05-04 09:24:31.352554298 +0900
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
 # Home Network Public Key for protecting with SUCI Profile A
@@ -31,7 +31,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
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
--- open5gs-ue.yaml.orig        2025-03-16 15:49:12.000000000 +0900
+++ open5gs-ue1.yaml    2025-11-21 20:15:31.242725049 +0900
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
 # Home Network Public Key for protecting with SUCI Profile A
@@ -31,7 +31,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
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
```

<a id="network_settings_ext"></a>

### Network settings of External Node

Set the routings towards UEs and Framed Routes.
```
ip route add 10.45.0.0/16 via 192.168.16.112
ip route add 192.168.20.0/24 via 192.168.16.112
ip route add 192.168.21.0/24 via 192.168.16.112
ip route add 192.168.22.0/24 via 192.168.16.112
```

<a id="network_settings_ue0"></a>

### Network settings of UERANSIM UE0

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, after starting UE0 of UERANSIM on VM4, change the default GW interface to `uesimtun0` as follows.
```
ip route change default dev uesimtun0
```
When stopping UE0 on VM4, the `uesimtun0` interface is lost, and the default GW interface associated with the `uesimtun0` interface is also lost. Therefore, after restarting UE0, add the default GW interface as follows.
```
ip route add default dev uesimtun0
```

<a id="network_settings_ue1"></a>

### Network settings of UERANSIM UE1

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, after starting UE1 of UERANSIM on VM5, change the default GW interface to `uesimtun0` as follows.
```
ip route change default dev uesimtun0
```
When stopping UE1 on VM5, the `uesimtun0` interface is lost, and the default GW interface associated with the `uesimtun0` interface is also lost. Therefore, after restarting UE1, add the default GW interface as follows.
```
ip route add default dev uesimtun0
```

<a id="network_settings_pc1"></a>

### Network settings of PC1 (Internal Node)

Set `192.168.20.1/24` of UE0 as the default GW on PC1.
```
ip route change default via 192.168.20.1
```

<a id="network_settings_pc2"></a>

### Network settings of PC2 (Internal Node)

Set `192.168.21.1/24` of UE1 as the default GW on PC2.
```
ip route change default via 192.168.21.1
```

<a id="network_settings_pc3"></a>

### Network settings of PC3 (Internal Node)

Set `192.168.22.1/24` of UE1 as the default GW on PC3.
```
ip route change default via 192.168.22.1
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
- Open5GS v2.7.6 (2025.11.19) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.7 (2025.10.25) - https://github.com/aligungr/UERANSIM/wiki/Installation

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
UERANSIM v3.2.7
[2025-11-21 21:08:57.639] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2025-11-21 21:08:57.642] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2025-11-21 21:08:57.642] [sctp] [debug] SCTP association setup ascId[5]
[2025-11-21 21:08:57.642] [ngap] [debug] Sending NG Setup Request
[2025-11-21 21:08:57.649] [ngap] [debug] NG Setup Response received
[2025-11-21 21:08:57.649] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
11/21 21:08:57.640: [amf] INFO: gNB-N2 accepted[192.168.0.131]:40674 in ng-path module (../src/amf/ngap-sctp.c:113)
11/21 21:08:57.640: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:900)
11/21 21:08:57.646: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1277)
11/21 21:08:57.647: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:947)
```

<a id="start_ue0"></a>

#### Start UE0

Start UE0 as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml
UERANSIM v3.2.7
[2025-11-21 21:09:42.559] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2025-11-21 21:09:42.560] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2025-11-21 21:09:42.560] [nas] [info] Selected plmn[001/01]
[2025-11-21 21:09:42.560] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2025-11-21 21:09:42.560] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2025-11-21 21:09:42.560] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2025-11-21 21:09:42.560] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2025-11-21 21:09:42.561] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-11-21 21:09:42.561] [nas] [debug] Sending Initial Registration
[2025-11-21 21:09:42.561] [rrc] [debug] Sending RRC Setup Request
[2025-11-21 21:09:42.561] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2025-11-21 21:09:42.562] [rrc] [info] RRC connection established
[2025-11-21 21:09:42.562] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2025-11-21 21:09:42.562] [nas] [info] UE switches to state [CM-CONNECTED]
[2025-11-21 21:09:42.568] [nas] [debug] Authentication Request received
[2025-11-21 21:09:42.568] [nas] [debug] Received SQN [000000000540]
[2025-11-21 21:09:42.568] [nas] [debug] SQN-MS [000000000000]
[2025-11-21 21:09:42.572] [nas] [debug] Security Mode Command received
[2025-11-21 21:09:42.572] [nas] [debug] Selected integrity[2] ciphering[0]
[2025-11-21 21:09:42.584] [nas] [debug] Registration accept received
[2025-11-21 21:09:42.584] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2025-11-21 21:09:42.584] [nas] [debug] Sending Registration Complete
[2025-11-21 21:09:42.584] [nas] [info] Initial Registration is successful
[2025-11-21 21:09:42.584] [nas] [debug] Sending PDU Session Establishment Request
[2025-11-21 21:09:42.586] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-11-21 21:09:42.792] [nas] [debug] Configuration Update Command received
[2025-11-21 21:09:42.805] [nas] [debug] PDU Session Establishment Accept received
[2025-11-21 21:09:42.805] [nas] [info] PDU Session establishment is successful PSI[1]
[2025-11-21 21:09:42.825] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
11/21 21:09:42.708: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:461)
11/21 21:09:42.708: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2775)
11/21 21:09:42.708: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:622)
11/21 21:09:42.708: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1912)
11/21 21:09:42.708: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1688)
11/21 21:09:42.708: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1623)
11/21 21:09:42.708: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:183)
11/21 21:09:42.708: [sbi] INFO: [d3c611be-c6d2-41f0-8386-594833b1d9b7] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:307)
11/21 21:09:42.708: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.709: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:09:42.709: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.709: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:09:42.710: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.712: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
11/21 21:09:42.713: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.714: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.714: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.716: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:337)
11/21 21:09:42.718: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:09:42.718: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.719: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.720: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:09:42.720: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.720: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.722: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.722: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.724: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.724: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.725: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:361)
11/21 21:09:42.725: [sbi] INFO: [d3c7fab0-c6d2-41f0-9d72-39782bf7d5df] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
11/21 21:09:42.725: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.726: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:114)
11/21 21:09:42.726: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:09:42.726: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.728: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
11/21 21:09:42.937: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:3001)
11/21 21:09:42.937: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:609)
11/21 21:09:42.937: [gmm] INFO:     UTC [2025-11-21T12:09:42] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
11/21 21:09:42.937: [gmm] INFO:     LOCAL [2025-11-21T21:09:42] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:556)
11/21 21:09:42.937: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2796)
11/21 21:09:42.937: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] LBO[0] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1416)
11/21 21:09:42.937: [gmm] INFO: V-SMF Instance [d3d8d40c-c6d2-41f0-b63a-258514c009d9](LIST) (../src/amf/gmm-handler.c:1493)
11/21 21:09:42.937: [gmm] INFO: [d3d8d40c-c6d2-41f0-b63a-258514c009d9] Setup NF Instance [type:SMF] (../src/amf/gmm-handler.c:1495)
11/21 21:09:42.937: [gmm] INFO: V-SMF Instance [d3d8d40c-c6d2-41f0-b63a-258514c009d9] (../src/amf/gmm-handler.c:1505)
11/21 21:09:42.937: [gmm] INFO: V-SMF discovered in Non-Roaming or LBO-Roaming[0] (../src/amf/gmm-handler.c:1574)
11/21 21:09:42.937: [gmm] INFO: nsmf_pdusession [1:0x6416a2a64c70:(nil)] (../src/amf/gmm-handler.c:1614)
11/21 21:09:42.937: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.938: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1069)
11/21 21:09:42.938: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3381)
11/21 21:09:42.938: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:331)
11/21 21:09:42.939: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:09:42.939: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.939: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.941: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.942: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:456)
11/21 21:09:42.942: [sbi] INFO: [d3c7fab0-c6d2-41f0-9d72-39782bf7d5df] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
11/21 21:09:42.942: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.943: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
11/21 21:09:42.943: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:450)
11/21 21:09:42.943: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:09:42.944: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.944: [sbi] INFO: [d3c45a36-c6d2-41f0-a349-a7749bea8301] Setup NF Instance [type:BSF] (../lib/sbi/path.c:307)
11/21 21:09:42.945: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.945: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:121)
11/21 21:09:42.946: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:369)
11/21 21:09:42.946: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:590)
11/21 21:09:42.947: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
11/21 21:09:42.948: [sbi] INFO: [d28e0cca-c6d2-41f0-ba9c-f3e1e7e34db1] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
11/21 21:09:42.948: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.951: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.951: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:09:42.952: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.952: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:09:42.952: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:09:42.953: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:947)
```
The Open5GS U-Plane log when executed is as follows.
```
11/21 21:09:42.955: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:212)
11/21 21:09:42.955: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
11/21 21:09:42.955: [upf] INFO: UE F-SEID[UP:0x983 CP:0x4fb] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:498)
11/21 21:09:42.959: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2025-11-21 21:09:42.825] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
9: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::9a1f:1162:694f:575c/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
**Don't forget [Network settings of UERANSIM UE0](#network_settings_ue0).**

<a id="start_ue1"></a>

#### Start UE1

Start UE1 as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue1.yaml 
UERANSIM v3.2.7
[2025-11-21 21:14:01.812] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2025-11-21 21:14:01.813] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2025-11-21 21:14:01.814] [nas] [info] Selected plmn[001/01]
[2025-11-21 21:14:01.814] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2025-11-21 21:14:01.814] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2025-11-21 21:14:01.814] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2025-11-21 21:14:01.814] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2025-11-21 21:14:01.816] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-11-21 21:14:01.816] [nas] [debug] Sending Initial Registration
[2025-11-21 21:14:01.816] [rrc] [debug] Sending RRC Setup Request
[2025-11-21 21:14:01.816] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2025-11-21 21:14:01.817] [rrc] [info] RRC connection established
[2025-11-21 21:14:01.817] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2025-11-21 21:14:01.817] [nas] [info] UE switches to state [CM-CONNECTED]
[2025-11-21 21:14:01.823] [nas] [debug] Authentication Request received
[2025-11-21 21:14:01.823] [nas] [debug] Received SQN [000000000181]
[2025-11-21 21:14:01.823] [nas] [debug] SQN-MS [000000000000]
[2025-11-21 21:14:01.827] [nas] [debug] Security Mode Command received
[2025-11-21 21:14:01.827] [nas] [debug] Selected integrity[2] ciphering[0]
[2025-11-21 21:14:01.839] [nas] [debug] Registration accept received
[2025-11-21 21:14:01.839] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2025-11-21 21:14:01.839] [nas] [debug] Sending Registration Complete
[2025-11-21 21:14:01.839] [nas] [info] Initial Registration is successful
[2025-11-21 21:14:01.839] [nas] [debug] Sending PDU Session Establishment Request
[2025-11-21 21:14:01.839] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2025-11-21 21:14:02.041] [nas] [debug] Configuration Update Command received
[2025-11-21 21:14:02.053] [nas] [debug] PDU Session Establishment Accept received
[2025-11-21 21:14:02.056] [nas] [info] PDU Session establishment is successful PSI[1]
[2025-11-21 21:14:02.077] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
11/21 21:14:01.746: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:461)
11/21 21:14:01.746: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2775)
11/21 21:14:01.746: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:622)
11/21 21:14:01.746: [amf] INFO: [suci-0-001-01-0000-0-0-0000000001] Unknown UE by SUCI (../src/amf/context.c:1912)
11/21 21:14:01.746: [amf] INFO: [Added] Number of AMF-UEs is now 2 (../src/amf/context.c:1688)
11/21 21:14:01.746: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1623)
11/21 21:14:01.746: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:183)
11/21 21:14:01.746: [sbi] INFO: [d3c611be-c6d2-41f0-8386-594833b1d9b7] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:307)
11/21 21:14:01.746: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.747: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:14:01.747: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.748: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:14:01.748: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.750: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
11/21 21:14:01.751: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.752: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.752: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.754: [ausf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/ausf/nudm-handler.c:337)
11/21 21:14:01.756: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:14:01.756: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.756: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.758: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:14:01.758: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.758: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.760: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.760: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.761: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.762: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.763: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:361)
11/21 21:14:01.763: [sbi] INFO: [d3c7fab0-c6d2-41f0-9d72-39782bf7d5df] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
11/21 21:14:01.763: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.764: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/pcf/npcf-handler.c:114)
11/21 21:14:01.764: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:14:01.764: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.766: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
11/21 21:14:01.969: [gmm] INFO: [imsi-001010000000001] Registration complete (../src/amf/gmm-sm.c:3001)
11/21 21:14:01.969: [amf] INFO: [imsi-001010000000001] Configuration update command (../src/amf/nas-path.c:609)
11/21 21:14:01.969: [gmm] INFO:     UTC [2025-11-21T12:14:01] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
11/21 21:14:01.969: [gmm] INFO:     LOCAL [2025-11-21T21:14:01] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:556)
11/21 21:14:01.969: [amf] INFO: [Added] Number of AMF-Sessions is now 2 (../src/amf/context.c:2796)
11/21 21:14:01.969: [gmm] INFO: UE SUPI[imsi-001010000000001] DNN[internet] LBO[0] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1416)
11/21 21:14:01.969: [gmm] INFO: V-SMF Instance [d3d8d40c-c6d2-41f0-b63a-258514c009d9](LIST) (../src/amf/gmm-handler.c:1493)
11/21 21:14:01.969: [gmm] INFO: [d3d8d40c-c6d2-41f0-b63a-258514c009d9] Setup NF Instance [type:SMF] (../src/amf/gmm-handler.c:1495)
11/21 21:14:01.969: [gmm] INFO: V-SMF Instance [d3d8d40c-c6d2-41f0-b63a-258514c009d9] (../src/amf/gmm-handler.c:1505)
11/21 21:14:01.969: [gmm] INFO: V-SMF discovered in Non-Roaming or LBO-Roaming[0] (../src/amf/gmm-handler.c:1574)
11/21 21:14:01.969: [gmm] INFO: nsmf_pdusession [1:0x6416a2a64c70:(nil)] (../src/amf/gmm-handler.c:1614)
11/21 21:14:01.969: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.970: [smf] INFO: [Added] Number of SMF-UEs is now 2 (../src/smf/context.c:1069)
11/21 21:14:01.970: [smf] INFO: [Added] Number of SMF-Sessions is now 2 (../src/smf/context.c:3381)
11/21 21:14:01.970: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:331)
11/21 21:14:01.970: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:14:01.970: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.971: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.973: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.974: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:456)
11/21 21:14:01.974: [sbi] INFO: [d3c7fab0-c6d2-41f0-9d72-39782bf7d5df] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
11/21 21:14:01.974: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.975: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
11/21 21:14:01.975: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/pcf/npcf-handler.c:450)
11/21 21:14:01.975: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:14:01.975: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.976: [sbi] INFO: [d3c45a36-c6d2-41f0-a349-a7749bea8301] Setup NF Instance [type:BSF] (../lib/sbi/path.c:307)
11/21 21:14:01.976: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.977: [pcf] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../src/pcf/nbsf-handler.c:121)
11/21 21:14:01.978: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:369)
11/21 21:14:01.978: [smf] INFO: UE SUPI[imsi-001010000000001] DNN[internet] IPv4[10.45.0.3] IPv6[] (../src/smf/npcf-handler.c:590)
11/21 21:14:01.979: [sbi] INFO: [d28e0cca-c6d2-41f0-ba9c-f3e1e7e34db1] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
11/21 21:14:01.979: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.982: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.982: [sbi] INFO: [d3c5b75a-c6d2-41f0-a9b4-e306ee97a519] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
11/21 21:14:01.982: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.983: [sbi] INFO: [d3c813c4-c6d2-41f0-9c70-ed45935f764a] Setup NF Instance [type:UDR] (../lib/sbi/path.c:307)
11/21 21:14:01.983: [scp] INFO: Setup NF EndPoint(addr) [127.0.0.20:7777] (../src/scp/sbi-path.c:463)
11/21 21:14:01.984: [amf] INFO: [imsi-001010000000001:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:947)
```
The Open5GS U-Plane log when executed is as follows.
```
11/21 21:14:01.988: [upf] INFO: [Added] Number of UPF-Sessions is now 2 (../src/upf/context.c:212)
11/21 21:14:01.988: [upf] INFO: UE F-SEID[UP:0x5fb CP:0xb5b] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:498)
```
Looking at the console log of the `nr-ue` command, UE1 has been assigned the IP address `10.45.0.3` from Open5GS 5GC.
```
[2025-11-21 21:14:02.077] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up.
```
Just in case, make sure it matches the IP address of the UE1's TUNnel interface.
```
# ip addr show
...
8: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.3/24 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::caef:7abf:f9d1:5fbc/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
**Don't forget [Network settings of UERANSIM UE1](#network_settings_ue1).**

<a id="run_pc1"></a>

### Run tcpdump on PC1

On PC1, run `tcpdump` on `ens20` to check Frame Routing of UE0 (`192.168.20.0/24`).
```
# tcpdump -i ens20 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens20, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc2"></a>

### Run tcpdump on PC2

On PC2, run `tcpdump` on `ens20` to check Frame Routing of UE1 (`192.168.21.0/24`).
```
# tcpdump -i ens20 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens20, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="run_pc3"></a>

### Run tcpdump on PC3

On PC1, run `tcpdump` on `ens20` to check Frame Routing of UE1 (`192.168.22.0/24`).
```
# tcpdump -i ens20 -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on ens20, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

<a id="ping"></a>

## Ping Framed Routes

<a id="ping_ue0"></a>

### Ping IP address (192.168.20.100/24) of Framed Routes of UE0 on PC1

On EXT (External Node), ping IP address (`192.168.20.100/24`) of Framed Routes of UE0 and confirm with `tcpdump` running on PC1.
```
# ping 192.168.20.100
PING 192.168.20.100 (192.168.20.100) 56(84) bytes of data.
64 bytes from 192.168.20.100: icmp_seq=1 ttl=62 time=1.31 ms
64 bytes from 192.168.20.100: icmp_seq=2 ttl=62 time=1.24 ms
64 bytes from 192.168.20.100: icmp_seq=3 ttl=62 time=1.23 ms
```
The `tcpdump` log on PC1 is as follows.
```
21:36:45.601899 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 1335, seq 1, length 64
21:36:45.601934 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 1335, seq 1, length 64
21:36:46.603340 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 1335, seq 2, length 64
21:36:46.603361 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 1335, seq 2, length 64
21:36:47.604794 IP 192.168.16.152 > 192.168.20.100: ICMP echo request, id 1335, seq 3, length 64
21:36:47.604814 IP 192.168.20.100 > 192.168.16.152: ICMP echo reply, id 1335, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC2 and PC3.**

<a id="ping_ue11"></a>

### Ping IP address (192.168.21.100/24) of Framed Routes of UE1 on PC2

On EXT (External Node), ping IP address (`192.168.21.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on PC2.
```
# ping 192.168.21.100
PING 192.168.21.100 (192.168.21.100) 56(84) bytes of data.
64 bytes from 192.168.21.100: icmp_seq=1 ttl=62 time=1.30 ms
64 bytes from 192.168.21.100: icmp_seq=2 ttl=62 time=1.27 ms
64 bytes from 192.168.21.100: icmp_seq=3 ttl=62 time=1.40 ms
```
The `tcpdump` log on PC2 is as follows.
```
21:39:33.385782 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 1337, seq 1, length 64
21:39:33.385823 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 1337, seq 1, length 64
21:39:34.387179 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 1337, seq 2, length 64
21:39:34.387198 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 1337, seq 2, length 64
21:39:35.388576 IP 192.168.16.152 > 192.168.21.100: ICMP echo request, id 1337, seq 3, length 64
21:39:35.388596 IP 192.168.21.100 > 192.168.16.152: ICMP echo reply, id 1337, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC1 and PC3.**

<a id="ping_ue12"></a>

### Ping IP address (192.168.22.100/24) of Framed Routes of UE1 on PC3

On EXT (External Node), ping IP address (`192.168.22.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on PC3.
```
# ping 192.168.22.100
PING 192.168.22.100 (192.168.22.100) 56(84) bytes of data.
64 bytes from 192.168.22.100: icmp_seq=1 ttl=62 time=1.14 ms
64 bytes from 192.168.22.100: icmp_seq=2 ttl=62 time=1.31 ms
64 bytes from 192.168.22.100: icmp_seq=3 ttl=62 time=1.44 ms
```
The `tcpdump` log on PC3 is as follows.
```
21:42:13.223560 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 1342, seq 1, length 64
21:42:13.223587 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 1342, seq 1, length 64
21:42:14.224933 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 1342, seq 2, length 64
21:42:14.224952 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 1342, seq 2, length 64
21:42:15.226317 IP 192.168.16.152 > 192.168.22.100: ICMP echo request, id 1342, seq 3, length 64
21:42:15.226338 IP 192.168.22.100 > 192.168.16.152: ICMP echo reply, id 1342, seq 3, length 64
```
**Note. Confirm that no packets have arrived at PC1 and PC2.**

<a id="ping_ue2"></a>

### Ping IP address (192.168.23.100/24) of Framed Routes (not exist)

On EXT (External Node), ping IP address (`192.168.23.100/24`) of Framed Routes which do not exist on either UE0 or UE1, and confirm no packets with `tcpdump` running on PC1, PC2 and PC3.
```
# ping 192.168.23.100
PING 192.168.23.100 (192.168.23.100) 56(84) bytes of data.
```
**Make sure there are no tcpdump logs on PC1, PC2 and PC3.**

---
I was able to confirm the very simple configuration for Framed Routing.
In practice, I think that PSA-UPF and UE will require more complex network routing configuration.
In this article, I kept the minimum settings necessary to check Framed Routing.
Also in this scenario, UE0 and UE1 serve as L3SW (i.e., routing only, no NAT), and you may run `ping` and `iperf3` commands bidirectionally between PC1, PC2, PC3, and EXT.

I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2025.11.22] Added information related to Open5GS Framed Routing feature to the top of this article.
- [2025.11.21] Modified the scenario to verify Framed Routing to explain it in a bit more detail.
- [2024.03.31] [This commit](https://github.com/open5gs/open5gs/commit/e8a3b76af395a9986234b7d339a7a96dc5bb537f) fixed the issue where SMF crashes without `gtpc` section in `smf.yaml`. So deleted the `gtpc` section in `smf.yaml` for 5G use.
- [2024.03.29] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.03.18] Updated to Open5GS v2.6.1 (2023.03.18) and UERANSIM v3.2.6 (2023.03.17).
- [2023.03.11] Added the description about ping between UE0 and UE1, and ping between Framed routes belonging to different UEs.
- [2023.01.29] Initial release.
