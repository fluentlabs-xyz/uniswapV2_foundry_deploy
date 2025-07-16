# uniswapv2_foundry_deployment

## Deploy

### Step 1: Prepare UniswapV2Factory

ℹ️ Note: The INIT_CODE_HASH public variable:

```solidity
bytes32 public constant INIT_CODE_PAIR_HASH = keccak256(abi.encodePacked(type(UniswapV2Pair).creationCode));
```

has been added to simplify UniswapV2Router02 deployment later. INIT_CODE_PAIR_HASH is added to UniswapV2Factory here:

https://github.com/MarcusWentz/uniswapV2_foundry_deployment/blob/main/src/UniswapV2Factory.sol#L405

⚠️  Caution: the constructor arguments set in a text file to avoid forge compilation issues. 

ℹ️  Note: this example sets feeToSetter to 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 (vitalik.eth).

### Step 2: Deploy UniswapV2Factory

```shell
forge create src/UniswapV2Factory.sol:UniswapV2Factory \
--constructor-args-path src/deployConstructor/UniswapV2Factory.txt \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

If contract verification fails after deployment, verify the contract at the deployed address:

```shell
forge verify-contract \
--rpc-url https://rpc.testnet.fluent.xyz \
--constructor-args-path src/deployConstructor/UniswapV2Factory.txt \
<contract_address> \
src/UniswapV2Factory.sol:UniswapV2Factory  \
--verifier blockscout \
--verifier-url https://blockscout.dev.gblend.xyz/api/
```

### Step 3: Update UniswapV2Router02 value INIT_CODE_HASH before deploying

Update library inside Router INIT_CODE_HASH on line 704 in this format:

https://github.com/MarcusWentz/uniswapV2_foundry_deployment/blob/main/src/UniswapV2Router02.sol#L704 


🔴 Warning: Do not use this exact INIT_CODE_HASH in UniswapV2Router02 unless it matches your new INIT_CODE_HASH deployed from UniswapV2Factory

```solidity
hex'cffad317eee0eafb23b811b0c051afaea213e52be9e2ab444e34b016651762ba' // init code hash
```

🔴 Warning: if the INIT_CODE_HASH set in UniswapV2Router02 does not match the INIT_CODE_HASH in UniswapV2Factory, certain transactions will revert:

https://ethereum.stackexchange.com/questions/88075/uniswap-addliquidity-function-transaction-revert/94852#94852

### Step 4: Deploy UniswapV2Router02:

```shell
forge create src/UniswapV2Router02.sol:UniswapV2Router02 \
--constructor-args-path src/deployConstructor/UniswapV2Router02.txt \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

Note: UniswapV2Router02.sol is a large contract, so the `foundry.toml` settings folder should have Solidity compiler optimization runs set to 200:
```toml
[profile.default]
optimizer = true
optimizer_runs = 200
```

If this isn't set or you want to enable different compiler optimization runs for certain contracts, you can add these flags:
```shell
forge create src/UniswapV2Router02.sol:UniswapV2Router02 \
--constructor-args-path src/deployConstructor/UniswapV2Router02.txt \
--private-key $devTestnetPrivateKey \
--rpc-url https://rpc.testnet.fluent.xyz \
--broadcast \
--verify \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/ \
--optimize \
--optimizer-runs 200 
```

If contract verification fails after deployment, verify the contract at the deployed address:
 
```shell
forge verify-contract \
--rpc-url https://rpc.testnet.fluent.xyz \
--constructor-args-path src/deployConstructor/UniswapV2Router02.txt \
<contract_address> \
src/UniswapV2Router02.sol:UniswapV2Router02 \
--verifier blockscout \
--verifier-url https://testnet.fluentscan.xyz/api/
```

### Step 5 (Optional): Verify UniswapV2Pair (ERC-20 LP token deployed from UniswapV2Factory): 

```shell
forge verify-contract \
--rpc-url https://rpc.dev.gblend.xyz/ \
<contract_address> \
src/UniswapV2Pair.sol:UniswapV2Pair \
--verifier blockscout \
--verifier-url https://blockscout.dev.gblend.xyz/api/
```

## Foundry Fork Test addLiquidityEth

🔴 Warning: make sure you approve enough tokens for allowance to avoid ERC-20 `transferFrom()` errors. 

🔴 Warning: make sure you supply enough liquidity amounts to avoid error `ds-math-sub-underflow':

https://ethereum.stackexchange.com/a/130087

`MINIMUM_LIQUIDITY` for minting Uniswap V2 LP tokens is generally 1000 wei:

https://docs.uniswap.org/contracts/v2/reference/smart-contracts/pair#minimum_liquidity

```shell
forge test \
--fork-url https://rpc.dev.gblend.xyz/ \
--match-test testAddLiquidityEth
```

## Debugging Resource:

https://github.com/shardeum/bug-reporting/issues/263#issuecomment-1495059281

## Sepolia Verified Deployments:

ℹ️ Note: this example from the deployed address for UniswapV2Factory INIT_CODE_HASH is :

```solidity
hex'f5cf876c1910617ea1749e7346a0c71cf8d678a08fc2fb10ae44fded41499415' // init code hash
```

UniswapV2Factory:

https://sepolia.etherscan.io/address/0xdbaa7dfbd9125b7a43457d979b1f8a1bd8687f37#code

UniswapV2Router02:

https://sepolia.etherscan.io/address/0xd49d79d476215bef1e5ac43c46ec9db6e7906dbd#code

## Base Sepolia Verified Deployments:

UniswapV2Factory:

https://sepolia.basescan.org/address/0xaec91b299dc7b7001e32112f2a72a3a51f3b9303

UniswapV2Router02:

https://sepolia.basescan.org/address/0x794c842Ef3EE218464068F0FFC4Bc084453CDeDD

## Fluent Testnet Verified Deployments:

UniswapV2Factory:

https://testnet.fluentscan.xyz/address/0xCcc41c8E5BeE781FAe36Af0a801C1eA523067c6F?tab=contract

UniswapV2Router02:

https://testnet.fluentscan.xyz/address/0x9E1D1631B5d08B9Ab5f75e560434c59235ec7AA3?tab=contract