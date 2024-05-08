<h1 align="center">zklink point system adapters</h1>
<p align="center"><i>Guide Document for Projects Integrated with ZKLink Points System</i></p>

# Preamble

This document provides detailed guidance for protocol developers interested in integrating their decentralized applications (dApps), decentralized exchanges (DEXes), or other protocols with the zkLink ecosystem to leverage the Nova points incentive system. By following this guide, developers can create compatible adapters, format data submissions and understand the integration workflow.

## Goal

The main purpose of this integration is to enable protocols that have successfully integrated with the zkLink Nova system to automatically allocate Nova points to their users.

As the final verification step in this integration process, we will conduct a rigorous comparison between the CSV file generated by your adapter and the actual on-chain data obtained from your contract deployed on the zkLink network.

This comparison is crucial to ensure the accuracy and consistency of the data before finalizing the integration and activating point allocation.

Once the verification is successful, zkLink will start distributing Nova points to users.

## Points

### Briefly Formulas:

$$
Points = Group Booster (Ecosystem Points + Asset Points) + Referral Points + Bonus Points
$$

Here is a brief overview of the zkLink points framework and components that need to be understood in order to clearly understand the types of points data you need to provide. When users transfer assets from cross-chain bridges to the zkLink Nova network, they can participate in various protocol interactions within the zkLink network ecosystem. We will calculate points based on the assets directly held by users and the metrics generated from interactions with dApps. In the **adapters**, we collect 3 types of metrics data provided by project parties to calculate _EcosystemPoints_, the _EcosystemPoints_ formulas is:

$$
Ecosystem Points = Vol{\text{\scriptsize u,t}} Points + TVL_{\text{\scriptsize u,t}} Points + TxNum_{\text{\scriptsize u,t }}Points
$$

1. Vol<sub>u,t</sub>,
   Vol signifies the total volume of trades (in ETH) executed by a user over a period of time.

   $$ VolPoints\_{\text{\scriptsize u,t}} = Booster · 1/1000 · Price · Quantity $$

   - Spot DEX, Spot PERPs, Lending, Bridge:
     The total trading/lending/bridging volume by the user based on the formula above.
   - GameFi, NFTFi, SocialFi:
     The total trading volume of assets such as NFT and FT.  
      The total spending in the game economy (eg. tokens spent in gameFi, in-game asset trading etc)
     In this sense, the total trading volume can signify total transaction volume in gameFi. - To prevent sybil attack, a multiplier of 1/x where x = 1000, 2000,..., N is used for each specific protocol. The multiplier is incorporated into the constant b.

2. TVL<sub>u,t</sub> (Total Locked Value):
   Total value of liquidity provided to DEX pools or PERPs or Lending , The TVL refers to the total value of different tokens owned by each user in a specific period of time.

   These tokens are not sTokens or lTokens; rather, they represent the quantity of underlying tokens corresponding to these collateral certificates in the pool.

   For example, if two users, A and B, each stake 20 ETH in a protocol pool and receive 20 lpETH, then the pool locks 40 ETH of underlying tokens. When user C borrows 20 ETH, the underlying token balance for users A and B in the protocol is each 10 ETH. The calculation formula is: `TVL_u =20 lpETH / 40 lpETH * 20 ETH`.

3. TxNum<sub>u,t</sub> signifies the total number of transactions
   - For Dex/Perps/Lending protocol, tx volume should be grater than 0.1 ETH or 500 USDC/USDT.
   - For Bridge, it should be Number of bridged transactions.
   - For GameFi/NFTFi, it is the total number of on-chain interactions in the protocol.

## Getting Started

### Sample Adapter

Check this [example](./example/)

You can use a subgraph or an existing service to process adapter data.

**Requirements:**

- You must provide an **npm project** with the correct npm dependencies included in the **package.json** and output an executable Node.js script in _dist/index_.
- We need to provide the functions `getUserTVolByBlock`, `getUserTVLByBlock`, and `getUserTxNumByBlock`, based on three different types of points: **TVol**, **TVL**, and **TxNum**, which takes _blockNumber_ and _blockTimestamp_ as a parameter to execute the script and outputs a CSV file.
- In the output CSV file, you need to include snapshot data of all users within the protocol at this _blockNumber_.

## Installation & Setup

