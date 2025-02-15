# Atomic Swaps with Local Verification

This document proposes an approach on solving asset interop beetween chains with Atomic Swaps. It doesn't require standardatized cross-chain messaging protocol, does not introduce new trust-assumptions and it's open and permisionless for any networks to join. [This protocol](https://ethereum-magicians.org/t/cross-chain-asset-bridging-with-atomic-swaps-and-local-verification/22444) leverages the improved version of HTLCs (called PreHTLC) along with recent develpomnents in Local Verificaiton methonds like running a Light Client in the browser. In essence this protocol introduces a mechanisms for Solving user's intents with Atomic Swaps.

## Fundamentals

This aproach prioritized designing the protocol around 2 foundamental pillars:

- **Trustless** - Users should be able to move assets between chains without ever losing control of their funds.
- **Permissionless** - Networks should be able to join the protocol without approvals or gatekeepers.

## Intents with PreHTLC and Local Verification

Let's walk through how the [PreHTLC protocol](https://docs.train.tech/protocol/atomic-swaps-prehtlc) works end-to-end when Alice wants to transfer 1 ETH from Starknet to Optimism.

- Alice opens the Bridge dApp, connects her wallet, and selects the route from Starknet to Optimism.  
- The dApp queries the Solver Discovery contract and retrieves addresses of all Solvers that support that route.
- Alice creates a **commit transaction with 1 ETH in Starknet** and passes the Solver addresses.

At this stage, the locked funds can either be updated with a `Hashlock` after the auction winner is decided or refunded once the `Timelock` expires.

- A Dutch Auction starts — Bob and John (Solvers) compete on price until one of them wins.  
- Bob wins the auction generates secret `S`, calculates `Haslock = HASH(S)` and **locks 1 ETH on Optimism**.

Now, the funds are secured for Alice and can be released once Bob reveals the secret.

- The dApp begins **local verification** of the Optimism network by tracking Bob’s transaction. If a Light Client is available, it is used; otherwise, the dApp retrieves data from multiple RPC endpoints or uses the node provider specified by the user. Once detected, it captures the `Hashlock`.
- Once the `Hashlock` is verified, Alice signs a message that adds the `Hashlock` and sets the `receiver` of her previously committed funds to Bob.

At this point, there are two locks (on Starknet and Optimism) tied to the same `Hashlock`. The funds can be unlocked by providing a secret `S` that satisfies `HASH(S) = Hashlock`.

- Bob monitors the source Starknet network. Once he confirms that the commitment is locked for him and secured with his `Hashlock`, he **reveals the secret** `S` on both networks, claiming his funds and releasing Alice's funds.

## Conclusion

- Process takes **<30 seconds**
- users' and solvers' funds are **never taken** into custody
- **any network** can be added, **anyone** can become a solver

