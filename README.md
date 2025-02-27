# DataNation
Cardano - Turning Dormant Data into Business Assets

Please note - the initial project draft WhitePaper and pitch deck are saved as PDFs in the docs directory. 

## DataNation Smart Contract Design Outline

**Overview**

DataNation is a decentralised platform that empowers businesses to monetise their datasets, particularly for training Large Language Models (LLMs), by leveraging Cardano’s blockchain technology. The platform connects dataset owners with AI engineers and data analysts to enhance dataset quality and facilitates a marketplace for selling datasets or fractional ownership stakes. Smart contracts automate token minting, marketplace interactions, income distribution, and dataset buyouts, ensuring transparency, security, and compliance with privacy and regulatory standards.

**The smart contracts are designed to:**

- Mint a reference NFT and 1,000 fractional tokens per dataset using the CIP-68 standard.
- Enable trading of fractional tokens on a marketplace.
- Manage income from dataset usage and distribute it to fractional owners via a treasury.
- Handle outright dataset purchases with proceeds distributed to token holders.
- This following is an outline of the architecture, data structures, and validation logic for these contracts in Aiken.

The following is an outline of the design for the DataNation project's smart contracts. This provides a detailed overview of the smart contract architecture, components, and logic, tailored for implementation in Cardano's Aiken language.

---

# DataNation Smart Contract Design Document

## Overview

DataNation is a decentralised platform that empowers businesses to monetize their datasets, particularly for training Large Language Models (LLMs), by leveraging Cardano’s blockchain technology. The platform connects dataset owners with AI engineers and data analysts to enhance dataset quality and facilitates a marketplace for selling datasets or fractional ownership stakes. Smart contracts automate token minting, marketplace interactions, income distribution, and dataset buyouts, ensuring transparency, security, and compliance with privacy and regulatory standards.

The smart contracts are designed to:
- Mint a reference NFT and 1,000 fractional tokens per dataset using the CIP-68 standard.
- Enable trading of fractional tokens on a marketplace.
- Manage income from dataset usage and distribute it to fractional owners via a treasury.
- Handle outright dataset purchases with proceeds distributed to token holders.

This document outlines the architecture, data structures, and validation logic for these contracts in Aiken.

---

## Smart Contract Components

### 1. Minting Policy

#### Purpose
The minting policy governs the creation of a reference NFT and 1,000 associated fractional tokens for each dataset uploaded to DataNation. The NFT serves as a unique identifier for the dataset, while the fractional tokens represent ownership stakes in the dataset’s income stream.

#### Parameters
- **Platform Public Key Hash (PKH)**: Ensures only the DataNation platform can mint tokens.
- **Dataset ID**: A unique identifier (e.g., `"Dataset123"`) embedded in the token asset name for traceability.

#### Minted Assets
- **Reference NFT**:
  - **Policy ID**: Unique per minting policy instance.
  - **Asset Name**: `"DataNation.<DatasetID>.Ref"` **TBD**.
  - **Quantity**: 1.
  - **Metadata (CIP-68)**:
    - `assetCertificate`: Hash linking to the dataset.
    - `versionHash`: Dataset version identifier.
    - `kycBarcode`: KYC compliance identifier.
    - `fractionalSale`: Indicates fractionalization status.
    - `lockInTimePeriod`: Duration (in days, e.g., 7, 15, 30, 90) set by the owner, affecting income generation.
- **Fractional Tokens**:
  - **Policy ID**: Same as the NFT.
  - **Asset Name**: `"DataNation.<DatasetID>.Fraction"` **TBD**.
  - **Quantity**: 1,000.
  - Fungible tokens representing equal shares of the dataset’s income.

#### Validation Logic
- The transaction must be signed by the platform’s PKH.
- Exactly one reference NFT and 1,000 fractional tokens are minted in a single transaction.
- Metadata for the NFT is set according to CIP-68 standards.
- No additional minting is allowed after the initial transaction (ensured by a unique redeemer or time constraint).

#### Aiken Code Outline
TBD

---

### 2. NFT Locking Script

#### Purpose
This script locks the reference NFT at a script address, allowing it to be transferred only when a buyer pays the dataset’s buyout price to the treasury, transferring ownership and ending the income stream for fractional holders.

