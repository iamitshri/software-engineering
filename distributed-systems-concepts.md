# Concepts
- Majority of these concepts are from  https://www.educative.io/courses/grokking-adv-system-design-intvw
##  Bloom Filters
- Use Bloom filters to quickly find if an element might be present in a set.
- It can **definitely tell if the element is not present** but give you a probabilistic value **if its present** in the set.
- BigTable and Cassandra uses bloom filters
##  Consistent Hashing
- Act of distributing data across the nodes
- Consistent Hashing technique stores the data managed by a distributed system in a ring. Each node in the ring is assigned a range of data.
- This scheme still faces hotspot & non-uniform data and load issue.  Vnodes solve this problem 
- Instead of assigning a single token to a node, the hash range is divided into multiple smaller ranges, and each physical node is assigned several of these smaller ranges. Each of these subranges is considered a Vnode. With Vnodes, instead of a node being responsible for just one token, it is responsible for many tokens (or subranges).
- challenges:
  - How do we know which exact node to store the data? 
  - Where the data lives?
  - If the node is removed or added, which data needs to be moved
  - Minimize the data movement when nodes join or leave the cluster
- Consistent Hashing maps data to physical nodes and ensures that only a small set of keys move when servers are added or removed.
##  Quorum
- In a distributed environment, a quorum is the minimum number of servers on which a distributed operation needs to be performed successfully before declaring the operation’s overall success.
##  Leader and Follower
- Allow only a single server (called leader) to be responsible for data replication and to coordinate work.
##  Write-ahead Log
- To guarantee durability and data integrity, each modification to the system is first written to an append-only log on the disk. This log is known as Write-Ahead Log (WAL) or transaction log or commit log. Writing to the WAL guarantees that if the machine crashes, the system will be able to recover and reapply the operation if necessary.
##  Segmented Log
- Break down the log into smaller segments for easier management.
- Easier management and better performance 
- Cassandra uses log segmentation for storing commit log
##  High-Water mark
- Keep track of the last log entry on the leader, which has been successfully replicated to a quorum of followers. The index of this entry in the log is known as the High-Water Mark index. The leader exposes data only up to the high-water mark index.
- When leader dies, there may be some record 
- For each data mutation, the leader first appends it to WAL and then sends it to all the followers. Upon receiving the request, the followers append it to their respective WAL and then send an acknowledgment to the leader. The leader keeps track of the indexes of the entries that have been successfully replicated on each follower. The high-water mark index is the highest index, which has been replicated on the quorum of the followers. The leader can propagate the high-water mark index to all followers as part of the regular Heartbeat message. The leader and followers ensure that the client can read data only up to the high-water mark index. This guarantees that even if the current leader fails and another leader is elected, the client will not see any data inconsistencies.
- Kafka: To deal with non-repeatable reads and ensure data consistency, Kafka brokers keep track of the high-water mark, which is the largest offset that all In-Sync-Replicas (ISRs) of a particular partition share. Consumers can see messages only until the high-water mark.
##  Lease
- Instead of lock, client gets a time bound lease which grants exclusive access to the resource.
- Heartbeating is one of the mechanisms for detecting failures in a distributed system.
##  Heartbeat
- Each server periodically sends a heartbeat message to a central monitoring server or other servers in the system to show that it is still alive and functioning.
##  Gossip Protocol
- Each node keeps track of state information about other nodes in the cluster and gossip (i.e., share) this information to one other random node every second. This way, eventually, each node gets to know about the state of every other node in the cluster.
- Used in a decentralized cluster with peer to peer communication mechanism
- Dynamo & Cassandra use gossip protocol which allows each node to keep track of state information about the other nodes in the cluster, like which nodes are reachable, what key ranges they are responsible for, etc.
##  Phi Accrual Failure Detection
- Helps in declaring a node dead 
- Phi Accrual Failure Detector makes a distributed system efficient as it takes into account fluctuations in the network environment and other intermittent server issues before declaring a system completely dead.
- Use adaptive failure detection algorithm as described by Phi Accrual Failure Detector
- This algorithm uses historical heartbeat information to make the threshold adaptive
- With Phi Accrual Failure Detector, if a node does not respond, its suspicion level is increased and could be declared dead later.
##  Split-brain
- A leader is declared dead and the new leader is elected. After some time the dead leader comes back to life. Now there are two active leaders
- The common scenario in which a distributed system has two or more active leaders is called split-brain.
- Split-brain is solved through the use of Generation Clock, which is simply a monotonically increasing number to indicate a server’s generat
##  Fencing
- Fencing is the idea of putting a fence around a previously active leader so that it cannot access cluster resources and hence stop serving any read/write request.
##  Checksum
- When a system is storing some data, it computes a checksum of the data, and stores the checksum with the data. When a client retrieves data, it verifies that the data it received from the server matches the checksum stored. If not, then the client can opt to retrieve that data from another replica.
##  Vector Clocks
- Use Vector clocks to keep track of value history and reconcile divergent histories at read time.
- A vector clock is effectively a (node, counter) pair. One vector clock is associated with every version of every object. If the counters on the first object’s clock are less-than-or-equal to all of the nodes in the second clock, then the first is an ancestor of the second and can be forgotten. Otherwise, the two changes are considered to be in conflict and require reconciliation. Such conflicts are resolved at read-time, and if the system is not able to reconcile an object’s state from its vector clocks, it sends it to the client application for reconciliation (since clients have more semantic information on the object and may be able to reconcile it). Resolving conflicts is similar to how Git works. If Git can merge different versions into one, merging is done automatically. If not, the client (i.e., the developer) has to reconcile conflicts manually.
##  CAP Theorem
- CAP theorem states that it is impossible for a distributed system to simultaneously provide all three of the following desirable properties:
  - Consistency ( C ): All nodes see the same data at the same time. It is equivalent to having a single up-to-date copy of the data.
  - Availability ( A ): Every request received by a non-failing node in the system must result in a response. Even when severe network failures occur, every request must terminate.
  - Partition tolerance ( P ): A partition-tolerant system continues to operate despite partial system failure or arbitrary message loss. Such a system can sustain any network failure that does not result in a failure of the entire network. Data is sufficiently replicated across combinations of nodes and networks to keep the system up through intermittent outages.
