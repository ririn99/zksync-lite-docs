# NFTs

NFTs are currently in testnet on the ropsten-beta and rinkeby-beta zkSync networks. This API reference provides descriptions for all functions regarding NFTs in zkSync 1.x. Currently it includes minting and transferring NFTs. Documentation for functions regarding withdrawals is coming very soon! Please start by reading our [high level overview](https://zksync.io/dev/nfts/).

- [Connecting to Rinkeby-beta testnet](https://zksync.io/api/sdk/js/nfts.html#connect-to-the-rinkeby-beta-testnet)
- [Mint NFT](https://zksync.io/api/sdk/js/nfts.html#mint-nft)
- [Transfer NFT](https://zksync.io/api/sdk/js/nfts.html#transfer-nft)
- [Utility Functions](https://zksync.io/api/sdk/js/nfts.html#utility-functions)
    - [Calculate Transaction Fee](https://zksync.io/api/sdk/js/nfts.html#calculate-transaction-fee)
    - [View NFT](https://zksync.io/api/sdk/js/nfts.html#view-nft)
    - [Get NFT](https://zksync.io/api/sdk/js/nfts.html#get-nft)
    - [Get a receipt](https://zksync.io/api/sdk/js/nfts.html#get-a-receipt)

## Connect to the Rinkeby-beta testnet

```typescript
const syncProvider = await zksync.getDefaultProvider("rinkeby-beta");
```

## Mint NFT

You can mint an NFT by calling the `mintNFT` function from the `Wallet` class, now available in the zksync@beta version.

> Signature

```typescript
async mintNFT(mintNft: {
    recipient: string;
    contentHash: string;
    feeToken: TokenLike;
    fee?: BigNumberish;
    nonce?: Nonce;
}): Promise<Transaction>
```

| Name        | Description                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------------- |
| recipient   | the recipient address represented as a hex string                                                   |
| contentHash | the unique identifier of the NFT represented as a 32-byte hex string (e.g. IPFS content identifier) |
| feeToken    | name of token in which fee is to be paid (typically ETH)                                            |
| fee         | transaction fee                                                                                     |

Example: 

```typescript
const contentHash = "0xbd7289936758c562235a3a42ba2c4a56cbb23a263bb8f8d27aead80d74d9d996"
const nft = await syncWallet.mintNFT({
    recipient: syncWallet.address(),
    contentHash,
    feeToken: "ETH",
    fee,
});
```

After an NFT is minted, it can be in two states: committed and verified. An NFT is committed if it has been included in a rollup block, and verified when a zero knowledge proof has been generated for that block and the root hash of the rollup block has been included in the smart contract on Ethereum mainnet.

## Transfer NFT

An NFT can only be transferred after the block with it's mint transaction is verified. This means the newly minted NFT may have to wait a few hours before it can be transferred. This only applies to the first transfer; all following transfers can be completed with no restrictions.

You can transfer an NFT by calling the `syncTransferNFT` function:

```typescript
async syncTransferNFT(transfer: {
    to: Address;
    token: NFT;
    feeToken: TokenLike;
    fee?: BigNumberish;
    nonce?: Nonce;
    validFrom?: number;
    validUntil?: number;
}): Promise<Transaction[]>
```

| Name     | Description                                              |
| -------- | -------------------------------------------------------- |
| to       | the recipient address represented as a hex string        |
| feeToken | name of token in which fee is to be paid (typically ETH) |
| token    | address of the NFT                                       |
| fee      | transaction fee                                          |

The `syncTransferNFT` function works as a batched transaction under the hood, so it will return an array of transactions where the first handle is the NFT transfer and the second is the fee.  

```typescript
const handles = await sender.syncTransferNFT({
  to: receiver.address(),
  feeToken,
  token: nft,
  fee,
});
```

## Utility Functions

### Calculate Transaction Fee

To calculate the transaction fee for minting an NFT, the `getTransactionFee` method from the `Provider` class now supports transaction type `'MintNFT'`.

> Signature

```typescript
async getTransactionFee(
    txType: 'Withdraw' | 'Transfer' | 'FastWithdraw' | 'MintNFT' | ChangePubKeyFee | LegacyChangePubKeyFee,
    address: Address,
    tokenLike: TokenLike
): Promise<Fee>
```

Example: 

```typescript
const { totalFee: fee } = await syncProvider.getTransactionFee("MintNFT", syncWallet.address(), feeToken);
```

### View an NFT

To view an account's NFTs:

```typescript
// Get state of account
const state = await syncProvider.getAccountState('<account-address>');
// View committed NFTs
console.log(state.committed.nfts);
// View verified NFTs
console.log(state.verified.nfts);
```

### Get an NFT

You may also find the `getNFT` function from the `Wallet` class useful.

> Signature

```typescript
async getNFT(tokenId: number, type: 'committed' | 'verified' = 'committed'): Promise<NFT>
```

### Get a Receipt

To get a receipt for a minted NFT:

```typescript
// mint nft
const nft = await wallet.mintNFT({...});
// get receipt
const receipt = await nft.awaitReceipt();
```

To get a receipt for a transfer:

```typescript
// transfer nft
const handles = await sender.syncTransferNFT({...});
// get receipt
const receipt = await handles[0].awaitReceipt();
```