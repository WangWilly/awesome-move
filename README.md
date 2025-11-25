# **The Move Financial Ecosystem: A Comprehensive Analysis of Smart Contract Architectures on Sui, Aptos, and Emerging Networks**

## **1\. Executive Summary and Ecosystem Overview**

The decentralized finance (DeFi) landscape is currently undergoing a structural paradigm shift, driven by the emergence and rapid maturation of the Move programming language. Originally developed by Meta (formerly Facebook) for the Libra/Diem blockchain, Move has evolved into a platform-agnostic smart contract language designed specifically for the rigorous manipulation of digital assets. Unlike Solidity, which treats assets as mutable entries in a hash map, Move treats assets as "resources"—linear types that cannot be copied or implicitly discarded, only moved between storage locations. This fundamental architectural difference addresses the most pervasive security vulnerabilities in Web3, such as reentrancy attacks and double-spending bugs, offering a new foundation for high-value financial engineering.

This report provides an exhaustive analysis of financial smart contract projects built using the Move language, focusing primarily on the two dominant Layer 1 blockchains: **Sui** and **Aptos**, while also covering emerging networks like **Starcoin** and **Movement Labs**. The analysis is derived from deep repository inspection, architectural documentation, and ecosystem directories. It identifies a robust ecosystem where financial primitives—Decentralized Exchanges (DEXs), Lending Protocols, and Derivatives—are being rebuilt with a focus on formal verification, parallel execution, and object-centric state management.1

The current state of Move-based financial engineering is characterized by a "bifurcation of dialect." While both Sui and Aptos utilize Move, they employ divergent storage models. Aptos adheres closer to the original Diem vision with global storage and account-based resources, whereas Sui introduces an object-centric model where smart contracts operate on distinct objects rather than account balances. This report dissects how these differences manifest in the codebases of major financial projects, offering a granular view of the open-source repositories available for study and integration.

The analysis reveals that the Move ecosystem has matured beyond experimental implementations into a sophisticated network of interoperable financial applications. Projects like Thala Labs on Aptos and Scallop on Sui demonstrate a high degree of tooling maturity, employing formal verification methods and modular SDKs to abstract complexity. Furthermore, the rise of "Super Apps"—platforms integrating swapping, lending, and stablecoin issuance—suggests a consolidation of financial services within highly integrated smart contract suites.

## **2\. The Move Language Paradigm in Financial Engineering**

To understand the projects listed in this report, one must first grasp the architectural imperatives that drive Move developers. Financial applications on Move leverage specific language features that differentiate them from Ethereum Virtual Machine (EVM) implementations. This paradigm shift is not merely syntactical but deeply structural, influencing how liquidity is pooled, how interest is accrued, and how assets are secured against malicious actors.

### **2.1 Asset Safety via Linear Logic**

In financial smart contracts, the primary risk vector is often the mishandling of asset balances. Move mitigates this through linear logic, a concept borrowed from linear logic systems in type theory. A resource in Move is a struct that has specific abilities: key, store, copy, and drop. Crucially, financial tokens usually lack the copy and drop abilities. This means a token cannot be duplicated (preventing inflation bugs) and cannot be destroyed accidentally (preventing fund loss). If a function takes a Coin resource as input, it _must_ either transfer it to another address or destroy it explicitly via a specific bytecode instruction. This enforces "conservation of mass" for digital assets at the compiler level.1

The implications of this for financial engineering are profound. In Solidity, a token transfer is a state change in a ledger mapping mapping(address \=\> uint256). In Move, a token transfer is the actual movement of a resource object from one storage slot to another. This prevents an entire class of "double spending" bugs where a developer might update a balance without deducting it from the sender. For example, in the implementation of the Coin module found in the Aptos Framework or Sui Framework, the withdraw function returns a Coin\<T\> resource that must be handled by the calling function. It cannot be ignored or left in a local variable scope; the code will simply fail to compile. This rigorous enforcement forces developers to account for every single unit of value within a transaction flow, creating a "safety-by-default" environment for DeFi protocols.

### **2.2 Formal Verification with the Move Prover**

High-value financial projects on Aptos and Sui heavily utilize the Move Prover, a formal verification tool capable of mathematically proving that a smart contract adheres to its specification.4 Unlike unit testing, which checks specific inputs against expected outputs, the Move Prover checks all possible inputs against a defined logic model. Projects like **Econia** and **DeepBook** utilize this to ensure their order matching engines are deterministic and error-free. The emphasis on verification is a recurring theme in the repositories analyzed, where specification files (.spec.move) often accompany the logic files.

The Move Prover operates by translating Move bytecode into the Boogie intermediate verification language, which is then checked by the Z3 theorem prover. This allows developers to write invariants—statements that must always be true—directly into their code. For instance, a lending protocol like Scallop can define an invariant stating that "the total assets in the vault must always equal the sum of user deposits plus accrued interest." The Move Prover will then mathematically verify if there is _any_ sequence of transactions that could violate this rule. If a reentrancy path or logic error exists that breaks this invariant, the verification fails before the code is even deployed. This level of rigor is rarely seen in other ecosystems and represents a significant maturation of smart contract engineering standards.

### **2.3 Object-Centricity vs. Global Storage**

The ecosystem is divided into two architectural approaches, creating distinct patterns for financial application development:

