# The Maxonrow Blockchain
The purpose of the new blockchain is to enables individuals and companies to make instantaneous transactions without an intermediary. Transaction information in the form of digital ‘blocks’ are stored and verified by distributed database on multiple computers (nodes) that are trusted and verified by the system—decentralized. These transactions information are immutable, therefore completely secure and valid.

### Node Roles
#### What is a Validator Node?
Validators are a group of infrastructure that take the responsibility to maintain the blockchain data and validate all the transactions. They join the consensus procedure and vote to produce blocks. The fees if any, are collected and distributed among all validators.

### Blocking
Maxonrow Blockchain uses a similar block structure as Tendermint proposes, with a size limit of 2.2 megabyte. It is expected a block will be produced on a-few-of-seconds level among validators, and can include from 0 up to several thousands of transactions.

### Blockchain State
Blockchain state stores the below information:
- Account and balances
- Fees
- Token information

### Cryptographic Design
#### Account and Address
For normal users, all the keys and addresses can be generated via the Web Wallet.

#### Signature
Maxonrow Blockchain uses a special signature against a SHA256 hash of the byte array of a JSON-encoded canonical representation of the transaction.

