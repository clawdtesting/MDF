# MDF (Monad DeFi Framework) Contracts Overview

This document provides an overview of the core smart contracts within the `MDF/contracts` directory, explaining their individual functionalities and how they interact to form a cohesive decentralized finance (DeFi) project centered around the **MMMToken**.

## 1. `MMMToken.sol`

This is the primary ERC20 token of the ecosystem, designed with unique tokenomics:

*   **Access Control**: Utilizes `Ownable` for administrative functions, allowing the contract owner to configure critical parameters such as the `taxVault` address, the Uniswap `pair` address, and the Uniswap `router` address.
*   **Dynamic Tax Model**: Implements a time-decaying tax mechanism for both buy and sell transactions. The tax percentages decrease over predefined time intervals after the token's launch, incentivizing long-term holding and discouraging rapid price manipulation.
*   **Tax Collection**: Overrides the standard ERC20 `_update` function to apply taxes during transfers, specifically targeting buy/sell transactions involving the Uniswap liquidity pair. Collected MMMTokens are automatically routed to the `TaxVault`.
*   **Holder Tracking**: Tracks `lastNonZeroAt` for each token holder, recording the timestamp when a user's balance was last non-zero. This data is crucial for the `RewardVault` to enforce minimum hold times for reward eligibility.

## 2. `TaxVault.sol`

The `TaxVault` serves as the central processing and distribution hub for the MMMToken taxes:

*   **Tax Aggregation**: Receives all MMMTokens collected as tax from the `MMMToken` contract during taxable transfers.
*   **Vault Wiring**: Configured with the addresses of the `RewardVault`, `MarketingVault`, and `TeamVestingVault`, defining the destinations for processed funds.
*   **Fund Allocation**: Specifies predefined split percentages (`bpsReward`, `bpsBurn`, `bpsMkt`, `bpsTeam`) dictating how the collected MMMTokens (after potential conversion) are distributed across different functions.
*   **Automated Processing**: Features a `processTaxes` function (which can be called by a designated `keeper` or manually). This function facilitates the conversion of a portion of the collected MMMTokens into USDC via a Uniswap V2 router (supporting direct MMM → USDC or MMM → WMON → USDC paths).
*   **Distribution Mechanism**: Distributes processed funds as follows:
    *   A portion of MMMToken is transferred to the `RewardVault`, triggering its `notifyRewardAmount` function to update reward calculations.
    *   A portion of MMMToken is sent to the `DEAD` address, effectively burning those tokens and reducing the total supply.
    *   The USDC generated from swaps is allocated between the `MarketingVault` and `TeamVestingVault` based on configured percentages.

## 3. `RewardVault.sol`

This contract is responsible for managing and distributing MMMToken rewards to eligible holders:

*   **Reward Accumulation**: Receives MMMToken from the `TaxVault` via the `notifyRewardAmount` function. This amount updates an `accRewardPerToken` accounting system, which tracks accumulated rewards per token.
*   **Claiming Mechanism**: Allows users to `claim` their pending MMMToken rewards, provided they meet specific eligibility criteria:
    *   Maintaining a `minBalance` of MMMTokens.
    *   Satisfying a `minHoldTimeSec` (an initial holding period after acquiring tokens).
    *   Adhering to a `claimCooldown` period between consecutive claims.
*   **NFT Integration**: Integrates directly with `BoostNFT.sol` to provide special benefits. Holders of `BoostNFT` can receive reductions to both their required hold time and claim cooldown periods, incentivizing NFT ownership within the ecosystem.
*   **Security**: Incorporates `nonReentrant` protection to prevent reentrancy attacks on critical functions.

## 4. `BoostNFT.sol`

An ERC721 NFT contract designed to enhance utility and incentivize participation:

*   **Rarity System**: Defines different NFT rarities (e.g., `COMMON`, `RARE`), each associated with distinct `holdReduction` and `cooldownReduction` values.
*   **Utility for Holders**: Owners of `BoostNFT`s receive tangible benefits in the `RewardVault` by having their hold times and claim cooldowns reduced, encouraging ecosystem engagement.
*   **Controlled Minting & Transfers**: NFTs are minted by the contract owner. Transferability can be locked (`transfersEnabled`) to prevent manipulative trading or gaming of the system.

## 5. `TwoOfThreeERC20Vault.sol`

A foundational multi-signature vault contract:

*   **Generic Multi-Sig**: A reusable contract designed to securely hold and manage any specified ERC20 token (e.g., USDC).
*   **Consensus Mechanism**: Requires a minimum of 2 out of 3 designated `owners` to confirm a proposed transaction before it can be executed, ensuring decentralized control over funds.
*   **Base for Specialized Vaults**: Serves as the base contract for more specialized vaults like `MarketingVault` and `TeamVestingVault`.

## 6. `MarketingVault.sol` & `TeamVestingVault.sol`

Specialized implementations inheriting from `TwoOfThreeERC20Vault.sol`:

*   These contracts are specific instances of the multi-signature vault, designed to receive and securely manage USDC distributed from the `TaxVault`.
*   Their multi-signature nature ensures transparent and accountable management of funds allocated for marketing initiatives and team vesting, requiring collective agreement for any outgoing transactions.

## Overall Project Synergy

The `MDF` contract suite establishes a robust and interconnected DeFi ecosystem:

1.  **Tokenomics**: The `MMMToken`'s dynamic tax model and reward distribution create incentives for holding and participation.
2.  **Fund Management**: The `TaxVault` efficiently collects and intelligently distributes protocol revenue (MMMTokens, USDC) to various ecosystem components.
3.  **Community Engagement**: `BoostNFT`s provide added utility and exclusive benefits, fostering a loyal community.
4.  **Security & Governance**: Multi-signature vaults (`MarketingVault`, `TeamVestingVault`) ensure secure and decentralized control over critical operational funds, requiring consensus for expenditures.

This architecture aims to build a sustainable and engaging DeFi platform on the Monad blockchain, combining innovative tokenomics with robust financial management and community incentives.
