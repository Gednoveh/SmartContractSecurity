Delegatecall
==

Delegatecall特点
--

1.部署合约A

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract A {
    address public c;
    address public a;

    function test() public returns (address b){
        b = address(this);
        a = b;

    }
}

2.部署合约B

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract B {
    address public a;
    address public c;

    address Aaddress = [contract A address];

    function testDelegatecall() public{
        Aaddress.delegatecall(abi.encodeWithSignature("test()"));
    }
}

3.使用A合约中的<code>test()</code>方法，查看a和c，a被修改为合约A地址

