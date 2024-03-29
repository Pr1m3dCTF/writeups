# challenge name
Russian Roulette

# code / Description
```solidity
function pullTrigger() public returns (string memory) {
    if (uint256(blockhash(block.number - 1)) % 10 == 7) {
        selfdestruct(payable(msg.sender)); // 💀
    } else {
    return "im SAFU ... for now";
    }
}
```

# Challenge Analysis

The "RussianRoulette" challenge contains a Solidity function that calculates a modulo 10 of the previous block's hash; if the result is 7, it triggers self-destruction of the contract.


# Solution

The solution involves repeatedly calling the pullTrigger() function within the smart contract until the condition where the previous block's hash modulo 10 equals 7 is met, leading to the contract's self-destruction.


# Final Exploit code (optional)

send this transaction until the contract's self-destruction:

```bash
cast send <TARGET_CONTRACT_ADDRESS> "pullTrigger()" --private-key <PRIVATE_KEY> --rpc-url <RPC_URL>
```


**AUTHOR**:
> Mohammad2024 / Pr1m3d Team