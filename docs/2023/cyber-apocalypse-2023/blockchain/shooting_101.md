# 1 - Challenge code and Description

```
Your metallic body might have advanced targeting systems, but hitting a target is not just about technical proficiency.
To truly master the art of targeting, you must learn to trust your instincts and develop a keen sense of intuition.
During this training, you will emerge as a skilled marksman who can hit the targets with deadly precision.
It's about time to train and prove yourself in the Shooting Area, can you make it?
```

**Setup.sol**:
```solidity
pragma solidity ^0.8.18;

import {ShootingArea} from "./ShootingArea.sol";

contract Setup {
    ShootingArea public immutable TARGET;

    constructor() {
        TARGET = new ShootingArea();
    }

    function isSolved() public view returns (bool) {
        return TARGET.firstShot() && TARGET.secondShot() && TARGET.thirdShot();
    }
}
```


**ShootingArea.sol**:

```solidity
pragma solidity ^0.8.18;

contract ShootingArea {
    bool public firstShot;
    bool public secondShot;
    bool public thirdShot;

    modifier firstTarget() {
        require(!firstShot && !secondShot && !thirdShot);
        _;
    }

    modifier secondTarget() {   
        require(firstShot && !secondShot && !thirdShot);
        _;
    }

    modifier thirdTarget() {
        require(firstShot && secondShot && !thirdShot);
        _;
    }

    receive() external payable secondTarget {
        secondShot = true;
    }

    fallback() external payable firstTarget {
        firstShot = true;
    }

    function third() public thirdTarget {
        thirdShot = true;
    }
}
```




# 2 - Solution

Accoring to this code section, to solve this challenge the 3 state variables `firstShot, secondShot, thirdShot` should be set to True
```solidity
function isSolved() public view returns (bool) {
    return TARGET.firstShot() && TARGET.secondShot() && TARGET.thirdShot();
}
```

We know that we can not change state variables of a contract directly and because we doon't have any setter function for these variables the only way to change their values is through `fallback, receive, third` function\
`third` function is a normal function but `fallback` and `receive` are special functions which will be triggered in special conditions and we can not call them directly\
According to this [video](https://www.youtube.com/watch?v=CMVC6Tp9gq4) `fallback` and `receive` can be triggered when facing errors like the calling function does not exist or lack of crypto-currency ...

If we wanna try to send a custom transaction with arbitrary data we will encounter error which we can use to trigger fallback and receive function\
Also one more important issue about the code is that we have three modifiers named `firstTarget, secondTarget, thirdTarget`.\
According to [this link](https://www.alchemy.com/overviews/solidity-modifier), A midifier puts some conditions on a function. If the conditions are met the function would be executed else not.\

If we look at the modifiers, `fallback` function is dependant on `firstTarget` modifier which tells us in order to this `fallback` function be executed all state variables `firstShot, secondShot, thirdShot` should be False\
Like that the `receive` function is dependant on `secondTarget` modifier which indicates that in order to trigger `receive` function the `firstShot` state variable should set to True
And the third function is dependant on `thirdTarget` modifier which indicates that in order to call that function the `firstShot` and `secondShot` state variable should set to True.

So to brief all we should trigger these 3 functions in this order
1. fallback
2. receive
3. third


Let's first get connection infomation from second service

```bash
nc 165.22.116.7 31860
1 - Connection information
2 - Restart Instance
3 - Get flag
action? 1

Private key     :  0xf1b75a27cee0e379746277b990bb7987815fd720d9fbbbbd0115b75d334c0272
Address         :  0x880D2D46b194678fe1990E0c859F0bEdB2A87F6f
Target contract :  0x5094b5864dbB733a98E2A201fd7419F4e908be7B
Setup contract  :  0xFc5becb1a0026dd785AbCe82b52A31045164E2CF
```

For first and second shot which is `fallback` and `receive` function I used [web3py](https://web3py.readthedocs.io/en/stable/):
```py
from web3 import Web3

url = 'http://165.22.116.7:30205/'
wallet = '0x880D2D46b194678fe1990E0c859F0bEdB2A87F6f'
target = '0x5094b5864dbB733a98E2A201fd7419F4e908be7B'

w3 = Web3(Web3.HTTPProvider(url))

# First Shot
# This transaction will trigger fallback because it has data
w3.eth.send_transaction({'to': target, 'from': wallet, 'data':'abcd'})


# Second Shot
# This transaction will trigger receive because it has no data
w3.eth.send_transaction({'to': target, 'from': wallet})
```

And for the third shot which is a normal function and we can call it directly I used [web3 cli](https://github.com/gochain/web3)
The address is ShootingArea contact address
```bash
web3 contract call --address '0x5094b5864dbB733a98E2A201fd7419F4e908be7B' --abi ShootingArea.abi --function third
```

After all these we can get the flag
```bash
nc 165.22.116.7 31860
1 - Connection information
2 - Restart Instance
3 - Get flag
action? 3
HTB{f33l5_n1c3_h1771n6_y0ur_74r6375}
```

And here is the flag
```
HTB{f33l5_n1c3_h1771n6_y0ur_74r6375}
```
