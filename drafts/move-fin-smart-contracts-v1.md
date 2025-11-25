Here is a curated list of open-source financial smart contract projects built with **Move**, categorized by ecosystem (Aptos and Sui). These repositories are excellent for learning how high-value DeFi protocols are structured in Move.

### **📘 Move Financial Smart Contract Projects**

| Project Name      | Ecosystem | Category   | GitHub Repository                                                                                                                                        | Description                                                                                                                            |
| :---------------- | :-------- | :--------- | :------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| **Liquidswap**    | Aptos     | DEX (AMM)  | [pontem-network/liquidswap](https://github.com/pontem-network/liquidswap)                                                                                | One of the first AMMs on Aptos. Excellent for learning standard swap logic, liquidity pools, and generic asset handling in Move.       |
| **Econia**        | Aptos     | Order Book | [econia-labs/econia](https://github.com/econia-labs/econia)                                                                                              | A hyper-parallelized on-chain central limit order book (CLOB). Great for studying high-performance execution and order matching logic. |
| **Aries Markets** | Aptos     | Lending    | [Aries-Markets/aries-markets-contract-public](https://www.google.com/search?q=https://github.com/Aries-Markets/aries-markets-contract-public&authuser=2) | A decentralized margin trading and lending protocol. Useful for understanding money markets and account management.                    |
| **Aux Exchange**  | Aptos     | DEX        | [aux-exchange/aux-exchange](https://github.com/aux-exchange/aux-exchange)                                                                                | A comprehensive exchange featuring both an AMM and a CLOB. A larger codebase that demonstrates complex financial architecture.         |
| **Thala Labs**    | Aptos     | DeFi Suite | [ThalaLabs/aptos-playground](https://github.com/ThalaLabs/aptos-playground)                                                                              | Contains experimental and playground code for Thala's stablecoin and swap products. Good for seeing newer Move features in action.     |
| **DeepBook**      | Sui       | Order Book | [MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3)                                                                                        | **Official** CLOB primitive for Sui built by Mysten Labs. The "gold standard" for efficient order book implementation on Sui.          |
| **Scallop**       | Sui       | Lending    | [scallop-io/sui-lending-protocol](https://github.com/scallop-io/sui-lending-protocol)                                                                    | An over-collateralized lending protocol. Highly readable code for understanding borrowing, supplying, and interest rate models on Sui. |
| **Navi Protocol** | Sui       | Lending    | [naviprotocol/navi-smart-contracts](https://www.google.com/search?q=https://github.com/naviprotocol/navi-smart-contracts&authuser=2)                     | Another leading liquidity protocol on Sui. Provides a clear look at how to structure interface interactions and protocol reserves.     |
| **Cetus**         | Sui       | DEX (CLMM) | [CetusProtocol/cetus-clmm-interface](https://github.com/CetusProtocol/cetus-clmm-interface)                                                              | Concentrated Liquidity Market Maker. While the core is complex, this repo shows the interface and interaction logic for advanced AMMs. |

---

### **💡 Tips for Reading These Repositories**

1. **Start with the sources/ folder:** In Move projects, the actual smart contract logic always lives here. Look for files ending in .move.
2. **Look for Move.toml:** This file defines the project dependencies. It tells you which frameworks (like AptosFramework or SuiFramework) the project relies on.
3. **Aptos vs. Sui Move:**
   - **Aptos** projects often use account::signer and global storage resources extensively.
   - **Sui** projects rely heavily on "Objects" and passing \&mut TxContext. When reading Sui code, look for structs with the key and store abilities.
