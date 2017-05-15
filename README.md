
If you apply these codes in your futher papers, you would cite these relevant papers:
- H. Li, H. Zhou, W. Quan, H. Zhang and S. Yu, "HCaching: High-speed Caching for Information-Centric Networking," submitted to *GLOBECOM*, 2017.
- W. Quan, C. Xu, J. Guan, H. Zhang, and L. A. Grieco, ''Scalable name lookup with adaptive prefix bloom filter for named data networking,'' *IEEE Communications Letters*, vol.~18, pp. 102--105, 2014.
- Thomas, Yannis *et al.*, "Object-oriented Packet Caching for ICN," *ICN*, 2015.


# Design Overview
On the basis of Thomas "ccncaching" in ns-3, CCNHCaching is equipped with **the capacity which simulates throughput of ICN cache routers**, and implements LRU, OPC [1], SSD\_DRAM [2], HCaching caching systems in three differenct cache placement policies: edge caching, univercal caching and betweenness caching[3].
CCNHCaching architecure is shown as follow:
![](https://github.com/iplab2016/wiki/blob/master/ccncaching-HCaching/images/ccnhcaching-v3.jpg)
Fig. 1. The CCNHCaching system structure.


Fig. 1a depicts CCNHCaching structural diagram.
Interfaces in CCNHCaching is realized based on point-to-point model of NS-3, which puts bits on a corresponding channel "wire".
Point-to-point network device models egress that can send bits at certain data-rate with egress queue.
The receiving device models ingress interface without ingress queue, that can receive any data rate queue.
Therefore, there is no need, nor way to set a receive data rate in this model.
By setting the data-rate on the transmitter of both devices connected to a given point-to-point-channel one can model a symmetric channel;or by setting different data-rates one can model an asymmetric channel.

Three main components form CCNHCaching: Pending Interest Table (PIT), Forwarding Information Base (FIB) and Content Store (CS).
PIT keeps track of the content items that have recently been requested and not yet responded.
FIB is a table that associates prefixes of content names to one or multiple next hop ICN routers.
CS plays the role of buffer and caching system, which store content after it have been forwarded to serve future requests.
Specially, we implement CS with our high-speed caching throughput simulating.

Fig. 1b presents CS structural diagram.
CS can be organized by one or multiple types of memories, whose access time is simulating by respective simulator.
These memories simulators include SSD simulator, SRAM simulator and DRAM simulator. 
Upon reception of an user request on an ingress \emph{I}, CCNHCaching checks for the requested content availability in CS.
If the content is available, CS send it back through \emph{I}.
Otherwise, CCNHCaching forwards out the request and wait for this content.
Read commands from memory simulators are pushed to Read Command Queue, and write commands are pushed to Write Command Queue.
Read/write commands are not popped from respective queue until routers finish reading or writing content from memories.
The popup operations are executed exactly at access time of memories.
After reading/writing content from CS, CCNHCaching can obtain the operating latency from memory simulators.
This latency is added to current simulation time, and used as scheduling time of next event. 
Based on these latency simulations, CCNHCaching can exactly simulate throughput of caching system.



This simulator is just alpha version. 
It was tested only in ns-3-allinone.3.18, please follow the ns-3 instractions to set up ns-3 and the module as follow. 

# Installation ns-3.18.1

## Download ns-3.18.1 

```shell
$ wget https://www.nsnam.org/release/ns-allinone-3.18.1.tar.bz2

$ tar -xvf ns-allinone-3.18.1.tar.bz

$ cd ns-allinone-3.18.1
```

## Modify point-to-point module
```shell
$ vi ns-3.18.1/src/point-to-point/model/point-to-point-net-device.cc +603
```

You should add ccn protocol support code as follows
```cpp
uint16_t
PointToPointNetDevice::PppToEther (uint16_t proto)
{
  switch(proto)
    {
    case 0x0021: return 0x0800;   //IPv4
    case 0x0057: return 0x86DD;   //IPv6
    case 0x0061: return 0x88DD;   //ccn
    default: NS_ASSERT_MSG (false, "PPP Protocol number not defined!");
    }
  return 0;
}

uint16_t
PointToPointNetDevice::EtherToPpp (uint16_t proto)
{
  switch(proto)
    {
    case 0x0800: return 0x0021;   //IPv4
    case 0x86DD: return 0x0057;   //IPv6
    case 0x88DD: return 0x0061;   //IPv6
    default: NS_ASSERT_MSG (false, "PPP Protocol number not defined!");
    }
  return 0;
}
```

```shell
vi ns-3.18.1/src/point-to-point/model/ppp-header.cc +70
```
Add ccn support in PppHeader::Print function!
```cpp
void
PppHeader::Print (std::ostream &os) const
{
  std::string proto;

  switch(m_protocol)
    {
    case 0x0021: /* IPv4 */
      proto = "IP (0x0021)";
      break;
    case 0x0057: /* IPv6 */
      proto = "IPv6 (0x0057)";
      break;
    case 0x0061: /* ccn */
      proto = "IPv6 (0x0061)";
      break;
    default:
      NS_ASSERT_MSG (false, "PPP Protocol number not defined!");
    }
  os << "Point-to-Point Protocol: " << proto;
}
```
## Build ns-3
Run the command as follows:
```shell
$ ./build.py
```


# Install ccncaching module

## Download ccnhcaching source code
```shell
$ cd ns-3.18.1/src/
```

```shell
$ git clone https://github.com/iplab2016/ccnhcaching
```




## configure and make it

```shell
$ cd ../../
$ CXXFLAGS="-std=c++11" LINKFLAGS="-lbf -lpthread" ./waf configure --enable-test
$ ./waf
```

# Note
If there is not libbf library, please install it as [libbf github wiki](https://github.com/iplab2016/libbf)

# CONTACT
Melon Li
melon.haifeng@gmail.com

# REFERENCES
[1] Thomas, Yannis, et al. "Object-oriented Packet Caching for ICN" Proceedings of the 2nd international conference on Information-centric networking. ACM, 2015.

[2] Rossini G, Rossi D, Garetto M, et al. Multi-terabyte and multi-gbps information centric routers[C]//INFOCOM, 2014 Proceedings IEEE. IEEE, 2014: 181-189.

[3] Chai, Wei Koong, et al. "Cache “less for more” in information-centric networks." NETWORKING 2012. Springer Berlin Heidelberg, 2012. 27-40.

[4] Kim Y, Tauras B, Gupta A, et al. Flashsim: A simulator for nand flash-based solid-state drives[C]//Advances in System Simulation, 2009. SIMUL'09. First International Conference on. IEEE, 2009: 125-131.
[5] Kim Y, Yang W, Mutlu O. Ramulator: A fast and extensible DRAM simulator[J]. IEEE Computer Architecture Letters, 2016, 15(1): 45-49.

