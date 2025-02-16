# Atomic Swaps with Local Verification

This proposal introduces a mechanism for achieving asset interoperability between Ethereum L2s and beyond using Atomic Swaps. The approach does not require a cross-chain messaging protocol, does not introduce new trust assumptions, and remains open and permissionless for any network to participate. It leverages an enhanced version of HTLCs (PreHTLC) in conjunction with recent advancements in local verification techniques, such as running a light client in the browser (e.g., Helios), to achieve two primary objectives:

- **Trustless** – Users must be able to transfer assets between chains without relinquishing control of their funds.  
- **Permissionless** – Networks should be able to integrate with the protocol without requiring approvals or external gatekeeping mechanisms.  

## Intents with PreHTLC

PreHTLC introduces three key improvements over [traditional HTLCs](https://en.bitcoin.it/wiki/Hash_Time_Locked_Contracts):  

- **Delegated secret management** – The Solver is responsible for managing secrets, reducing operational complexity for users.  
- **Multiple Solver selection** – Users can designate multiple Solvers to fulfill the transaction, mitigating Solver liveness risks.  
- **Incentive alignment** – A reward/slash mechanism ensures that Solvers are economically incentivized to execute transactions promptly.  

This document does not exhaustively cover all edge cases and [implementation details](https://docs.train.tech) but instead provides a high-level overview of the core process.  

## Protocol Walkthrough  

The following outlines the end-to-end execution when Alice transfers 1 ETH from Starknet to Optimism.  

1. **Transaction initiation:**  
   - Alice accesses the Bridge dApp, connects her wallet, and selects the Starknet → Optimism transfer route.  
   - The dApp queries the Solver Discovery contract to retrieve a list of available Solvers supporting the route.  
   - Alice submits a **commit transaction** on Starknet, locking 1 ETH and specifying the set of Solvers.  

At this point, the locked funds are subject to one of two conditions:  

- A `Hashlock` is added once the auction winner is determined.  
- If no Solver successfully claims the transaction before the `Timelock` expires, Alice can reclaim her funds.  

2. **Solver selection via Dutch Auction:**  
   - A Dutch auction is initiated, where Solvers (e.g., Bob and John) compete by progressively lowering their bid price.  
   - Bob wins the auction, generates a secret `S`, computes `Hashlock = HASH(S)`, and **locks 1 ETH for Alice on Optimism**.  

Once Bob has locked the corresponding amount on the destination chain, Alice's funds can be securely released upon disclosure of the secret.  

3. **Local verification of the destination transaction:**  
   - The dApp performs **local verification** by monitoring Bob’s transaction on Optimism.  
   - If a light client is available, it is utilized for validation. Otherwise, the dApp queries multiple RPC endpoints or a user-specified RPC provider.  
   - Upon confirming the transaction, the `Hashlock` is retrieved.  

4. **Finalization and settlement:**  
   - Once Alice verifies the `Hashlock`, she signs a transaction that assigns the `Hashlock` to her previously committed funds and sets the `receiver` to Bob.  

At this stage, both chains (Starknet and Optimism) hold funds locked under the same `Hashlock`. The only remaining step is secret revelation.  

5. **Secret disclosure and fund release:**  
   - Bob monitors Starknet for confirmation that the commitment is locked in his favor with the expected `Hashlock`.  
   - Upon verification, Bob **reveals the secret** `S` on both chains, proving `HASH(S) = Hashlock`, thereby unlocking and claiming his funds while simultaneously releasing Alice's funds.  

## Conclusion  

By integrating an intent/solver-based framework with PreHTLCs and local verification, this protocol achieves:  

- **Sub-30-second settlement times** for cross-chain transfers.  
- **Non-custodial security** – neither users’ nor Solvers’ funds are ever controlled by an intermediary.  
- **Open network integration** – any blockchain network can be onboarded without requiring explicit permission.  

This approach ensures a scalable and trustless mechanism for cross-chain asset transfers without relying on third-party validators or external security mechanisms.