##  PACELEC Theorem
- We cannot avoid partition in a distributed system, therefore, according to the CAP theorem, a distributed system should choose between consistency or availability.
- One place where the CAP theorem is silent is what happens when there is no network partition? What choices does a distributed system have when there is no partition
- The PACELC theorem states that in a system that replicates data:
- if there is a partition (‘P’), a distributed system can tradeoff between availability and consistency (i.e., ‘A’ and ‘C’);
- else (‘E’), when the system is running normally in the absence of partitions, the system can tradeoff between latency (‘L’) and consistency (‘C’).
##  Hinted Handoff
- If we have replication factor of 3 nodes and the client is writing with quorum consistency level. This means that even if one node is down client can still write to remaining two nodes to fulfill consistency level.
- For nodes that are down, the coordinator node keep notes (or hints) of all the write requests they have missed. Once the failing nodes recover, the write requests are forwarded to them based on the hints stored.
##  Read Repair
- In Distributed Systems, where data is replicated across multiple nodes, some nodes can end up having stale data
- The read repair operation pushes the latest version of data to nodes with the older version.
- Cassandra and Dynamo use ‘Read Repair’ to push the latest version of the data to nodes with the older versions.
## Anti-entropy
- Anti-entropy is a process of comparing the data of all replicas and updating each replica to the newest version
##  Merkle Trees
 - Replicas can contain a lot of data, so comparing them involves transfer of huge amounts of data
 - We use Merkle trees to compare replicas and figure out if the range of data residing on them are different.
 - Examples:
   - For anti-entropy and to resolve conflicts in the background, Dynamo uses Merkle trees.
   - In order to compare object values between replicas without using more resources than necessary, Riak relies on Merkle tree hash exchanges between nodes.
   - The active anti-entropy (AAE) subsystem was added to Riak in versions 1.3 and later to enable conflict resolution to run as a continuous background process, in contrast with read repair, which does not run continuously. AAE is most useful in clusters containing so- called “cold data” that may not be read for long periods of time, even months or years, and is thus not reachable by read repair.
     - https://docs.riak.com/riak/kv/latest/learn/concepts/active-anti-entropy/index.html
   - Cassandra accomplishes anti-entropy repair using Merkle trees, similar to Dynamo and Riak.
     - https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/operations/opsRepairNodesManualRepair.html
   