If you utilize subgraph on Nova network, you must include the subgraph package in the project directory.

The zkLink Nova network is not yet supported in The Graph's list of supported networks. As a result, to create a subgraph on the zkLink Nova network, it is necessary to set up your own hosted service. We have already set up a functional hosted service specifically for creating subgraphs on the zkLink Nova network. Please note the following requirements before proceeding:

- We will provide you with the deployment scripts necessary for local development. However, you need to set a secret environment variable to distinguish the deployment path from other projects and prevent it from being arbitrarily overwritten. see [env](#how-to-deploy-and-test-your-own-subgraph)
- Once the deployment is complete, teams must upload their source code to the **subgraph** directory.
- We will deploy it from our repository to our official production environment.

### How to deploy and test your own subgraph?

Then check the **.env.example** file in the **subgraph** folder of your project, create your own **.env** file, and change the `SUBGRAPH_PATH_SECRET` to a random value which is known only within your team.

```bash
cd src/adapters/<project folder>/subgraph/ && cp .env.example .env
```

### Launch

Return to the root directory of the adapters, you can use this script to deploy subgraph onto Nova subgraph service. Execute the deploySubgraph.js script with the following command:

```bash
cd src/adapters/<project folder>/ && node ../deploySubgraph.js --directory <project folder>
```

Upon successful deployment, a subgraph URL will be returned and displayed in your terminal.

## Building the Adapter

### Data requirement

Capture a snapshot every 8 hours at 02:00 UTC, 10:00 UTC, and 18:00 UTC of the Total Value Locked (TVL) distributed by assets per user on a daily basis.

### Output data schema

#### Vol

| Data Field   | Notes                                                                       |
| ------------ | --------------------------------------------------------------------------- |
| userAddress  | 0xe75FF0b77c1859AC30EB1EEBC53C5BAdD28d19F7                                  |
| tokenAddress | 0x2F8A25ac62179B31D62D7F80884AE57464699059                                  |
| poolAddress  | Each pool’s contract address eg: 0xE8a8f1D76625B03b787F6ED17bD746e0515F3aEf |
| volume       | Transaction volume, eg 2300 USD                                             |

#### TVL

| Data Field   | Notes                                                                       |
| ------------ | --------------------------------------------------------------------------- |
| userAddress  | 0xe75FF0b77c1859AC30EB1EEBC53C5BAdD28d19F7                                  |
| tokenAddress | 0x2F8A25ac62179B31D62D7F80884AE57464699059                                  |
| poolAddress  | Each pool’s contract address eg: 0xE8a8f1D76625B03b787F6ED17bD746e0515F3aEf |
| balance      | Historical balances raw data. 0.1 ETH should be 100000000000000000          |

#### TxNum

| Data Field        | Notes                                                                       |
| ----------------- | --------------------------------------------------------------------------- |
| userAddress       | 0xe75FF0b77c1859AC30EB1EEBC53C5BAdD28d19F7                                  |
| tokenAddress      | 0x2F8A25ac62179B31D62D7F80884AE57464699059                                  |
| poolAddress       | Each pool’s contract address eg: 0xE8a8f1D76625B03b787F6ED17bD746e0515F3aEf |
| transactionNumber | Transaction Number, eg: 321                                                 |

## Data Format & Submission

Guidelines on how to generate and format data in the required CSV format for submission.

### CSV Format Specification

Required fields, referencing to:

```typescript
type OutputDataSchemaRow = {
  userAddress: string;
  tokenAddress: string;
  poolAddress: string;
  balance: string;
};
```

## 5.2 Testing & Validation

Teams can use these test scripts to validate the csv results such that it meets our requirements.

```bash
cd src/adapters/<project folder>/ && node ../testScript.js <project folder>
```

We will conduct a sampling verification of your CSV results. Teams are required to provide detailed methods of verification, including the approach and information utilized in the verification process.

Teams should submit some transactions, data reference on the **blockchain explorer** or **analytics platform** such as **Defillama** and a detailed verification method etc. We will utilize the data or tools provided you have provided to verify the accuracy and authenticity of the on-chain data.

We will facilitate our comparison of the provided data against the data provided in the generated CSV.

Important Note: If we are unable to verify the authenticity of CSV data, then the PR will be rejected.