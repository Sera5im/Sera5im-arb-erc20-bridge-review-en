# Out-of-Flow Review

This review covers the config, init, routing-mutation, and upgrade surface around the deposit/withdraw lifecycle. It is not about the normal asset path itself, but about the surrounding admin/config assumptions that keep that path valid.

## 1. L1GatewayRouter.initialize(...)

What it does:

- initializes the L1 router
- fixes `owner`, `defaultGateway`, `counterpartGateway`, and `inbox`

Invariants:

- router initialization must fix the correct `owner`
- router initialization must fix the correct `counterpartGateway` and `defaultGateway`
- router initialization must fix the correct `inbox`

## 2. L1GatewayRouter.setDefaultGateway(...)

What it does:

- changes the default routing path
- initiates the corresponding L2 router update

Invariants:

- default routing mutation must be privileged
- default routing mutation must not remain only on L1

## 3. L1GatewayRouter.setOwner(...)

What it does:

- transfers privileged authority over the L1 router

Invariants:

- owner rotation must be privileged
- owner must not be moved to `address(0)`

## 4. L1GatewayRouter.setGateway(...)

What it does:

- registers a gateway for a token path
- forwards the mutation into the downstream routing layer

Invariants:

- the gateway registration path must go through the intended registration logic
- a single-token gateway mutation must not bypass downstream correspondence checks

## 5. L1GatewayRouter.setGateways(...)

What it does:

- batch-updates router-level gateway mappings
- sends the corresponding update to L2

Invariants:

- batch routing mutation must be privileged
- batch update must not bypass token/gateway correspondence checks

## 6. L2GatewayRouter.initialize(...)

What it does:

- initializes the L2 router
- fixes `counterpartGateway` and `defaultGateway`

Invariants:

- L2 router initialization must fix the counterpart routing model

## 7. L2GatewayRouter.setGateway(...)

What it does:

- accepts a counterpart-driven batch routing mutation on L2
- updates the L2 view of the token-to-gateway mapping

Invariants:

- L2 routing mutation must come only from `counterpartGateway`
- batch correspondence between the token list and the gateway list must remain aligned

## 8. L2GatewayRouter.setDefaultGateway(...)

What it does:

- changes the destination-side default routing branch on L2

Invariants:

- the L2 default routing mutation must be counterpart-gated
- the default routing mutation must explicitly change the stored destination-side route

## 9. L1ArbitrumGateway.postUpgradeInit()

What it does:

- acts as a post-upgrade hook for the proxy-controlled L1 gateway

Invariants:

- the post-upgrade hook must be callable only by the proxy admin

## 10. L1ArbitrumGateway._initialize(...)

What it does:

- initializes the L1 gateway
- fixes counterpart, router, and inbox

Invariants:

- L1 gateway initialization must not start without a valid router
- L1 gateway initialization must not start without a valid inbox

## 11. L2ArbitrumGateway.postUpgradeInit()

What it does:

- acts as a post-upgrade hook for the proxy-controlled L2 gateway

Invariants:

- the post-upgrade hook must be callable only by the proxy admin

## 12. L2ArbitrumGateway._initialize(...)

What it does:

- initializes the L2 gateway
- fixes the L1 counterpart and router

Invariants:

- L2 gateway initialization must not start without a valid router
