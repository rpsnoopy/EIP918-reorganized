---
eip: 918
title: ERC-918 Minable Token Standard
author: Jay Logelin <jlogelin@fas.harvard.edu>, Infernal_toast <admin@0xbitcoin.org>, Michael Seiler <mgs33@cornell.edu>, Brandon Grill <bg2655@columbia.edu>, Rick Park <ceo@xbdao.io>
type: Standards Track
category: ERC
status: Draft
created: 2018-10-23
---

## Simple Summary

A specification for a standardized Minable Token that uses a Proof of Work algorithm for distribution.


## Abstract

he following standard allows for the implementation of a standard API for tokens within smart contracts when including a POW (Proof of Work) mining distribution facility.
In this kind of token contract, tokens are locked within a token smart contract and slowly dispensed by means of a mint() function which acts like a POW faucet when the user submit a valid solution of some Proof of Work algorithm. The tokens are dispensed in chunks formed by some tokens (reward).


## Motivation

The rationale for this model is that this approach can both minimize the gas fees paid by miners in order to obtain tokens and precisely control the distribution rate.
The standardization of the API will allow the development of standardized CPU and GPU token mining software, token mining pools and other external tools in the token mining ecosystem.
This standard is intended to be integrable in any token smart contract which eventually implements any other EIP standard for the non-mining-related functions, like ERC20, ERC223, ERC721, etc.
It can be here mentioned that token distribution via POW is considered very interesting in order to minimize the investor's risks exposure related to eventual illicit behavior of the 'human actors' who launch and manage the distribution, like those implementing ICO's (Initial Coin Offer) and derivatives. The POW model can be totally realized by means of a smart contract, excluding human interferences.


## Specification

### Mandatory methods

**NOTES**:
 - The following specifications use syntax from Solidity `0.4.25` (or above)
 - Callers MUST handle `false` from `returns (bool success)`.  Callers MUST NOT assume that `false` is never returned!



#### challengeNumber

Returns the current challengeNumber, i.e. a byte32 number to be included (with other elements, see later) in the POW algorithm input in order to synthetize a valid solution. It is expected that a new challengeNumber is generated after that the valid solution has been found and the reward tokens have been assigned.

``` solidity
function challengeNumber() view public returns (bytes32)
```

NOTES: in a common implementation, challengeNumber is calculated starting from some immutable data, like elements derived from some past ethereum blocks.



#### difficulty

Returns the current difficulty, i.e. a number useful to estimate (by means of some known algorithm) the mean time required to find a valid POW solution. It is expected that the difficulty varies if the smart contract controls the mean time between valid solutions by means of some control loop.

``` solidity 
function difficulty() view public returns (uint256)
```

NOTES: in a common implementation, difficulty varies when computational power is added/subtracted to the network, in order to maintain stable the mean time between valid solutions found.



#### decimals

Returns the number of decimals the token uses - e.g. `8`, means to divide the token amount by `100000000` to get its user representation.

OPTIONAL - This method can be used to improve usability,
but interfaces and other contracts MUST NOT expect these values to be present.

``` js
function decimals() public view returns (uint8)
```


#### totalSupply

Returns the total token supply.

``` js
function totalSupply() public view returns (uint256)
```



#### balanceOf

Returns the account balance of another account with address `_owner`.

``` js
function balanceOf(address _owner) public view returns (uint256 balance)
```



#### transfer

Transfers `_value` amount of tokens to address `_to`, and MUST fire the `Transfer` event.
The function SHOULD `throw` if the `_from` account balance does not have enough tokens to spend.

*Note* Transfers of 0 values MUST be treated as normal transfers and fire the `Transfer` event.

``` js
function transfer(address _to, uint256 _value) public returns (bool success)
```



#### transferFrom

Transfers `_value` amount of tokens from address `_from` to address `_to`, and MUST fire the `Transfer` event.

The `transferFrom` method is used for a withdraw workflow, allowing contracts to transfer tokens on your behalf.
This can be used for example to allow a contract to transfer tokens on your behalf and/or to charge fees in sub-currencies.
The function SHOULD `throw` unless the `_from` account has deliberately authorized the sender of the message via some mechanism.

*Note* Transfers of 0 values MUST be treated as normal transfers and fire the `Transfer` event.

``` js
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
```



#### approve

Allows `_spender` to withdraw from your account multiple times, up to the `_value` amount. If this function is called again it overwrites the current allowance with `_value`.

**NOTE**: To prevent attack vectors like the one [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/) and discussed [here](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729),
clients SHOULD make sure to create user interfaces in such a way that they set the allowance first to `0` before setting it to another value for the same spender.
THOUGH The contract itself shouldn't enforce it, to allow backwards compatibility with contracts deployed before

``` js
function approve(address _spender, uint256 _value) public returns (bool success)
```


#### allowance

Returns the amount which `_spender` is still allowed to withdraw from `_owner`.

``` js
function allowance(address _owner, address _spender) public view returns (uint256 remaining)
```



### Events


#### Transfer

MUST trigger when tokens are transferred, including zero value transfers.

A token contract which creates new tokens SHOULD trigger a Transfer event with the `_from` address set to `0x0` when tokens are created.

``` js
event Transfer(address indexed _from, address indexed _to, uint256 _value)
```



#### Approval

MUST trigger on any successful call to `approve(address _spender, uint256 _value)`.

``` js
event Approval(address indexed _owner, address indexed _spender, uint256 _value)
```



## Implementation

There are already plenty of ERC20-compliant tokens deployed on the Ethereum network.
Different implementations have been written by various teams that have different trade-offs: from gas saving to improved security.

#### Example implementations are available at
- [OpenZeppelin implementation](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/9b3710465583284b8c4c5d2245749246bb2e0094/contracts/token/ERC20/ERC20.sol)
- [ConsenSys implementation](https://github.com/ConsenSys/Tokens/blob/fdf687c69d998266a95f15216b1955a4965a0a6d/contracts/eip20/EIP20.sol)


## History

Historical links related to this standard:

- Original proposal from Vitalik Buterin: https://github.com/ethereum/wiki/wiki/Standardized_Contract_APIs/499c882f3ec123537fc2fccd57eaa29e6032fe4a
- Reddit discussion: https://www.reddit.com/r/ethereum/comments/3n8fkn/lets_talk_about_the_coin_standard/
- Original Issue #20: https://github.com/ethereum/EIPs/issues/20



## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
