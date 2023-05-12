# P4INT_P4PI
SDN P4 INT deployed in P4PI raspberry and security analysis



##INSTALL P4PI
1 start here https://github.com/p4lang/p4pi/wiki/Installing-P4Pi







##DEBUG

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
