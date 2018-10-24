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

A specification for a standardized minable Token that uses a Proof of Work algorithm for distribution.


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

Returns the current `challengeNumber`, i.e. a byte32 number to be included (with other elements, see later) in the POW algorithm input in order to synthesize a valid solution. It is expected that a new `challengeNumber` is generated after that the valid solution has been found and the reward tokens have been assigned.

```js
function challengeNumber() view public returns (bytes32)
```

**NOTES**: in a common implementation `challengeNumber` is calculated starting from some immutable data, like elements derived from some past ethereum blocks.



#### difficulty

Returns the current difficulty, i.e. a number useful to estimate (by means of some known algorithm) the mean time required to find a valid POW solution. It is expected that the `difficulty` varies if the smart contract controls the mean time between valid solutions by means of some control loop.

```js
function difficulty() view public returns (uint256)
```

**NOTES**: in a common implementation, difficulty varies when computational power is added/subtracted to the network, in order to maintain stable the mean time between valid solutions found.



#### epochCount

Returns the current epoch, i.e. the number of successful minting operation so far (starting from zero).


```js
function epochCount() view public returns (uint256)
```


#### adjustmentInterval

Returns the interval, in seconds, between two successive difficulty adjustment.

```js
function adjustmentInterval () view public returns (uint256)
```

**NOTES**: in a common implementation, while difficulty varies when computational power is added/subtracted to the network, the adjustmentInterval is fixed at deploy time.


#### miningTarget

Returns the miningTarget, i.e. a number which is a threshold useful to evaluate if a given submitted POW solution is valid.

```js
function miningTarget () view public returns (uint256)
```

**NOTES**: in a common implementation, the solution is valid if lower than the miningTarget.


#### miningReward

Returns the number of tokens that POW faucet shall dispense as next reward.

```js
function miningReward() view public returns (uint256)
```

**NOTES**: in a common implementation, the reward progressively diminishes toward zero trough the epochs (“epoch” is mining cycle started by the generation of a new challengeNumber and ended by the reward assignment), in order to have a maximum number of tokens dispensed in the whole life of the token smart contract, i.e. after that the maximum number of tokens has been dispensed, no more tokens will be dispensed.



#### tokensMinted

Returns the total number of tokens dispensed so far by POW faucet

```js
function tokensMinted() view public returns (uint256)
```


#### mint

Returns a flag indicating that the submitted solution has been considered the valid solution for the current epoch and rewarded, and that all the activities needed in order to launch the new epoch have been successfully completed.

```js
function mint(uint256 nonce) public returns (bool success)
```

**NOTES**:
1) In particular, the method must verify a submitted solution, described by the nonce (see later):
* IF the solution found is the first valid solution submitted for the current epoch:
    a) rewards the solution found sending No. miningReward tokens to msg.sender;
    b) creates a new challengeNumber valid for the next POW epoch;
    c) eventually adjusts the POW difficulty;
    d) return true
* ELSE the solution is not the first valid solution submitted for the current epoch and it returns false (or revert).

2) The first phase (hash check) MUST BE implemented using the below specified public function hash(), while an internal function structure is recommended (see Reccomendation), but it is not mandatory.



#### hash

Returns the digest calculated by the algorithm of hashing used in the particular implementation, whatever it will be.

```js
function hash(uint256 nonce, address minter, bytes32 challengeNumber) public returns (bytes32 digest)
```

**NOTES**: hash() is to be declared public and it MUST include explicitly uint256 nonce, address minter, bytes32 challengeNumber in order to be useful as test function for the mining software development and debugging.


### Events

#### Mint

The Mint event indicates the rewarded address, the reward amount, the epoch count and the challenge number used.

```js
event Mint(address indexed _to, uint _reward, uint _epochCount, bytes32 _challengeNumber)
```

**NOTES**: TO BE MANDATORY EMITTED immediately after that the submitted solution is rewarded.


## Recommendation

#### MITM attacks

To prevent man-in-the-middle attacks, the msg.sender address, which is the address eventually rewarded, should be part of the hash so that any nonce solution found is valid only for that particular Ethereum account and it is not susceptible to be used by other. This also allows pools to operate without being easily cheated by the miners because pools can force miners to mine using the pool’s address in the hash algorithm. In that a case, indeed, the pool is the only address able to collect rewards.

#### Anticipated mining

In order to avoid that miners are in condition to calculate anticipated solutions for later epoch, a “challengeNumber”, i.e. a number somehow derived from existing but mutable conditions, should be part of the hash so that future blocks cannot be mined before. The “challengeNumber” acts like a random piece of data that is not revealed until a mining round starts.

### Hash functions
The use of solidity keccak256 algorithm is strongly recommended, even if not mandatory, because it is a very cost effective one-way algorithm to compute in the EVM environment and it is available as built-in function in solidity.

