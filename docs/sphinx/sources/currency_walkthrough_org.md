## [Example "Currency" Contract Walkthrough (original)](#https://raw.githubusercontent.com/ekkis/eos/master/README.md)

EOS comes with example contracts that can be uploaded and run for testing purposes. Next we demonstrate how to upload and interact with the sample contract "currency".

<a name="smartcontractexample"></a>
### Example smart contracts

First, run the node

```bash
cd ~/eos/build/programs/nodeos/
./nodeos
```

<a name="walletimport"></a>
### Setting up a wallet and importing account key

As you've previously added `plugin = eosio::wallet_api_plugin` into `config.ini`, EOS wallet will be running as a part of `nodeos` process. Every contract requires an associated account, so first, create a wallet.

```bash
cd ~/eos/build/programs/cleos/
./cleos wallet create # Outputs a password that you need to save to be able to lock/unlock the wallet
```

For the purpose of this walkthrough, import the private key of the `eosio` account, a system account included within genesis.json, so that you're able to issue API commands under authority of an existing account. The private key referenced below is found within your `config.ini` and is provided to you for testing purposes.

```bash
./cleos wallet import 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

<a name="createaccounts"></a>
### Creating accounts for sample "currency" contract

First, generate some public/private key pairs that will be later assigned as `owner_key` and `active_key`.

```bash
cd ~/eos/build/programs/cleos/
./cleos create key # owner_key
./cleos create key # active_key
```

This will output two pairs of public and private keys

```
Private key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Public key: EOSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

**Important:**
Save the values for future reference.

Run the `create` command where `eosio` is the account authorizing the creation of the `currency` account and `PUBLIC_KEY_1` and `PUBLIC_KEY_2` are the values generated by the `create key` command

```bash
./cleos create account eosio currency PUBLIC_KEY_1 PUBLIC_KEY_2
```

You should then get a JSON response back with a transaction ID confirming it was executed successfully.

Go ahead and check that the account was successfully created

```bash
./cleos get account currency
```

If all went well, you will receive output similar to the following:

```json
{
  "account_name": "currency",
  "eos_balance": "0.0000 EOS",
  "staked_balance": "0.0001 EOS",
  "unstaking_balance": "0.0000 EOS",
  "last_unstaking_time": "2035-10-29T06:32:22",
...
```

Now import the active private key generated previously in the wallet:

```bash
./cleos wallet import XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

<a name="uploadsmartcontract"></a>
### Upload sample "currency" contract to blockchain

Before uploading a contract, verify that there is no current contract:

```bash
./cleos get code currency
code hash: 0000000000000000000000000000000000000000000000000000000000000000
```

With an account for a contract created, upload a sample contract:

```bash
./cleos set contract currency ../../contracts/currency/currency.wast ../../contracts/currency/currency.abi
```

As a response you should get a JSON with a `transaction_id` field. Your contract was successfully uploaded!

You can also verify that the code has been set with the following command:

```bash
./cleos get code currency
```

It will return something like:
```bash
code hash: 9b9db1a7940503a88535517049e64467a6e8f4e9e03af15e9968ec89dd794975
```

Before using the currency contract, you must issue the currency.

```bash
./cleos push action currency issue '{"to":"currency","quantity":"1000.0000 CUR","memo":""}' --permission currency@active
```

Next verify the currency contract has the proper initial balance:

```bash
./cleos get table currency currency account
{
  "rows": [{
     "currency": 1381319428,
     "balance": 10000000
     }
  ],
  "more": false
}
```

<a name="pushamessage"></a>
### Transfering funds with the sample "currency" contract

Anyone can send any message to any contract at any time, but the contracts may reject messages which are not given necessary permission. Messages are not sent "from" anyone, they are sent "with permission of" one or more accounts and permission levels. The following commands show a "transfer" message being sent to the "currency" contract.

The content of the message is `'{"from":"currency","to":"eosio","quantity":"20.0000 CUR","memo":"any string"}'`. In this case we are asking the currency contract to transfer funds from itself to someone else. This requires the permission of the currency contract.

```bash
./cleos push action currency transfer '{"from":"currency","to":"eosio","quantity":"20.0000 CUR","memo":"my first transfer"}' --permission currency@active
```

Below is a generalization that shows the `currency` account is only referenced once, to specify which contract to deliver the `transfer` message to.

```bash
./cleos push action currency transfer '{"from":"${usera}","to":"${userb}","quantity":"20.0000 CUR","memo":""}' --permission ${usera}@active
```

As confirmation of a successfully submitted transaction, you will receive JSON output that includes a `transaction_id` field.

<a name="readingcontract"></a>
### Reading sample "currency" contract balance

So now check the state of both of the accounts involved in the previous transaction.

```bash
./cleos get table eosio currency account
{
  "rows": [{
      "currency": 1381319428,
      "balance": 200000
       }
    ],
  "more": false
}
./cleos get table currency currency account
{
  "rows": [{
      "currency": 1381319428,
      "balance": 9800000
    }
  ],
  "more": false
}
```

As expected, the receiving account **eosio** now has a balance of **20** tokens, and the sending account now has **20** less tokens than its initial supply.