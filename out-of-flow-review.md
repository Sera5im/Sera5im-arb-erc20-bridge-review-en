# Out-of-Flow Review

This review covers the config, init, routing-mutation, and upgrade surface around the deposit/withdraw lifecycle. It is not about the normal asset path itself, but about the surrounding admin/config assumptions that keep that path valid.

## 1. L1GatewayRouter.initialize(...)

```solidity
function initialize(
    address _owner,
    address _defaultGateway,
    address,
    address _counterpartGateway,
    address _inbox
) public {
    GatewayRouter._initialize(_counterpartGateway, address(0), _defaultGateway);
    owner = _owner;
    WhitelistConsumer.whitelist = address(0);
    inbox = _inbox;
}
```

What it does:

- initializes the L1 router
- fixes `owner`, `defaultGateway`, `counterpartGateway`, and `inbox`

Invariants:

- router initialization must fix the correct `owner`
- router initialization must fix the correct `counterpartGateway` and `defaultGateway`
- router initialization must fix the correct `inbox`

## 2. L1GatewayRouter.setDefaultGateway(...)

```solidity
function setDefaultGateway(
    address newL1DefaultGateway,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    uint256 _maxSubmissionCost
) external payable virtual onlyOwner returns (uint256) {
    return
        _setDefaultGateway(
            newL1DefaultGateway,
            _maxGas,
            _gasPriceBid,
            _maxSubmissionCost,
            msg.value
        );
}
```

What it does:

- changes the default routing path
- initiates the corresponding L2 router update

Invariants:

- default routing mutation must be privileged
- default routing mutation must not remain only on L1

## 3. L1GatewayRouter.setOwner(...)

```solidity
function setOwner(address newOwner) external onlyOwner {
    require(newOwner != address(0), "INVALID_OWNER");
    owner = newOwner;
}
```

What it does:

- transfers privileged authority over the L1 router

Invariants:

- owner rotation must be privileged
- owner must not be moved to `address(0)`

## 4. L1GatewayRouter.setGateway(...)

```solidity
function setGateway(
    address _gateway,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    uint256 _maxSubmissionCost,
    address _creditBackAddress
) public payable virtual override returns (uint256) {
    return
        _setGatewayWithCreditBack(
            _gateway,
            _maxGas,
            _gasPriceBid,
            _maxSubmissionCost,
            _creditBackAddress,
            msg.value
        );
}
```

What it does:

- registers a gateway for a token path
- forwards the mutation into the downstream routing layer

Invariants:

- the gateway registration path must go through the intended registration logic
- a single-token gateway mutation must not bypass downstream correspondence checks

## 5. L1GatewayRouter.setGateways(...)

```solidity
function setGateways(
    address[] memory _token,
    address[] memory _gateway,
    uint256 _maxGas,
    uint256 _gasPriceBid,
    uint256 _maxSubmissionCost
) external payable virtual onlyOwner returns (uint256) {
    return
        _setGateways(
            _token,
            _gateway,
            _maxGas,
            _gasPriceBid,
            _maxSubmissionCost,
            msg.sender,
            msg.value
        );
}
```

What it does:

- batch-updates router-level gateway mappings
- sends the corresponding update to L2

Invariants:

- batch routing mutation must be privileged
- batch update must not bypass token/gateway correspondence checks

## 6. L2GatewayRouter.initialize(...)

```solidity
function initialize(
    address _counterpartGateway,
    address _defaultGateway
) public {
    GatewayRouter._initialize(_counterpartGateway, address(1), _defaultGateway);
}
```

What it does:

- initializes the L2 router
- fixes `counterpartGateway` and `defaultGateway`

Invariants:

- L2 router initialization must fix the counterpart routing model

## 7. L2GatewayRouter.setGateway(...)

```solidity
function setGateway(
    address[] memory _token,
    address[] memory _gateway
) external override onlyCounterpartGateway {
    require(_token.length == _gateway.length, "WRONG_LENGTH");

    for (uint256 i = 0; i < _token.length; i++) {
        l1TokenToGateway[_token[i]] = _gateway[i];
        emit GatewaySet(_token[i], _gateway[i]);
    }
}
```

What it does:

- accepts a counterpart-driven batch routing mutation on L2
- updates the L2 view of the token-to-gateway mapping

Invariants:

- L2 routing mutation must come only from `counterpartGateway`
- batch correspondence between the token list and the gateway list must remain aligned

## 8. L2GatewayRouter.setDefaultGateway(...)

```solidity
function setDefaultGateway(address newL2DefaultGateway)
    external
    override
    onlyCounterpartGateway
{
    defaultGateway = newL2DefaultGateway;
    emit DefaultGatewayUpdated(newL2DefaultGateway);
}
```

What it does:

- changes the destination-side default routing branch on L2

Invariants:

- the L2 default routing mutation must be counterpart-gated
- the default routing mutation must explicitly change the stored destination-side route

## 9. L1ArbitrumGateway.postUpgradeInit()

```solidity
function postUpgradeInit() external {
    address proxyAdmin = ProxyUtil.getProxyAdmin();
    require(msg.sender == proxyAdmin, "NOT_FROM_ADMIN");
}
```

What it does:

- acts as a post-upgrade hook for the proxy-controlled L1 gateway

Invariants:

- the post-upgrade hook must be callable only by the proxy admin

## 10. L1ArbitrumGateway._initialize(...)

```solidity
function _initialize(
    address _l2Counterpart,
    address _router,
    address _inbox
) internal {
    TokenGateway._initialize(_l2Counterpart, _router);
    require(_router != address(0), "BAD_ROUTER");
    require(_inbox != address(0), "BAD_INBOX");
    inbox = _inbox;
}
```

What it does:

- initializes the L1 gateway
- fixes counterpart, router, and inbox

Invariants:

- L1 gateway initialization must not start without a valid router
- L1 gateway initialization must not start without a valid inbox

## 11. L2ArbitrumGateway.postUpgradeInit()

```solidity
function postUpgradeInit() external {
    address proxyAdmin = ProxyUtil.getProxyAdmin();
    require(msg.sender == proxyAdmin, "NOT_FROM_ADMIN");
}
```

What it does:

- acts as a post-upgrade hook for the proxy-controlled L2 gateway

Invariants:

- the post-upgrade hook must be callable only by the proxy admin

## 12. L2ArbitrumGateway._initialize(...)

```solidity
function _initialize(address _l1Counterpart, address _router) internal override {
    TokenGateway._initialize(_l1Counterpart, _router);
    require(_router != address(0), "BAD_ROUTER");
}
```

What it does:

- initializes the L2 gateway
- fixes the L1 counterpart and router

Invariants:

- L2 gateway initialization must not start without a valid router
