---
id: q1y7wa01neow17n9dnr1qb4
title: ERC20
desc: ""
updated: 1669183345602
created: 1669182413235
---

## ERC20 Sepc

ERC20 is the standard for fungible tokens on Ethereum.

```solidity
pragma solidity ^0.6.0;

interface IERC20 {

    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function allowance(address owner, address spender) external view returns (uint256);

    function transfer(address recipient, uint256 amount) external returns (bool);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);


    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

## Transfer vs TransferFrom

The `transfer` methods is 2 party transfer:

- Sender -> `transfer(receiver_address, amount)`

The `transferFrom` method is 3 part transfer:

- Sender -> `approve(exchange_address, amount)`
- Buyer -> trade on the exchange
- Exchange -> `transferFrom(sender_address, buyer_address, amount)`
