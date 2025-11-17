CAP: The CAP theorem states that a distributed data store can only have two of the following three properties at any given time:
Consistency (all nodes see the same data),
Availability (every request receives a response), and 
Partition Tolerance (the system continues to operate despite network failures). 

Since network failures (partitions) are a reality in distributed systems, designers must choose between prioritizing Consistency or Availability during a network partition


ACID: ACID (Atomicity, Consistency, Isolation, Durability) applies to traditional databases to ensure transactions are reliable and correct.

---

ACID properties
- Applies to: Traditional, single-node relational databases.
- Purpose: To ensure the reliability and correctness of transactions.

<> Properties:
- Atomicity: All parts of a transaction are completed, or none are.
- Consistency: Transactions bring the database from one valid state to another.
- Isolation: Simultaneous transactions do not interfere with each other.
- Durability: Committed transactions are permanent and survive system failures. 

CAP theorem
- Applies to: Distributed systems, like many modern NoSQL databases.
- Purpose: To understand the trade-offs in distributed systems, particularly during network partitions.

<> Properties: A distributed system can only guarantee two of the following three properties at once:
- Consistency: Every read receives the most recent write or an error.
- Availability: Every non-failing node in the system can always process a request.
- Partition Tolerance: The system continues to operate despite a network partition (communication breakdown between nodes). 
