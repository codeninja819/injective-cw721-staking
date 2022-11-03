# Cw721 NFT staking
NFT staking refers to the locking up of NFTs on a platform or protocol to receive staking rewards and other privileges. Most of NFT staking contracts are based ERC. This `NFT staking contract` is constructed of [CW721](https://github.com/CosmWasm/cw-nfts/blob/main/packages/cw721/README.md) to make this work in other environments based on [Cosmos-sdk](https://github.com/cosmos/cosmos-sdk). It is inspired by [solidity contract of Animocabrands](https://github.com/animocabrands/ethereum-contracts-nft_staking).

## Prerequisites
1. Increase allowance `NFT staking contract` about contract owner(=payer) of cw20 that is rewards token. (rewards pool)
2. Approve `NFT staking contract` about staker's token ID of CW721 NFT contract. (for transferNft)

Considering of prerequisites change. This `NFT staking contract` version is after the `increaseAllowance` for the payer of the rewards token contract is preceded, the amount of rewards is transferred from the payer to the `NFT staking contract` through the executing in the this contract. Also, the NFT of staker in the CW721 contract executes `Approve` and `NFT staking contract` can execute `transferNft` when staker stakes NFT in order to replace NFT owner from staker to `NFT staking contract`. 
In other way, payer transfers cw20 token amount to `NFT staking contract` directly to add rewards pool, and staker executes `transferNft` directly to change NFT owner.

## Usage
### Store & Instantiate
To make a WASM file, run `build.sh`
```shell
./build.sh
```
After store wasm file, send instantiate message.
The implementer contract's instantiate message needs to provide the following arguments.
- `cycle_length_in_seconds`: Length of a cycle, in seconds.
- `period_length_in_cycles`: Length of a period, in cycles.
- `white_listed_nft_contract`: The CW721-metadata-onchain contract to whitelist for performing NFT staking operations.
- `rewards_token_contract`: CW20-based token used as staking rewards.

e.g.
```json
{
    "owner":"xpla1j55tymfdys9n7k0dq6xmyd4hgfelp9jghzympt",
    "cycle_length_in_seconds":60,
    "period_length_in_cycles":3,
    "white_listed_nft_contract":"xpla15d33tr7llwwfw9w7uy4wyrfhfc37ma9fflkjt6dhsdlrpmwegjeqa20agz",
    "rewards_token_contract":"xpla1vrvp5w6lm3zj9cwevx4ljnt6e9khhfvqawc8f4qa872lcelkdhcqznz49p"
}
```

## Concepts
### Staking
Staking is the mechanism by-which a CW721-NFT is transferred to the `NftStaking` contract, to be held for a period of time, in exchange for a claimable CW20-based token payout (rewards). While staked, the `NFT staking contract` maintains ownership of the NFT and unlocks claimable rewards over time. When the owner decides to withdraw, or unstake, the NFT from the `NFT staking contract`, it will be transferred back to staker, but will stop generating rewards.

### Cycles, Period and Rewards Schedule
Discrete units of time in staking are expressed in terms of `periods` and `cycles` A cycle is defined as a duration in time, measured in seconds. Periods are a larger duratino expressed in the number of cycles. When the contract starts, the first cycle of the first period begins. The length of cycles and periods are set at contract's deployment through `cycle_length_in_seconds` and `period_length_in_cycles` instantiate arguments.

Through executing functions are `add_rewards_for_period` and `add_rewards_pool`, the contract owner is able to set rewards schedule and amounts of pool. `add_rewards_for_period` function saves rewards per cycle even after contract starts. If rewards per cycle are replaced to new value, computing rewards are changed immediatly when staker claims rewards. `add_rewards_pool` function executes that the cw20 token amount of contract owner is transferred to `NFT staking contract`.

### Claiming
Rewards can be claimed at any moment by the stakers if they already accumulated some again. Claims are computed by periods, summing up the gains over the schedule, starting from the last unclaimed up until the previous period relative to now. This means that at least one period must elapse before the accumulated rewards for staking an NFT, in any given period, can be claimed. Or in other words, a staker can claim rewards once per payout period. If a staker stakes multi NFT, each NFT is managed by `staker_tokenid_key` which is key, mapping staker address and NFT token ID. So, staker is able to claim rewards about each NFT. 

### Snapshots
Snapshots are historical records of changes staked/unstated over time. For every cycle in which an NFT is staked or unstaked, a new snapshot is created. This provides a means for calculating a staker's entitled proportion of rewards for every cycle of a period that they are claiming. A snapshot history for each `staker_tokenid_key` to track stake changes.

Snapshots have the following properties:
- Spans at least one cycle.
- Can span multiple cycles over multiple periods.
- The span of one snapshot will never overlap with another (for any given staker).
- Are arranged consecutively in sequence without skipping over cycles (i.e. there will never be a cycle in between two snapshots).
- Are removed from a staker's snapshot history as soon as a rewards claim is made for the periods that cover the span of the snapshot.

### Abuse prevention
Upon the initial staking of an NFT to the contract, the NFT will be "frozen" for a duration of up to 2 cycles before being allowed to be unstaked. As well, an NFT cannot be staked again during the same cycle after unstaking.



