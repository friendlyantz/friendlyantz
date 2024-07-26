---
excerpt: "Book notes: High PerformanceBrowser Networking"
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - template
---

https://hpbn.co/

# Ch 1

- Propagation delay =  Time sender -> receiver, 
	- function of distance over ~`speed of light resisted by fibre/medium` (refractive index value of fibre is 1.4-1.6, so add approx 50% to time in ref to vacuum)
	
- Transmission delay - Time to push packet’s bits into the link, 
	- function of the packet’s length and data rate of the link.
- Processing delay - Time to process:
	- the packet header, 
	- check for bit-level errors,
	- determine destination.
- Queuing delay

- i.e. New York -> Sydney 
	- ~16000km, 
	- 80ms in fibre vs 53ms in vac
	- 160ms for round trip. 
	- Total is 300ms approx with other delays
	- NOTE: 100-200ms is human perceptible “lag”
Content delivery network (CDN) is addressing this

> Bufferbloat in Your Local Router - routers ome with large buffers, but it breaks TCP’s congestion avoidance mechanisms. CoDel active queue management algorithm is solving this ([see more](https://queue.acm.org/detail.cfm?id=2209336))

## Last-Mile Latency

> *Fiber-to-the-home, on average, has the best performance in terms of latency, with 18 ms average during the peak period, with cable having 26 ms latency and DSL 44 ms latency.*

```sh
traceroute google.com
traceroute to google.com (142.250.76.110), 64 hops max, 40 byte packets
 1  192.168.41.168 (192.168.41.168)  6.242 ms  7.192 ms  5.198 ms
 2  * * *
 3  * * *
 4  * * *
 5  255.0.0.4 (255.0.0.4)  73.592 ms  22.153 ms  21.414 ms
 6  255.0.0.4 (255.0.0.4)  20.029 ms  21.712 ms  22.307 ms
 7  * * *
 8  bundle-ether26.exi-core30.melbourne.telstra.net (203.50.61.160)  61.505 ms  18.472 ms *
 9  bundle-ether1.lon-edge903.melbourne.telstra.net (203.50.11.199)  85.512 ms  31.766 ms  27.146 ms
10  142.250.162.46 (142.250.162.46)  25.318 ms  22.595 ms  26.489 ms
11  * * *
12  142.250.230.160 (142.250.230.160)  26.950 ms
    216.239.59.108 (216.239.59.108)  32.450 ms
    142.250.230.156 (142.250.230.156)  19.845 ms
13  192.178.98.132 (192.178.98.132)  24.315 ms
    192.178.98.214 (192.178.98.214)  23.034 ms
    192.178.98.62 (192.178.98.62)  26.535 ms
14  192.178.244.29 (192.178.244.29)  37.345 ms
    192.178.245.1 (192.178.245.1)  41.555 ms
    192.178.243.225 (192.178.243.225)  36.744 ms
15  216.239.51.72 (216.239.51.72)  47.287 ms
    142.251.240.248 (142.251.240.248)  39.488 ms  37.153 ms
16  192.178.97.163 (192.178.97.163)  35.953 ms
    192.178.97.167 (192.178.97.167)  35.766 ms
    192.178.97.235 (192.178.97.235)  35.521 ms
17  142.250.212.137 (142.250.212.137)  45.843 ms  49.993 ms  35.098 ms
18  syd09s24-in-f14.1e100.net (142.250.76.110)  37.290 ms  37.704 ms  42.905 ms
```

```sh
traceroute rmit.edu.au
traceroute to rmit.edu.au (131.170.0.105), 64 hops max, 40 byte packets
 1  192.168.41.168 (192.168.41.168)  5.491 ms  5.034 ms  7.123 ms
 2  * * *
 3  255.0.0.1 (255.0.0.1)  28.771 ms * *
 4  * * *
 5  255.0.0.4 (255.0.0.4)  77.356 ms  24.782 ms  22.336 ms
 6  255.0.0.4 (255.0.0.4)  24.182 ms  35.384 ms  26.657 ms
 7  * * *
 8  ae20-20.cla-ice302.melbourne.telstra.net (203.50.61.163)  95.129 ms  26.061 ms  26.782 ms
 9  bundle-ether25.cla-core30.melbourne.telstra.net (203.50.61.176)  29.641 ms  35.206 ms  20.190 ms
10  bundle-ether3.hay-core30.sydney.telstra.net (203.50.13.132)  36.528 ms  33.684 ms  43.443 ms
11  * * *
12  bundle-ether1.sydo-core04.telstraglobal.net (203.50.13.94)  148.803 ms  35.028 ms  34.955 ms
13  bundle-ether1.sydo-core04.telstraglobal.net (203.50.13.94)  202.046 ms  261.575 ms  177.922 ms
14  i-10604.1wlt-core02.telstraglobal.net (202.84.141.225)  174.757 ms  169.941 ms  182.748 ms
15  i-10604.1wlt-core02.telstraglobal.net (202.84.141.225)  176.272 ms  242.165 ms  184.791 ms
16  i-93.tlot02.bi.telstraglobal.net (202.84.253.86)  173.080 ms  174.665 ms  171.359 ms
17  l3-peer.eqnx03.pr.telstraglobal.net (134.159.61.106)  176.523 ms  173.110 ms  174.848 ms
18  * * *
19  aarnet-pty.ear4.london1.level3.net (217.163.113.74)  377.656 ms  301.677 ms  324.889 ms
20  et-5-1-0.pe1.crlt.vic.aarnet.net.au (113.197.15.20)  295.749 ms  284.176 ms  295.449 ms
21  138.44.64.77 (138.44.64.77)  355.950 ms  430.433 ms  336.025 ms
22  * * *
----
64  * * *
```

> *Latency, not bandwidth, is the performance bottleneck for most websites!*

Optical fibers have a distinct advantage when it comes to bandwidth because each fiber can carry many different wavelengths (channels) of light through a process known as wavelength-division multiplexing (WDM)
	as of 2010: multiplex was possible with over 400 wavelengths with the peak capacity of 171 Gbit/s per channel, which translates to over 70 Tbit/s of total bandwidth for a single fiber link!

half of all Internet traffic is video streaming 

- Delivering Higher Bandwidth is not really limited, as we can put more cables, improve WDM, etc
- Delivering Lower Latencies is limited to speed of light / material science. (~30% max improvements possible)
Hence:
- we need to reduce round trips,
- move the data closer to the client, and
- build applications that can hide the latency through
	- caching,
	- pre-fetching, and a variety of similar techniques
