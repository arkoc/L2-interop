# Atomic Swaps with Local Verification

This document proposes an approach on solving asset interop beetween chains. It doesn't require standardatized cross-chain messaging protocol, does not introduce new trust-assumptions and it's open and permisionless for any networks to join.

The aproach is also described in [Ethereum magicians](https://ethereum-magicians.org/t/cross-chain-asset-bridging-with-atomic-swaps-and-local-verification/22444)

## Fundamentals

This aproach prioritized designing the protocol around 2 foundamental pillars:

- Trustless
Users should be able to move assets between chains without ever losing control of their funds.
- Permissionless
Networks should be able to join the protocol without approvals or gatekeepers.

## Intens with PreHTLC and Local Verification

1. Alice opens the [Bridge dApp](https://docs.train.tech/protocol/dApp), connects her wallet, and selects the route from Starknet to Optimism.  
2. The dApp queries the [Solver Discovery contract](https://docs.train.tech/protocol/discovery) and retrieves addresses of all Solvers that support that route.
3. Alice creates a **commit transaction with 1 ETH in Starknet** and passes the Solver addresses.
At this stage, the locked funds can either be updated with a \`Hashlock\` after the auction winner is decided or refunded once the \`Timelock\` expires.

4. A [Dutch Auction](https://docs.train.tech/protocol/auction) starts — Bob and John (Solvers) compete on price until one of them wins.  
5. Bob wins the auction generates secret `S`, calculates `Haslock = HASH(S)` and **locks 1 ETH on Optimism**.
Now, the funds are secured for Alice and can be released once Bob reveals the secret.
6. The dApp begins **local verification** of the Optimism network by tracking Bob’s transaction. If a [Light Client](https://docs.train.tech/protocol/dApp#observing-the-destination-network) is available, it is used; otherwise, the dApp retrieves data from multiple RPC endpoints or uses the node provider specified by the user. Once detected, it captures the \`Hashlock\`.
7. Once the \`Hashlock\` is verified, Alice signs a message that adds the \`Hashlock\` and sets the \`receiver\` of her previously committed funds to Bob.

At this point, there are two locks (on Starknet and Optimism) tied to the same \`Hashlock\`. The funds can be unlocked by providing a secret \`S\` that satisfies \`HASH(S) = Hashlock\`.
8. Bob monitors the source Starknet network. Once he confirms that the commitment is locked for him and secured with his \`Hashlock\`, he **reveals the secret** \`S\` on both networks, claiming his funds and releasing Alice's funds.

## Conclusion

- Process takes **<30 seconds**
- users' and solvers' funds are **never taken** into custody
- **any network** can be added, **anyone** can become a solver

This is a bit different route compared to collective effort of standardatizing cross-chain messaging, but it introduces major improvements as this is currently the only aproach that can ensure a neutral, trustless asset interop beetwin L2s, it can scale beyond Ethereum and is completly permissionless for new networks and solvers to join.
