# The big ideas behind reliable, scalable, and maintainable systems

# Chapter 5  Replication
## Leaders and followers
- The leader handles all writes and reads
- Followers replicate the leader's log and handle only reads
- Followers can be promoted to leader if the leader fails
### Synchronous Versus Asynchronous Replication
- Synchronous: The leader waits until followers have confirmed that they received the write before reporting success to the user
- Asynchronous: The leader sends the write to the followers without waiting, and does not wait for acknowledgement from followers
- Synchronous is safer but slower
- Asynchronous is faster but risk losing data
![Alt text](image-2.png)
![Alt text](image-3.png)
### Setting up new followers
- Snapshot: The leader reads a snapshot of its database and sends the snapshot to the follower
- Log: The leader sends the follower the entire log from the beginning of time
- The follower can start processing queries as soon as it has received the snapshot or log
- The follower can request missing log entries from the leader
- The follower can request missing log entries from other followers
### Handling node outages
- If a follower fails, it can be discarded and replaced with a fresh copy
- If the leader fails, one of the followers must be promoted to be the new leader
- The leader must not accept writes from clients while it is recovering from a failure
- The leader must not accept writes from clients until the followers have caught up with the writes that were lost while the leader was down
### implementaion of replication logs
- Statement-based replication: The leader logs each write request (statement) that it executes, and sends that statement log to its followers
- Write-ahead log (WAL) shipping: The leader sends its entire WAL to its followers  
- Logical (row-based) log replication: The leader sends the data in its log in a form that is closer to the internal storage format of the database
- Trigger-based replication: The leader sends the data in its log in a form that is closer to the internal storage format of the database
## Problems with replication lag
### Reading your own writes
- If a client reads from the leader, it will always see the latest data
- If a client reads from a follower, it may not see the latest data
![Alt text](image-1.png)
### Monotonic reads
#### How to define the  monotonic reads
    If a client has seen the data at some point, it should not later see the data in an older state
    If a client has read the value of a key and later reads the value of that same key again, it should see the same value or a newer value.
##### How to achieve monitonic reads
    Read from the leader or same user reads from the same replica. For example, the replica can be chosen base on a hash of the userId, rather than randomly

### Consistent prefix reads
#### what is the consistent prefix reads
    If a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order,
    but they may see only some of the writes.
#### How to achieve consistent prefix reads
    Read from the leader or same user reads from the same replica. For example, the replica can be chosen base on a hash of the userId, rather than randomly

### Solutions for replication lag
#### what is replication lag? why it is matter?
    Replication lag is the delay between the time a write is made on the leader and the time the change has been replicated to the followers.
    Replication lag is a problem because it means that reads from followers may be stale.
#### How to solve replication lag?
    1. Read-your-own-writes consistency
    2. Monotonic reads
    3. Consistent prefix reads
    4. Read-after-write consistency
    5. Bounded staleness
    6. Causal consistency
    7. Session guarantees
    8. Monotonic writes
    9. Consistent cross-channel reads
    10. Global snapshots
    11. Serializable snapshot isolation
    12. Repeatable read snapshot isolation
    13. External consistency
    14. Distributed transactions
    15. Multi-leader replication
    16. Leaderless replication
    17. Write conflicts
    18. Custom conflict resolution logic
    19. Multi-datacenter operation
    20. Handling node outages
    21. Handling network interruptions
    22. Detecting concurrent writes
    23. Resolving conflicts at read time
    24. Resolving conflicts at write time
    25. Custom conflict resolution logic
    26. Multi-leader replication
    27. Leaderless replication
    28. Write conflicts
    29. Custom conflict resolution logic
    30. Multi-datacenter operation
    31. Handling node outages
    32. Handling network interruptions
    33. Detecting concurrent writes
    34. Resolving conflicts at read time
    35. Resolving conflicts at write time
    36. Custom conflict resolution logic
    37. Multi-leader replication
    38. Leaderless replication
## Multi-leader replication
### what is the multi-leader replication
    Multi-leader replication means that writes can be made to any node and are then asynchronously replicated to other nodes.
### Use cases for multi-leader replication
    1. Multi-datacenter operation
        - If you have datacenters in different geographical regions, you want to serve users from their nearest datacenter
        - performance
        - tolerance of datacenter outages
        - tolerance of network problems

    2. Clients with offline operation
        appliation needs to be continue to work while it disconnected from the internet. Calender App.
        CouchDB
    3. Collaborative editing
        Real-time collaberative editing applications allow serveral people to edit a document simulataneously. For example, Google Docs, When one user edits the document, the changes are instantly applied to their local replica and asynchronously replicaed to the server and other users who are editting the same documents.
    4. Multi-user chat
        Multi-user chat is similar to collaborative editing, except that the data being edited is a chat room rather than a document.

### Handling write conflicts
    The biggest problem with multi-leader replication is that write conflicts can occur, which means that conflict resolution is required.
![Alt text](image-4.png)
- Last write wins (LWW)
- Timestamp ordering
- Application-level conflict resolution
- Conflict avoidance
- Converging toward a consistent state
- Custom conflict resolution logic
    - on write
         - As soon as the database sysem detects a conflict in the log of replicated changes, it calls the conflict handler.
    - on read
        - when a conflict is detected, all the conflicting writes are stored, and the application is notified of the conflict when it reads one of the conflicting writes.
    - Automatic conflict resolution
        - The database system automatically resolves conflicts, without calling the application code.
        - Conflict-free replicated datatypes
        - mergeable persistent data structures
        - operational transformation
### Multi-leader replication topologies
   A replication topology is the arrangement of replication links between nodes.
   ![Alt text](image-5.png)
    The most general toplogy is all-to-all, in which every leaders sends its writes to every other leaders.
    .   Mysql  by default supports only a circular toplogy, in which each leader only has one upstream leader and one downstream leader.
    
## Leaderless replication
### what is leaderless replication?
    Leaderless replication means that any node can accept a write and that writes are propagated asynchronously to other nodes.
    Single-leader replication is a special case of leaderless replication, where there is only one node that is allowed to accept writes at any given time.
    multi-leader replication is a special case of leaderless replication, where all nodes are allowed to accept writes at any given time.
    Some data storage systems take a different approach, abondoning the concept of leader and allowing any replica to directly accept writes from clients, this idea is forgotten during  the era of relational databases. And it once again became an popular architecture of database after Amazon Dynamo.Riak, Cassandra, and Voldemort are open source datastores with leaderless replication model inspired by Dynamo.
    - In some headerless implementations, the clients directly sends its writes to several replicas, while in others, a a

