# Getting Started

### Run a single node
```sh
# Build the binaries
make build

# Initialize configuration files and genesis file
make init

# Start the blockchain
make start
```


### Running Full node
`MAXONROW` blockchain can be run as a full node, syncing it's state with another node or validator.

```sh
# Initial the blockchain network with a valid chain-id, 
# which will generate the config, genesis and account in respective folder
mxwd init --home ~/.mxw

# Generate validator transcation, 
# which will create the gentx folder with gentx transcation of validator account-1
mxwd gentx --name acc-1 --home ~/.mxw

# Start the node
mxwd start --home ~/.mxw
```

#### Examples of Query Account

```sh
# Get all the account details
mxwcli keys list --home ~/.mxw

# Query the address of acc-1
mxwcli keys show acc-1 --address

# Query the account, which verify the balance of acc-1
mxwcli query account $(mxwcli keys show acc-1 --address) --chain-id maxonrow-chain
```


#### Examples of Send Transaction between TWO accounts

```sh
# Query the address of acc-1
mxwcli keys show acc-1 --address

# Query the address of acc-2
mxwcli keys show acc-2 --address

# Send Transaction
mxwcli tx send $(mxwcli keys show acc-1 --address) $(mxwcli keys show acc-2 --address) 1000cin --fees 10000000000000000cin --gas 0 --memo "TRANSFER" --chain-id maxonrow-chain
```

### Testing

`MAXONROW` come with the many test cases, you can find all our test case under test folder in project soruce.

```sh
# Run tests
make test
```

