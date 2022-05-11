# EVM puzzles

A collection of EVM puzzles. Each puzzle consists on sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction data that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.

## Solutions

Following are the solutions for each puzzle. But first, a high level overview how this actually works:

For each new puzzle, the play.js script is creating a smart contract with the code of the puzzle, e.g. for puzzle #1 a smart contract with code `0x3456FDFDFDFDFDFD5B00` is being created. If you enter a solution, a transaction with your calldata/callvalue is being sent to that generted smart contract. If the tx is successfull, you passed, if the tx reverts, you failed.

### Puzzle #1

First opcode is `CALLVALUE`, which moves the msg value on top of the stack.
Second opcode is `JUMP`, which reads the topmost value of the stack and jumps to that destination.

We can see that `JUMPDEST` is at 08, so callvalue has to be 8.

[EVM Playground](https://www.evm.codes/playground?callValue=8&unit=Wei&codeType=Mnemonic&code='CALLVALUEy~~~~~~yDESTzSTOP'~zREVERTz%5CnyzJUMP%01yz~_)

### Puzzle #2

First opcode is `CALLVALUE` again, but this time the second opcode is `CODESIZE`, which puts the size of the code (so last address + 1) on the stack. The third opcode `SUB` subtracts `CALLVALUE` from `CODESIZE`.

The result of `SUB` has to be 6, which is the jump destination. `CODESIZE` returns 10, so `CALLVALUE` has to be 4.

[EVM Playground](https://www.evm.codes/playground?callValue=4&unit=Wei&codeType=Mnemonic&code='CALLVALUEzCODESIZEzSUBy~~yDESTzSTOP~~'~zREVERTz%5CnyzJUMP%01yz~_)

### Puzzle #3

First opcode is `CALLDATASIZE`, which puts the byte size of the calldata on top of the stack, e.g. `0x01FF` -> 2
`JUMPDEST` is at address 4, so `CALLDATASIZE` has to return 4, e.g. correct calldata could be `0x01020304`

[EVM Playground](https://www.evm.codes/playground?unit=Wei&callData=0xff010203FF&codeType=Mnemonic&code='CALLDATASIZEyzzyDEST~STOP~'~%5Cnz~REVERTy~JUMP%01yz~_)

### Puzzle #4

This puzzle takes the `CALLVALUE` and `CODESIZE` and uses them as arguments for a bitwise exclusive or (XOR) operation.

Truth table of A XOR B

| A   | B   | Y   |
| --- | --- | --- |
| 0   | 0   | 0   |
| 0   | 1   | 1   |
| 0   | 0   | 1   |
| 0   | 1   | 0   |

`JUMPDEST` is at address 0A (10), so the result of our XOR operation has to be 10.
`CODESIZE` is `0x0B + 1 = 0x0C => 12 => 0b1100`, our task is to submit the correct calldata so that `12 XOR CALLVALUE` returns 10, which is `0b0110 => 6`.

| CODESIZE  | 1   | 1   | 0   | 0   |
| --------- | --- | --- | --- | --- |
| CALLVALUE | 0   | 1   | 1   | 0   |
| XOR       | 1   | 0   | 1   | 0   |

### Puzzle #5

This puzzle puts the callvalue on the stack, duplicates it and multiplicates it by itself. After that it pushes the value `0x0100 = 256` to the stack and checks if the result of the multiplication before is equal.

So the puzzle basically squares the callvalue and checks if the result is equal to 256. If yes, it jumps to the desired jumpdestination, if not it reverts.

The `JUMPI` opcode is used to implement conditions, it takes two values from the stack, the first (topmost) value is the jump destination, the second value is the conditional value: If its different from 0, it jumps to `JUMPDEST`, if not it will simply continue.

The solution is the square root of 256, which is 16.

[EVM Playground](https://www.evm.codes/playground?callValue=100&unit=Wei&codeType=Mnemonic&code='CALLVALUE~DUP1~MULy2%200x0100~EQy1%200x0CwIzzwDEST~STOPzz'~%5Cnz~REVERTy~PUSHw~JUMP%01wyz~_)

### Puzzle #6

This puzzle takes the calldata via `CALLDATALOAD` and pads it to 32 bytes, the result is being used as `JUMPDEST`.

To solve this puzzle we have to send the jump destination 0x0A padded to 32 bytes:
`0x000000000000000000000000000000000000000000000000000000000000000A`

[EVM Playground](https://www.evm.codes/playground?callValue=100&unit=Wei&callData=0x000000000000000000000000000000000000000000000000000000000000000a&codeType=Mnemonic&code='PUSH1%200x00zCALLDATALOADy~~~~~~yDESTzSTOP'~zREVERTz%5CnyzJUMP%01yz~_)

### Puzzle #7

This puzzle is one of the most difficult so far.

`CALLDATACOPY` saves the calldata to the memory. It takes three input arguments: The memory offset, the calldata offset, and the length to copy. In our puzzle `CALLDATACOPY` copies the entire calldata to memory, so the memory and calldata content is now identical.

It continues with the same three opcodes, but then executes `CREATE`. What this opcode does is creating a new contract, with the calldata as creation bytecode.

`CREATE` returns the address of the new contract, which is consumed by `EXTCODESIZE`. The following opcodes basically check if the bytecode size of the newly created contract is equal to one, if yes, jump to the jump destination.

To achieve this, we have to submit a creation bytecode which just returns a single byte as calldata

`0x60016000F3` -> push a single byte to memory and return it via `RETURN`.

For more info on creation vs runtime bytecode, check out [this](https://ventral.digital/posts/2022/3/12/evm-puzzles-second-wind) link or [this](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-ii-creation-vs-runtime-6b9d60ecb44c/).

[EVM Playground](https://www.evm.codes/playground?callValue=100&unit=Wei&callData=0x60026000F3&codeType=Mnemonic&code='yw0zDUP1zyCOPYzyw0~00zCREATEzEXTCODEw1zEQ~13vIzREVERTvDESTzSTOP'~zPUSH1%200xz%5CnyCALLDATAwSIZE~0vzJUMP%01vwyz~_)