- **The Aptos Model:** Follows the Diem legacy. Data is stored within resources attached to accounts. To access a liquidity pool, the transaction interacts with the resource stored at the protocol's address. This "global storage" model is similar to a traditional file system where data lives at specific paths (addresses). Financial contracts on Aptos often use a central resource account to hold the state of the protocol (e.g., the liquidity pool reserves), and users interact with this central state. This centralization simplifies the mental model but can create contention points if not managed carefully with parallel execution engines like Block-STM.
- **The Sui Model:** Data exists as distinct "Objects" with unique IDs. A liquidity pool in a Sui DEX (like Cetus or Turbos) is a shared object. This allows for massive parallelism; transactions touching different pools can execute simultaneously because they do not contend for the same global state.4 The object-centric model changes how developers think about asset ownership. Instead of an account "having" a balance in a contract, an account "owns" a Coin object. Financial protocols on Sui are composed of shared objects (pools, order books) and owned objects (user positions, NFTs). This distinction allows Sui to parallelize transactions that don't touch the same shared objects, enabling extremely high throughput for non-overlapping financial activities.

## **3\. The Sui Ecosystem: Financial Architectures**

Sui has rapidly cultivated a dense DeFi ecosystem, heavily leveraging its object-centric model to enable low-latency trading and on-chain order books. The platform's unique ability to handle shared objects allows for complex financial primitives that would be prohibitively expensive or slow on other networks. The following analysis details the key financial projects, their architectural focus, and their repository structures.

### **3.1 Decentralized Exchanges (AMM & CLMM)**

The standard for trading on Sui has shifted towards Concentrated Liquidity Market Makers (CLMM), mirroring Uniswap V3 but optimized for Sui's parallel execution. This shift is driven by the capital efficiency of CLMMs and the capability of the Move language to handle the complex math and tick management required for such systems.

#### **Cetus Protocol**

Cetus is a pioneer in bringing CLMM to Move-based chains. Its architecture is critical for developers studying high-efficiency liquidity provision.

- **Architectural Insight:** Cetus utilizes a permissionless standard where liquidity positions are represented as NFTs (objects) on Sui. This leverages Sui's object model natively; users own their positions as tangible assets in their wallets. The protocol separates its interface logic from the core CLMM logic to facilitate integration, allowing other protocols to easily build on top of Cetus liquidity.
- **Repository Analysis:** The cetus-clmm-sui-sdk 6 is the primary entry point for developers, written in TypeScript (99.5%).7 It abstracts the complexity of math and object management. The cetus-clmm-interface 8 repository provides the Move definitions required for other smart contracts to interact with Cetus pools, demonstrating the "composability" aspect of Move.
- **Key Insight:** The existence of a dedicated "interface" repository highlights a trend in Move development: avoiding monolithic codebases in favor of modular, interactable packages that allow third-party contracts to call into the protocol without redeploying logic. This modularity reduces contract bloat and gas costs for integrators.
- **Operational Mechanism:** The aggregator repository 9 shows how Cetus integrates multiple liquidity sources (Cetus, DeepBook, Kriya, FlowX) into a single routing engine. This aggregator uses a "Router Swap" model where the client calculates the best path off-chain (using the SDK) and submits the route to the on-chain aggregator contract for execution.

#### **Turbos Finance**

Turbos Finance, backed by Jump Crypto, emphasizes a non-custodial and hyper-efficient DEX model.10

- **Architectural Insight:** Turbos focuses on simplifying the CLMM experience. Their open-source approach includes a public interface that allows developers to build "vaults" or "strategy managers" on top of their pools.11 This design choice encourages a layered ecosystem where Turbos provides the base liquidity layer, and other developers build asset management products on top.
- **Repository Analysis:** The turbos-sui-move-interface 12 is actively maintained, with recent commits referencing Access Control List (ACL) configurations. This indicates a focus on institutional-grade permissioning layers, likely to support regulated or compliant trading environments. The turbos-clmm-sdk 13 mirrors the industry standard of providing heavy TypeScript wrappers around Move calls, enabling complex off-chain calculations for optimal swap routes.
- **Ecosystem Integration:** The turbos-vault-sc repository 14 suggests the development of automated rebalance vaults. These vaults would automatically adjust liquidity positions within the CLMM ticks to maximize fee generation, a sophisticated financial product that requires the safety and expressiveness of Move to implement securely.

#### **Aftermath Finance**

Aftermath takes a broader approach, acting as a DEX, a Liquid Staking Derivative (LSD) platform, and an aggregator.

- **Architectural Insight:** Aftermath's AMM implementation, found in move-amm-public 15, reveals how multi-asset pools (similar to Balancer) can be implemented on Sui. Unlike standard pairs (X/Y), multi-asset pools allow for the creation of index-like products where a single liquidity pool holds a basket of assets (e.g., SUI, USDC, ETH, BTC) with defined weights.
- **Smart Order Routing:** The presence of sui-move-inline 15 in their repository list suggests they are optimizing gas consumption by analyzing the cost of Move functions. This is a critical step for aggregators that must split trades across multiple pools efficiently. By analyzing the gas cost of various execution paths, Aftermath can offer users the most cost-effective routes.
- **Repository Structure:** The aftermath-ts-sdk 7 is a robust toolkit for interacting with all facets of the protocol. The SDK includes logic for the "Router," "Staking," and "Perpetuals" (indicated by jest.config.ts references to perpetual testing 7), showcasing the breadth of their financial suite.

### **3.2 On-Chain Central Limit Order Books (CLOB)**

