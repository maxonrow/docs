# Maxonrow Blockchain FAQ
* What is Maxonrow Blockchain?
* What can you do on Maxonrow Blockchain?
* What is the native coin on Maxonrow Blockchain?
* Where can I see my assets and transactions?
* What is the Fee Structure?
* Can I see orders/balances of others or can other parties see my orders/balances?
* How can I issue an asset?
* Can I run a full node for Maxonrow Blockchain?
* How would a third-party integrate with Maxonrow Blockchain?


### What is Maxonrow Blockchain?
Maxonrow Blockchain is the blockchain initially developed by Maxonrow and community.

### What can you do on Maxonrow Blockchain?
You can:

* Send and receive MXW
* Issue new tokens to digitalize assets, and use Maxonrow Blockchain as underlying
    third-party transfer network for the assets
* Send, receive, burn/mint and freeze/unfreeze tokens
* Explore the transaction history and blocks on the chain, via node RPC interfaces.
* Run a full node to listen to and broadcast live updates on transactions, blocks, and consensus activities
* Extract other data of Maxonrow Blockchain via full node or APIs
* Apply to run a validator node
* Develop tools and application to help users use Maxonrow Blockchain 

### What is the native coin on Maxonrow Blockchain?
The Maxonrow Coin, MXW, is the native asset on Maxonrow Blockchain. There are 9.33 Billion MXW coins in total. There will be no mining. The existing coin burns and freezes will still be in effect on the new Maxonrow Blockchain.

### Where can I see my assets and transactions?
You can always use wallets that support Maxonrow Blockchain to check your asset balances, transactions history. Maxonrow Blockchain Explorer is another tool to check balances and transactions.

### What is the Fee Structure?
Fees are charged and shared among the block producers (i.e. Validators) to run the network, in order to pay for the network usage and prevent abuse and attack. Since all user transactions, include transfer, new order etc, they are all recorded in blocks and blockchain state, the fee will be shared among different transactions. 

### Can I see orders/balances of others or can other parties see my orders/balances?
Yes, anyone can see anyone's orders and balances if they know the corresponding addresses. Maxonrow Blockchain is 100% transparent for transactions and balances.

### How can I issue an asset?
Anyone can pay a fee and issue an asset as Token on Maxonrow Blockchain, as long as they provide proper information for the fields below, and then execute the command through the command line or http interfaces.

* Name: a description string of less than 100 characters
* Symbol: an identifier string less than 100 characters
* Total Supply: depends on the settings
* Mint-able: whether the token can increase Total Supply in later time or not

### Can I run a full node for Maxonrow Blockchain?
Yes, you can. A full node contains all the information and application logic for Maxonrow Blockchain. It can receive and broadcast blocks and transactions with other full nodes and even validators. The only exception is it will not participate in the consensus if the full node is not a Validator.


### How would a third-party integrate with Maxonrow Blockchain?
A wallet provider may choose to only support the feature set of Maxonrow Blockchain, which would just cover wallets, addresses, balances and transfers.



