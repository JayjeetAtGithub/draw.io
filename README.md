# skyhook-overhead-observations

## Node Configuration
```
m510
CPU: Eight-core Intel Xeon D-1548 at 2.0 GHz
RAM: 64GB ECC Memory (4x 16 GB DDR4-2133 SO-DIMMs)
Disk: 256 GB NVMe flash storage
NIC: Dual-port Mellanox ConnectX-3 10 GB NIC (PCIe v3.0, 8 lanes
```

## Benchmarks

### 1. Network Benchmarks
<img height="300" width="350" src="https://user-images.githubusercontent.com/33978990/89516252-d47ad300-d7f5-11ea-90d0-d1f3e794356e.png"/>

This machines has a 10Gb/s NIC and we are using that.

* Min. Bandwidth: `6.5 Gb/s`
* Max. Bandwidth: `8.25 Gb/s`

The average is `7.61 Gb/s` (951 MB/s).

### 2. Rados Benchmarks

The goal of this benchmark is to run the rados bench as aggressively as possible and try to find out when the 10Gb network becomes the bottleneck.

We used 2 machines, one acting the ceph client and one acting as the OSD backend. I would call them client and osd. So the osd machine has a 256GB NVMe block device.

## Case I:

### 1. 4 OSD on a single NVMe device

#### 1 thread
<img height="300" width="350" src="https://user-images.githubusercontent.com/33978990/89557300-35bf9800-d830-11ea-8ed4-48f95296558d.png"/>

As seen from the Graph, a 1 thread couldn't fully utilize the entire througput of the 4 OSDs and thus gives only the `ceph tell` limit, i-e `350 MB/s`.

#### 4 threads
<img height="300" width="350" src="https://user-images.githubusercontent.com/33978990/89557605-af578600-d830-11ea-9709-1a1d27af7e2e.png"/>

With 4 threads working concurrently, the throughput goes almost double as that with 1 thread as expected. Our client is able to read more amount of data / second.

#### 8 threads (using all the threads available)
<img height="300" width="350" src="https://user-images.githubusercontent.com/33978990/89557838-0fe6c300-d831-11ea-8069-04ca61ddb383.png"/>

Througput continues to improve.

#### 10 (using few threads than available) respectively
<img height="300" width="350" src="https://user-images.githubusercontent.com/33978990/89563799-a7501400-d839-11ea-8355-7f49389a6fe9.png"/>

There is no significant improvement in going from 8 to 10 threads. 
The maximum througput given by the OSDs combined is around the networks limit of (~950 MB/s). 
So, maybe we can say the network has become the bottleneck.

## Case II:

A single OSD on a single NVMe

#### 10 threads (all the threads available)
<img height="300" width="350" src="https://user-images.githubusercontent.com/33978990/89565891-d916aa00-d83c-11ea-8ce2-4bcaaaff643e.png"/>

Its almost half of what given by 4 OSDs on a single NVMe. So, the througput increases with no of OSDs and bottlenecks the network eventually.


### 3. FIO benchmarks
```
sudo fio --filename=/dev/nvme0n1p4 --direct=1 --rw=read --bs=4m --ioengine=libaio --iodepth=16 --runtime=120 --numjobs=1 --time_based --group_reporting --name=throughput-test-job --eta-newline=1 --readonly
```

The `/dev/nvme0n1p4` performs at `2540MB/s` (read) which is almost thrice our network link capacity at nominal configuration. (I happen to find out some bug is FIO plot, so didnt use the popper workflow)
