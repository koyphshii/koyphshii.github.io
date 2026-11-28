---
title: "Bilinear Pairings"
date: 2026-11-28T00:00:00+01:00
draft: false
author: "Koyphshi"
description: "My learning notes on bilinear pairings from the RareSkills ZK Book, covering bilinear functions, elliptic curve pairings, and Ethereum precompile 0x08 in Solidity."
categories: ["Cryptography"]
tags: ["bilinear-pairings", "elliptic-curves", "zk", "solidity", "evm"]
math: true
toc:
  enable: true
  auto: true
code:
  copy: true
  maxShownLines: 50
---

My learning notes on bilinear pairings from the RareSkills ZK Book, covering bilinear functions, elliptic curve pairings, and Ethereum precompile `0x08` in Solidity.

<!--more-->

{{< admonition type="info" title="Related Challenges" open="true" >}}
- **CTF Challenge 1**: [BBLS](/posts/bbls/) (exploiting a modified BLS signature scheme)
- **CTF Challenge 2**: [BBLSv2](/posts/bblsv2/) (field prime vs group order mismatch in BN128 pairings)
- **Original Repo**: [`koyphshii/rareskills-bilinear-pairings`](https://github.com/koyphshii/rareskills-bilinear-pairings)
{{< /admonition >}}

## How bilinear pairings work

i was learning about bilinear pairings from the rareskills zk book, and in my initial read of the chapter i had so many questions because i didn't understand due to my low iq, so i listed them and then went through them, at first i will share the questions and then talk about what i understood after. (the screenshots are from the book)

Here is the link of the book : [RareSkills Bilinear Pairing](https://rareskills.io/post/bilinear-pairing#end-to-end-solidity-example-of-bilinear-pairings)

![pairing intro](pairing-intro.webp)

here they said that the pairing allows us to prove that a number is the product of 2 other numbers without ever leaking the numbers, is this the only use case ? i don't know

and they said it allows us to encrypt them and then do the computation on the encrypted values, the pairing allows us to do what exactly here ? the encryption is the thing that the pairing gives us ? or what

for me i think the pairing is the notation of the multiplication of $E(a)$ with $E(b)$

![pairing definition](pairing-definition.webp)

from this what i understand is :

a `bilinear pairing` is a function that takes 2 elliptic curve points as input and returns an output that corresponds to the product of the `dl` of the points

now they are limiting the pairing to elliptic curves only ?

![pairing dl product](pairing-dl-product.webp)

if we stick to the definition they gave us, this is clear, the function $e$ just checks the `dl` product

![bilinear mapping general](bilinear-mapping-general.webp)

now this is again confusing me, because they are giving a general definition to the `bilinear` mapping

they are saying that it is simply a function that if we fix one argument and the other varies, it gives us a linear function

but then they jump into that property, i have no idea where it did come from

and then the book stops explaining how this property did come from!! i am really confused is the problem with me ??

1 - is the mapping a function that works only with elliptic curves and gives the product of the `dl` ?

2 - is it a more general function that allows us to prove that $a \cdot b = c$ from $E(a) \cdot E(b) = E(c)$ ??

3 - or it is a more general function that comes from linear functions

## AFTER DARK

so after fighting with my stupid questions, i was able to find a quite of understanding that made sense for me

### 1 - what a bilinear function is

In its simplest form, a bilinear function is a function that takes 2 arguments where if you fix one and the other varies, the output will vary with that input that is not fixed

so it is a function $f : A \times B \to C$ where $f(a, b) = c$, if we fix $a$

$b$ and $c$ will vary in the same way (meaning they scale together, if we multiply $b$ by 2, $c$ will scale by 2 also, and so on)

for example $f(x, y) = 5xy$, if we fix $x$ or $y$, lets say we set $y = 2$, it will become a linear function $f(x) = 10x$

i know this may sound stupid, but lets talk about linear functions :

a linear function is a function where the input and output behave in the same way when working with either addition or multiplication, so it preserves the structure of these 2 operations, and that is why we always define a linear function with 2 properties :

1 - $f(x + y) = f(x) + f(y)$

2 - $f(k \cdot x) = k \cdot f(x)$

and from these, we can conclude that bilinear functions also have these 2 properties

### 2 - bilinear functions in the context of cryptography

here the headaches may start, but hopefully it is simple if we look at it from a good angle

we talked about bilinear functions before, right? cool. Now when we take the same exact function and write $f : G_1 \times G_2 \to G_t$ where :

- $G_1, G_2$ are 2 elliptic curve groups (they can be same or different but same order)
- $G_t$ is not an elliptic curve group, probably all you need to know about is that it is a finite cyclic group, but i will add some details later

we can absolutely do that because we didn't restrict what group elements the function $f$ takes

now with this $f$ function that works with elliptic curves, our goal is to achieve :

for $P = p \cdot G$ and $Q = q \cdot G$ and $R = r \cdot G$, we prove that we know $p \cdot q = r$ without ever leaking their values, only by using the public points $P, Q, R$

and there is a bilinear mapping that allows us to do that :

$$\Large f(P, Q) = f(R, G)$$

and we call it a bilinear pairing

> **NOTE** : we use the word "pairing" in the context of elliptic curves only

so a bilinear pairing is a function that corresponds to the product of the discrete log of the arguments, which are the points

now where did this come from ?

lets talk a bit more about $G_t$

$G_t$ is a subgroup of the multiplicative group of a finite field extension

as we know a finite field $\mathbb{F}_p$ is composed from 2 groups, one under addition and the other under multiplication excluding 0, an extension of the field can be $\mathbb{F}_{p^{12}}$, so we work with elements of dimension 12 instead of 1 dimension elements following some rules

and the multiplicative group of the field is simply the group that is working with multiplication

and by subgroup we simply take less elements of the whole group, but a subgroup is still considered a group by itself

so $G_t$ works with elements under multiplication, Cool!

now if we take this pairing : $f(k \cdot A, B)$, what we can do with it ?

remember that the pairing is a linear function when we fix one argument !

so if we fix $B = H_1$ we get $f(k \cdot A, H_1)$ and this is a linear function, and using the addition property we get :

$$\Large f(k \cdot A, H_1) = f(A, H_1) \cdot f(A, H_1) \cdots k \text{ times} = f(A, H_1)^k$$

now we do the same thing for $B = H_2$ and $B = H_3$ and so on, we notice that it is true for every point we set $B$ to, so we can directly say :

$$\Large f(k \cdot A, B) = f(A, B)^k$$

from this, all properties are easier to understand

for example in the book :

![book properties](book-properties.webp)

they can all be written : $e(G_1, G_2)^{a \cdot b}$

why is $G_t$ really complex? well remember that the pairing is a function, it takes inputs and gives output, we need it to satisfy the properties of a linear function, so $G_t$ is probably the group that satisfies it.

hopefully from my perspective, what i have written here answers my questions, remember that i am just a beginner in this field, so don't take it as official resource to work on, instead my goal is to have people that review it and help me improve by tagging mistakes i did

And from now, i will be showing my POC of this book chapter because i liked it so i read it

## Bilinear Pairings in Ethereum

### precompiles

when we are working with a programming language that produces `evm` bytecode, for example solidity, if the function we did implement in solidity was complicated, it will produce so many opcodes to store and the `evm` interpreter will execute each opcode one by one, that will cost gas, so complicated logic can lead to consuming so much gas

when talking about a pairing function, it is actually really really complicated, the codomain group elements have a dimension of 12, and when we are restricted to the native opcodes of the evm that can only work with 256 bits words, we will need so many opcodes to work with the $G_t$ elements, so if we implement it using solidity directly (meaning in the smart contract we write all the logic) it will produce a very big amount of opcodes and this can lead to problems like hitting the maximum size of the block

for this reason `ethereum precompiles` exist, what they do is basically when we call `pairing` in solidity, it will not produce bytecode that corresponds to the logic of the `pairing`, instead it produces a call opcode, specifically `STATICCALL 0x08`, and then we run the pairing math directly in the host cpu, bypassing the `evm` interpreter loops

as we know, depending on what client we are running, the `evm` will use a certain programming language, for example if we were using `geth`, then all the `evm` logic is implemented using go, for example the `ADD` opcode is implemented using go, so when hitting the `STATICCALL 0x08` opcode, we will be running the pairing logic using go, a language like go running on the cpu can perform much more efficiently

and the `0x08 precompile` follows the `EIP 197 Specification`, which tells the `evm implementation` we are running what to do exactly when hitting that `0x08` call, like how much gas it must cost, what to return, the format, the groups we should use, and pretty much everything, you can find it here : [EIP-197](https://eips.ethereum.org/EIPS/eip-197)

`PyEVM` is a python implementation of the `EVM` specification, but there is no running client with this implementation participating in the `ethereum` network, because `PyEVM` was built mainly for testing and research, the project is now archived and you can find it here : [ethereum/py-evm](https://github.com/ethereum/py-evm)

previously in this chapter, we worked with the `py_ecc` library, this library is actually the code that is implemented for the `0x08 precompile` in `PyEVM`

### example in solidity

so we know there is one precompile at address `0x08` that implements the pairing math following the specifications of EIP 197

here is one way to do that in solidity :

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;
contract Pairings {
    /**
     *  returns true if == 0,
     *  returns false if != 0,
     *  reverts with "Wrong pairing" if invalid pairing
     */
    function run(uint256[12] memory input) public view returns (bool) {
        assembly {
            let success := staticcall(gas(), 0x08, input, 0x0180, input, 0x20)
            if success {
                return(input, 0x20)
            }
        }
        revert("Wrong pairing");
    }
}
```

so what will happen is that we call the `run` function and the calldata will be a contiguous 12 words (each one 32 bytes)

why 12 exactly ? remember the groups we are working on, $G_1$ elements are points of the form $(x, y)$, meaning 2 words, meanwhile $G_2$ elements are of the form $((x_1, y_1), (x_2, y_2))$, meaning 4 words, so one pairing function takes 6 words, and in this example we are comparing 2 pairings together, so 12 words

now since we specified that input will be in memory, before even calling the staticcall, the raw data is already in memory, what is left is reading it

`staticcall(gas(), 0x08, input, 0x0180, input, 0x20)` here the variable `input` corresponds to the offset of the data in memory, and `0x0180` is `32 * 12`

so it is basically starting offset + length, and the second `input` is the return offset, since we wont need the points we used in the pairing after the math, we can overwrite it with the result, and `0x20` (32 bytes) is the length of the result which will be a boolean

after that, the precompile that is getting called will read from memory and perform the math

and this is how we call this run function :

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;
import "forge-std/Test.sol";
import "../src/Pairings.sol";

contract PairingsTest is Test {
    Pairings public pairings;

    function setUp() public {
        pairings = new Pairings();
    }

    function testPairings() public view {
        uint256 aG1_x = 3010198690406615200373504922352659861758983907867017329644089018310584441462;
        uint256 aG1_y = 17861058253836152797273815394432013122766662423622084931972383889279925210507;

        uint256 bG2_x1 = 2725019753478801796453339367788033689375851816420509565303521482350756874229;
        uint256 bG2_x2 = 7273165102799931111715871471550377909735733521218303035754523677688038059653;
        uint256 bG2_y1 = 2512659008974376214222774206987427162027254181373325676825515531566330959255;
        uint256 bG2_y2 = 957874124722006818841961785324909313781880061366718538693995380805373202866;

        uint256 cG1_x = 4503322228978077916651710446042370109107355802721800704639343137502100212473;
        uint256 cG1_y = 6132642251294427119375180147349983541569387941788025780665104001559216576968;

        uint256 dG2_x1 = 18029695676650738226693292988307914797657423701064905010927197838374790804409;
        uint256 dG2_x2 = 14583779054894525174450323658765874724019480979794335525732096752006891875705;
        uint256 dG2_y1 = 2140229616977736810657479771656733941598412651537078903776637920509952744750;
        uint256 dG2_y2 = 11474861747383700316476719153975578001603231366361248090558603872215261634898;

        uint256[12] memory points = [
            aG1_x,
            aG1_y,
            bG2_x2,
            bG2_x1,
            bG2_y2,
            bG2_y1,
            cG1_x,
            cG1_y,
            dG2_x2,
            dG2_x1,
            dG2_y2,
            dG2_y1
        ];

        bool x = pairings.run(points);
        console2.log("result:", x);
    }
}
```

after running this code, this is the result :

![Solidity pairing test result](solidity-result.webp)

---

### Related CTF Challenges

If you want to see how bilinear pairings get used (and broken) in practice, check out the 2 challenges I built around BLS signatures and BN128 pairings :

1 - [BBLS](/posts/bbls/)
2 - [BBLSv2](/posts/bblsv2/)
