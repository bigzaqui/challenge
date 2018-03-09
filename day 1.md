#day 1

##TCP/IP book
- Frame is for lower layers (ethernet), datagram is for higher .

- baud: number of transitions on a signal in each second

- a 28.8 kbps modem encodes nine bits into each transition; it runs at 3,200 baud

##network performance in AWS, learnings

- AWS tests recommend using iperf3, TODO read man iperf3
-  **Remember!** When testing the network in a server-client arch, avoid I/O by serving the data directly from memory
-  [Measuring throughput on AWS same region, multi region... good baselines](http://epamcloud.blogspot.se/2013/03/testing-amazon-ec2-network-speed.html?m=1)
-  [comparing plain HTTP against SSH for throughput can be deceiving](https://www.rightscale.com/blog/cloud-industry-insights/network-performance-within-amazon-ec2-and-amazon-s3) CPU might be a bottleneck, since there's no parallelism
-  Do not use SCP for copying big data, use pigz instead! [up to 10x](http://intermediatesql.com/linux/scrap-the-scp-how-to-copy-data-fast-using-pigz-and-nc/). Gzip is single threaded, pigz solves that.
-  AWS provides 1 Gbit limitiation in performance with normal instances, but it can reach something close to 25Gbps by using special kernel module + virtual intel network interface