One of the defining features of the Sui ecosystem is the viability of fully on-chain order books, made possible by the network's high throughput and low latency. This represents a significant deviation from the AMM-dominated landscape of Ethereum, bringing traditional finance (TradFi) market structures to DeFi.

#### **DeepBook**

DeepBook is a public good primitive on Sui, providing a shared liquidity layer for other protocols.16 It acts as a foundational "liquidity plumbing" for the entire ecosystem.

- **Architectural Insight:** DeepBook V3 introduces flash loans and governance directly into the matching engine. It leverages Sui's parallel execution to match orders without blocking the entire network. This is a significant departure from Ethereum, where order books are prohibitively expensive due to the cost of state updates. DeepBook creates a "Custodial" model where the order book contract holds the assets, but the matching logic is executed efficiently via shared objects.
- **Integration:** The sui-move-deepbook 17 repositories demonstrate how to interact with this layer. Because it is a "native" shared object, other DeFi protocols (like Kriya or Aftermath) can route trades through DeepBook to access deep liquidity without building their own matching engines. This promotes liquidity concentration rather than fragmentation.
- **Transparency and Privacy:** As a CLOB, DeepBook functions as a transparent ledger of bids and asks. The docs.sui.io snippet 18 highlights that while the ledger is open, the matching engine respects user parameters, allowing for sophisticated trading strategies that rely on visible market depth.

#### **Kriya DEX**

Kriya specializes in derivatives and spot trading, also utilizing an order book model.

- **Repository Analysis:** The kriya-dex-interface 19 provides the exact Move structs used in their system: KriyaLPToken and Pool. The code snippets reveal specific fee scaling mechanisms (scaleX, scaleY) and permission flags (is_swap_enabled, is_stable), offering a clear template for developers looking to build managed liquidity pools.
- **Derivatives Logic:** The structure of the Pool object in Kriya includes fields for lsp_locked (locked liquidity provider tokens) and protocol_fee balances.19 This indicates a more complex fee distribution model suited for perpetual swaps where funding rates and protocol insurance funds must be managed programmatically. The interface allows for the creation of pools that can be toggled between "stable" (curve-like) and "volatile" (CLMM/CLOB) modes.

### **3.3 Lending and Borrowing Protocols**

Lending on Move leverages the "Hot Potato" pattern (a struct with no abilities that must be returned to the contract) to handle flash loans and ensuring solvency. This pattern makes flash loans safer and easier to implement than in Solidity, where developers must rely on callbacks and balance checks.

#### **Scallop**

Scallop describes itself as a "Next Generation Money Market".20

