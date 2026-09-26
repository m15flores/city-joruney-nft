# City Journey NFT

Extended version of the ERC-721 exercise from the Blockchain Accelerator program (Module 2). A personal, soulbound NFT collection representing the cities I have lived in, deployed live on Arbitrum One.

Original exercise: https://github.com/m15flores/blockchain-accelerator-journal/tree/main/module-2/nft-collection

## What changed vs. the original exercise

The original `BANFTCollection.sol` was a bare ERC-721 with sequential minting and no on-chain metadata beyond the token URI. This version adds:

- **On-chain `CityData` struct per token**: city name, latitude/longitude (scaled `int256`), and a from/to date range (Unix timestamps), verifiable directly on-chain without depending on IPFS staying available
- **`Ownable` with mint restricted to the owner**. This collection tells one person's story; minting isn't open to the public
- **A richer `MintNFT` event** including the city name, not just the token ID
- **Soulbound (non-transferable) tokens**. Each piece represents a lived chapter, not a tradeable asset. Enforced via an override of `_update`, the single internal hook OpenZeppelin routes every transfer through

## Contract

- Address: `0xA066b02716BEFaAb59B370224Af1c8C0bBEA1eDd` (Arbitrum One)
- [Arbiscan](https://arbiscan.io/address/0xa066b02716befaab59b370224af1c8c0bbea1edd)
- [View on OpenSea](https://opensea.io/fr/collection/city-journey-nft)

## Testing

9 tests covering minting (with on-chain city data), soulbound transfer restrictions, and token URI resolution.

​```bash
forge test -vv
​```

100% coverage (lines, statements, branches, functions) on the contract itself:

​```bash
forge coverage
​```

| File                    | % Lines         | % Statements    | % Branches    | % Funcs       |
|-------------------------|------------------|------------------|----------------|----------------|
| src/CityJourneyNFT.sol  | 100.00% (20/20)  | 100.00% (19/19)  | 100.00% (4/4)  | 100.00% (5/5)  |


(The deployment script is excluded from this table. It's a one-off tool, not tested via Foundry's suite; its correctness was verified by the successful live deployment.)

### A note on hardcoded token IDs in tests

Some tests assert against literal `cityData(0)` / `cityData(1)` instead of computing the expected token ID dynamically. This is intentional: these tests exist specifically to confirm sequential minting starts at 0 and increments correctly. A dynamic calculation would silently adapt to a future change in that behavior instead of catching it, which is the opposite of what the test is for.

## Metadata & minting

- Images and metadata JSON hosted on IPFS via Pinata
- A small Go script (`tools/mint-cities/`) reads city data from a JSON file and mints each token sequentially via `go-ethereum`, using auto-generated bindings from `abigen`

## Notes

- `totalSupply` is fixed at deploy time, with no owner function to increase it, and tokens are soulbound (non-transferable). Both are deliberate, not just technical defaults: this collection is a fixed set of ten chapters, from where the story started to where it stands right now. Some chapters already lived, the last one just beginning. A future chapter means a new collection, not an expansion of this one.

- City data (name, coordinates, date range) lives on-chain in a `CityData` struct per token, verifiable and independent of any third-party service staying available. The descriptive phrase for each city stays in IPFS metadata instead: it's narrative content with no need for the stronger guarantees on-chain storage provides, and keeping it off-chain avoids the gas cost of storing long strings. A Go-based event indexer (planned) will read city data directly from the contract rather than from IPFS.

- The minting script manages transaction nonces manually, fetching the starting nonce once and incrementing it locally between calls, rather than relying on automatic nonce resolution per transaction. Sequential mints submitted in quick succession triggered "nonce too low" errors when the nonce was recalculated on every call, since Arbitrum confirms fast enough for consecutive lookups to race each other.
