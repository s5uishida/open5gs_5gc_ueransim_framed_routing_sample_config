# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Framed Routing
This describes a very simple configuration that uses Open5GS and UERANSIM for Framed Routing.

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
  - [Network settings of UERANSIM UE0](#network_settings_ue0)
  - [Network settings of UERANSIM UE1](#network_settings_ue1)
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
- [Ping Framed Routes](#ping)
  - [Ping IP address (192.168.20.100/24) of Framed Routes of UE0](#ping_ue0)
  - [Ping IP address (192.168.21.100/24) of Framed Routes of UE1](#ping_ue11)
  - [Ping IP address (192.168.22.100/24) of Framed Routes of UE1](#ping_ue12)
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
- 5GC - Open5GS v2.7.0 (2024.03.24) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 22.04 | 2GB | 20GB |
| VM2 | Open5GS 5GC U-Plane  | 192.168.0.112/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | UERANSIM RAN (gNodeB) | 192.168.0.131/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM4 | UERANSIM UE0 | 192.168.0.132/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM5 | UERANSIM UE1 | 192.168.0.133/24 | Ubuntu 22.04 | 1GB | 10GB |

Subscriber Information (other information is the same) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files. As of 2023.01.29, Framed Routes cannot be set with the WebUI. Also, if you change the `open5gs-dbctl` script, it seems that you can register these with this script, but I could not register.**
| UE # | IMSI | DNN | OP/OPc | Framed Routes |
| --- | --- | --- | --- | -- |
| UE0 | 001010000000000 | internet | OPc | 192.168.20.0/24 |
| UE1 | 001010000000001 | internet | OPc | 192.168.21.0/24<br>192.168.22.0/24 |

I registered these information with the Open5GS WebUI.
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

The DN is as follows.
| DN | TUNnel interface of DN | DNN | TUNnel interface of UE |
| --- | --- | --- | --- |
| 10.45.0.0/16 | ogstun | internet | uesimtun0 |

<a id="changes"></a>

## Changes in configuration files of Open5GS 5GC and UERANSIM UE / RAN

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ amf.yaml    2024-03-24 20:19:58.000000000 +0900
@@ -19,27 +19,27 @@
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
--- nrf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ nrf.yaml    2024-03-24 20:58:02.000000000 +0900
@@ -10,8 +10,8 @@
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
--- smf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ smf.yaml    2024-03-31 23:01:38.941943102 +0900
@@ -19,35 +19,31 @@
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
         port: 9090
   session:
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
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
--- upf.yaml.orig       2024-03-24 15:36:48.000000000 +0900
+++ upf.yaml    2024-03-29 16:12:53.684589991 +0900
@@ -10,16 +10,17 @@
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
     - subnet: 10.45.0.1/16
-    - subnet: 2001:db8:cafe::1/48
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
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue0.yaml    2024-03-29 16:27:36.390313563 +0900
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
@@ -28,7 +28,7 @@
 
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
--- open5gs-ue.yaml.orig        2023-12-02 06:14:20.000000000 +0900
+++ open5gs-ue1.yaml    2024-03-29 16:30:10.175137471 +0900
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
@@ -28,7 +28,7 @@
 
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
Next, configure the TUNnel interface and NAPT.
```
ip tuntap add name ogstun mode tun
ip addr add 10.45.0.1/16 dev ogstun
ip link set ogstun up

iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE
```

<a id="network_settings_ue0"></a>

### Network settings of UERANSIM UE0

After starting UE0 of UERANSIM on VM4, set the Framed Routes IP address (ex.`192.168.20.100/24`) and the routing of DN (`10.45.0.0/16`) to the `uesimtun0` interface as follows.
```
ip addr add 192.168.20.100/24 dev uesimtun0
ip route add 10.45.0.0/16 dev uesimtun0
```

<a id="network_settings_ue1"></a>

### Network settings of UERANSIM UE1

After starting UE1 of UERANSIM on VM5, set the Framed Routes IP addresses (ex.`192.168.21.100/24` and `192.168.22.100/24`) and the routing of DN (`10.45.0.0/16`) to the `uesimtun0` interface as follows.
```
ip addr add 192.168.21.100/24 dev uesimtun0
ip addr add 192.168.22.100/24 dev uesimtun0
ip route add 10.45.0.0/16 dev uesimtun0
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
    "$oid": "660010644d5ed1026c8029aa"
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
      "$numberLong": "705"
    }
  },
  "slice": [
    {
      "_id": {
        "$oid": "660010644d5ed1026c8029ab"
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
            "$oid": "660010644d5ed1026c8029ac"
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
    "$oid": "66066e30a4ac04026e67b4d6"
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
      "$numberLong": "129"
    }
  },
  "slice": [
    {
      "_id": {
        "$oid": "66066e30a4ac04026e67b4d7"
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
            "$oid": "66066e30a4ac04026e67b4d8"
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
- Open5GS v2.7.0 (2024.03.24) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

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
UERANSIM v3.2.6
[2024-03-29 17:29:05.380] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2024-03-29 17:29:05.392] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2024-03-29 17:29:05.392] [sctp] [debug] SCTP association setup ascId[9]
[2024-03-29 17:29:05.393] [ngap] [debug] Sending NG Setup Request
[2024-03-29 17:29:05.410] [ngap] [debug] NG Setup Response received
[2024-03-29 17:29:05.410] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/29 17:29:05.413: [amf] INFO: gNB-N2 accepted[192.168.0.131]:39014 in ng-path module (../src/amf/ngap-sctp.c:113)
03/29 17:29:05.413: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:754)
03/29 17:29:05.429: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1236)
03/29 17:29:05.429: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:793)
```

<a id="start_ue0"></a>

#### Start UE0

Start UE0 as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml
UERANSIM v3.2.6
[2024-03-29 17:29:52.625] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-29 17:29:52.626] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-29 17:29:52.628] [nas] [info] Selected plmn[001/01]
[2024-03-29 17:29:52.629] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-29 17:29:52.630] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-29 17:29:52.630] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-29 17:29:52.631] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-29 17:29:52.632] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-29 17:29:52.633] [nas] [debug] Sending Initial Registration
[2024-03-29 17:29:52.634] [rrc] [debug] Sending RRC Setup Request
[2024-03-29 17:29:52.635] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-29 17:29:52.636] [rrc] [info] RRC connection established
[2024-03-29 17:29:52.637] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-29 17:29:52.638] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-29 17:29:52.665] [nas] [debug] Authentication Request received
[2024-03-29 17:29:52.666] [nas] [debug] Received SQN [0000000002E1]
[2024-03-29 17:29:52.666] [nas] [debug] SQN-MS [000000000000]
[2024-03-29 17:29:52.684] [nas] [debug] Security Mode Command received
[2024-03-29 17:29:52.685] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-29 17:29:52.720] [nas] [debug] Registration accept received
[2024-03-29 17:29:52.721] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-29 17:29:52.721] [nas] [debug] Sending Registration Complete
[2024-03-29 17:29:52.721] [nas] [info] Initial Registration is successful
[2024-03-29 17:29:52.722] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-29 17:29:52.722] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-29 17:29:52.924] [nas] [debug] Configuration Update Command received
[2024-03-29 17:29:52.943] [nas] [debug] PDU Session Establishment Accept received
[2024-03-29 17:29:52.948] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-29 17:29:52.972] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/29 17:29:52.638: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/29 17:29:52.638: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2656)
03/29 17:29:52.638: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/29 17:29:52.638: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1840)
03/29 17:29:52.639: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1621)
03/29 17:29:52.639: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/29 17:29:52.639: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:172)
03/29 17:29:52.646: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.647: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.647: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.648: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.648: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.649: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.659: [sbi] INFO: [UDM] (SCP-discover) NF registered [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/path.c:211)
03/29 17:29:52.712: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.713: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.713: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.714: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.716: [sbi] INFO: [UDR] (SCP-discover) NF registered [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/path.c:211)
03/29 17:29:52.923: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:2321)
03/29 17:29:52.923: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:591)
03/29 17:29:52.923: [gmm] INFO:     UTC [2024-03-29T08:29:52] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/29 17:29:52.923: [gmm] INFO:     LOCAL [2024-03-29T17:29:52] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/29 17:29:52.923: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2677)
03/29 17:29:52.924: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/29 17:29:52.924: [gmm] INFO: SMF Instance [60254b14-eda6-41ee-ba70-290c6ff4e539] (../src/amf/gmm-handler.c:1324)
03/29 17:29:52.925: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1019)
03/29 17:29:52.925: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3090)
03/29 17:29:52.926: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.926: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.926: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.927: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.927: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.927: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.929: [sbi] INFO: [UDM] (SCP-discover) NF registered [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/path.c:211)
03/29 17:29:52.931: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [601a21bc-eda6-41ee-a31a-2f41d8b49ba5:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.931: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.931: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.931: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.931: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.932: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [601a21bc-eda6-41ee-a31a-2f41d8b49ba5:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.933: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.933: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.933: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.933: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.934: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:2] (../lib/sbi/path.c:216)
03/29 17:29:52.935: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [601239de-eda6-41ee-99ab-b77236a98db8:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.935: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.936: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.936: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [601239de-eda6-41ee-99ab-b77236a98db8:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.937: [sbi] INFO: [BSF] (SCP-discover) NF registered [601239de-eda6-41ee-99ab-b77236a98db8:1] (../lib/sbi/path.c:211)
03/29 17:29:52.938: [sbi] INFO: [PCF] (SCP-discover) NF registered [601a21bc-eda6-41ee-a31a-2f41d8b49ba5:1] (../lib/sbi/path.c:211)
03/29 17:29:52.938: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:542)
03/29 17:29:52.939: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
03/29 17:29:52.944: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:52.944: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 17:29:52.944: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.945: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.945: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:52.945: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:52.946: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:2] (../lib/sbi/path.c:216)
03/29 17:29:52.947: [amf] INFO: [imsi-001010000000000:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane log when executed is as follows.
```
03/29 17:29:52.928: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:208)
03/29 17:29:52.928: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
03/29 17:29:52.928: [upf] INFO: UE F-SEID[UP:0x18c CP:0x88b] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
03/29 17:29:52.928: [upf] INFO: UE F-SEID[UP:0x18c CP:0x88b] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:485)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2024-03-29 17:29:52.972] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
10: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::3348:b3:4cba:2947/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
- Run `tcpdump` on VM4 to confirm Framed Routing of UE0
```
# tcpdump -i uesimtun0 -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on uesimtun0, link-type RAW (Raw IP), capture size 262144 bytes
```
**Don't forget [Network settings of UERANSIM UE0](#network_settings_ue0).**

<a id="start_ue1"></a>

#### Start UE1

Start UE1 as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue1.yaml 
UERANSIM v3.2.6
[2024-03-29 17:29:59.366] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-29 17:29:59.367] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-29 17:29:59.369] [nas] [info] Selected plmn[001/01]
[2024-03-29 17:29:59.370] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-29 17:29:59.370] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-29 17:29:59.371] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-29 17:29:59.371] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-29 17:29:59.375] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-29 17:29:59.375] [nas] [debug] Sending Initial Registration
[2024-03-29 17:29:59.376] [rrc] [debug] Sending RRC Setup Request
[2024-03-29 17:29:59.377] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-29 17:29:59.378] [rrc] [info] RRC connection established
[2024-03-29 17:29:59.379] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-29 17:29:59.379] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-29 17:29:59.401] [nas] [debug] Authentication Request received
[2024-03-29 17:29:59.401] [nas] [debug] Received SQN [0000000000C1]
[2024-03-29 17:29:59.402] [nas] [debug] SQN-MS [000000000000]
[2024-03-29 17:29:59.417] [nas] [debug] Security Mode Command received
[2024-03-29 17:29:59.418] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-29 17:29:59.460] [nas] [debug] Registration accept received
[2024-03-29 17:29:59.460] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-29 17:29:59.461] [nas] [debug] Sending Registration Complete
[2024-03-29 17:29:59.461] [nas] [info] Initial Registration is successful
[2024-03-29 17:29:59.462] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-29 17:29:59.462] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-29 17:29:59.669] [nas] [debug] Configuration Update Command received
[2024-03-29 17:29:59.728] [nas] [debug] PDU Session Establishment Accept received
[2024-03-29 17:29:59.733] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-29 17:29:59.767] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/29 17:29:59.367: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:401)
03/29 17:29:59.367: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2656)
03/29 17:29:59.367: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:562)
03/29 17:29:59.368: [amf] INFO: [suci-0-001-01-0000-0-0-0000000001] Unknown UE by SUCI (../src/amf/context.c:1840)
03/29 17:29:59.368: [amf] INFO: [Added] Number of AMF-UEs is now 2 (../src/amf/context.c:1621)
03/29 17:29:59.368: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1224)
03/29 17:29:59.368: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:172)
03/29 17:29:59.373: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.373: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.374: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.375: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.375: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.376: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.384: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:2] (../lib/sbi/path.c:216)
03/29 17:29:59.437: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.438: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.438: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.439: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.441: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:3] (../lib/sbi/path.c:216)
03/29 17:29:59.652: [gmm] INFO: [imsi-001010000000001] Registration complete (../src/amf/gmm-sm.c:2321)
03/29 17:29:59.652: [amf] INFO: [imsi-001010000000001] Configuration update command (../src/amf/nas-path.c:591)
03/29 17:29:59.653: [gmm] INFO:     UTC [2024-03-29T08:29:59] Timezone[0]/DST[0] (../src/amf/gmm-build.c:558)
03/29 17:29:59.654: [gmm] INFO:     LOCAL [2024-03-29T17:29:59] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:563)
03/29 17:29:59.655: [amf] INFO: [Added] Number of AMF-Sessions is now 2 (../src/amf/context.c:2677)
03/29 17:29:59.656: [gmm] INFO: UE SUPI[imsi-001010000000001] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef [NULL] (../src/amf/gmm-handler.c:1285)
03/29 17:29:59.656: [gmm] INFO: SMF Instance [60254b14-eda6-41ee-ba70-290c6ff4e539] (../src/amf/gmm-handler.c:1324)
03/29 17:29:59.660: [smf] INFO: [Added] Number of SMF-UEs is now 2 (../src/smf/context.c:1019)
03/29 17:29:59.660: [smf] INFO: [Added] Number of SMF-Sessions is now 2 (../src/smf/context.c:3090)
03/29 17:29:59.664: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.665: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.666: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.666: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.667: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.668: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.677: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:3] (../lib/sbi/path.c:216)
03/29 17:29:59.682: [sbi] WARNING: [PCF] (NRF-discover) NF has already been added [601a21bc-eda6-41ee-a31a-2f41d8b49ba5:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.683: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.684: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.684: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.685: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.13:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.685: [sbi] INFO: [PCF] (NF-discover) NF Profile updated [601a21bc-eda6-41ee-a31a-2f41d8b49ba5:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.691: [sbi] WARNING: [UDR] (NRF-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.692: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.692: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.20:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.692: [sbi] INFO: [UDR] (NF-discover) NF Profile updated [6019e95e-eda6-41ee-8072-392145ccd024:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.696: [sbi] WARNING: [UDR] (SCP-discover) NF has already been added [6019e95e-eda6-41ee-8072-392145ccd024:4] (../lib/sbi/path.c:216)
03/29 17:29:59.699: [sbi] WARNING: [BSF] (NRF-discover) NF has already been added [601239de-eda6-41ee-99ab-b77236a98db8:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.700: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.700: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.15:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.701: [sbi] INFO: [BSF] (NF-discover) NF Profile updated [601239de-eda6-41ee-99ab-b77236a98db8:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.703: [sbi] WARNING: [BSF] (SCP-discover) NF has already been added [601239de-eda6-41ee-99ab-b77236a98db8:2] (../lib/sbi/path.c:216)
03/29 17:29:59.706: [sbi] WARNING: [PCF] (SCP-discover) NF has already been added [601a21bc-eda6-41ee-a31a-2f41d8b49ba5:2] (../lib/sbi/path.c:216)
03/29 17:29:59.707: [smf] INFO: UE SUPI[imsi-001010000000001] DNN[internet] IPv4[10.45.0.3] IPv6[] (../src/smf/npcf-handler.c:542)
03/29 17:29:59.719: [sbi] WARNING: [UDM] (NRF-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1162)
03/29 17:29:59.720: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:80] (../lib/sbi/context.c:2210)
03/29 17:29:59.720: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.721: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.721: [sbi] WARNING: NF EndPoint(addr) updated [127.0.0.12:7777] (../lib/sbi/context.c:1946)
03/29 17:29:59.721: [sbi] INFO: [UDM] (NF-discover) NF Profile updated [6012ae5a-eda6-41ee-a502-39adaacc8462:1] (../lib/sbi/nnrf-handler.c:1200)
03/29 17:29:59.725: [sbi] WARNING: [UDM] (SCP-discover) NF has already been added [6012ae5a-eda6-41ee-a502-39adaacc8462:4] (../lib/sbi/path.c:216)
03/29 17:29:59.726: [amf] INFO: [imsi-001010000000001:1:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:867)
```
The Open5GS U-Plane log when executed is as follows.
```
03/29 17:29:59.697: [upf] INFO: [Added] Number of UPF-Sessions is now 2 (../src/upf/context.c:208)
03/29 17:29:59.697: [upf] INFO: UE F-SEID[UP:0x38 CP:0x382] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:485)
03/29 17:29:59.697: [upf] INFO: UE F-SEID[UP:0x38 CP:0x382] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:485)
```
Looking at the console log of the `nr-ue` command, UE1 has been assigned the IP address `10.45.0.3` from Open5GS 5GC.
```
[2024-03-29 17:29:59.767] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up.
```
Just in case, make sure it matches the IP address of the UE1's TUNnel interface.
```
# ip addr show
...
8: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.3/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::85c5:5c11:ad58:2ed0/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```
- Run `tcpdump` on VM5 to confirm Framed Routing of UE1
```
# tcpdump -i uesimtun0 -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on uesimtun0, link-type RAW (Raw IP), capture size 262144 bytes
```
**Don't forget [Network settings of UERANSIM UE1](#network_settings_ue1).**

<h2 id="ping">Ping Framed Routes</h4>

<h3 id="ping_ue0">Ping IP address (192.168.20.100/24) of Framed Routes of UE0</h4>

On UPF (VM2), ping IP address (`192.168.20.100/24`) of Framed Routes of UE0 and confirm with `tcpdump` running on VM4.

```
# ping 192.168.20.100 -I ogstun 
PING 192.168.20.100 (192.168.20.100) from 10.45.0.1 ogstun: 56(84) bytes of data.
64 bytes from 192.168.20.100: icmp_seq=1 ttl=64 time=2.55 ms
64 bytes from 192.168.20.100: icmp_seq=2 ttl=64 time=2.82 ms
64 bytes from 192.168.20.100: icmp_seq=3 ttl=64 time=3.00 ms
```
The `tcpdump` log on UE0 is as follows.
```
17:36:27.690302 IP 10.45.0.1 > 192.168.20.100: ICMP echo request, id 13, seq 1, length 64
17:36:27.690343 IP 192.168.20.100 > 10.45.0.1: ICMP echo reply, id 13, seq 1, length 64
17:36:28.690866 IP 10.45.0.1 > 192.168.20.100: ICMP echo request, id 13, seq 2, length 64
17:36:28.690900 IP 192.168.20.100 > 10.45.0.1: ICMP echo reply, id 13, seq 2, length 64
17:36:29.692084 IP 10.45.0.1 > 192.168.20.100: ICMP echo request, id 13, seq 3, length 64
17:36:29.692117 IP 192.168.20.100 > 10.45.0.1: ICMP echo reply, id 13, seq 3, length 64
```
**Note. Confirm that no packets have arrived at UE1 (VM5).**

<h3 id="ping_ue11">Ping IP address (192.168.21.100/24) of Framed Routes of UE1</h4>

On UPF (VM2), ping IP address (`192.168.21.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on VM5.

```
# ping 192.168.21.100 -I ogstun 
PING 192.168.21.100 (192.168.21.100) from 10.45.0.1 ogstun: 56(84) bytes of data.
64 bytes from 192.168.21.100: icmp_seq=1 ttl=64 time=2.67 ms
64 bytes from 192.168.21.100: icmp_seq=2 ttl=64 time=2.85 ms
64 bytes from 192.168.21.100: icmp_seq=3 ttl=64 time=2.89 ms
```
The `tcpdump` log on UE1 is as follows.
```
17:37:06.682357 IP 10.45.0.1 > 192.168.21.100: ICMP echo request, id 14, seq 1, length 64
17:37:06.682407 IP 192.168.21.100 > 10.45.0.1: ICMP echo reply, id 14, seq 1, length 64
17:37:07.683585 IP 10.45.0.1 > 192.168.21.100: ICMP echo request, id 14, seq 2, length 64
17:37:07.683623 IP 192.168.21.100 > 10.45.0.1: ICMP echo reply, id 14, seq 2, length 64
17:37:08.684557 IP 10.45.0.1 > 192.168.21.100: ICMP echo request, id 14, seq 3, length 64
17:37:08.684599 IP 192.168.21.100 > 10.45.0.1: ICMP echo reply, id 14, seq 3, length 64
```
**Note. Confirm that no packets have arrived at UE0 (VM4).**

<h3 id="ping_ue12">Ping IP address (192.168.22.100/24) of Framed Routes of UE1</h4>

On UPF (VM2), ping IP address (`192.168.22.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on VM5.

```
# ping 192.168.22.100 -I ogstun 
PING 192.168.22.100 (192.168.22.100) from 10.45.0.1 ogstun: 56(84) bytes of data.
64 bytes from 192.168.22.100: icmp_seq=1 ttl=64 time=0.740 ms
64 bytes from 192.168.22.100: icmp_seq=2 ttl=64 time=0.694 ms
64 bytes from 192.168.22.100: icmp_seq=3 ttl=64 time=0.761 ms
```
The `tcpdump` log on UE1 is as follows.
```
17:37:49.900156 IP 10.45.0.1 > 192.168.22.100: ICMP echo request, id 15, seq 1, length 64
17:37:49.900171 IP 192.168.22.100 > 10.45.0.1: ICMP echo reply, id 15, seq 1, length 64
17:37:50.931587 IP 10.45.0.1 > 192.168.22.100: ICMP echo request, id 15, seq 2, length 64
17:37:50.931600 IP 192.168.22.100 > 10.45.0.1: ICMP echo reply, id 15, seq 2, length 64
17:37:51.955624 IP 10.45.0.1 > 192.168.22.100: ICMP echo request, id 15, seq 3, length 64
17:37:51.955637 IP 192.168.22.100 > 10.45.0.1: ICMP echo reply, id 15, seq 3, length 64
```
**Note. Confirm that no packets have arrived at UE0 (VM4).**

<h3 id="ping_ue2">Ping IP address (192.168.23.100/24) of Framed Routes (not exist)</h4>

On UPF (VM2), ping IP address (`192.168.23.100/24`) of Framed Routes which do not exist on either UE0 or UE1, and confirm no packets with `tcpdump` running on VM4 and VM5.

```
# ping 192.168.23.100 -I ogstun 
PING 192.168.23.100 (192.168.23.100) from 10.45.0.1 ogstun: 56(84) bytes of data.
```
**Make sure there are no tcpdump logs on UE0 and UE1.**

---
I was able to confirm the very simple configuration for Framed Routing.
In practice, I think that PSA-UPF and UE will require more complex network routing configuration.
In this article, I kept the minimum settings necessary to check Framed Routing.
See [here](https://github.com/s5uishida/open5gs_5gc_ueransim_framed_routing_sample_config/issues/2#issuecomment-1464891842) for ping between UE0 and UE1, and ping between Framed routes belonging to different UEs.

I would like to thank the excellent developers and all the contributors of Open5GS and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.29] Updated to Open5GS v2.7.0 (2024.03.24).
- [2023.03.18] Updated to Open5GS v2.6.1 (2023.03.18) and UERANSIM v3.2.6 (2023.03.17).
- [2023.03.11] Added the description about ping between UE0 and UE1, and ping between Framed routes belonging to different UEs.
- [2023.01.29] Initial release.
