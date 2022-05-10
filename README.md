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

| 1 | 1 | 0 | 0 |
| 0 | 1 | 1 | 0 |
|--- |--- |--- |--- |
| 1 | 0 | 1 | 0 |
