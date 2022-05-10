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

First opcode is `CALLVALUE` again, but this time the second opcode is `CODESIZE`, which puts the size of the code (so last address + 1) on the stack. The third opcode `SUB` subtracts the `CALLVALUE` from `CODESIZE`.

The result of `SUB` has to be 6, which is the jump destination. `CODESIZE` returns 10, so `CALLVALUE` has to be 4.

[EVM Playground](https://www.evm.codes/playground?callValue=4&unit=Wei&codeType=Mnemonic&code='CALLVALUEzCODESIZEzSUBy~~yDESTzSTOP~~'~zREVERTz%5CnyzJUMP%01yz~_)
