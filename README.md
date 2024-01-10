# MongoDB Transacton Shard Affinity Demo

This project provides a basic example of achieving shard affinity for transactions to optimise the performance of an application that applies multi-record transactions in a sharded MongoDB database cluster. The example is based on a contrived bank account balances and bank transactions maintenance use case where every evening, each bank account holder has their bank transactions from the last day persisted and their corresponding account balance updated atomically. As a result of wrapping this in a transaction, the failure scenario can't occur where the payments have been ingested into the database, but the balance was not updated.

This project provides a demo MongoDB Shell script, which can be run in one of two modes: 'non-affinity' or 'affinity':

  1. In 'non-affinity' mode, new bank transactions are placed into an `items` collection, and the corresponding bank account balance is updated with the new end-of-day value in the `balances` collection. The script intentionally forces all bank transaction items to persist in one shard only and all the bank account balances to persist on the other shard. This helps to show the impact when all subsequent transactions invoked by the script result in 'two-phase-commit' transactions. 
  1. In 'affinity' mode, new bank transactions and the corresponding bank account balance are placed into a single combined collection (the `items` collection). The script ensures the single collection's documents are evenly balanced across both shards. This helps to show the impact when all subsequent transactions invoked by the script result in  'single-shard' transactions.

Due to the potential need to hold both bank transaction items and bank account balances in the same collection (in the 'affinity' mode), each document includes a `type` field with a value of `ITEM` or `BALANCE`. This `type` field isn't really needed for the 'non-affinity' mode but is nevertheless always included in the documents to keep things simple and consistent. 


## Prerequisites

* You have a local or remote MongoDB sharded cluster containing 2 Shards available to use, accessible from your workstation. Note:
    - You can't use a MongoDB Atlas hosted Database Cluster because Atlas [doesn't allow the use of](https://www.mongodb.com/docs/atlas/unsupported-commands/) some of the sharding commands required by this demo.
    - For convenience, you could use the [sharded-mongodb-docker](sharded-mongodb-docker) project to easily run a MongoDB sharded cluster in a set of Docker containers on your workstation.
* The [MongoDB Shell](https://docs.mongodb.com/mongodb-shell/install/) is installed on your workstation .


## Steps To Run

1. PERFORM NON-AFFINITY TEST: From the terminal, execute the following MongoDB Shell script to configure two sharded collections, `items` (i.e.,  bank account transactions) and balances (i.e.,  bank account balances) and loop ingesting data using a transaction for each batch of items and balances updates.

    ```console
    mongosh "mongodb://localhost:27017/" mdb-tx-shard-affinity-demo.mongodb
    ```

   Note: If you do not have a `mongos` process listening on `localhost:27017`, correct the MongoDB URL in the above command before running the command.

1. PERFORM AFFINITY TEST: From the terminal, execute the following command and MongoDB Shell script to configure one sharded collection, `items` to hosl before bank account transactions and balances, with transaction shard affinity being simulated.

    ```console
    export AFFINITY=true
    mongosh "mongodb://localhost:27017/" mdb-tx-shard-affinity-demo.mongodb
    ```    

   Note: If you do not have a `mongos` process listening on `localhost:27017`, correct the MongoDB URL in the above command before running the command.


## Optional Help

1. To see how chunks are partioned in the sharded collections, once you've run the MongoDB Shell `mdb-tx-shard-affinity-demo.mongodb` script, from the terminal, execute the following command and MongoDB Shell script to output the relevant collecitons' statistics. 

    ```console
    mongosh $MDB_CONNECTION_STRING show-chunks-stats.mongodb
    ```