#### Parameters
- **NFT Policy ID and Asset Name**: Identifies the specific reference NFT.
- **Treasury Script Address**: Where buyout proceeds are sent.

#### Datum
TBD

#### Validation Logic
- **Buyout Action**:
  - The transaction consumes the NFT UTXO.
  - At least `buyoutPrice` lovelace is sent to `treasuryAddress`.
  - The NFT is sent to the buyer’s address.
- **No Other Action**: The NFT cannot be moved unless the above conditions are met.

#### Aiken Code Outline
TBD

---

### 3. Marketplace Script

#### Purpose
The marketplace script enables listing and purchasing of fractional tokens, allowing stakeholders to trade ownership stakes in dataset income.

#### Parameters
- **Platform Fee Address**: Where transaction fees are sent (optional).
- **Fee Rate**: Percentage of the sale price (e.g., 1%).

#### Datum for Listing
TBD

#### Redeemers
- `List`: Lists tokens for sale.
- `Purchase`: Buys listed tokens.

#### Validation Logic
- **Listing**:
  - Tokens are sent to the script address with a `ListingDatum`.
  - Transaction is signed by the seller.
- **Purchasing**:
  - Consumes a listing UTXO.
  - Sends `amount * pricePerToken` lovelace to the seller (minus fees, if applicable).
  - Transfers `amount` tokens to the buyer’s address.
  - Optionally sends a fee to the platform.

#### Aiken Code Outline
TBD

---

### 4. Treasury Script

#### Purpose
The treasury script manages income from dataset usage and buyouts, distributing it to fractional token holders based on their ownership stakes and the time since their last withdrawal.

#### Parameters
- **Token Policy ID and Asset Name**: Identifies the fractional tokens for this dataset.
- **Scale Factor**: `1_000_000` (1e6) for integer precision.

#### Treasury Datum
TBD

#### User Token UTXO Datum
TBD

#### Redeemers
- `AddIncome { amount: Int }`: Adds income from usage or buyout.
- `Withdraw`: Allows a fractional owner to claim their share.

#### Validation Logic
- **AddIncome**:
  - Consumes the treasury UTXO.
  - Increases `cumulativeIncomePerToken` by `(amount * SCALE) / 1000`.
  - Outputs a new treasury UTXO with updated datum and increased value.
- **Withdraw**:
  - Consumes the treasury UTXO and a user’s token UTXO.
  - Calculates owed amount: `(currentCumulativeIncomePerToken - lastClaimCumulativeIncomePerToken) * numberOfTokens / SCALE`.
  - Pays the owed amount to the user.
  - Outputs a new treasury UTXO with reduced value.
  - Outputs a user UTXO with the same tokens and updated `lastClaimCumulativeIncomePerToken`.

#### Aiken Code Outline
TBD

---

## Additional Considerations

### Merkle Patricia Forestry (MPF)
Originally in my whitepaper I suggested suggested using an MPF to update fractional token metadata (e.g., timestamps). However, since fractional tokens are fungible and their datum is tied to UTXOs (not individual tokens), the treasury script’s approach above I believe suffices for now. MPF will be explored for more complex metadata updates in future iterations, using the Aiken library at [GitHub](https://aiken-lang.github.io/merkle-patricia-forestry).

### Workflow
1. **Dataset Upload**: Platform mints NFT and tokens, locks NFT, and distributes tokens to the owner.
2. **Marketplace Trading**: Owners list tokens; buyers purchase them.
3. **Income Generation**: Usage payments are sent to the treasury, increasing `cumulativeIncomePerToken`.
4. **Withdrawals**: Token holders claim income after the lock-in period.
5. **Buyout**: A buyer pays the buyout price, NFT is transferred, proceeds are added to the treasury, and income generation ceases.

### Security
- Validate token authenticity using policy IDs and asset names.
- Ensure only authorised entities (e.g., platform, owners) can trigger actions.
- Use large scale factors to prevent precision loss in calculations.

---

This design provides a foundation for DataNation’s smart contracts in Aiken, aligning with the project’s vision of decentralised data monetisation. Further refinements can incorporate additional features like subscription management or DeFi staking as the platform evolves.

--- 
