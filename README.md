# Temple Mitigations

## Prize Pool 
Total Prize Pool: $15,000
  - High: $11,000
  - Medium: $3,500
  - Low: $500

Start Date: August 21, 2024 Noon UTC

End Date: August 28, 2024 Noon UTC

nSLOC - 1115

[//]: # (scope-open)
## Scope (contracts)

All Contracts in `contracts/templegold` are in scope.
```js
contracts/
└── templegold
    ├── AuctionBase.sol
    ├── DaiGoldAuction.sol
    ├── EpochLib.sol
    ├── SpiceAuction.sol
    ├── SpiceAuctionFactory.sol
    ├── TempleGold.sol
    ├── TempleGoldAdmin.sol
    ├── TempleGoldStaking.sol
    └── TempleTeleporter.sol
```

| #     | File              | nSLOC | Description |
|  :-:  | :---------------- | :------: | :------------------- |
|   1   | contracts/templegold/AuctionBase.sol       |   13   | Base auction contract. Inherited by `DaiGoldAuction.sol` and `SpiceAuction.sol` |
|   2   | contracts/templegold/DaiGoldAuction.sol          |   176   | An auction contract for deposits of bid token to bid on a share of distributed TGLD for every epoch. Once bid, users cannot withdraw their bid token and can claim their share of Temple Gold for epoch after auction finishes. |
|   3   | contracts/templegold/EpochLib.sol    |  13   | A library for `DaiGoldAuction` and `SpiceAuction` for epoch time validations |
|   4   | contracts/templegold/SpiceAuction.sol |  294   | A special auction configured with a "spice" token and TGLD. For every auction epoch spice token or TGLD can be the auction token or vice versa. Spice auctions are controlled by governance and an operator (specific actions). Configuration for an epoch is set before the epoch auction starts. After an auction, redeemed TGLD can be burned and circulating supply updated |
|   5   | contracts/templegold/SpiceAuctionFactory.sol |  51   | Factory to create and track spice auction contracts |
|   6   | contracts/templegold/TempleGold.sol |  193   | Temple Gold is a non-transferrable ERC20 token with LayerZero integration for cross-chain transfer for holders. On mint, Temple Gold is distributed to DaiGoldAuction, Staking contracts and team multisig using distribution share parameters percentages set at `DistributionParams`. Users can get Temple Gold by staking Temple for Temple Gold rewards on the staking contract or in auctions. |
|   7   | contracts/templegold/TempleGoldAdmin.sol |  50   | Temple Gold Admin is an admin to Temple Gold contract. From the setup of layerzero, `Ownable` is used for admin executions. Avoids a manual import to change `Ownable` to `ElevatedAccess` by using this contract for admin executions |
|   8   | contracts/templegold/TempleGoldStaking.sol |  287   | Stakers deposit Temple and claim rewards in Temple Gold. This is a fork of the synthetix staking contract with some modifications (voting function, unstake cooldown and flexible reward durations). Duration for distributing staking rewards is set with `setRewardDuration`. An unstake cooldown is used to encourage longer staking times. During unstake cooldown, stakers can't unstake their original stake. Governance contracts will read a staker's vote power with `getCurrentVotes` and `getPriorVotes`. |
|   9   | contracts/templegold/TempleTeleporter.sol |  38   |  Temple Teleporter transfers Temple token cross-chain with layer zero integration |
|     |  |  998   |  |

[//]: # (scope-open)

## Not in Scope
- Any findings from previous audits are out of scope
    - TempleGold - [Cyfrin audit](https://github.com/Cyfrin/cyfrin-audit-reports/blob/main/reports/2024-06-17-cyfrin-templedao-v2.1.pdf)
    - Code Hawks Competitive [Audit](https://codehawks.cyfrin.io/c/2024-07-templegold/results?lt=contest&page=1&sc=reward&sj=reward&t=leaderboard)

- ERC20 Tokens:
    - Non standard 18dp ERC20 Tokens (eg USDT, other fee taking ERC20s) and USDC are out of scope

- Any `slither` output is considered public and out of scope
    - slither output: [protocol/slither.db.json](https://github.com/TempleDAO/temple/blob/templegold/protocol/slither.db.json)

- Centralization risks are for policy/emergency/operational behaviour, and owned by the TempleDAO multisig. This is acceptable and out of scope as it's required for the protocol to work as intended and protect user funds

- External libraries (prbmath, openzeppelin, layerszerolabs) are out of scope.



## Major Changes

- Changes from Code Hawks competitive audit results
- `TempleGoldStaking` uses unstake cooldown. No more vesting rate for rewards, stakers get their full rewards. During unstake cooldown, stakers can't unstake their original stake and any new stakes resets the unstake cooldown to start.
- `TempleGold` introduces a way to update circulating supply. Max supply is changed to max circulating supply. Another change is allowing burns after redemption (successful spice auctions) from whitelisted contracts when `to` is set to `address(0)`. Allowing burns is followed by updating circulating supply.
- `SpiceAuction` introduces a way to update `TempleGold` on source chain (arbitrum one) with redeemed TGLD tokens.
- `TempleGold` vesting factor changed to use a literal numerical value and a week multiplier to calculate amount of tokens to mint in `_getMintAmount()`.