- **Architectural Insight:** Scallop’s architecture separates the query logic (ScallopQuery) from the transaction building logic (ScallopBuilder).7 This separation of concerns in their SDK (sui-scallop-sdk) 1 allows frontend developers to query chain state efficiently via indexers before constructing complex transaction blocks. This significantly reduces the latency perceived by the user, as the heavy lifting of state retrieval is decoupled from transaction construction.
- **Repository Status:** The sui-lending-protocol repo 20 contains the core logic. Recent commits regarding "supply limits" and "interest models" suggest an active governance process tweaking the risk parameters of the protocol.21 The sui-scallop-sdk is extensively maintained, with a recent V2 release focusing on modularity.
- **Models and Utilities:** The SDK includes a ScallopAddress model for managing contract addresses across mainnet environments and a ScallopUtils module for helper functions.7 This structure is critical for maintaining upgradability; as the underlying contracts are upgraded (via Sui's package upgrade mechanism), the SDK can simply update the address registry without breaking the application logic.

#### **Navi Protocol**

Navi is a leading liquidity protocol on Sui, evolving into a comprehensive DeFi hub.

- **SDK Evolution:** The transition from navi-sdk (legacy) to a modular new SDK 7 highlights the maturation of the ecosystem. The new SDK splits functionality into Lending, Bridge, and Aggregator packages. This modularity is essential as protocols expand from single-use cases to "Super Apps" where a single interface might offer lending, swapping, and cross-chain bridging simultaneously.
- **Repository Analysis:** While some core logic is proprietary, the protocol-interface and navi-smart-contracts 22 are public, allowing for composability. The navi-sdk repository reveals a focus on cross-chain functionality ("Bridge SDK") 7, suggesting Navi aims to be a liquidity gateway for assets entering Sui from other chains.
- **Legacy vs. Modern:** The deprecation of the old SDK in favor of the new modular architecture 7 serves as a case study in technical debt management within rapidly evolving DeFi ecosystems. Developers integrating Navi must ensure they are pulling the scoped packages (e.g., @naviprotocol/lending-sdk) rather than the monolithic legacy package.

### **3.4 Aggregators and Infrastructure**

Aggregators are the connective tissue of the financial ecosystem, ensuring users get the best execution price by routing orders across multiple DEXs.

- **7k Aggregator:** The 7k-sdk-ts 7 repository provides the tools to quote and swap across multiple venues. A notable technical detail is the requirement of @pythnetwork/pyth-sui-js as a peer dependency.7 This indicates that the aggregator likely uses Pyth oracles for real-time price reference checks or for securing limit orders against stale price data. The SDK supports "BluefinX" routing, implying integration with off-chain order books or specialized derivative layers.
- **Hop Aggregator:** Provides a similar service, emphasizing API integration for other dApps to earn revenue shares.7 The "Hop SDK" allows developers to embed swap functionality directly into wallets or other dApps, monetizing the volume via a fee share model. This represents the "B2B" side of DeFi protocols.
- **Bucket Protocol:** Bucket is a Collateralized Debt Position (CDP) protocol issuing the $BUCK stablecoin. Its bucket-protocol-sdk 7 allows users to interact with the protocol programmatically—drawing loans, depositing collateral (SUI, BTC, ETH), and managing positions. The protocol enforces a minimum collateral ratio of 110%, a parameter critical for solvency which is managed directly by the Move contracts referenced in the SDK.

## **4\. The Aptos Ecosystem: Financial Architectures**

Aptos, sharing the same Move lineage, focuses heavily on "Block-STM" (Software Transactional Memory) to achieve parallelism. Its financial ecosystem is characterized by highly integrated, often monolithic "Super Apps" and robust standard libraries. Unlike Sui's object-centric model, Aptos relies on global resource storage, leading to different design patterns for liquidity pools and state management.

### **4.1 Integrated DeFi Suites**

#### **Thala Labs**

Thala is a prime example of a DeFi "Super App" on Aptos, combining a stablecoin (MOD), a swap (ThalaSwap), and a launchpad.

- **Architectural Insight:** Thala utilizes a "weighted pool" concept similar to Balancer. This allows for pools with arbitrary asset weights (e.g., 80/20 pools), enabling liquidity bootstrapping pools (LBPs) and ETF-like products. Their repositories, such as thalaswap-integration-starter 23 and integrate-thala 24, provide developers with pre-packaged Move interfaces.
- **Type Safety Innovation:** Thala developed surf 25, a TypeScript library that infers types directly from Move ABIs (Application Binary Interfaces). This eliminates the need for manual encoding/decoding of arguments, significantly reducing frontend bugs. This demonstrates a high level of developer tooling maturity, moving beyond basic SDKs to sophisticated type-generation systems that improve the developer experience (DX).
- **Fixed Point Math:** Thala maintains fixed_point64 26, a specialized library for high-precision arithmetic. Financial math in DeFi often requires precision beyond standard integer types to calculate interest and exchange rates accurately. Thala's library provides Q64.64 fixed-point numbers, addressing a common pain point in Solidity where math overflows or precision loss can lead to substantial financial discrepancies.
- **Resource Account Model:** Thala uses a specific resource account (0x4827...) to hold pool resources.24 This centralization simplifies the discovery of pools—one can simply query the resource account to find all active pools—but requires careful management of signer capabilities to ensure only authorized contracts can modify the state.

#### **Aries Markets**

Aries offers a decentralized margin trading protocol.

- **Repository Analysis:** aries-markets-contract-public 27 and aries-tssdk 28 are the primary resources. The presence of supply apy calculation in commit messages 28 indicates that the calculation logic for interest rates is transparently managed within the SDKs and contracts.
- **Lending Integration:** Aries functions as a money market, allowing users to lend assets to earn yield while others borrow for leverage. The codebase likely implements a "utilization-based" interest rate model, where the cost of borrowing increases as the pool becomes more utilized.

### **4.2 Order Book Primitives**

#### **Econia Labs**

Econia is the foundational order book layer for Aptos, similar to DeepBook on Sui but architecturalized differently for Aptos's global storage.

- **Architectural Insight:** Econia uses an "atomic matching engine" that settles trades within a single transaction. This allows for composability where a user can swap, lend, and borrow in one atomic move. The engine leverages Aptos's optimistic concurrency control to process non-conflicting orders in parallel.
- **Repository Analysis:** The econia repository 29 is highly structured, containing the Move source code. It is designed to be a "hyper-parallelized" CLOB. The repository includes strict code ownership (CODEOWNERS) and auditing branches (v4.2.0-audited).30 This tagging of audited code is a best practice in high-stakes financial software, providing assurance to integrators that the code they are interacting with has been reviewed by security firms.
- **Multisig Governance:** Issue trackers in the Econia repository reveal discussions about multisig transaction verification.30 This highlights the governance layer of the protocol, where upgrades and parameter changes are managed by a multisignature wallet, adding a layer of human security to the automated code.

### **4.3 Automated Market Makers (AMM)**

#### **Liquidswap (Pontem Network)**

Liquidswap was one of the first AMMs on Aptos and remains a cornerstone of the ecosystem.

- **Repository Analysis:** liquidswap 31 contains the core logic. They also provide liquidswap_router_v2 32, enabling third-party dependency integration. The use of Move.toml dependency definitions 32 shows how deeply modular the Aptos Move ecosystem is; projects can pull in Liquidswap as a local or git dependency to build on top of its liquidity.
- **Curve Mechanisms:** Liquidswap implements both uncorrelated (Uniswap V2 style) and stable (Curve style) swapping curves. This flexibility allows it to serve as a general-purpose DEX for volatile assets and a highly efficient venue for stablecoin swaps.

#### **PancakeSwap (Aptos Deployment)**

The BSC giant expanded to Aptos, bringing its massive user base and brand recognition.

- **Repository Analysis:** pancake-contracts-move 33 contains the Move adaptation of their famous Uniswap V2 fork. It includes modules for Exchange Protocol, Farms, and IFO (Initial Farm Offerings).34 This serves as an excellent reference for developers translating Solidity logic (Uniswap V2) into Move. The repository structure is segmented into "projects" 34, separating the core exchange logic from the peripheral farming and auction systems.

#### **Aux Exchange**

Aux offers a hybrid AMM and CLOB, attempting to unify the two dominant liquidity models.

- **Repository Analysis:** aux-exchange 35 is a comprehensive repo containing the Move contracts (aptos folder) and a Go-based SDK (go-util). This suggests a focus on backend integration for market makers who prefer Go over TypeScript. The repository includes a router implementation designed to execute trades across both the AMM and CLOB to find the best price execution.

### **4.4 NFT Marketplaces and Financialization**

#### **BlueMove**

BlueMove is the leading NFT marketplace on Aptos (and Sui), supporting NFT trading and launchpads.

- **Integration:** While the core contracts are often closed-source or not fully public in a single repo, the aptos-nft-aggregator repository 36 contains configuration files specifically for indexing BlueMove events (listing_created, token_offer_created). This suggests that financial data aggregators treat BlueMove as a primary data source.
- **Event Structure:** The aggregator config reveals the event types supported by BlueMove: OfferEvent, ListEvent, BuyEvent.36 Understanding these event structures is crucial for developers building NFT analytics tools or trading bots that react to market activity in real-time.

## **5\. Emerging and Niche Move Ecosystems**

While Sui and Aptos dominate, the Move story extends to other networks that offer unique architectural variations. These emerging chains are exploring different consensus mechanisms and layers, broadening the scope of what is possible with Move.

### **5.1 Starcoin**

Starcoin is unique for implementing Move on a Proof-of-Work (PoW) consensus mechanism, aiming to combine Bitcoin's security model with Move's smart contract safety.

- **Repository Analysis:** starcoin-framework 38 contains the standard library. The starcoin core repo 39 is written in Rust. They utilize a DaoSpace and Starswap (Uniswap style DEX) 1, demonstrating that Move's utility extends beyond high-throughput PoS chains. The Starswap project represents a deployment of standard AMM logic in a PoW environment, an interesting case study for security researchers comparing PoS vs PoW finality in Move.

### **5.2 Movement Labs**

Movement Labs is building a "Move L2" on Ethereum (M1 and M2).

- **Architectural Insight:** This project attempts to bring the safety of the MoveVM to the liquidity of Ethereum. By running the MoveVM as a rollup on Ethereum, they aim to offer the high throughput and safety of Move while settling on the world's most liquid blockchain.
- **Repository Analysis:** The movement-ecosystem 40 and M1 41 repositories indicate a focus on compatibility. They are building a "Fractal" transpiler to allow Solidity developers to interact with Move.40 This creates a hybrid development environment where Solidity contracts can potentially call Move modules, bridging the gap between the two developer communities.

## **6\. Detailed Data Tables of Financial Projects**

The following tables categorize the identified projects by ecosystem and function, providing direct links to their repositories and key architectural notes. These tables serve as a quick reference for developers and researchers to locate specific codebases.

### **Table 1: Sui Ecosystem Financial Projects**

| Project Name  | Category         | Primary GitHub Repository                                                                                          | SDK/Interface Repo                                                                       | Key Architectural Features                                                                                           |
| :------------ | :--------------- | :----------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------- |
| **Cetus**     | DEX (CLMM)       | [CetusProtocol/cetus-clmm-sui-sdk](https://github.com/CetusProtocol/cetus-clmm-sui-sdk)                            | [cetus-clmm-interface](https://github.com/CetusProtocol/cetus-clmm-interface)            | Concentrated liquidity via NFTs; permissionless pools; separate interface for composability; aggregator logic.       |
| **Turbos**    | DEX (CLMM)       | [turbos-finance/turbos-clmm-sdk](https://github.com/turbos-finance/turbos-clmm-sdk)                                | [turbos-sui-move-interface](https://github.com/turbos-finance/turbos-sui-move-interface) | Jump Crypto backed; open interfaces for vault strategies; institutional ACL features for permissioned pools.         |
| **DeepBook**  | CLOB             | [MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3)                                                  | [sui-move-deepbook](https://github.com/dacadeorg/sui-move-deepbook)                      | Shared object order book; minimal gas costs; public good primitive for routing; supports flash loans.                |
| **Scallop**   | Lending          | [scallop-io/sui-lending-protocol](https://github.com/scallop-io/sui-lending-protocol)                              | [sui-scallop-sdk](https://github.com/scallop-io/sui-scallop-sdk)                         | Separate Query/Builder architecture for efficiency; institutional grade risk models; indexer support.                |
| **Navi**      | Lending/LSDeFi   | [naviprotocol/navi-smart-contracts](https://github.com/naviprotocol/navi-smart-contracts)                          | [navi-sdk](https://github.com/naviprotocol/navi-sdk)                                     | Modular SDK (Lending, Bridge, Aggregator); Flash loan support; leading TVL; transition to modular package structure. |
| **Kriya**     | Derivatives      | [efficacy-finance/kriya-dex-interface](https://github.com/efficacy-finance/kriya-dex-interface)                    | N/A                                                                                      | Spot and perp trading; specific fee scaling logic in Move structs; stable/volatile pool modes.                       |
| **Aftermath** | Aggregator/DEX   | [AftermathFinance/move-amm-public](https://github.com/AftermathFinance/move-amm-public)                            | [aftermath-ts-sdk](https://github.com/AftermathFinance/aftermath-ts-sdk)                 | Multi-asset pools; Smart Order Routing with gas optimization analysis; staking and perp integration.                 |
| **Bucket**    | CDP (Stablecoin) | ([https://github.com/Bucket-Protocol/bucket-protocol-sdk](https://github.com/Bucket-Protocol/bucket-protocol-sdk)) | N/A                                                                                      | Collateralized Debt Position protocol; 0% interest loans against SUI/BTC/ETH; minimum collateral ratio of 110%.      |
| **7k**        | Aggregator       | [7k-ag/7k-sdk-ts](https://github.com/7k-ag/7k-sdk-ts)                                                              | N/A                                                                                      | Aggregation SDK; requires Pyth Oracle peer dependency; supports BluefinX routing.                                    |
| **Hop**       | Aggregator       | \[N/A\]                                                                                                            | N/A                                                                                      | Focus on B2B integration via SDK; revenue sharing model for integrating dApps.                                       |

### **Table 2: Aptos Ecosystem Financial Projects**

| Project Name   | Category            | Primary GitHub Repository                                                                                     | SDK/Interface Repo                                                         | Key Architectural Features                                                                                    |
| :------------- | :------------------ | :------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| **Thala**      | Super App (CDP+DEX) | (https://github.com/ThalaLabs/thalaswap-integration-starter)                                                  | [surf](https://github.com/ThalaLabs/surf)                                  | Weighted pools; Surf library for type-safe Move interactions; stablecoin (MOD); fixed-point math lib.         |
| **Econia**     | CLOB                | [econia-labs/econia](https://github.com/econia-labs/econia)                                                   | N/A                                                                        | Hyper-parallelized order book; atomic matching engine; audited branches; multisig governance integration.     |
| **Liquidswap** | DEX (AMM)           | [pontem-network/liquidswap](https://github.com/pontem-network/liquidswap)                                     | [liquidswap-sdk](https://github.com/pontem-network/liquidswap-sdk)         | First AMM on Aptos; extensively used as a dependency in Move.toml files; stable and uncorrelated curves.      |
| **Aries**      | Margin Trading      | [Aries-Markets/aries-markets-contract-public](https://github.com/Aries-Markets/aries-markets-contract-public) | [aries-tssdk](https://github.com/Aries-Markets/aries-tssdk)                | Integrated money markets; transparent interest rate calculation logic; margin trading on top of AMMs.         |
| **Pancake**    | DEX                 | [pancakeswap/pancake-contracts-move](https://github.com/pancakeswap/pancake-contracts-move)                   | N/A                                                                        | Port of Uniswap V2/MasterChef to Move; includes Farm and IFO modules; highly recognizable architecture.       |
| **Aux**        | DEX (Hybrid)        | [aux-exchange/aux-exchange](https://github.com/aux-exchange/aux-exchange)                                     | N/A                                                                        | Combined AMM and CLOB; offers Go-based SDK for backend integration; sophisticated router.                     |
| **BlueMove**   | NFT Marketplace     |                                                                                                               | [aptos-nft-aggregator](https://github.com/aptos-labs/aptos-nft-aggregator) | Leading NFT market; Aggregator repo contains config for indexing BlueMove events like ListEvent and BuyEvent. |

### **Table 3: Emerging Chains & Infrastructure**

| Project         | Ecosystem | GitHub Repository                                                                           | Description                                                                                    |
| :-------------- | :-------- | :------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------------------------------- |
| **Starcoin**    | Starcoin  | [starcoinorg/starcoin-framework](https://github.com/starcoinorg/starcoin-framework)         | Proof-of-Work blockchain using Move; Layered scaling architecture; implements DaoSpace.        |
| **Movement**    | Movement  | [movementlabsxyz/movement-ecosystem](https://github.com/movementlabsxyz/movement-ecosystem) | L2 for Move on Ethereum; establishing an ecosystem registry and Fractal transpiler.            |
| **Pyth**        | Oracle    | [pyth-network/pyth-crosschain](https://github.com/pyth-network/pyth-crosschain)             | Critical oracle infrastructure used by virtually all derivatives on Sui/Aptos for price feeds. |
| **Switchboard** | Oracle    | [switchboard-xyz/sbv2-aptos](https://github.com/switchboard-xyz/sbv2-aptos)                 | On-demand oracle feeds; integrated into Thala and others; Move native implementation.          |

## **7\. Technical Synthesis and Developer Recommendations**

The analysis of these projects reveals several recurring themes and best practices that define the current state of Move financial engineering. These insights offer a roadmap for new developers entering the space.

### **7.1 The Rise of the SDK Wrapper and Type Safety**

A clear trend observed across both ecosystems is the abstraction of Move complexity. While the smart contracts (Move files) define the logic, the developer experience is dominated by TypeScript SDKs (cetus-clmm-sdk, scallop-sdk, surf). For a researcher or developer, this means that understanding the Move code is necessary for security audits, but building _on top_ of these protocols is largely a TypeScript activity. The repositories frequently contain src folders full of TS code rather than just Move modules.7

Thala's surf library represents the pinnacle of this trend, providing "end-to-end type safety".25 By generating TypeScript types directly from Move ABIs, it prevents mismatch errors where the frontend sends a string when the contract expects a u64. This reduces the cognitive load on developers, who no longer need to constantly reference the Move source code to understand function signatures.

### **7.2 Interoperability via Move.toml and On-Chain Dependencies**

The Move.toml file is the package manifest for Move projects. A crucial insight for finding financial projects is to look for dependencies in this file. For example, a project building a yield aggregator will likely list Liquidswap or PancakeSwap as a dependency in its TOML file.32 This dependency graph creates a "lego" system where protocols like **Umi** or **7k** can import the bytecode of DEXs directly to execute swaps, ensuring atomic composability.

This mechanism is superior to Solidity's interface system because Move allows for the direct import of the _module_, ensuring that the types match exactly. When a developer imports Liquidswap::Router, they are compiling against the exact bytecode deployed on-chain, not just an interface definition that might be outdated.

### **7.3 The Importance of Indexers and Data Services**

Due to the complexity of on-chain state (especially Sui's object model), projects rely heavily on indexers. The **Scallop** repository explicitly mentions a ScallopIndexer module.7 This suggests that querying the blockchain directly via RPC for complex financial data (e.g., "all user loans across all pools") is inefficient. Developers should look for projects that provide open-source indexer implementations (often using GraphQL or specialized APIs) alongside their smart contracts.

BlueMove's integration with the aptos-nft-aggregator further highlights this dependency. The aggregator configures specific event listeners for BlueMove's contracts 36, effectively "listening" for market activity to update off-chain databases. For any data-intensive financial application (like a portfolio tracker or tax tool), building or using an indexer is mandatory.

### **7.4 Security, Audits, and Formal Verification**

The ecosystem places a premium on security, likely a reaction to the high-profile exploits in the EVM space. Repositories like **Econia** maintain specific branches tagged audited.30 When evaluating a project for integration, checking the repository tags for audit markers is a reliable heuristic for project maturity.

Furthermore, the widespread use of the Move Prover is a strong indicator of a project's commitment to safety. The presence of .spec.move files in a repository allows anyone to verify the mathematical proofs of the contract's correctness. This transparency builds trust; a user doesn't just have to trust the developer's code, they can verify the _proof_ that the code behaves as intended. This is a unique value proposition of the Move ecosystem that financial projects are leveraging heavily.

## **8\. Conclusion**

The Move financial ecosystem has matured from experimental primitives into a sophisticated landscape of high-performance applications. **Sui** has successfully carved out a niche for high-frequency trading and central limit order books (DeepBook) by leveraging its object-centric parallelism. **Aptos** has cultivated a suite of integrated "Super Apps" (Thala, Aries) that utilize the language's safety features and global storage model to build complex margin and lending products.

For the user seeking to read and analyze these projects, the GitHub repositories listed above represent the "source of truth." They reveal not just the financial logic (interest rate models, swap math) but also the emerging standards of Move development: heavy use of formal verification, modular interfaces for third-party integration, and robust TypeScript SDKs to bridge the gap between the blockchain and the frontend. The data suggests that while the "MoveVM wars" between Sui and Aptos continue, the ultimate winner is the developer, who now has access to a library of open-source financial building blocks that are safer, faster, and more composable than their EVM counterparts. The future of DeFi on Move is likely to see further convergence of these tools, with cross-chain aggregators and unified SDKs blurring the lines between the individual chains.

#### **Works cited**

1. MystenLabs/awesome-move: Code and content from the Move community. \- GitHub, accessed November 25, 2025, [https://github.com/MystenLabs/awesome-move](https://github.com/MystenLabs/awesome-move)
2. diem/move: Home of the Move programming language \- GitHub, accessed November 25, 2025, [https://github.com/diem/move](https://github.com/diem/move)
3. aptos-labs/move \- GitHub, accessed November 25, 2025, [https://github.com/aptos-labs/move](https://github.com/aptos-labs/move)
4. GitHub \- move-cn/awesome-sui, accessed November 25, 2025, [https://github.com/move-cn/awesome-sui](https://github.com/move-cn/awesome-sui)
5. sui · GitHub Topics, accessed November 25, 2025, [https://github.com/topics/sui](https://github.com/topics/sui)
6. CetusProtocol · GitHub, accessed November 25, 2025, [https://github.com/CetusProtocol](https://github.com/CetusProtocol)
7. sui-foundation/awesome-sui: A curated list of Move code and resources. \- GitHub, accessed November 25, 2025, [https://github.com/sui-foundation/awesome-sui](https://github.com/sui-foundation/awesome-sui)
8. CetusProtocol/cetus-clmm-interface \- GitHub, accessed November 25, 2025, [https://github.com/CetusProtocol/cetus-clmm-interface](https://github.com/CetusProtocol/cetus-clmm-interface)
9. CetusProtocol/aggregator \- GitHub, accessed November 25, 2025, [https://github.com/CetusProtocol/aggregator](https://github.com/CetusProtocol/aggregator)
10. Turbos Finance \- Sui Directory, accessed November 25, 2025, [https://sui.directory/project/turbos-finance/](https://sui.directory/project/turbos-finance/)
11. Turbos Finance by Turbos | QuickNode, accessed November 25, 2025, [https://www.quicknode.com/builders-guide/tools/turbos-finance?category=defi-tools](https://www.quicknode.com/builders-guide/tools/turbos-finance?category=defi-tools)
12. turbos-finance/turbos-sui-move-interface \- GitHub, accessed November 25, 2025, [https://github.com/turbos-finance/turbos-sui-move-interface](https://github.com/turbos-finance/turbos-sui-move-interface)
13. turbos-finance \- GitHub, accessed November 25, 2025, [https://github.com/turbos-finance](https://github.com/turbos-finance)
14. turbos · GitHub Topics, accessed November 25, 2025, [https://github.com/topics/turbos](https://github.com/topics/turbos)
15. Aftermath \- GitHub, accessed November 25, 2025, [https://github.com/Aftermathfinance](https://github.com/Aftermathfinance)
16. MystenLabs/deepbookv3: Deepbook V3 \- GitHub, accessed November 25, 2025, [https://github.com/MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3)
17. dacadeorg/sui-move-deepbook \- GitHub, accessed November 25, 2025, [https://github.com/dacadeorg/sui-move-deepbook](https://github.com/dacadeorg/sui-move-deepbook)
18. DeepBookV3 \- Sui Documentation, accessed November 25, 2025, [https://docs.sui.io/standards/deepbook](https://docs.sui.io/standards/deepbook)
19. efficacy-finance/kriya-dex-interface \- GitHub, accessed November 25, 2025, [https://github.com/efficacy-finance/kriya-dex-interface](https://github.com/efficacy-finance/kriya-dex-interface)
20. Scallop Protocol (Smart Contract): Program Info \- HackenProof, accessed November 25, 2025, [https://hackenproof.com/programs/scallop-protocol-smart-contract](https://hackenproof.com/programs/scallop-protocol-smart-contract)
21. scallop-io/sui-lending-protocol: An over collateralized lending protocol on SUI network \- GitHub, accessed November 25, 2025, [https://github.com/scallop-io/sui-lending-protocol](https://github.com/scallop-io/sui-lending-protocol)
22. Navi Protocol · GitHub, accessed November 25, 2025, [https://github.com/naviprotocol](https://github.com/naviprotocol)
23. ThalaLabs/thalaswap-integration-starter \- GitHub, accessed November 25, 2025, [https://github.com/ThalaLabs/thalaswap-integration-starter](https://github.com/ThalaLabs/thalaswap-integration-starter)
24. ThalaLabs/integrate-thala \- GitHub, accessed November 25, 2025, [https://github.com/ThalaLabs/integrate-thala](https://github.com/ThalaLabs/integrate-thala)
25. ThalaLabs/surf: Type-Safe TypeScript Interfaces & React Hooks for Aptos. \- GitHub, accessed November 25, 2025, [https://github.com/ThalaLabs/surf](https://github.com/ThalaLabs/surf)
26. Thala Labs \- GitHub, accessed November 25, 2025, [https://github.com/thalalabs](https://github.com/thalalabs)
27. Aries Markets · GitHub, accessed November 25, 2025, [https://github.com/Aries-Markets](https://github.com/Aries-Markets)
28. Aries-Markets/aries-tssdk \- GitHub, accessed November 25, 2025, [https://github.com/Aries-Markets/aries-tssdk](https://github.com/Aries-Markets/aries-tssdk)
29. econia-labs/econia: Hyper-parallelized on-chain order book for the Aptos blockchain \- GitHub, accessed November 25, 2025, [https://github.com/econia-labs/econia](https://github.com/econia-labs/econia)
30. \[Multisig\]\[CLI\]\[Compiler\] Different CLI versions generate different multisig v2 transaction proposals · Issue \#10934 · aptos-labs/aptos-core \- GitHub, accessed November 25, 2025, [https://github.com/aptos-labs/aptos-core/issues/10934](https://github.com/aptos-labs/aptos-core/issues/10934)
31. BlockEdenHQ/awesome-aptos: A curated list of code and ... \- GitHub, accessed November 25, 2025, [https://github.com/BlockEdenHQ/awesome-aptos](https://github.com/BlockEdenHQ/awesome-aptos)
32. Code Example \- Depend on Third-Party Smart Contracts \- Aptos Learn, accessed November 25, 2025, [https://learn.aptoslabs.com/en/code-examples/third-party-contract-dependency](https://learn.aptoslabs.com/en/code-examples/third-party-contract-dependency)
33. Smart Contracts (Aptos) \- PancakeSwap Developer, accessed November 25, 2025, [https://developer.pancakeswap.finance/contracts-aptos](https://developer.pancakeswap.finance/contracts-aptos)
34. pancakeswap/pancake-smart-contracts \- GitHub, accessed November 25, 2025, [https://github.com/pancakeswap/pancake-smart-contracts](https://github.com/pancakeswap/pancake-smart-contracts)
35. aux-exchange/aux-exchange: https://aux.exchange \- GitHub, accessed November 25, 2025, [https://github.com/aux-exchange/aux-exchange](https://github.com/aux-exchange/aux-exchange)
36. Bluemove \- Aptos Documentation, accessed November 25, 2025, [https://aptos.dev/build/indexer/nft-aggregator/marketplaces/bluemove](https://aptos.dev/build/indexer/nft-aggregator/marketplaces/bluemove)
37. aptos-labs/aptos-nft-aggregator \- GitHub, accessed November 25, 2025, [https://github.com/aptos-labs/aptos-nft-aggregator](https://github.com/aptos-labs/aptos-nft-aggregator)
38. starcoinorg/starcoin-framework: The Starcoin Move framework \- GitHub, accessed November 25, 2025, [https://github.com/starcoinorg/starcoin-framework](https://github.com/starcoinorg/starcoin-framework)
39. starcoinorg/starcoin: Starcoin \- A Move smart contract blockchain network that scales by layering \- GitHub, accessed November 25, 2025, [https://github.com/starcoinorg/starcoin](https://github.com/starcoinorg/starcoin)
40. movementlabsxyz/movement-ecosystem \- GitHub, accessed November 25, 2025, [https://github.com/movementlabsxyz/movement-ecosystem](https://github.com/movementlabsxyz/movement-ecosystem)
41. movementlabsxyz/M1: An L1 for Move VM built on Avalanche. \- GitHub, accessed November 25, 2025, [https://github.com/movementlabsxyz/M1](https://github.com/movementlabsxyz/M1)
