# Open5GS 5GC & UERANSIM UE / RAN Sample Configuration - Framed Routing
This describes a very simple configuration that uses Open5GS and UERANSIM for Framed Routing.

---

<a id="conf_list"></a>

## List of Sample Configurations

1. [One SGW-C/PGW-C, one SGW-U/PGW-U and one APN](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)
2. [One SGW-C/PGW-C, Multiple SGW-Us/PGW-Us and APNs](https://github.com/s5uishida/open5gs_epc_oai_sample_config)
3. [One SMF, one UPF and one DNN](https://github.com/s5uishida/open5gs_5gc_srsran_sample_config)
4. [One SMF, Multiple UPFs and DNNs](https://github.com/s5uishida/open5gs_5gc_ueransim_sample_config)
5. [Select nearby UPF(PGW-U) according to the connected eNodeB](https://github.com/s5uishida/open5gs_epc_srsran_nearby_upf_sample_config)
6. [Select nearby UPF according to the connected gNodeB](https://github.com/s5uishida/open5gs_5gc_ueransim_nearby_upf_sample_config)
7. [Select UPF based on S-NSSAI](https://github.com/s5uishida/open5gs_5gc_ueransim_snssai_upf_sample_config)
8. [SCP Indirect communication Model C](https://github.com/s5uishida/open5gs_5gc_ueransim_scp_model_c_sample_config)
9. [VoLTE and SMS Configuration for docker_open5gs](https://github.com/s5uishida/docker_open5gs_volte_sms_config)
10. [Monitoring Metrics with Prometheus](https://github.com/s5uishida/open5gs_5gc_ueransim_metrics_sample_config)
11. Framed Routing (this article)
12. [VPP-UPF(PGW-U) with DPDK](https://github.com/s5uishida/open5gs_epc_srsran_vpp_upf_dpdk_sample_config)
13. [VPP-UPF with DPDK](https://github.com/s5uishida/open5gs_5gc_ueransim_vpp_upf_dpdk_sample_config)
14. [eUPF(eBPF/XDP UPF(PGW-U))](https://github.com/s5uishida/open5gs_epc_srsran_eupf_sample_config)
15. [eUPF(eBPF/XDP UPF)](https://github.com/s5uishida/open5gs_5gc_ueransim_eupf_sample_config)
---

<a id="misc"></a>

## Miscellaneous Notes

- [Install MongoDB 6.0 and Open5GS WebUI](https://github.com/s5uishida/open5gs_install_mongodb6_webui)
- [Install MongoDB 4.4.18 on Ubuntu 20.04 for Raspberry Pi 4B](https://github.com/s5uishida/install_mongodb_on_ubuntu_for_rp4b)
- [A Note for 5G SUCI Profile A/B Scheme](https://github.com/s5uishida/note_5g_suci_profile_ab)
- [A Note for Changing Network Interface of UPF from TUN to TAP in Open5GS](https://github.com/s5uishida/change_from_tun_to_tap_in_open5gs)
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
- 5GC - Open5GS v2.6.1 (2023.03.18) - https://github.com/open5gs/open5gs
- UE / RAN - UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | Open5GS 5GC C-Plane | 192.168.0.111/24 | Ubuntu 22.04 | 1GB | 20GB |
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
- Open5GS v2.6.1 (2023.03.18) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of Open5GS 5GC C-Plane

- `open5gs/install/etc/open5gs/amf.yaml`
```diff
--- amf.yaml.orig       2023-03-10 21:49:28.000000000 +0900
+++ amf.yaml    2023-03-11 18:37:14.000000000 +0900
@@ -416,26 +416,26 @@
       - addr: 127.0.0.5
         port: 7777
     ngap:
-      - addr: 127.0.0.5
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.5
         port: 9090
     guami:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         amf_id:
           region: 2
           set: 1
     tai:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         tac: 1
     plmn_support:
       - plmn_id:
-          mcc: 999
-          mnc: 70
+          mcc: 001
+          mnc: 01
         s_nssai:
           - sst: 1
     security:
```
- `open5gs/install/etc/open5gs/smf.yaml`
```diff
--- smf.yaml.orig       2023-03-10 21:49:28.000000000 +0900
+++ smf.yaml    2023-03-11 18:38:30.000000000 +0900
@@ -602,20 +602,17 @@
       - addr: 127.0.0.4
         port: 7777
     pfcp:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.111
     gtpc:
       - addr: 127.0.0.4
-      - addr: ::1
     gtpu:
-      - addr: 127.0.0.4
-      - addr: ::1
+      - addr: 192.168.0.111
     metrics:
       - addr: 127.0.0.4
         port: 9090
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
     dns:
       - 8.8.8.8
       - 8.8.4.4
@@ -813,7 +810,8 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
+        dnn: internet
 
 #
 #  o Disable use of IPv4 addresses (only IPv6)
```

<a id="changes_up"></a>

### Changes in configuration files of Open5GS 5GC U-Plane

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2023-03-10 21:58:54.000000000 +0900
+++ upf.yaml    2023-03-11 18:41:22.000000000 +0900
@@ -196,12 +196,13 @@
 #
 upf:
     pfcp:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     gtpu:
-      - addr: 127.0.0.7
+      - addr: 192.168.0.112
     subnet:
       - addr: 10.45.0.1/16
-      - addr: 2001:db8:cafe::1/48
+        dnn: internet
+        dev: ogstun
     metrics:
       - addr: 127.0.0.7
         port: 9090
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran"></a>

#### Changes in configuration files of RAN

- `UERANSIM/config/open5gs-gnb.yaml`
```diff
--- open5gs-gnb.yaml.orig       2022-07-03 13:06:44.000000000 +0900
+++ open5gs-gnb.yaml    2023-01-12 21:26:20.000000000 +0900
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
--- open5gs-ue.yaml.orig        2023-03-17 19:17:14.000000000 +0900
+++ open5gs-ue0.yaml    2023-03-18 20:09:12.143031846 +0900
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
 # Routing Indicator
 routingIndicator: '0000'
 
@@ -22,7 +22,7 @@
 
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
--- open5gs-ue.yaml.orig        2023-03-17 19:17:13.000000000 +0900
+++ open5gs-ue1.yaml    2023-03-18 20:10:52.872658000 +0900
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
 # Routing Indicator
 routingIndicator: '0000'
 
@@ -22,7 +22,7 @@
 
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
    "$oid": "63d51ff360e6c465ca52edbb"
  },
  "schema_version": 1,
  "imsi": "001010000000000",
  "msisdn": [],
  "imeisv": "4370816125816151",
  "mme_host": [],
  "mme_realm": [],
  "purge_flag": [],
  "security": {
    "k": "465B5CE8 B199B49F AA5F0A2E E238A6BC",
    "op": null,
    "opc": "E8ED289D EBA952E4 283B54E8 8E6183CA",
    "amf": "8000",
    "sqn": {
      "$numberLong": "129"
    }
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
  "slice": [
    {
      "sst": 1,
      "default_indicator": true,
      "session": [
        {
          "name": "internet",
          "type": 3,
          "qos": {
            "index": 9,
            "arp": {
              "priority_level": 8,
              "pre_emption_capability": 1,
              "pre_emption_vulnerability": 1
            }
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
            "$oid": "63d51ff360e6c465ca52edbd"
          },
-->       "ipv4_framed_routes": [
-->         "192.168.20.0/24"
-->       ],
          "pcc_rule": []
        }
      ],
      "_id": {
        "$oid": "63d51ff360e6c465ca52edbc"
      }
    }
  ],
  "access_restriction_data": 32,
  "subscriber_status": 0,
  "network_access_mode": 0,
  "subscribed_rau_tau_timer": 12,
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
    "$oid": "63d53f7360e6c465ca52edcf"
  },
  "schema_version": 1,
  "imsi": "001010000000001",
  "msisdn": [],
  "imeisv": "4370816125816151",
  "mme_host": [],
  "mme_realm": [],
  "purge_flag": [],
  "security": {
    "k": "465B5CE8 B199B49F AA5F0A2E E238A6BC",
    "op": null,
    "opc": "E8ED289D EBA952E4 283B54E8 8E6183CA",
    "amf": "8000",
    "sqn": {
      "$numberLong": "161"
    }
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
  "slice": [
    {
      "sst": 1,
      "default_indicator": true,
      "session": [
        {
          "name": "internet",
          "type": 3,
          "qos": {
            "index": 9,
            "arp": {
              "priority_level": 8,
              "pre_emption_capability": 1,
              "pre_emption_vulnerability": 1
            }
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
            "$oid": "63d53f7360e6c465ca52edd1"
          },
-->       "ipv4_framed_routes": [
-->         "192.168.21.0/24",
-->         "192.168.22.0/24"
-->       ],
          "pcc_rule": []
        }
      ],
      "_id": {
        "$oid": "63d53f7360e6c465ca52edd0"
      }
    }
  ],
  "access_restriction_data": 32,
  "subscriber_status": 0,
  "network_access_mode": 0,
  "subscribed_rau_tau_timer": 12,
  "__v": 0
}
```

<a id="build"></a>

## Build Open5GS and UERANSIM

Please refer to the following for building Open5GS and UERANSIM respectively.
- Open5GS v2.6.1 (2023.03.18) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- UERANSIM v3.2.6 (2023.03.17) - https://github.com/aligungr/UERANSIM/wiki/Installation

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
[2023-03-18 20:29:47.584] [sctp] [info] Trying to establish SCTP connection... (192.168.0.111:38412)
[2023-03-18 20:29:47.586] [sctp] [info] SCTP connection established (192.168.0.111:38412)
[2023-03-18 20:29:47.586] [sctp] [debug] SCTP association setup ascId[8]
[2023-03-18 20:29:47.587] [ngap] [debug] Sending NG Setup Request
[2023-03-18 20:29:47.593] [ngap] [debug] NG Setup Response received
[2023-03-18 20:29:47.593] [ngap] [info] NG Setup procedure is successful
```
The Open5GS C-Plane log when executed is as follows.
```
03/18 20:29:47.598: [amf] INFO: gNB-N2 accepted[192.168.0.131]:41600 in ng-path module (../src/amf/ngap-sctp.c:113)
03/18 20:29:47.598: [amf] INFO: gNB-N2 accepted[192.168.0.131] in master_sm module (../src/amf/amf-sm.c:733)
03/18 20:29:47.599: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1034)
03/18 20:29:47.599: [amf] INFO: gNB-N2[192.168.0.131] max_num_of_ostreams : 10 (../src/amf/amf-sm.c:772)
```

<a id="start_ue0"></a>

#### Start UE0

Start UE0 as follows. This will register the UE with 5GC and establish a PDU session.
```
# ./nr-ue -c ../config/open5gs-ue0.yaml
UERANSIM v3.2.6
[2023-03-18 20:31:21.873] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 20:31:21.874] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 20:31:21.874] [nas] [info] Selected plmn[001/01]
[2023-03-18 20:31:21.874] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 20:31:21.875] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 20:31:21.875] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 20:31:21.875] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 20:31:21.877] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 20:31:21.877] [nas] [debug] Sending Initial Registration
[2023-03-18 20:31:21.877] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 20:31:21.878] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 20:31:21.878] [rrc] [info] RRC connection established
[2023-03-18 20:31:21.878] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 20:31:21.879] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 20:31:21.896] [nas] [debug] Authentication Request received
[2023-03-18 20:31:21.897] [nas] [debug] Sending Authentication Failure due to SQN out of range
[2023-03-18 20:31:21.905] [nas] [debug] Authentication Request received
[2023-03-18 20:31:21.914] [nas] [debug] Security Mode Command received
[2023-03-18 20:31:21.914] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 20:31:21.944] [nas] [debug] Registration accept received
[2023-03-18 20:31:21.944] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 20:31:21.944] [nas] [debug] Sending Registration Complete
[2023-03-18 20:31:21.945] [nas] [info] Initial Registration is successful
[2023-03-18 20:31:21.945] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 20:31:21.945] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 20:31:22.151] [nas] [debug] Configuration Update Command received
[2023-03-18 20:31:22.181] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 20:31:22.184] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 20:31:22.212] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/18 20:31:21.890: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:372)
03/18 20:31:21.890: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2327)
03/18 20:31:21.890: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:533)
03/18 20:31:21.892: [amf] INFO: [suci-0-001-01-0000-0-0-0000000000] Unknown UE by SUCI (../src/amf/context.c:1634)
03/18 20:31:21.892: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1419)
03/18 20:31:21.892: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:985)
03/18 20:31:21.892: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000000]    SUCI (../src/amf/gmm-handler.c:149)
03/18 20:31:21.896: [sbi] WARNING: [23d96522-c580-41ed-a39a-eb99a8bbfee4] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:21.896: [sbi] WARNING: NF EndPoint updated [127.0.0.11:80] (../lib/sbi/context.c:1618)
03/18 20:31:21.896: [sbi] WARNING: NF EndPoint updated [127.0.0.11:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.897: [sbi] INFO: [23d96522-c580-41ed-a39a-eb99a8bbfee4] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:21.898: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:21.898: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:31:21.899: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.899: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.899: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.899: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:21.908: [gmm] WARNING: Authentication failure(Synch failure) (../src/amf/gmm-sm.c:1367)
03/18 20:31:21.929: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:21.930: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:31:21.930: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.930: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.930: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.930: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:21.933: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:21.934: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:31:21.934: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.934: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.934: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.934: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:21.947: [sbi] WARNING: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:21.947: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/18 20:31:21.947: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.947: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.948: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.948: [sbi] INFO: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:21.951: [sbi] WARNING: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:21.951: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1618)
03/18 20:31:21.951: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1527)
03/18 20:31:21.951: [sbi] INFO: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:22.160: [gmm] INFO: [imsi-001010000000000] Registration complete (../src/amf/gmm-sm.c:1917)
03/18 20:31:22.160: [amf] INFO: [imsi-001010000000000] Configuration update command (../src/amf/nas-path.c:612)
03/18 20:31:22.160: [gmm] INFO:     UTC [2023-03-18T11:31:22] Timezone[0]/DST[0] (../src/amf/gmm-build.c:545)
03/18 20:31:22.160: [gmm] INFO:     LOCAL [2023-03-18T20:31:22] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:550)
03/18 20:31:22.161: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2348)
03/18 20:31:22.161: [gmm] INFO: UE SUPI[imsi-001010000000000] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1186)
03/18 20:31:22.165: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1012)
03/18 20:31:22.165: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3108)
03/18 20:31:22.167: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:22.167: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:31:22.167: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.168: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.168: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.168: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:22.176: [sbi] WARNING: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:22.176: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/18 20:31:22.176: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.177: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.177: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.177: [sbi] INFO: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:22.179: [sbi] WARNING: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:22.179: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1618)
03/18 20:31:22.179: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.179: [sbi] INFO: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:22.182: [sbi] WARNING: [23d85aa6-c580-41ed-9618-9f018b8d4b52] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:31:22.182: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1618)
03/18 20:31:22.182: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1527)
03/18 20:31:22.182: [sbi] INFO: [23d85aa6-c580-41ed-9618-9f018b8d4b52] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:31:22.185: [smf] INFO: UE SUPI[imsi-001010000000000] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:528)
03/18 20:31:22.186: [gtp] INFO: gtp_connect() [192.168.0.112]:2152 (../lib/gtp/path.c:60)
```
The Open5GS U-Plane log when executed is as follows.
```
03/18 20:31:22.166: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:194)
03/18 20:31:22.166: [gtp] INFO: gtp_connect() [192.168.0.111]:2152 (../lib/gtp/path.c:60)
03/18 20:31:22.166: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:467)
03/18 20:31:22.167: [upf] INFO: UE F-SEID[UP:0x1 CP:0x1] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:467)
03/18 20:31:22.173: [gtp] INFO: gtp_connect() [192.168.0.131]:2152 (../lib/gtp/path.c:60)
```
Looking at the console log of the `nr-ue` command, UE0 has been assigned the IP address `10.45.0.2` from Open5GS 5GC.
```
[2023-03-18 20:31:22.212] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.2] is up.
```
Just in case, make sure it matches the IP address of the UE0's TUNnel interface.
```
# ip addr show
...
12: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::db7a:216e:ff4:5c1b/64 scope link stable-privacy 
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
[2023-03-18 20:34:43.700] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2023-03-18 20:34:43.701] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2023-03-18 20:34:43.701] [nas] [info] Selected plmn[001/01]
[2023-03-18 20:34:43.702] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2023-03-18 20:34:43.702] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2023-03-18 20:34:43.702] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2023-03-18 20:34:43.703] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2023-03-18 20:34:43.705] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 20:34:43.706] [nas] [debug] Sending Initial Registration
[2023-03-18 20:34:43.707] [rrc] [debug] Sending RRC Setup Request
[2023-03-18 20:34:43.707] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2023-03-18 20:34:43.708] [rrc] [info] RRC connection established
[2023-03-18 20:34:43.708] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2023-03-18 20:34:43.708] [nas] [info] UE switches to state [CM-CONNECTED]
[2023-03-18 20:34:43.719] [nas] [debug] Authentication Request received
[2023-03-18 20:34:43.720] [nas] [debug] Sending Authentication Failure due to SQN out of range
[2023-03-18 20:34:43.725] [nas] [debug] Authentication Request received
[2023-03-18 20:34:43.729] [nas] [debug] Security Mode Command received
[2023-03-18 20:34:43.729] [nas] [debug] Selected integrity[2] ciphering[0]
[2023-03-18 20:34:43.747] [nas] [debug] Registration accept received
[2023-03-18 20:34:43.747] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2023-03-18 20:34:43.747] [nas] [debug] Sending Registration Complete
[2023-03-18 20:34:43.748] [nas] [info] Initial Registration is successful
[2023-03-18 20:34:43.748] [nas] [debug] Sending PDU Session Establishment Request
[2023-03-18 20:34:43.748] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2023-03-18 20:34:43.957] [nas] [debug] Configuration Update Command received
[2023-03-18 20:34:43.979] [nas] [debug] PDU Session Establishment Accept received
[2023-03-18 20:34:43.984] [nas] [info] PDU Session establishment is successful PSI[1]
[2023-03-18 20:34:44.006] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up.
```
The Open5GS C-Plane log when executed is as follows.
```
03/18 20:34:43.707: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:372)
03/18 20:34:43.707: [amf] INFO: [Added] Number of gNB-UEs is now 2 (../src/amf/context.c:2327)
03/18 20:34:43.707: [amf] INFO:     RAN_UE_NGAP_ID[2] AMF_UE_NGAP_ID[2] TAC[1] CellID[0x10] (../src/amf/ngap-handler.c:533)
03/18 20:34:43.707: [amf] INFO: [suci-0-001-01-0000-0-0-0000000001] Unknown UE by SUCI (../src/amf/context.c:1634)
03/18 20:34:43.707: [amf] INFO: [Added] Number of AMF-UEs is now 2 (../src/amf/context.c:1419)
03/18 20:34:43.707: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:985)
03/18 20:34:43.707: [gmm] INFO: [suci-0-001-01-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:149)
03/18 20:34:43.708: [sbi] WARNING: [23d96522-c580-41ed-a39a-eb99a8bbfee4] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.709: [sbi] WARNING: NF EndPoint updated [127.0.0.11:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.709: [sbi] WARNING: NF EndPoint updated [127.0.0.11:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.709: [sbi] INFO: [23d96522-c580-41ed-a39a-eb99a8bbfee4] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.711: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.711: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.712: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.712: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.712: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.712: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.719: [gmm] WARNING: Authentication failure(Synch failure) (../src/amf/gmm-sm.c:1367)
03/18 20:34:43.729: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.729: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.730: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.730: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.730: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.730: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.732: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.733: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.733: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.733: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.733: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.733: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.739: [sbi] WARNING: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.740: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.740: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.740: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.740: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.740: [sbi] INFO: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.742: [sbi] WARNING: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.743: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.743: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.743: [sbi] INFO: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.952: [gmm] INFO: [imsi-001010000000001] Registration complete (../src/amf/gmm-sm.c:1917)
03/18 20:34:43.952: [amf] INFO: [imsi-001010000000001] Configuration update command (../src/amf/nas-path.c:612)
03/18 20:34:43.953: [gmm] INFO:     UTC [2023-03-18T11:34:43] Timezone[0]/DST[0] (../src/amf/gmm-build.c:545)
03/18 20:34:43.953: [gmm] INFO:     LOCAL [2023-03-18T20:34:43] Timezone[32400]/DST[0] (../src/amf/gmm-build.c:550)
03/18 20:34:43.954: [amf] INFO: [Added] Number of AMF-Sessions is now 2 (../src/amf/context.c:2348)
03/18 20:34:43.955: [gmm] INFO: UE SUPI[imsi-001010000000001] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] (../src/amf/gmm-handler.c:1186)
03/18 20:34:43.958: [smf] INFO: [Added] Number of SMF-UEs is now 2 (../src/smf/context.c:1012)
03/18 20:34:43.958: [smf] INFO: [Added] Number of SMF-Sessions is now 2 (../src/smf/context.c:3108)
03/18 20:34:43.960: [sbi] WARNING: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.961: [sbi] WARNING: NF EndPoint updated [127.0.0.12:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.961: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.961: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.962: [sbi] WARNING: NF EndPoint updated [127.0.0.12:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.962: [sbi] INFO: [23d9855c-c580-41ed-9ecf-61104054e617] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.966: [sbi] WARNING: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.966: [sbi] WARNING: NF EndPoint updated [127.0.0.13:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.966: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.966: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.966: [sbi] WARNING: NF EndPoint updated [127.0.0.13:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.967: [sbi] INFO: [23e2c87e-c580-41ed-8376-054e8bb9724b] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.968: [sbi] WARNING: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.968: [sbi] WARNING: NF EndPoint updated [127.0.0.20:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.968: [sbi] WARNING: NF EndPoint updated [127.0.0.20:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.968: [sbi] INFO: [23e31824-c580-41ed-b9eb-49a4ba28df6e] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.970: [sbi] WARNING: [23d85aa6-c580-41ed-9618-9f018b8d4b52] (NF-discover) NF has already been added (../lib/sbi/nnrf-handler.c:839)
03/18 20:34:43.971: [sbi] WARNING: NF EndPoint updated [127.0.0.15:80] (../lib/sbi/context.c:1618)
03/18 20:34:43.971: [sbi] WARNING: NF EndPoint updated [127.0.0.15:7777] (../lib/sbi/context.c:1527)
03/18 20:34:43.971: [sbi] INFO: [23d85aa6-c580-41ed-9618-9f018b8d4b52] (NF-discover) NF Profile updated (../lib/sbi/nnrf-handler.c:862)
03/18 20:34:43.973: [smf] INFO: UE SUPI[imsi-001010000000001] DNN[internet] IPv4[10.45.0.3] IPv6[] (../src/smf/npcf-handler.c:528)
```
The Open5GS U-Plane log when executed is as follows.
```
03/18 20:34:43.957: [upf] INFO: [Added] Number of UPF-Sessions is now 2 (../src/upf/context.c:194)
03/18 20:34:43.957: [upf] INFO: UE F-SEID[UP:0x2 CP:0x2] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:467)
03/18 20:34:43.957: [upf] INFO: UE F-SEID[UP:0x2 CP:0x2] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:467)
```
Looking at the console log of the `nr-ue` command, UE1 has been assigned the IP address `10.45.0.3` from Open5GS 5GC.
```
[2023-03-18 20:34:44.006] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.45.0.3] is up.
```
Just in case, make sure it matches the IP address of the UE1's TUNnel interface.
```
# ip addr show
...
4: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.3/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::d320:a1d8:9991:ec03/64 scope link stable-privacy 
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
64 bytes from 192.168.20.100: icmp_seq=1 ttl=64 time=2.60 ms
64 bytes from 192.168.20.100: icmp_seq=2 ttl=64 time=1.97 ms
64 bytes from 192.168.20.100: icmp_seq=3 ttl=64 time=1.22 ms
```
The `tcpdump` log on UE0 is as follows.
```
20:38:29.549365 IP 10.45.0.1 > 192.168.20.100: ICMP echo request, id 2, seq 1, length 64
20:38:29.549388 IP 192.168.20.100 > 10.45.0.1: ICMP echo reply, id 2, seq 1, length 64
20:38:30.551727 IP 10.45.0.1 > 192.168.20.100: ICMP echo request, id 2, seq 2, length 64
20:38:30.551747 IP 192.168.20.100 > 10.45.0.1: ICMP echo reply, id 2, seq 2, length 64
20:38:31.551911 IP 10.45.0.1 > 192.168.20.100: ICMP echo request, id 2, seq 3, length 64
20:38:31.551930 IP 192.168.20.100 > 10.45.0.1: ICMP echo reply, id 2, seq 3, length 64
```
**Note. Confirm that no packets have arrived at UE1 (VM5).**

<h3 id="ping_ue11">Ping IP address (192.168.21.100/24) of Framed Routes of UE1</h4>

On UPF (VM2), ping IP address (`192.168.21.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on VM5.

```
# ping 192.168.21.100 -I ogstun 
PING 192.168.21.100 (192.168.21.100) from 10.45.0.1 ogstun: 56(84) bytes of data.
64 bytes from 192.168.21.100: icmp_seq=1 ttl=64 time=1.75 ms
64 bytes from 192.168.21.100: icmp_seq=2 ttl=64 time=1.66 ms
64 bytes from 192.168.21.100: icmp_seq=3 ttl=64 time=2.24 ms
```
The `tcpdump` log on UE1 is as follows.
```
20:39:16.595023 IP 10.45.0.1 > 192.168.21.100: ICMP echo request, id 3, seq 1, length 64
20:39:16.595046 IP 192.168.21.100 > 10.45.0.1: ICMP echo reply, id 3, seq 1, length 64
20:39:17.596053 IP 10.45.0.1 > 192.168.21.100: ICMP echo request, id 3, seq 2, length 64
20:39:17.596083 IP 192.168.21.100 > 10.45.0.1: ICMP echo reply, id 3, seq 2, length 64
20:39:18.597262 IP 10.45.0.1 > 192.168.21.100: ICMP echo request, id 3, seq 3, length 64
20:39:18.597295 IP 192.168.21.100 > 10.45.0.1: ICMP echo reply, id 3, seq 3, length 64
```
**Note. Confirm that no packets have arrived at UE0 (VM4).**

<h3 id="ping_ue12">Ping IP address (192.168.22.100/24) of Framed Routes of UE1</h4>

On UPF (VM2), ping IP address (`192.168.22.100/24`) of Framed Routes of UE1 and confirm with `tcpdump` running on VM5.

```
# ping 192.168.22.100 -I ogstun 
PING 192.168.22.100 (192.168.22.100) from 10.45.0.1 ogstun: 56(84) bytes of data.
64 bytes from 192.168.22.100: icmp_seq=1 ttl=64 time=1.96 ms
64 bytes from 192.168.22.100: icmp_seq=2 ttl=64 time=2.08 ms
64 bytes from 192.168.22.100: icmp_seq=3 ttl=64 time=1.63 ms
```
The `tcpdump` log on UE1 is as follows.
```
20:39:56.267711 IP 10.45.0.1 > 192.168.22.100: ICMP echo request, id 4, seq 1, length 64
20:39:56.267773 IP 192.168.22.100 > 10.45.0.1: ICMP echo reply, id 4, seq 1, length 64
20:39:57.269269 IP 10.45.0.1 > 192.168.22.100: ICMP echo request, id 4, seq 2, length 64
20:39:57.269303 IP 192.168.22.100 > 10.45.0.1: ICMP echo reply, id 4, seq 2, length 64
20:39:58.270011 IP 10.45.0.1 > 192.168.22.100: ICMP echo request, id 4, seq 3, length 64
20:39:58.270037 IP 192.168.22.100 > 10.45.0.1: ICMP echo reply, id 4, seq 3, length 64
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

- [2023.03.18] Updated to Open5GS v2.6.1 (2023.03.18) and UERANSIM v3.2.6 (2023.03.17).
- [2023.03.11] Added the description about ping between UE0 and UE1, and ping between Framed routes belonging to different UEs.
- [2023.01.29] Initial release.
