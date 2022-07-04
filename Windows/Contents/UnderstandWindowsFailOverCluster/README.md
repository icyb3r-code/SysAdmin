# Understand Windows Failover Cluster



## Option 1 - Traditional 

This is the traditional cluster configuration, this option means connect two servers or two nodes to the shared storage using iSCSI or FC SAN device. so the key point here is that the shared storage is on storage solution and  it can be reachable by all the cluster node and available for read and write (IO Operations)

![](attachments/Pasted%20image%2020220429083304.png)

### Network Configuration 

![](attachments/Pasted%20image%2020220429091833.png)


## Option 2 - Locally-attached SAS

![](attachments/Pasted%20image%2020220429085458.png)




## Option 3 - Two-Tier and Remote Storage

We seperate our cluster node or compute node from storage, we interconnect between these nodes, Note: SOFS means Scale-Out File Server. 

![](attachments/Pasted%20image%2020220429084208.png)



## Option 4 - Hyperconverged S2D
This options called Hyperconverged Storage Space Direct Implementation.
no need for SAS or iSCSI or SAN. the storage connected direct and localy to the server and replicated to the second node same for the second node replicated to the first ndoe.

![](attachments/Pasted%20image%2020220429084704.png)

## Option 5 - Disaggregated S2D

We can see here we seperate the Compute nodes and Storage nodes and interconnect between them, note that the local storage are have direct replication that means the are replicated directly to each others.

![](attachments/Pasted%20image%2020220429085208.png)

