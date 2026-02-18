---
description: Create production grade smart contracts. Use this skill when the user asks to write smart contracts that are going to be deployed to production (to a mainnet, or used in a mainnet script).
disable-model-invocation: true
---

# Solidity Development Standards

Instructions for how to write solidity code, from the [Cyfrin security team.](https://www.cyfrin.io/)

## Philosophy

- **Everything will be attacked** - Assume that any code you write will be attacked and write it defensively.

## Code Quality and Style

1. Absolute and named imports only — no relative (`..`) paths

```solidity
// good
import {MyContract} from "contracts/MyContract.sol";

// bad
import "../MyContract.sol";
```


2. Prefer `revert` over `require`, with custom errors that are prefix'd with the contract name and 2 underscores.

```solidity
error ContractName__MyError();

// Good
myBool = true;
if (myBool) {
    revert ContractName__MyError();
}

// bad
require(myBool, "MyError");
```

3. In tests, prefer to use fuzz tests over unit tests
```solidity
// good - using foundry's built in stateless fuzzer
function testMyTest(uint256 randomNumber) { }

// bad
function testMyTest() { 
    uint256 randomNumber = 0;
}
```

4. Functions should be grouped according to their visibility and ordered:

```
constructor
receive function (if exists)
fallback function (if exists)
user-facing state-changing functions
    (external or public, not view or pure)
user-facing read-only functions
    (external or public, view or pure)
internal state-changing functions
    (internal or private, not view or pure)
internal read-only functions
    (internal or private, view or pure)
```

5. Headers should look like this:

```solidity
    /*//////////////////////////////////////////////////////////////
                      INTERNAL STATE-CHANGING FUNCTIONS
    //////////////////////////////////////////////////////////////*/
```

6. Layout of file

```
Pragma statements
Import statements
Events
Errors
Interfaces
Libraries
Contracts
```

Layout of contract:

````
Type declarations
State variables
Events
Errors
Modifiers
Functions
```

7. Use the branching tree technique when creating tests
Credit for this to [Paul R Berg](https://x.com/PaulRBerg/status/1682346315806539776)

- Target a function
- Create a `.tree` file
- Consider all possible execution paths 
- Consider what contract state leads to what path
- Consider what function params lead to what paths
- Define "given state is x" nodes
- Define "when parameter is x" node 
- Define final "it should" tests

Example:
```
├── when the id references a null stream
│   └── it should revert
└── when the id does not reference a null stream
    ├── given assets have been fully withdrawn
    │   └── it should return DEPLETED
    └── given assets have not been fully withdrawn
        ├── given the stream has been canceled
        │   └── it should return CANCELED
        └── given the stream has not been canceled
            ├── given the start time is in the future
            │   └── it should return PENDING
            └── given the start time is not in the future
                ├── given the refundable amount is zero
                │   └── it should return SETTLED
                └── given the refundable amount is not zero
                    └── it should return STREAMING
```

Example:
```solidity
function test_RevertWhen_Null() external {
    uint256 nullStreamId = 1729;
    vm.expectRevert(abi.encodeWithSelector(Errors.SablierV2Lockup_Null.selector, nullStreamId));
    lockup.statusOf(nullStreamId);
}

modifier whenNotNull() {
    defaultStreamId = createDefaultStream();
    _;
}

function test_StatusOf()
    external
    whenNotNull
    givenAssetsNotFullyWithdrawn
    givenStreamNotCanceled
    givenStartTimeNotInFuture
    givenRefundableAmountNotZero
{
    LockupLinear.Status actualStatus = lockup.statusOf(defaultStreamId);
    LockupLinear.Status expectedStatus = LockupLinear.Status.STREAMING;
    assertEq(actualStatus, expectedStatus);
}
```

8. Prefer strict pragma versions for contracts, and floating pragma versions for tests, libraries, abstract contracts, interfaces, and scripts.

9. Add a security contact to the natspec at the top of your contracts

```solidity
/**
  * @custom:security-contact mycontact@example.com
  * @custom:security-contact see https://mysite.com/ipfs-hash
  */  
```

10. Remind people to get an audit if they are deploying to mainnet, or trying to deploy to mainnet