### Solution representation
The recommended representation of the solution found is by a ‘nonce’, i.e. a number, that miners try to find, that being part of the digest make the value of the hash of the digest itself under the required threshold for validity.

### mint() internal structure
From the miner point of view, submitting a solution for possible reward means to call the mint() function with the suitable arguments and waiting for evaluation results.
It is recommended that internally the `mint()` function be realized invoking 4 separate successive phases: hash check, rewarding, epoch increment, difficulty adjustment.
The first phase (hash check) MUST BE implemented using the specified `public function hash()`, while an internal `function mint()` structure is recommended, but it is not mandatory. In particular the following phases, being totally internal to the contract, cannot be specified as mandatory, but the schema where four explicit and subsequent phases are evidenced (hash check, rewarding, epoch increment and difficulty adjustment) is recommended.

In the preferred realization, for each of those steps a suitable function is declared and called:
1) hash check -> MANDATORY by above spec. 	`function hash()`
2) rewarding -> by means of some 		`function _reward() internal returns (uint)`
3) epoch increment -> by means of some 	`function _epoch() internal returns (uint)`
4) difficulty adj. -> by means of some 	`function _adjustDifficulty() internal returns (uint)`

It may be useful to recall that a Mint event MUST BE emitted before returning a boolean success flag.

In a sample compliant realization, the mint can be then roughly described as follows:

```js
function mint(uint256 nonce) public returns (bool success) {
    require (hash(nonce, minter, challengeNumber) < byte32(miningTarget), “Invalid solution”);
    emit Mint(minter, _reward(), _epochCount, _challengeNumber);
    _epoch();
    _adjustDifficulty();
    return(true);
}
```            

## Backwards Compatibility
In order to facilitate the use of both existing mining programs and existing pool software already used to mine minable tokens deployed before the emission of the present standard, the following functions can be included in the contract. They are simply a wrapping of some of the above defined functions:

```js
function getAdjustmentInterval() public view returns (uint) {
            return adjustmentInterval();
}

function getChallengeNumber() public view returns (bytes32) {
            return challengeNumber();
}

function getMiningDifficulty() public view returns (uint) {
            return difficulty();
}
function getMiningReward() public view returns (uint) {
            return miningReward();
}

function mint(uint256 _nonce, bytes32 _challenge_digest) public returns (bool success) {
            return mint (_nonce);
}
```

**NOTES**: Any already existing token implementing this interface can be declared compliant to EIP918-B (B for Backwards). **EIP918-B compliance is deprecated.**



## Implementation notes and examples

< here, properly reorganized, all the suitable elements from the current draft (interface, abstract contract, etc.) >

### Abstract contracts

In order to implement the standard, the following abstract contract can be included and inheritated by the smart contract.

`contract AEIP918B  {
    function challengeNumber() public view returns (bytes32);
    function difficulty() public view returns (uint256);
    function epochCount() public view returns (uint256);
    function adjustmentInterval () public view returns (uint256);
    function miningTarget () public view returns (uint256);
    function miningReward() public view returns (uint256);
    function tokensMinted() public view returns (uint256);
    function mint(uint256 nonce) public returns (bool success);
    function hash(uint256 nonce, address minter, bytes32 challengeNumber) public returns (bytes32 digest);
    event Mint(	address indexed _to, uint _reward, uint _epochCount, bytes32 _challengeNumber);
} //END OF EIP918 `

**NOTES**: GIVEN THAT THE CURRENT VERSION OF THE SOLIDITY COMPILER (0.4.25) IS NOT YET ABLE TO MANAGE IMPLICIT PUBLIC VARIABLES GETTER AS VALID OVERLOADS ON INTERFACES AND ABSTRACT CONTRACTS, INCLUDING THE PREVIOUS VERSION OF THE ABSTRACT CONTRACT IN ORDER TO BE COMPLIANT CAN GENERATE SYNTAX ERRORS IF THE OVERLOADING FUNCTIONS ARE INTENDED TO BE THE AUTOMATICLY CREATED GETTER OF PUBLIC VARIABLES WITH THE SAME NAME. THE CURRENT SOLUTIONS ARE: (i) to move public variables declarations in the abstract contract and to omit the related method declaration, or (ii) to name the public variable differently and to write the getter using the naming convention declared by the standard.


#### Test Cases
-
-
-

## History

Historical links related to this standard:

- Original proposal from Jay Logelin: https://github.com/ethereum/wiki/wiki/Standardized_Contract_APIs/499c882f3ec123537fc2fccd57eaa29e6032fe4a
- Reddit discussion: https://www.reddit.com/r/ethereum/comments/3n8fkn/lets_talk_about_the_coin_standard/
- Original Issue EIP918: https://github.com/ethereum/EIPs/issues/918
-
-


## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
