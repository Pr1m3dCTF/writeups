# 1 - Challenge code and Description

```
Your advanced sensory systems make it easy for you to navigate familiar environments, but you must rely on intuition to navigate in unknown territories.
Through practice and training, you must learn to read subtle cues and become comfortable in unpredictable situations.
Can you use your software to find your way through the blocks?
```


**Setup.sol**
```solidity
pragma solidity ^0.8.18;

import {Unknown} from "./Unknown.sol";

contract Setup {
    Unknown public immutable TARGET;

    constructor() {
        TARGET = new Unknown();
    }

    function isSolved() public view returns (bool) {
        return TARGET.updated();
    }
}
```

**Unknown.sol**
```solidity
pragma solidity ^0.8.18;

contract Unknown {
    
    bool public updated;

    function updateSensors(uint256 version) external {
        if (version == 10) {
            updated = true;
        }
    }
}
```


# 2 - Solution
Here we have two smart contracts named `Setup` and `Unknown`\
We also have remote intances with two services first one is RPC service with http protocol and second one gives us two options:

1. Connection information
2. Restart Instance
3. Get flag

We can get our connection information with options 1 and here is the information we get

+ Wallet Address
+ Private Key
+ Setup contract address
+ Unknown contract address


If we look at the contract source codes we can get the flag when `Unknown` contract `updated` field is set to true\
We should interact with contract `Unknown` and call `updateSensors` function with input 10 to set it to True\
I used [web3 cli](https://github.com/gochain/web3) to call the `Unknown` contract `updateSensors` function with input 10

First Let's get information from second service
```bash
nc 206.189.112.129 30228
1 - Connection information
2 - Restart Instance
3 - Get flag
action? 1

Private key     :  0xa2944fbcda9390aefb92358c82125e88ac383342b938098ad7cf301fb97eef3e
Address         :  0x667da262319cc42Ef6621B8c2d185CDc7Ee8bbDf
Target contract :  0x0D660A6e10114bee123Cca7f7712Bda372c4eFb3
Setup contract  :  0x759b5313b4B8A1bf71A86c91E4178C316f41fA10
```

Here is the information we have:
```
Wallet Address : 0x667da262319cc42Ef6621B8c2d185CDc7Ee8bbDf
Private key : 0xa2944fbcda9390aefb92358c82125e88ac383342b938098ad7cf301fb97eef3e
Target contract address :  0x0D660A6e10114bee123Cca7f7712Bda372c4eFb3
Setup contract address :  0x759b5313b4B8A1bf71A86c91E4178C316f41fA10
```

To use web3 cli we should set our `private key` and `RPC url`

```bash
export WEB3_RPC_URL=http://206.189.112.129:32380/
export WEB3_PRIVATE_KEY=0xa2944fbcda9390aefb92358c82125e88ac383342b938098ad7cf301fb97eef3e
```

Now we should build our contract to generate abi files which are necessary for interacting with the network(You can read about more [ABI](https://www.alchemy.com/overviews/what-is-an-abi-of-a-smart-contract-examples-and-usage))

```bash
web3 contract build Unknown.sol
```

We don't need to deploy the contract because it's allready deployed and we have it's address.\
Now we can call the funtion we need like this with generated abi

```bash
web3 contract call --address '0x0D660A6e10114bee123Cca7f7712Bda372c4eFb3' --abi Unknown.abi --function updateSensors 10
```

With above command we call `updateSensors(10)` for Unknown contract which will satisfy the conditions to get flag from `Setup` contract
```bash
nc 206.189.112.129 30228
1 - Connection information
2 - Restart Instance
3 - Get flag
action? 3
FLAG=HTB{9P5_50FtW4R3_UPd4t3D}
```

And here is the flag
```
FLAG=HTB{9P5_50FtW4R3_UPd4t3D}
```
