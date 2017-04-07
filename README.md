On the basis of Yannis's "ccncaching" in ns-3, ccnhcaching is equipped with the capacity which simulates throughput of ICN cache routers, and implements LRU, OPC [1], SSD\_DRAM [2], HCaching caching systems in three differenct cache placement policies: edge caching, univercal caching and betweenness caching[3]. The architecure is shown as follow:
![](https://github.com/iplab2016/wiki/blob/master/ccncaching-HCaching/images/ccnhcaching.jpg)

This module was tested only in ns-3-allinone.3.18, please follow the ns-3 instractions to set up ns-3 and the module as follow. 

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

