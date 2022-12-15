# Sql Queries

[All Answers](../All_Answers.md)

## August 2021

**a) Briefly discuss (a) what consistency means in the context of ACID transactions, and (b) how relational systems support such consistency.**

- In the context of ACID transactions, consistency refers broadly to ``correctness'' of data. If the database has correct data before a transaction, and the transaction is correctly implemented, then the database will have correct data after the transaction. Relational 16 systems provide PRIMARY KEY, UNIQUE and FOREIGN KEY definitions, as well as some CHECK constraints, but the support is limited to what the database developers can specify using these constraints.

**b) Select the true statements:**

(a) Locking can be used to implement isolation. (50%)

(b) A buffer manager with a NO STEAL policy complicates the implementation of atomicity. (0%)

(c) Isolation and durability are mutually exclusive. (0%)

(d) Transactions are a general concept that could potentially be applied to, for example,  le systems. (50%)

## June 2021

**7a) Argue why the FORCE bu er manager policy is not appropriate for an HDD-based DBMS., Write your de nition here:**

- The crux of your answer should be: Writing all the updated pages would be very expensive (and cannot be done atomically). But with WAL we don't have to, as sequential writes of log buffer are orders of magnitude cheaper than random writes of data. Some side points were given for good related points.

**7b) In the WAL protocol, when are log buffers ushed to disk? Select all statements that apply:, Select the true statements:**

(a) Before a transaction commits. (50%)

(b) Before dirty data is stolen (i.e., written to disk at any time before the transaction that modified it commits). (50%)

(c) After dirty data is stolen. (0%)

(d) After a transaction commits. (0%)
