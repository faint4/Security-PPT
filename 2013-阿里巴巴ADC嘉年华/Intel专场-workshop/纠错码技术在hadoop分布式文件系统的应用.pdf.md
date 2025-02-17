Enable and Optimize Erasure Code for Big data on Hadoop
High Performance Computing, Intel Jun Jin (jun.i.jin@intel.com)

Agenda
ˇ Background and overview ˇ Codec performance ˇ Block placement Policy ˇ Performance evaluation ˇ Local Repairable Codes ˇ Summary

Overview
ˇ Hadoop and HDFS is popular now, the driver of growth in IPDC market
ˇ Data is growing faster than infrastructure ˇ 3x replication of everything is too expensive, especially on
Petabyte scale ˇ Use erasure codes for cold data
LRU to track data block, code cold files Classical codes are unsuitable for distributed environment Google GFS2(Colossus), Microsoft Azure and facebook(hdfs-20)

Revisit HDFS *
* From Apache website http://hadoop.apache.org/docs/stable/hdfs_design.html

HDFS Raid overview *

RaidNode RaidShell

NameNode
ˇ Obtain missing blocks ˇ Get files to raid
ˇ Create parity files ˇ Fix missing blocks
App Master

DataNodes
ˇ Recover files while reading Raid File System

DRFS: provides access to files and transparently recovers any corrupt or missing blocks encountered when reading a file RaidNode: a daemon that creates and maintains parity files for all data files stored in the DRFS BlockFixer, which periodically recomputes blocks that have been lost or corrupted RaidShell, allows administrator to manually trigger the recomputation of missing or corrupt blocks and check for files ErasureCode, which provides the encode and decode of the bytes in blocks
* From Apache website http://wiki.apache.org/hadoop/HDFS-RAID

Replication vs Erasure codes
ˇ Erasure coding, facebook's approach for code data

640MB data => 10 blocks

40% overhead

1

2

3

4

5

6

7

8

9 10

P1 P2 P3 P4 Parity file

Source file

In facebook, older files are switched from 3 replication to (10, 4) Reed Solomon

Overview - Performance characterization

ˇ Perf. characterization for baseline HDFS Raid (v2.0.2)
­ Reed Solomon(RS) encoding is CPU intensive, takes 80+% of time ­ Network and disk IO takes ~20% ­ High bin CPU could greatly improve the codec performance

Overall Encoding time ratio - Reed Solomon (10,4)

7.58%

9.92%

HDFS Block Read HDFS write fixed block RS Encoding

82.50%

NN: 1 EP server (X5680) ˇ 64G DDR3 1333Mhz

ˇ 2 x 1TB SATA

ˇ 2 x Broadcom NetXtreme II BCM5709 Gigabit, bind by mode 6

ˇ RH 6.2 64 bit, hadoop2.0.2

ˇ 36G DDR3 1333Mhz

DN: 12 x EP

ˇ 5 x 500GB SATA

server(E5645):

ˇ 2 x Broadcom NetXtreme II BCM5709 Gigabit, bind by mode 6

ˇ RH 6.2 64 bit, hadoop2.0.2

Agenda
ˇ Background and overview ˇ Codec performance ˇ Block placement Policy ˇ Performance evaluation ˇ Local Repairable Codes ˇ Summary

Performance of Encoder

Summary:
ˇ Pure encode/decode performance, no IO included ˇ Best performance based on successfully vectorize XOR loop for CRS2

ENCODE PERFORMANCE (MB/S)

E5-2680(2.7Ghz) Encode (MB/s)

E3-1270 v2 @3.5GHz Encode

2662 2110

50 90
JAVA VRS (eva)

312 378

341 402

959 1051

623 709

1383 1203

913 1035

VRS w=8

VRS w=16

CRS1 w=8

CRS1 w=16

CRS2 w=8

CRS2 w=16

CRS2 w=8 vectorized

VRS = Vandermonde Reed Solomon CRS1 = Cauchy Reed Solomon Original CRS2 = Cauchy Reed Solomon Good
Note: Test based on Jerasure library: http://web.eecs.utk.edu/~plank/plank/papers/CS-08-627.html

Performance of Decoder
Summary:
ˇ Pure encode/decode performance, no IO included ˇ Best performance based on successfully vectorize XOR loop for CRS2

ENCODE PERFORMANCE (MB/S)

E5-2680(2.7Ghz) Decode (MB/s)

E3-1270 v2 @3.5GHz Decode

1395 1163

1384 1060

10551127

868 822

1655 1488

55 60
JAVA VRS (eva)

VRS w=8

368 402

248 258

VRS w=16

CRS1 w=8

CRS1 w=16

CRS2 w=8

CRS2 w=16

CRS2 w=8 vectorized

Note: Test based on Jerasure library: http://web.eecs.utk.edu/~plank/plank/papers/CS-08-627.html

New codec integration
ˇ The original codec in HDFS implemented Classical ReedSolomon (RS), which proved very slow
ˇ Cauchy RS is much better than Classical RS ˇ Native C code for Cauchy RS, from Jerasure 1.2 ˇ Used direct buffer for Java and C communication to
eliminate unnecessary memory copy

New codec integration in HDFS

ˇ Performance compare

Encoding time ratio RS (10,4)

HDFS Block Read HDFS write fixed block

7.58% 9.92%

RS Encoding

82.50%

NN: 1 EP

ˇ

server

ˇ

(X5680)

ˇ

ˇ ˇ DN: 12 x EP ˇ server(E5645) ˇ

