## Set a private key for Demo
1. Generate a new address and print its private key with `-p` (for mainnet, append `-n mainnet`):
```
$ ./index address -p
private key: 99fdadee1065805c41f4134f2dbc0ed80fd2b82fac2d7d945c41c221533e1c8b
tb1qp877w90x3vuu6t79kx5u9mswpf2sz0t3hkqdlx
```
2. Set the environment variable with the above private key:
```
$ export PRIVATE_KEY=99fdadee1065805c41f4134f2dbc0ed80fd2b82fac2d7d945c41c221533e1c8b
```
3. Send some balance to the address printed above (0.001 should be enough).
For testnet, you can use [this faucet](https://testnet-faucet.mempool.co/).