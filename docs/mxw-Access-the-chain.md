# How to Access Maxonrow Blockchain
There are TWO ways to read and write data from Maxonrow Blockchain:

### Node RPC
There are public data seed nodes that joins  network. They usually provide RPC calls. Please check the Node RPC Reference.

You can also run a full node by yourself, so that you will have a local server to send RPC requests and read Chain information off.

### Command Line Interface CLI
Essentially command line interfaces are just tools that wrap the incoming command line arguments and call RPCs. Please check the Command Line Referenace.

### Write APIs
You can only write to Maxonrow Blockchain via Transactions. The Node RPC provide a broadcastTx API to submit a signed and encoded transaction onto the Maxonrow Blockchain. The detailed process is outlined below:

### Message Composition
The transaction message and related information will be packed into payload, which is the so called Standard Transaction. The transaction body, memo, signature, etc. all fill in the Standard Transaction, encode and then broadcast out together onto Maxonrow Blockchain.

### Transaction Encoding
Encoding defines the way how transactions are serialized and transferred between clients and nodes, and different nodes themselves. here is a detailed specification on the transaction types and encoding logic.

### Signature
Signature is the evidence to prove the sender owns the transaction. The signature will be encoded together with transaction message and sent as payload to Maxonrow Blockchain node via RPC, as described in the above section.

### Account and Sequence Number
After Account is created, besides the balances, Account also contains:
- Account Number: an internal identifier for the account
- Sequence Number: an ever-changing integer.

The Sequence Number is the way how Maxonrow Blockchain prevents Replay Attack (the idea is borrowed from Cosmos network, but varies a bit in handling). Every transaction should have a new Sequence Number increased by 1 from the current latest sequence number of the Account, and after this transaction is recorded on the block chain, the Sequence Number will be set to the same number as the one of latest transaction.

This logic forces the client to be aware of the current Sequence Number, either by reading from the blockchain via API, or keep the counting locally by themselves. The recommended way is to keep counting locally and re-synchronize from the blockchain periodically.

### Examples
SDK in different languages are provided to simplify use of APIs to access Maxonrow Blockchain.
