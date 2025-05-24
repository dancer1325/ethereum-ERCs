---
eip: 20
title: Token Standard
author: Fabian Vogelsteller <fabian@ethereum.org>, Vitalik Buterin <vitalik.buterin@ethereum.org>
type: Standards Track
category: ERC
status: Final
created: 2015-11-19
---

## Simple Summary

* 💡standard interface -- for -- tokens💡

## Abstract

* enable
  * implementing standard API -- for -- tokens | smart contracts

* provide
  * basic functionality / 
    * transfer tokens
    * enable tokens -- to be -- approved
      * -> they can be spent -- by -- ANOTHER on-chain third party

## Motivation

* enable tokens | Ethereum -- to be -- re-used by OTHER applications
  * _Example:_ wallets, decentralized exchanges

## Specification

## Token
### Methods

* requirements
  * Solidity v`0.4.17`+
  * Callers MUST handle `false` -- from -- `returns (bool success)`
    * == NOT assume that `false` is NEVER returned

#### `.name()`

* returns the name of the token
* OPTIONAL
* uses
  * improve usability

``` js
function name() public view returns (string)
```

#### `.symbol()`

* returns the symbol of the token
* OPTIONAL
* uses
  * improve usability

``` js
function symbol() public view returns (string)
```

#### decimals

* returns the number of decimals / token uses
  * _Example:_ `8` -> token amount / `100000000` == user representation
* OPTIONAL
* uses
  * improve usability,

``` js
function decimals() public view returns (uint8)
```

#### `.totalSupply()`

* returns the total token supply

``` js
function totalSupply() public view returns (uint256)
```

#### `.balanceOf(address)`

* returns the `address`'s account balance

``` js
function balanceOf(address _owner) public view returns (uint256 balance)
```

#### `.transfer(address, uint256)`

* `uint256` tokens -- are transferred to -- `address` /
  * ⚠️MUST fire the `Transfer` event⚠️
  * ⚠️if the message caller's account balance does NOT have enough tokens -> `throw` ⚠️
  * `uint256` = 0 is VALID

``` js
function transfer(address _to, uint256 _value) public returns (bool success)
```

#### `.transferFrom(address _from, address _to, uint256 _value)`

* `uint256` tokens -- are transferred `_from` to -- `_to` 
  * ⚠️MUST fire the `Transfer` event⚠️
  * `uint256` = 0 is VALID
  * if `_from` does NOT authorize ->  `throw` 

* uses
  * withdraw workflow | smart contract
  * charge fees in sub-currencies (TODO: ❓)

``` js
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
```

#### `.approve(address _spender, uint256 _value)`

* enable
  * `_spender` to withdraw -- from -- your account TILL `_value` amount

* recommendations
  * clients SHOULD create UI / | BEFORE ALL, they set FIRSTLY allowance == `0` 
    * Reason: 🧠prevent attack vectors
      * [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/) 
      * discussed [here](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729)🧠

``` js
function approve(address _spender, uint256 _value) public returns (bool success)
```

#### `.allowance(address _owner, address _spender)`

* returns the amount / `_spender` is STILL ALLOWED, FROM `_owner`, -- TO -- withdraw 

``` js
function allowance(address _owner, address _spender) public view returns (uint256 remaining)
```

### Events

#### `.transfer(address indexed _from, address indexed _to, uint256 _value)`

* uses
  * | transfer tokens
  * token contract / creates NEW tokens -> SHOULD trigger a Transfer event / `_from` == `0x0`

``` js
event Transfer(address indexed _from, address indexed _to, uint256 _value)
```

#### `.Approval(address indexed _owner, address indexed _spender, uint256 _value)`

* uses
  * | ANY SUCCESSFUL call -- to -- `approve(address _spender, uint256 _value)` 

``` js
event Approval(address indexed _owner, address indexed _spender, uint256 _value)
```

## Implementation

* EXIST ALREADY ERC20-compliant
  * tokens / deployed | Ethereum network
  * implementations / have DIFFERENT trade-offs
    * _Example:_ gas saving -- to -- improved security

#### Example implementations are available at
- [OpenZeppelin implementation](../assets/eip-20/OpenZeppelin-ERC20.sol)
- [ConsenSys implementation](../assets/eip-20/Consensys-EIP20.sol)

## History

- Original proposal from Vitalik Buterin: https://github.com/ethereum/wiki/wiki/Standardized_Contract_APIs/499c882f3ec123537fc2fccd57eaa29e6032fe4a
  - ❌NOT AVAILABLE ❌
- Reddit discussion: https://www.reddit.com/r/ethereum/comments/3n8fkn/lets_talk_about_the_coin_standard/
- Original Issue #20: https://github.com/ethereum/EIPs/issues/20

