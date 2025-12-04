+++
title = 'Benchmarking EC2 NVMe Storage: EBS vs Instance Store Performance'
date = '2023-10-03T14:20:00-05:00'
description = "A practical guide to benchmarking AWS EC2 NVMe storage performance using fio. Learn the differences between EBS volumes and instance store, decode fio output, and understand IOPS, throughput, and latency."
excerpt = "Ever wondered how fast your EC2 storage really is? Let's dive into benchmarking NVMe storage on AWS and understand what those numbers actually mean."
tags = ['aws', 'ec2', 'nvme', 'storage', 'performance', 'benchmarking', 'fio', 'ebs', 'devops']
categories = ['Cloud', 'Performance']
keywords = ['aws ec2', 'nvme benchmarking', 'ebs performance', 'instance store', 'fio tutorial', 'storage iops', 'throughput', 'latency']
draft = false
+++

## Quick Primer: EBS vs Instance Store

Before we start throwing benchmarks around, let's clarify what we're actually testing here.

**Instance Store NVMe**: This is the physical SSD attached directly to your EC2 host. Think of it as the internal hard drive of your server. It's blazingly fast because there's no network in the way. The catch? It's ephemeral - if your instance stops or terminates, poof, your data is gone. Perfect for caches, temporary processing, or anything you can rebuild.

**EBS (Elastic Block Store)**: This is network-attached storage that persists independently of your instance. It's like having an external drive that you can unplug from one computer and attach to another. More flexible, but there's a network hop involved which affects performance.

### EBS Volume Types: The Simple Version

AWS offers several EBS types, and honestly, the naming could be better:

**gp3 (General Purpose SSD)** - The new default and probably what you want 99% of the time. You get 3,000 IOPS and 125 MB/s baseline, and you can tune IOPS and throughput independently. It's like ordering a burger where you can customize the toppings separately from the patty size.

**gp2 (General Purpose SSD)** - The old default. IOPS scale with volume size (3 IOPS per GB), which means you sometimes had to buy a bigger volume just to get more performance. Kind of like having to order a large drink to get free refills.

**io2/io2 Block Express** - For when you need serious, consistent IOPS. We're talking up to 64,000 IOPS per volume (or 256,000 with Block Express). These are expensive and you'll know if you need them. Think databases with heavy random I/O patterns.

**st1 (Throughput Optimized HDD)** - Spinning disks optimized for sequential throughput. Good for big data workloads where you're reading large files sequentially. Cheaper, but slow for random access.

**sc1 (Cold HDD)** - The budget option for infrequently accessed data. If you're considering this, you might want to think about S3 instead.

## Benchmarking with fio

`fio` (Flexible I/O Tester) is the de facto standard for storage benchmarking. It's powerful, which also means it has about a million options. This post will only discuss the basics of it.

First, install it:
```bash
# Amazon Linux / RHEL / CentOS
sudo yum install -y fio

# Ubuntu / Debian
sudo apt-get install -y fio
```

### A Real-World Benchmark

Here's a benchmark I run regularly to get a feel for storage performance. This tests random read/write with a 4K block size, which mimics what most databases do:

```bash
sudo fio --name=random-rw \
  --ioengine=libaio \
  --iodepth=32 \
  --rw=randrw \
  --rwmixread=70 \
  --bs=4k \
  --direct=1 \
  --size=4G \
  --numjobs=4 \
  --runtime=60 \
  --group_reporting \
  --filename=/dev/nvme1n1
```

Let me break down what this actually does:

- `--name=random-rw`: Just a label for this test
- `--ioengine=libaio`: Use Linux native async I/O (fast and realistic)
- `--iodepth=32`: Keep 32 I/O operations in flight simultaneously
- `--rw=randrw`: Random read AND write mixed together
- `--rwmixread=70`: 70% reads, 30% writes (typical for many apps)
- `--bs=4k`: 4 kilobyte blocks (database-like workload)
- `--direct=1`: Bypass OS cache (we want to test the actual disk)
- `--size=4G`: Test with 4GB of data
- `--numjobs=4`: Run 4 parallel jobs (simulate concurrent access)
- `--runtime=60`: Run for 60 seconds
- `--filename=/dev/nvme1n1`: The device to test

### Decoding the fio Output

When fio finishes, it dumps a wall of text at you. Here's what actually matters:

```
random-rw: (groupid=0, jobs=4): err= 0: pid=1234: Tue Oct  3 14:30:00 2023
  read: IOPS=12.5k, BW=48.8MiB/s (51.2MB/s)(2932MiB/60001msec)
    slat (usec): min=2, max=15234, avg=23.45, stdev=89.23
    clat (usec): min=45, max=98234, avg=1876.34, stdev=2345.67
     lat (usec): min=51, max=98256, avg=1899.79, stdev=2348.12
  write: IOPS=5357, BW=20.9MiB/s (21.9MB/s)(1256MiB/60001msec)
    slat (usec): min=3, max=24567, avg=34.56, stdev=123.45
    clat (usec): min=67, max=145678, avg=2234.56, stdev=3456.78
     lat (usec): min=73, max=145701, avg=2269.12, stdev=3459.23
```

**IOPS (Input/Output Operations Per Second)**: How many read or write operations the disk can handle per second. Higher is better. In this example, we're getting 12.5k read IOPS and 5.3k write IOPS.

**BW (Bandwidth/Throughput)**: How much data we're actually moving. In this case, 48.8 MiB/s for reads and 20.9 MiB/s for writes. This is the "how fast can I copy files" number.

**Latency Numbers** (this is where it gets interesting):
- `slat` (submission latency): Time to submit the I/O request to the kernel
- `clat` (completion latency): Time waiting for the I/O to complete  
- `lat` (total latency): The full round trip time - **this is what you feel**

The `avg` (average) latency is important, but the `max` tells you about those annoying hiccups. In the example above, average read latency is ~1.9ms, but max spiked to 98ms. That spike is what makes your app feel sluggish occasionally.

