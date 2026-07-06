---
title: "Magic Number"
date: 2026-01-07T00:00:00+00:00
draft: false
author: "Koyphshi"
description: "Deploying a custom EVM bytecode contract under 10 bytes to return 42"
categories: ["Blockchain"]
tags: ["ethernaut", "solidity", "evm", "bytecode", "assembly"]
math: true
code:
  maxShownLines: 50
toc:
  enable: true
  auto: true
---

this challenge is about calling the `setSolver` function and we give it an address of a contract that responds to `whatIsTheMeaningOfLife()` with the number **42**

<!--more-->

{{< admonition type="info" title="Challenge Info" open="true" >}}
- **Platform**: Ethernaut
- **Challenge**: Magic Number
- **Category**: Blockchain
{{< /admonition >}}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MagicNum {
    address public solver;

    constructor() {}

    function setSolver(address _solver) public {
        solver = _solver;
    }

    /*
    ____________/\\\_______/\\\\\\\\\_____
     __________/\\\\\_____/\\\///////\\\___
      ________/\\\/\\\____\///______\//\\\__
       ______/\\\/\/\\\______________/\\\/___
        ____/\\\/__\/\\\___________/\\\//_____
         __/\\\\\\\\\\\\\\\\_____/\\\//________
          _\///////////\\\//____/\\\/___________
           ___________\/\\\_____/\\\\\\\\\\\\\\\_
            ___________\///_____\//////////////__
    */
}
```

the plot is that this contract should be 10 bytes at most, if we create the contract using the normal way it will be way larger than 10 bytes

when we do an external call to a smart contract, we provide the function signature of the function we want to execute in this call, now someone may misunderstand the process and think that the evm will first look at this function signature and then look at it in the contract code stored in the blockchain and start executing the specific code block, but instead what the evm does is that it starts executing the contract we are calling from the first ever line, and when using the solidity compiler this line will include a jump instruction that redirects the evm into the right code block for the right function, so whatever the function we are calling in the contract, the evm will always start from offset 0, so if we deploy a custom bytecode without using the compiler, when the `MagicNum` contract calls that specific function, the evm will start executing our bytecode

our goal will be to deploy a bytecode that has only one job: return 42 to the caller contract

in order to do that we should store 42 in the memory and then return it, this can be done through the `mstore` opcode followed by a `return` instruction, in detail it will be:

- `mstore(0, 42)` — store 42 in memory at offset 0
- but before that we pass the opcode arguments into the stack using `push1 2a` and `push1 00`
- after we store it, we push the values that `return` will use: we return 32 bytes from memory starting from offset 0, so `push1 20` followed by `push1 00`

the run time bytecode:
- `push1 2a` -> `602a`
- `push1 00` -> `6000`
- `mstore` -> `52`
- `push1 20` -> `6020`
- `push1 00` -> `6000`
- `return` -> `f3`

so `602a60005260206000f3`

but we still need the constructor logic that gets executed when the contract gets deployed and returns the run time bytecode into memory to get stored into the blockchain:

- `push10 0x602a60005260206000f3`
- `push1 00`
- `mstore`
- `push1 0a`
- `push1 16`
- `return`

this will execute `return(22, 32)` and the reason we did that is because when the bytes memory was stored it gets left padded with zeros so in memory slot 0 we have 22 zeros followed by `602a60005260206000f3`, and we want to return exactly the run time bytecode, so we start from offset 22

the final bytecode we deploy is `69602a60005260206000f3600052600a6016f3`

now all left is deploying this bytecode using another smart contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "forge-std/console.sol";

contract attack {
  MagicNum mag;
  constructor(MagicNum _mag) payable {
    mag = _mag;
  }
  function pwn() external {
    bytes memory code = hex"69602a60005260206000f3600052600a6016f3";
    address addr;
    assembly {
      addr := create(0, add(code, 32), 19)
    }

    mag.setSolver(addr);
  }

}

contract Solver is Script {
  MagicNum instance = MagicNum(0x0B5592dD7512dDcb9F16C69c6d580743a54607C2);

  function run() external {
     vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
     attack p = new attack(instance);
     p.pwn();
  }
}
```

gg the challenge is solved
