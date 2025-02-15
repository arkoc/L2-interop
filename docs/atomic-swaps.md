# Atomic Swaps with Local Verification

This proposal introduces an approach to solving asset interoperability between chains using Atomic Swaps. It does not require a standardized cross-chain messaging protocol, does not introduce new trust assumptions, and is open and permissionless for any network to join. [This protocol](https://ethereum-magicians.org/t/cross-chain-asset-bridging-with-atomic-swaps-and-local-verification/22444) leverages an improved version of HTLCs (called PreHTLC) alongside recent developments in local verification methods, such as running a light client in the browser (e.g. Helios).

## Fundamentals

- **Trustless** - Users should be able to move assets between chains without ever losing control of their funds.
- **Permissionless** - Networks should be able to join the protocol without approvals or gatekeepers.

## Intents with PreHTLC

PreHTLC introduces three major improvements over classic HTLCs:

- Secret management is delegated to the Solver
- Users can select multiple Solvers to fulfill the transaction, addressing the liveness issue
- Introduces a reward/slash mechanism to incentivize Solvers to act in a timely manner

## Wolkthrough

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
- Users' and solvers' funds are **never taken** into custody
- **Any network** can be added, **anyone** can become a solver
