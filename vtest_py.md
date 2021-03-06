```
1) High level class diagram:
-------------------------

+----------------------+
|       TestCase       |
|          +           |
|          |           |
|      VtestBase       |
|          |           |
|          |           |
|     VtestCommon      |
|          |           |
|          |           |
|   VtestObjectBase    |
|                      |
+----------------------+


2)
          VtestBase
+----------------------------------+
|                                  |
|  def setUpClass()                |
|  def tearDownClass()             |
|  def create_vrouter_instance()   |
|  def destroy_vrouter_instance()  |
|  def launch_vtest_ut()           |
|  def get_sandesh_req_num()       |
|  def get_xml_proto_file_handle() |
|  def create_sandesh_req()        |
|  def run_command()               |
|                                  |
+----------------------------------+

Notes:
setUpClass() => Launches vrouter by calling create_vrouter_instance()
launch_vtest_ut() => Launches the vtest binary
run_command() => Runs a generic shell command

3)
          VtestCommon
+---------------------------+
|  def htonll()             |
|  def ntohll()             |
|  def htonl()              |
|  def ntohl()              |
|  def vt_mac()             |
|  def vt_encap()           |
|  def vt_ipv4()            |
|  def vt_ipv6()            |
|                           |
+---------------------------+

Notes: Utility functions

4) Example Object class hierarchy
---------------------------------

4.1) Vif class

            VtestObjectBase
+--------------------------------+              +----------------------------------+
|  _resp_file = ""               |              |                                  |
|  def sync()                    |              |                                  |
|  def delete()                  |              |        vr_interface_req          |
|  def get(key)                  |              |                                  |
|  def is_sent()                 |              |                                  |
|  def auto_clean()              |              |                                  |
|  cls._obj_dict = {}            |              |                                  |
+--------------------+-----------+              +-----------------+----------------+
                     |                                            |
                     +-----------+                   +------------+
                                 |                   |
                                 |       Vif         |
                      +----------+-------------------+------------+
                      |  send_packet()                            |
                      |  send_and_receive_packet()                |
                      |  send_and_compare_received_packet()       |
                      |                                           |
                      +---+--------+-------------------+------+---+
                          |        |                   |      |
                          |        |                   |      |
               +----------+        |                   |      +--------------+
               +                   +                   +                     +
         VirtualVif            VhostVif             AgentVif              FabricVif
      +---------------+    +----------------+   +----------------+    +-----------------+
      |               |    |                |   |                |    |                 |
      |               |    |                |   |                |    |                 |
      |               |    |                |   |                |    |                 |
      +---------------+    +----------------+   +----------------+    +-----------------+

Notes:
1) is_sent() => Checks if the object has been sent to vrouter or not
2) _resp_file => Gets filled up by vtest when the response is available from vrouter
3) sync() => Sync the object to vrouter (call vtest with proper args)
4) get(key) => Parses the key field from the xml file (_resp_file)
5) vif.send_packet() => Sends a packet in the vif
6) vif.send_and_receive_packet() => Send a packet and receive the reply packet
7) vif.send_and_compare_recieved_packet(packet1, rx_vif, packet2) 
    => Send packet1 and compare the reply with packet2 recieved on interface rx_vif

4.2) 
     +-----------------------------------------------+
     |                                               |
     |      vr_interface_req, VtestObjectBase        |
     |                    +                          |
     |                    |                          |
     |                    +                          |
     |                   Vif                         |
     |                    +                          |
     |                    |                          |
     |                    +                          |
     | VirtualVif   AgentVif   VhostVif   FabricVif  |
     |                                               |
     |                                               |
     +-----------------------------------------------+

Notes:
1) Example usage:
    vif = VitualVif(name="tap1", ip="10.0.3.4")
    vif.sync()
    print "Added vif with idx = " + str(vif.vif_idx)
2) vif_idx can be auto-generated if not given

4.3) NextHop class

+---------------------------------------------------------------------+
|                                                                     |
|            vr_nexthop_req, VtestObjectBase                          |
|                          +                                          |
|                          |                                          |
|                +---------+-------+                                  |
|                |                 |                                  |
|                +                 +                                  |
|            CompositeNextHop   NextHop                               |
|                                  +                                  |
|                                  |                                  |
|                                  +                                  |
|                     EncapNextHop   TunnelNextHop   ReceiveNextHop   |
|                                                                     |
|                                                                     |
+---------------------------------------------------------------------+

Notes:
1) Example usage:
    encap_nh1 = EncapNextHop(if_idx=3, dst_mac="02:00:01:00:33:22")
    encap_nh1.sync()
    print "Added encap_nh with idx " + str(encap_nh1.idx)

4.4) Flow class

+-----------------------------------------------+                      Flow
|                                               |         +--------------------------------+
|      vr_flow_req, VtestObjectBase             |         |                                |
|                   +                           |         |  sync_and_add_reverse_flow()   |
|                   |                           |         |  send()                        |
|                   +                           |         |  link_flow(flow1)              |
|                  Flow                         |         |  add_multiple(num,             |
|                   +                           |         |               incr_sip,        |
|                   |                           |         |               incr_sport,      |
|                   +                           |         |               ...)             |
|    InetFlow   Ip6Flow   NatFlow   MirrorFlow  |         |                                |
|                                               |         +--------------------------------+
|                                               |
+-----------------------------------------------+

Notes:
1) sync() => Sends message to vrouter to add a standalone forward flow
2) sync_and_add_reverse_flow() => Sends a message to add forward flow and
                                  create corresponding reverse flow, then link both
3) link_flow(flow1) => Links the current flow with "flow1"
4) add_multiple() => Add 'n' flows with incrementing sip/dip/sport/dport

4.5) Route class

+----------------------------------------+
|     vr_route_req, VtestObjectBase      |
|                  +                     |
|                  |                     |
|                  +                     |
|                Route                   |
|                  +                     |
|                  |                     |
|                  +                     |
|    BridgeRoute   IpRoute   Ip6route    |
|                                        |
+----------------------------------------+

5) Packet class

+----------------------------------------------+
|              VtestPacketBase                 |
|                    +                         |
|                    |                         |
|                    +                         |
|                VtestPacket                   |
|                    +                         |
|                    |                         |
|                    +                         |
|  IcmpPacket   MPLSoUdpPacket   VxLanPacket   |
|                                              |
|                                              |
+----------------------------------------------+

Notes:
1) Class to easily create different types of packets using scapy
```