ˇ

64G DDR3 1333Mhz 2 x 1TB SATA 2 x Broadcom NetXtreme II BCM5709 Gigabit, bind by mode 6 RH 6.2 64 bit, hadoop2.0.2-rc1 36G DDR3 1333Mhz 5 x 500GB SATA 2 x Broadcom NetXtreme II BCM5709 Gigabit, bind by mode 6 RH 6.2 64 bit, hadoop2.0.2-rc1

Encoding time ratio JE (10,4)

18% 32%

50%
Overall performance greatly improved by 20% - 2X Bottleneck move to network now.

HDFS Block Read HDFS write fixed block RS Encoding

IntelŽISA-L Library
ˇ IntelŽIntelligent Storage Acceleration Library ˇ Algorithmic Library to address key Storage market
segment needs
 Optimized for Intel Architecture  Enhances efficiency, data integrity, security/encryption, erasure
codes, compression, CRC, AES, etc.
ˇ Benefits of using IntelŽISA-L
 Highly optimized based on Intel New Instructions
ˇ Working on ISA-L library for HDFS erasure coding

Agenda
ˇ Background and overview ˇ Codec performance ˇ Block placement Policy ˇ Performance evaluation ˇ Local Repairable Codes ˇ Summary

New Block Placement Policy
ˇ Problem: Encoding brings heavy network and disk IO ˇ Blocks are randomly placed in HDFS by default, data locality
can't be guaranteed. ˇ Need a new block placement policy.

Agenda
ˇ Background and overview ˇ Codec performance ˇ Block placement Policy ˇ Performance evaluation ˇ Local Repairable Codes ˇ Summary

Performance evaluation

ˇ Workload 1: 1TB data, each file 5GB, block size 256MB ˇ Workload 2: 145 GB data, each file 2.5GB, block size 256MB ˇ Used 60 containers (5 on each server), 59 for map tasks
and 1 for Application Master
ˇ Compared 3 different code
1. Baseline code with VRS codes 2. Integrated JE code with default block placement 3. Integrated JE code with new block placement

NN: 1 EP server (X5680)
DN: 12 x EP server(E5645)

ˇ 64G DDR3 1333Mhz ˇ 2 x 1TB SATA ˇ 2 x Broadcom NetXtreme II BCM5709 Gigabit, bind by mode 6 ˇ RH 6.2 64 bit, hadoop2.0.2-rc1
ˇ 36G DDR3 1333Mhz ˇ 5 x 500GB SATA ˇ 2 x Broadcom NetXtreme II BCM5709 Gigabit, bind by mode 6 ˇ RH 6.2 64 bit, hadoop2.0.2-rc1

New Block Placement

ˇ Performance overview on two data set

Workload 1: 145GB data Baseline: Default + VRS
Integrated JE codec
Integrated JE + new block placement

Time 165s 135s
69s

Speedup 1
1.22X
2.4X

Workload 2: 1 TB data
Baseline: Default + VRS Integrated JE codec Integrated JE + new block placement

Time 1086s 738s
427s

Speedup 1
1.47X
2.53X

Performance improved up to 2.53X over the baseline

New Block Placement
ˇ Network BW greatly reduced based on new block placement

Teragen Encoding: baseline Teragen

Encoding: JE

Teragen

Encoding: Final

Notes: Use Teragen to generate source data then do encode

New Block Placement
ˇ CPU utilization for different optimization steps

Teragen

Encoding: baseline Teragen

Encoding: JE

Teragen

Encoding: Final

New Block Placement

ˇ Disk utilization for different optimization steps

Teragen

Encoding: baseline Teragen

Encoding: JE

Teragen

Encoding: Final

TCO analysis (E3 vs E5)
ˇ Assume to store 1PB data, EC with (10,4) codec needs 70 servers, while 3 copy solution needs about 150 servers.
ˇ TCO for 3 copy 940K vs EC 614K
ˇ $/GB : 0.9 vs 0.59

Agenda
ˇ Background and overview ˇ Codec performance ˇ Block placement Policy ˇ Performance evaluation ˇ Local Repairable Codes ˇ Summary

Local Repairable Codes*

ˇ Local repairable codes to reduce network/disk IO during

decoding

60% overhead

Source file

1

2

3

4

5

c2 c1

c3 c4

c5

S1

6

7

8

9 10

S2

P1

P2 P3 P4 P1, P2, P3, P4: Reed Solomon encoding

Use the first 5 and second 5 blocks create 2 local parity

blocks,S1, S2

One lost block requires only 5 other blocks to be read.

S3

Choose coefficients to let s3 = s1 + s2

Single failures represent 99.75% of recoveries**
* Erasure Coding in Windows Azure Storage ** Source: Rethinking Erasure Codes for Cloud File Systems: Minimizing I/O for Recovery and Degraded Reads
Simple Regenerating Codes: Network Coding for Cloud Storage, Novel Codes for Cloud Storage:https://mhi.usc.edu/files/2012/04/Sathiamoorthy-Maheswaran.pdf

Other considerations
ˇ HAR support ˇ Block balance
How to balance the blocks when new node added or deleted
ˇ Set replication ˇ Raid Shell tool ˇ Parallel reader ˇ And many others...

Summary
ˇ Jerasure CRS codec proved bring good performance advantage over baseline Java implementation, we working on ISA-L now
ˇ Developed a new block placement, which effectively reduces the network bandwidth
ˇ Working on local repairable codes. ˇ Erasure Code is an effective way to reduce storage cost for
cold data.

