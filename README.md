# 4.5


Problem Statement :


1.Explain what is checksum and the importance of checksum and how hadoop performs checksum

2.Explain the anatomy of file write to HDFS

3.Explain how HDFS handles failures during file write





Q1. Explain what is checksum and the importance of checksum and how hadoop performs checksum


Checksums are used to ensure the integrity of a file after it has been transmitted from one storage device to another. This can be across the Internet or simply between two computers on the same network. Either way, if you want to ensure that the transmitted file is exactly the same as the source file, you can use a checksum.

The checksum is calculated using a hash function and is normally posted along with the download. To verify the integrity of the file, a user calculates the checksum using a checksum calculator program and then compares the two to make sure they match.

Checksums are used not only to ensure a corrupt-free transmission, but also to ensure that the file has not been tampered with. When a good checksum algorithm is used, even a tiny change to the file will result in a completely different checksum value.

Hadoop checksum :

It works in below way.

The HDFS client software implements checksum checker. When a client creates an HDFS file, it computes a checksum of each block of the file and stores these checksums in a separate hidden file in the same HDFS namespace.
When a client retrieves file contents, it verifies that the data it received from each DataNode matches the checksum stored in the associated checksum file.
If not, then the client can opt to retrieve that block from another DataNode that has a replica of that block.
If checksum of another Data node block matches with checksum of hidden file, system will serve these data blocks.


Q2.Explain the anatomy of file write to HDFS


1.DistributedFileSystem object do a RPC call to namenode to create a new file in filesystem namespace with no blocks associated to it

2.Namenode process performs various checks like 
a) client has required permissions to create a file or not 
b) file should not exists earlier. In case of above exceptions it will throw an IOexception to client

3.Once the file is registered with the namenode then client will get an object i.e. FSDataOutputStream which in turns embed DFSOutputStream object for the client to start writing data to.       DFSoutputStream handles communication with the datanodes and namenode.

4.As client writes data DFSOutputStream split it into packets and write it to its internal queue i.e. data queue and also maintains an acknowledgement queue.

5.Data queue is then consumed by a Data Streamer process which is responsible for asking namenode    to allocate new blocks by picking a list of suitable datanodes to store the replicas.

6.The list of datanodes forms a pipeline and assuming a replication factor of three, so there will be three nodes in the pipeline.

7.The data streamer streams the packets to the first datanode in the pipeline, which then stores the packet and forward it to second datanode in the pipeline. Similarly the second node stores the packet and forward it to next datanode or last datanode in the pipeline

8.Once each datanode in the pipeline acknowledge the packet the packet is removed from the acknowledgement queue.




Q3.Explain how HDFS handles failures during file write



1.First, the pipeline is closed, and any packets in the ack queue are added to the front of the data queue so that datanodes that are downstream from the failed node will not miss any packets.

2.The current block on the good datanodes is given a new identity, which is communicated to the namenode, so that the partial block on the failed datanode will be deleted if the failed datanode recovers later on.

3.The failed datanode is removed from the pipeline, and the remainder of the block’s data is written to the two good datanodes in the pipeline.

4.The namenode notices that the block is under-replicated, and it arranges for a further replica to be created on another node. Subsequent blocks are then treated as normal.

5.It’s possible, but unlikely, that multiple datanodes fail while a block is being written. As long as dfs.replication.min replicas (which default to one) are written, the write will succeed, and the block will be asynchronously replicated across the cluster until its target replication factor is reached (dfs.replication, which defaults to three).
