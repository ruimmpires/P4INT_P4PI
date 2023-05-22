# P4INT_P4PI
SDN P4 INT deployed in P4PI raspberry and security analysis.
You can find a similar version running in Mininet,  in https://github.com/ruimmpires/P4INT_Mininet

## WHY
This is an SDN implementation of P4 INT-MD for bmv2 in P4Pi. This project is ongoing until hopefuly July 2023 as part of my thesis "SECURITY FOR SDN ENVIRONMENTS WITH P4" to be also available publicly. The code is mostly not original and you may find most of it in the following repositories:
- https://github.com/lifengfan13/P4-INT
- https://github.com/GEANT-DataPlaneProgramming/int-platforms
- https://github.com/mandaryoshi/p4-int. You may also check the Mandar Joshi's thesis "Implementation and evaluation of INT in P4" here https://www.diva-portal.org/smash/get/diva2:1609265/FULLTEXT01.pdf.
- [P4PI ARP](https://github.com/p4lang/p4pi/blob/master/packages/p4pi-examples/bmv2/arp_icmp/arp_icmp.p4)

You may also look into the official INT code available in the [p4 official repository for INT](https://github.com/p4lang/p4factory/tree/master/apps/int)

## INTRODUCTION
SDN with P4 brings a new set of possibilities as the way the packets are processed is not defined by the vendor, but rather by the P4 program. Using this language, developers can define data plane behavior, specifying how switches shall process the packets. P4 lets developers define which headers a switch shall parse, how tables match on each header, and which actions the switch shall perform on each header. This new programmability extends the capabilities of the data plane into security features, such as stateful packet inspection and filtering, thus relieving the control plane. This offloading of security is enhanced by the ability to run at line speed as P4 runs on the programmed devices.

You may find some courses in universities such as:
* [Stanford Building an Internet Router](https://cs344-stanford.github.io/documentation/internet-router/)
* https://p4campus.cs.princeton.edu/
* https://www.princeton.edu/~hyojoonk/

Find below the specs:
* [P4Runtime Specification](https://p4.org/p4-spec/p4runtime/main/P4Runtime-Spec.html#sec-client-arbitration-and-controller-replication)
* [P416 Language Specification working spec](https://p4.org/p4-spec/docs/P4-16-working-spec.html)
* [P416 Language Specification version 1.2.3](https://p4.org/p4-spec/docs/P4-16-v1.2.3.pdf)
* [In-band Network Telemetry (INT) Dataplane Specification Version 2.1](https://p4.org/p4-spec/docs/INT_v2_1.pdf)
* [INT spec source](https://github.com/p4lang/p4-applications/tree/master/telemetry/specs)

## TOPOLOGY
For the sake of simplicity and due to the shortage of Raspberry Pis, we streamlined the topology to three P4 switches. The WiFi interfaces are configured as APs, so we may regard those as L2 switches. We used 2 laptops with several WiFi interfaces to simulate the hosts: h1 (client), h2, hc (collector), ha (attacker) and hs (server). We used the following Raspberry PIs as described in the figure:
1. Raspberry Pi 3 Model B, Rev 1.2, 1GB of RAM. This device will act as INT source.
2. Raspberry Pi 4 Model B, Rev 1.5, 8GB of RAM. This device will act as INT transit.
3. Raspberry Pi 4 Model B, Rev 1.1, 4GB of RAM. This device will act as INT sink.
![Diagram of the INT solution with three P4Pis.](/pictures/p4pi_intv11.png)

As the Raspberry Pis only have one on-board Ethernet and one WiFi interface, we added the necessary USB devices as in the picture:
![INT lab with three P4Pis](/pictures/p4pi_hw.jpg)

## PRE-REQUISITES
We started with following the steps in the P4Pi wiki, downloaded the image, burned it to the SD-cards and then boot and configure each one (example code below for s3):
* connect via WiFi, papi/raspberry;
* ssh, pi/raspberry, sudo su, apt update;
* change static IPs, br0 and eth0, example below for the s3:
```
nano /etc/dhcpcd.conf
interface br0
  static ip_address=10.0.3.1/24
  nohook wpa_supplicant
interface eth0
  static ip_address=10.2.0.2/30
  interface wlx38a28c80c2ee
  static ip_address=10.0.4.1/24
  nohook wpa_supplicant
```
* Change DHCP
```
nano /etc/dnsmasq.d/p4edge.conf
interface=br0
dhcp-range=set:br0,10.0.3.2,10.0.3.10,255.255.255.0,24h
domain=p4pi3
address=/gw.p4pi3/10.0.3.1

interface=wlx38a28c80c2ee
dhcp-range=set:wlx38a28c80c2ee,10.0.4.2,10.0.4.10,255.255.255.0,24h
domain=p4pis
address=/gw.p4pis/10.0.4.1
```
* Change WiFi
```
cp /etc/hostapd/hostapd.conf /etc/hostapd/wlan0.conf
cp /etc/hostapd/hostapd.conf /etc/hostapd/wlx38a28c80c2ee.conf
nano /etc/hostapd/wlan0.conf
  ssid=p4pi3
nano /etc/hostapd/wlx38a28c80c2ee.conf
  interface=wlx38a28c80c2ee
  ssid=p4pis
```
* enable the new hostapd services
```
systemctl disable hostapd
systemctl enable hostapd@wlan0
systemctl enable hostapd@wlx38a28c80c2ee
```
* reboot
* connect via WiFi, papi3/raspberry
* ssh, pi/raspberry, sudo su
* add static routing as needed
```
ip r add 10.0.1.0/24 via 10.2.0.1
ip r add 10.0.2.0/24 via 10.2.0.1
```
* copy all the P4 code to /root/bmv2/intv8/. Be sure to name the main p4 file the same as the folder.
* configure the bmv2 service and change as needed:
```
nano /usr/bin/bmv2-start
#!/bin/bash
export P4PI=/root/PI
export GRPCPP=/root/P4Runtime_GRPCPP
export GRPC=/root/grpc
BM2_WDIR=/root/bmv2
P4_PROG=l2switch
T4P4S_PROG_FILE=/root/t4p4s-switch
if [ -f "${T4P4S_PROG_FILE}" ]; then
P4_PROG=$(cat "${T4P4S_PROG_FILE}")
else
echo "${P4_PROG}" > "${T4P4S_PROG_FILE}"
fi
rm -rf ${BM2_WDIR}/bin
mkdir ${BM2_WDIR}/bin
echo "Compiling P4 code"
p4c-bm2-ss -I /usr/share/p4c/p4include --std p4-16 --p4runtime-files ${BM2_WDIR}
echo "Launching BMv2 switch"
sudo simple_switch_grpc -i 0@veth0 -i 1@eth0 -i 2@wlx38a28c80c2ee \
${BM2_WDIR}/bin/${P4_PROG}.json -- --grpc-server-addr 127.0.0.1:50051
```
* configure the switch
```
echo intv8 > /root/t4p4s-switch
```
* stop and disable t4p4s, stop bmv2, enable bmv2:
```
systemctl stop t4p4s | systemctl disable t4p4s
systemctl stop bmv2 | systemctl enable bmv2
```
* start the bmv2 service and check its status:
```
systemctl start bmv2
systemctl status bmv2
```
Confirm the service is running before the next step.
* load the static tables into the P4 switch
```
simple_switch_CLI < /root/bmv2/intv8/r3commands.txt
```















[P4PI main site](https://github.com/p4lang/p4pi)

[Creating a new Bmv2 project on P4Pi](https://github.com/p4lang/p4pi/wiki/Creating-a-new-Bmv2-project-on-P4Pi)

##INSTALL P4PI
1 start here https://github.com/p4lang/p4pi/wiki/Installing-P4Pi








##DEBUG

### Switch forwarding
```sysctl -w net.ipv4.ip_forward=1```

### tcpdump R3 eth0, no watchlist
```
19:12:58.957528 b8:27:eb:83:45:95 (oui Unknown) > dc:a6:32:40:1b:03 (oui Unknown), ethertype IPv4 (0x0800), length 60: 10.0.1.9.63924 > hpelite.p4pi3.8080: UDP, length 5
19:12:58.959642 dc:a6:32:40:1b:03 (oui Unknown) > b8:27:eb:83:45:95 (oui Unknown), ethertype IPv4 (0x0800), length 75: hpelite.p4pi3 > 10.0.1.9: ICMP hpelite.p4pi3 udp port 8080 unreachable, length 41
19:13:04.007698 b8:27:eb:83:45:95 (oui Unknown) > dc:a6:32:40:1b:03 (oui Unknown), ethertype ARP (0x0806), length 60: Request who-has 10.1.0.3 tell 10.1.0.1, length 46
19:13:04.007783 dc:a6:32:40:1b:03 (oui Unknown) > b8:27:eb:83:45:95 (oui Unknown), ethertype ARP (0x0806), length 42: Reply 10.1.0.3 is-at dc:a6:32:40:1b:03 (oui Unknown), length 28
19:13:04.161584 dc:a6:32:40:1b:03 (oui Unknown) > b8:27:eb:83:45:95 (oui Unknown), ethertype ARP (0x0806), length 42: Request who-has 10.1.0.1 tell 10.1.0.3, length 28
19:13:04.162080 b8:27:eb:83:45:95 (oui Unknown) > dc:a6:32:40:1b:03 (oui Unknown), ethertype ARP (0x0806), length 60: Reply 10.1.0.1 is-at b8:27:eb:83:45:95 (oui Unknown), length 46
```
**no watchlist**
```
20:33:02.900323 b8:27:eb:83:45:95 (oui Unknown) > dc:a6:32:40:1b:03 (oui Unknown), ethertype IPv4 (0x0800), length 60: 10.0.1.9.63830 > hpelite.p4pi3.5432: UDP, length 5
20:33:02.908563 dc:a6:32:40:1b:03 (oui Unknown) > e4:a4:71:cd:52:99 (oui Unknown), ethertype IPv4 (0x0800), length 107: 10.0.1.9.63830 > hpelite.p4pi3.5432: UDP, length 65
20:33:02.911075 e4:a4:71:cd:52:99 (oui Unknown) > e4:a4:71:cd:52:99 (oui Unknown), ethertype IPv4 (0x0800), length 151: 10.0.1.9.63830 > hpelite.p4pi3.5432: UDP, length 109
```

**debug mode**
```
sudo simple_switch -i 0@veth0 -i 1@eth0 intv7.json --nanolog ipc:///tmp/bm-log.ipc
Calling target program-options parser
Adding interface veth0 as port 0
Adding interface eth0 as port 1

sudo python3 /usr/lib/python3/dist-packages/nanomsg_client.py
type: PACKET_IN, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, port_in: 1
type: PARSER_START, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, parser_id: 0 (parser)
type: PARSER_EXTRACT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 2 (ethernet)
type: PARSER_EXTRACT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 4 (ipv4)
type: PARSER_EXTRACT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 5 (udp)
type: PARSER_EXTRACT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 10 (intl4_shim)
type: PARSER_EXTRACT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 11 (int_header)
type: PARSER_EXTRACT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 20 (int_data)
type: PARSER_DONE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, parser_id: 0 (parser)
type: PIPELINE_START, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, pipeline_id: 0 (ingress)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 0 (node_2), result: False
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 2 (node_5), result: True
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 3 (node_6), result: True
type: TABLE_HIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 1 (MyIngress.l3_forward.ipv4_lpm), entry_hdl: 1
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 7 (MyIngress.l3_forward.ipv4_forward)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 4 (node_8), result: True
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 2 (MyIngress.process_int_source_sink.tb_set_source)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 1 (NoAction)
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 3 (MyIngress.process_int_source_sink.tb_set_sink)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 2 (NoAction)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 5 (node_11), result: False
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 6 (node_13), result: False
type: PIPELINE_DONE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, pipeline_id: 0 (ingress)
type: PIPELINE_START, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, pipeline_id: 1 (egress)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 7 (node_17), result: True
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 8 (node_18), result: False
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 7 (tbl_act)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 52 (act)
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 8 (MyEgress.process_int_transit.tb_int_insert)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 16 (MyEgress.process_int_transit.init_metadata)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 9 (node_22), result: True
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 10 (node_24), result: False
type: TABLE_HIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 10 (MyEgress.process_int_transit.tb_int_inst_0003), entry_hdl: 15
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 32 (MyEgress.process_int_transit.int_set_header_0003_i15)
type: TABLE_HIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 11 (MyEgress.process_int_transit.tb_int_inst_0407), entry_hdl: 15
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 48 (MyEgress.process_int_transit.int_set_header_0407_i15)
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 12 (tbl_int_transit400)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 55 (int_transit400)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 11 (node_28), result: True
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 13 (tbl_int_transit404)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 54 (int_transit404)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 12 (node_30), result: True
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 14 (tbl_int_transit407)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 56 (int_transit407)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 13 (node_32), result: True
type: TABLE_MISS, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, table_id: 15 (tbl_int_transit410)
type: ACTION_EXECUTE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, action_id: 57 (int_transit410)
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 14 (node_34), result: False
type: CONDITION_EVAL, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, condition_id: 15 (node_36), result: False
type: PIPELINE_DONE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, pipeline_id: 1 (egress)
type: DEPARSER_START, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, deparser_id: 0 (deparser)
type: CHECKSUM_UPDATE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, cksum_id: 0 (cksum)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 2 (ethernet)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 4 (ipv4)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 5 (udp)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 10 (intl4_shim)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 11 (int_header)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 12 (int_switch_id)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 13 (int_level1_port_ids)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 14 (int_hop_latency)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 15 (int_q_occupancy)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 16 (int_ingress_tstamp)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 17 (int_egress_tstamp)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 18 (int_level2_port_ids)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 19 (int_egress_tx_util)
type: DEPARSER_EMIT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, header_id: 20 (int_data)
type: DEPARSER_DONE, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, deparser_id: 0 (deparser)
type: PACKET_OUT, switch_id: 0, cxt_id: 0, sig: 15579444982436141299, id: 462, copy_id: 0, port_out: 0
```
